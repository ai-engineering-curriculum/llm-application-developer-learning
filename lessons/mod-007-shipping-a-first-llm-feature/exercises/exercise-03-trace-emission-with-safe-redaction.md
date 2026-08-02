# Exercise 03 — Trace emission with safe redaction

Paired with [chapter 3 — emitting traces you can actually search, without leaking](../03-traces-with-safe-redaction.md).

**Estimated effort:** 2 hours.

## Objective

Add a redacted, JSON, one-record-per-request trace to the endpoint. By the end of the exercise, every request that arrives at the endpoint produces exactly one JSON log line containing every field chapter 3 lists as "load-bearing" — and none of the fields chapter 3 lists as "keep out." You will also have a smoke test that asserts the redaction actually redacts.

## Problem statement

Extend `first-feature/` from exercise 02:

1. **Emit one JSON trace record per request**, on both success and error paths. The record has the shape from chapter 3's "What one trace record looks like" section — at minimum: `ts`, `request_id`, `user_id_hash`, `route`, `model`, `prompt_version`, `prompt_hash`, `input_tokens`, `output_tokens`, `max_tokens`, `stop_reason`, `latency_ms`, `time_to_first_token_ms`, `cost_usd_estimate`, `status`, `error_class`.
2. **Compute `prompt_hash` from the template and slot names, not the values.** A `PromptSpec` dataclass or equivalent that produces a stable fingerprint of the system prompt + the template shape. Verify the fingerprint changes when you edit the prompt and does *not* change when you send different user inputs.
3. **Compute `cost_usd_estimate` from a small price table** you keep in config (`PRICES = {"claude-opus-4-7": {"input": ..., "output": ..., "cached_input": ...}}`). The price table has a comment linking the pricing page you copied from and the date you copied it.
4. **Hash the user id, do not log the raw id.** A `sha256("acct_" + user_id + PEPPER)` shape, with the pepper read from the secret store. For a caller-authentication-less first feature, hash the request's IP + `User-Agent` as a proxy — the point is that the field exists and is not a raw identifier.
5. **Emit the record to stdout** as JSON. Do not write your own JSON serialisation — use `python-json-logger` (<https://github.com/madzak/python-json-logger>), `structlog` (<https://www.structlog.org/>), or the Node equivalent.
6. **Add a redaction layer at the logging boundary.** A logging filter (or `structlog` processor) that scans every emitted record for API-key patterns (`sk-ant-[A-Za-z0-9-]{20,}`, `sk-[A-Za-z0-9]{20,}`, `Bearer [A-Za-z0-9._-]+`) and replaces the match with `[redacted:api-key]`.
7. **A smoke test — `tests/test_trace_redaction.py`** — that constructs a fake log record whose message contains a real-looking API-key string, runs it through your logging layer, and asserts the emitted line has `[redacted:api-key]` and does *not* have the original key.
8. **Under no circumstances log the raw request body, the model's reply text, or any outbound HTTP header.** If exercise 01's response-body accumulation code exists anywhere, delete it now — you do not need to see the reply to trace the request.

## Requirements

- **One record per request.** Two records means the shape is wrong; zero records on the error path means the trace has a blind spot exactly where you need visibility.
- **The record is valid JSON** — `python -c "import json; [json.loads(l) for l in open('server.log')]"` on your captured log stream is silent. A downstream log collector must be able to parse every line.
- **`prompt_hash` is deterministic and template-shaped.** Two requests with the same template and different `text` values have the same hash; changing the system prompt (or bumping `SYSTEM_PROMPT_VERSION`) produces a different hash.
- **`cost_usd_estimate` is present and non-null on success.** Zero is a valid answer (cached call, empty output) but the field must exist.
- **`user_id_hash` never contains the raw id.** A `grep` for the raw id in the log stream returns nothing.
- **The `X-Request-Id` response header value matches the trace record's `request_id`.** If they diverge, one of them is generated in the wrong place — chapter 3 warns about exactly this.
- **The redaction filter runs on every record.** The test in step 7 passes. A second test that constructs a record with two different key patterns confirms both are redacted.
- **No secret ever hits stdout.** Verify by unsetting the SDK-internal debug flag (if you turned it on for local development) and by not calling any `logger.debug(request_headers)`-style line yourself.

## Starter guidance

- Read the "Emitting the record" section of chapter 3 carefully — the handler shape shown there is what you are implementing. The main additions are the JSON formatter, the redaction filter, and the price table.
- If you are using `structlog`, wire a `format_exc_info` processor and a `JSONRenderer` processor; put your redaction as a *processor*, not a downstream filter — that way it applies to every emission uniformly.
- If you are using `python-json-logger`, the redaction can be a `logging.Filter` subclass whose `filter` method rewrites `record.msg` and any relevant fields on the record.
- For the pricing table, read the provider page once and hard-code the numbers with a comment naming the URL and the date. Do not fetch prices at runtime.
  - OpenAI pricing: <https://openai.com/api/pricing/>
  - Anthropic pricing: <https://www.anthropic.com/pricing>
- The `time_to_first_token_ms` field is computed by starting a monotonic clock at the top of the handler and stamping the *first* iteration through the streaming generator. `time.perf_counter()` is the right clock in Python. Reference: <https://docs.python.org/3/library/time.html#time.perf_counter>.
- Twelve-factor logs (log to stdout, let the platform handle transport) is the shape you are following: <https://12factor.net/logs>.

## Acceptance criteria

- One `curl` against `POST /summarise` produces exactly one JSON line on stdout that includes every required field and no forbidden field (raw prompt, raw reply, key, header value, PII).
- Editing `prompts/summarise-v1.md` and rerunning changes `prompt_hash`. Sending a different `text` value with the same prompt file does *not* change `prompt_hash`.
- The trace record's `input_tokens` matches the provider's `usage.input_tokens` from the response — not your pre-flight estimate.
- Sending a request that trips the 413 in exercise 01 still emits a trace record, with `status: "error"` and `error_class: "input_too_large"`. The record does *not* contain the request body.
- `pytest tests/test_trace_redaction.py` passes. Adding a second test with a `Bearer` token and another with an OpenAI-prefixed key confirms both are redacted.
- `X-Request-Id` from a live response equals the `request_id` field in the corresponding trace record. Run it three times and cross-check.
- No line in the log stream matches the current API key when you `grep` for its first eight characters.

## Stretch goals

- **Add a `cached_input_tokens` field** when using prompt caching (mod-004). Confirm that a repeat request with the same system prompt increments the cached counter and *decreases* the `cost_usd_estimate` compared to the first request. This is what makes the mod-004 cache visible in your traces.
- **Wire the trace record into a real log collector locally.** Vector (<https://vector.dev/docs/>), Fluent Bit (<https://docs.fluentbit.io/manual/>), or the Datadog / Grafana / Loki agent. Send it to a local sink and confirm the fields are searchable. This makes the "one line per request" discipline pay off immediately.
- **Sample a `raw_prompt_archive` write to a separate directory.** For 1% of requests, write the full prompt + reply to `archive/<request_id>.json`. Purge the directory after 24 hours with a cron / systemd timer. This is chapter 3's "separate short-retention store" shape at hobby scale.
- **Compute p50 / p95 latency** over the last 15 minutes of traces from a simple `jq` one-liner. Post the numbers to your local README as the operational baseline. Exercise 04's rollout will move these; you will want to know the pre-rollout numbers.
