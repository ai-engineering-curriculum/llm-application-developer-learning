# Chapter 3 — Emitting traces you can actually search, without leaking

An LLM endpoint you cannot observe is a mystery box. When a customer says "the summariser gave me nonsense at 4:12 pm", you need to walk from that timestamp to *the exact prompt the model saw*, the model version, the sampling parameters, and the reply — without a single one of those fields containing a piece of PII you now have to explain to legal. This chapter is the shape of that record.

## Motivation

Two things go wrong when a first LLM feature does not have proper traces:

1. **You cannot debug in production.** A user reports a hallucination. You have no way to know which model version answered, whether retrieval fired, what the prompt actually was, or whether the reply was truncated by `max_tokens`. The best you can do is guess — and guessing at LLM behaviour is exactly the habit mod-001 chapter 5 taught you not to develop.
2. **You leak data through your logs.** The traces work. They also contain the user's email, their support ticket number, a snippet of a customer conversation, and — on the day someone adds a "log the raw request" line for debugging — the API key. Your log store is now a PII store, and every retention policy on it has to change.

The right shape avoids both. This chapter is a small catalog of fields to emit and a shorter but stricter catalog of fields not to.

## What one trace record looks like

One request → one trace record → one line in your log stream. Structure it as JSON so a search engine (or `jq`) can slice it later. A minimal record for the summariser from chapter 1:

```json
{
  "ts": "2026-08-02T14:12:03.117Z",
  "request_id": "req_2W7hR9c...",
  "user_id_hash": "sha256:9c8f...",
  "route": "/summarise",
  "model": "claude-opus-4-7",
  "prompt_version": "summarise-v3",
  "prompt_hash": "sha256:44a1...",
  "input_tokens": 4127,
  "output_tokens": 189,
  "max_tokens": 512,
  "temperature": 0.2,
  "cached_input_tokens": 3900,
  "stop_reason": "end_turn",
  "latency_ms": 4218,
  "time_to_first_token_ms": 342,
  "cost_usd_estimate": 0.0148,
  "status": "ok",
  "error_class": null
}
```

Everything in that record is either derivable from your program's own state (the request id, the model, the prompt version) or from the provider's response (`usage`, `stop_reason`). No user text. No model text. No secrets. That is the discipline: the trace tells you *what happened*, not *what was said*.

## Fields worth emitting, and why each is load-bearing

- **`ts`** — ISO-8601 timestamp with milliseconds and UTC (`Z`). If your log store adds one, still emit your own — clock skew between the app and the collector is real, and the app's timestamp is authoritative for the request's own timeline.
- **`request_id`** — a UUID or ULID generated at the top of the handler and echoed in the response headers (`X-Request-Id`). The one field that makes "the user's report" and "the row in the log store" collide. If you serve this endpoint behind a load balancer that already generates a request id, use *that* value — do not create a competing one.
- **`user_id_hash`** — not the raw user id. A stable hash (e.g., `sha256("acct_" + user_id + PEPPER)`) that lets you group requests per user without landing a joinable identifier in the log store. If you need to reverse it during an incident, that is what the pepper-in-the-secret-store is for.
- **`route`** — the endpoint path. Cheap and enormously useful when the same log stream serves several features.
- **`model`** — the exact model id used, matching what the SDK sent. If you use dated snapshots (`claude-opus-4-7-20260601`), record the snapshot. A silent model roll is the single most missed source of behaviour drift, and this field is how you catch it (see mod-006 chapter 4).
- **`prompt_version`** — a stable human-readable label for the system prompt in effect. Chapter 4 makes this a flag; chapter 6 uses it to prove which prompt handled a suspect request.
- **`prompt_hash`** — `sha256(system_prompt + fingerprint_of_message_shape)`. The **fingerprint**, not the content. The point is: if the prompt template is unchanged, the hash is unchanged, and you can group calls by "this specific prompt version." *Do not include the user's text in the hash* — that defeats the whole point of hashing (chapter 6 will make this concrete for injection detection).
- **`input_tokens` / `output_tokens`** — from the provider's `usage` block. Never from your own estimate; the server's count is authoritative. The pair is your cost input and your drift-detection signal.
- **`max_tokens`** — the cap you sent. Recording both this and `output_tokens` lets you diagnose truncation at a glance.
- **`temperature`** and any other sampling parameters you set explicitly. Recording defaults is optional; recording what you *changed* is not.
- **`cached_input_tokens`** — when using prompt caching (mod-004). Without this field you cannot compute your cache hit rate.
- **`stop_reason`** — `end_turn` / `max_tokens` / `tool_use` / `refusal`. The single most useful field for post-hoc triage — the "was this reply cut off, refused, or completed cleanly" question.
- **`latency_ms`** — server-side wall clock from the top of the handler to the last byte written. Distinct from the provider's own latency, though usually dominated by it.
- **`time_to_first_token_ms`** — the perceived-latency number for streaming endpoints. Chapter 5 of mod-003 explains why this and not `latency_ms` is what users experience.
- **`cost_usd_estimate`** — a small function of `input_tokens`, `output_tokens`, `cached_input_tokens`, and the price table you keep in config. Not a billing source of truth; a useful running signal on the day something concatenates the whole conversation history into every prompt.
- **`status`** and **`error_class`** — `"ok"` for successful requests, `"error"` with an `error_class` like `"input_too_large"` or `"upstream_5xx"` for failures. Do not put the exception message here; it may contain data.

That is the whole minimum record. Everything else — retrieval passages, tool calls, spans for individual sub-operations — is nice-to-have for a first feature. Get the basics right first.

## Fields to keep OUT of traces

The strict list. Every one of these has been leaked by someone before you.

- **The raw prompt text as sent to the model.** Includes the user's message, any retrieved passages, and any concatenated context. This is the field people are most tempted to log ("just to see what the model sees") and it is the one that most reliably contains PII, IP, and — if you concatenated headers by mistake — API keys.
- **The raw model output.** Same reason. Especially for features that return text the user asked you to summarise or answer questions from — the reply contains whatever the input contained.
- **API keys, bearer tokens, cookies, session ids.** Never in a log. If your logging middleware defaults to "log all request headers," reconfigure it before you ship. HTTP client interceptors that dump outbound requests are the most common source of provider-key leakage.
- **Full email addresses, phone numbers, national ids, addresses.** Even in the "user id" field. The hash-with-pepper pattern from above is what you use instead.
- **IP addresses**, unless you have a documented reason (abuse mitigation) and a documented retention policy that matches your privacy notice.
- **Whole request bodies** for endpoints that accept user text. The whole shape is: the trace has the *fingerprint* (hash, size, token counts) but never the *content*.
- **The exception's full traceback with local variables.** Excellent for debugging in a private dev environment; a data-exfil vector when it goes to a shared log store. Emit the exception class and message-with-values-redacted, and keep local-variable dumps behind an on-demand debug flag.

The general rule: **traces answer "what happened," not "what was said."** If you cannot answer a debugging question without the raw text, that is a signal for a *separate*, access-controlled prompt-archive store — see the section below.

## Prompt hashing done right

`prompt_hash` is one of the two or three most useful fields in the record. Two failure modes to avoid:

1. **Hashing the user text into the hash.** If every request has a unique input, every hash is unique, and grouping is impossible. The point of `prompt_hash` is that all requests that ran the *same prompt template* against *the same model* group together, whatever the user typed. Hash the *template* and its *variable names*, not the values.
2. **Forgetting to update the hash when the template changes.** The whole reason the field exists is drift detection. If you edit the prompt and the hash does not move, you cannot tell before-and-after apart.

A workable shape:

```python
import hashlib
from dataclasses import dataclass

@dataclass(frozen=True)
class PromptSpec:
    version: str          # "summarise-v3"
    system_prompt: str    # the whole system text
    template_slots: tuple[str, ...]  # ("text",)

    def fingerprint(self) -> str:
        payload = "\n".join([self.version, self.system_prompt, *self.template_slots])
        return "sha256:" + hashlib.sha256(payload.encode("utf-8")).hexdigest()
```

The fingerprint changes when the prompt version, system text, or *shape* of the slots changes. It does not change when the values in the slots change. That is exactly the invariant you want.

## Emitting the record

Two decisions, both easy for a first feature:

- **Where the record goes.** `stdout` is fine, and it is what most log collectors (Fluent Bit, Vector, Datadog agent, CloudWatch Logs agent) expect. Do not build a custom pipeline for this until you have one working. Reference for the "log to stdout" pattern in twelve-factor apps: <https://12factor.net/logs>.
- **The library.** Python's stdlib `logging` with a JSON formatter (`python-json-logger`) is the cheapest option. `structlog` (<https://www.structlog.org/>) is nicer once you want context-vars threaded through automatically. Whatever you use, emit **one JSON object per record**, not a formatted string — the whole point is that a downstream tool can parse it.

The emission point in the handler:

```python
import time
import uuid
import logging

log = logging.getLogger("summarise")

@app.post("/summarise")
async def summarise(body: SummariseRequest):
    request_id = f"req_{uuid.uuid4().hex[:20]}"
    started = time.perf_counter()
    first_token_ms: float | None = None
    input_tokens = await check_input_budget(body)

    async def gen():
        nonlocal first_token_ms
        try:
            async with client.messages.stream(...) as stream:
                async for text_chunk in stream.text_stream:
                    if first_token_ms is None:
                        first_token_ms = (time.perf_counter() - started) * 1000
                    yield text_chunk
                final = await stream.get_final_message()

            log.info("summarise.ok", extra={
                "request_id": request_id,
                "route": "/summarise",
                "model": MODEL,
                "prompt_version": PROMPT.version,
                "prompt_hash": PROMPT.fingerprint(),
                "input_tokens": final.usage.input_tokens,
                "output_tokens": final.usage.output_tokens,
                "max_tokens": OUTPUT_TOKEN_LIMIT,
                "stop_reason": final.stop_reason,
                "latency_ms": int((time.perf_counter() - started) * 1000),
                "time_to_first_token_ms": int(first_token_ms) if first_token_ms else None,
                "cost_usd_estimate": estimate_cost(MODEL, final.usage),
                "status": "ok",
            })
        except Exception as exc:
            log.error("summarise.error", extra={
                "request_id": request_id,
                "route": "/summarise",
                "model": MODEL,
                "status": "error",
                "error_class": type(exc).__name__,
                "latency_ms": int((time.perf_counter() - started) * 1000),
            })
            raise

    response = StreamingResponse(gen(), media_type="text/plain; charset=utf-8")
    response.headers["X-Request-Id"] = request_id
    return response
```

Notice what is *not* in the record: the user's `body.text`, the model's chunks, any header, any part of the API key. Notice what *is*: everything you need to reproduce and diagnose the call.

## When you really do need the prompt: a separate store

Sometimes a bug is genuinely un-diagnosable without the raw prompt. The right shape for this is a **separate, access-controlled, short-retention prompt-archive store** — not a log line. Concretely:

- Store the raw prompt + reply keyed by `request_id` in a private bucket (encrypted at rest) with a short retention window (7 or 14 days).
- Access is on-demand, audited, and separate from your regular log store's permissions.
- Sampling: 100% is fine at low traffic; a per-user sampling ratio is fine at higher traffic. Do not sample by request outcome, or you will lose the failures.
- Redact at the boundary: strip anything that looks like a secret pattern (API keys, JWTs, credit-card numbers) at write time, not at read time.

This is the shape production LLM observability platforms formalise. Chapter 7 names the ones you look at once this module stops being enough.

## Redaction: three cheap defences

You will still get PII in the log by accident. Defence in depth:

1. **Redact secrets at the logging layer.** A logging filter that scans emitted records for anything matching common secret patterns (`sk-[A-Za-z0-9]{20,}`, `sk-ant-[A-Za-z0-9-]{20,}`, Bearer tokens, JWTs) and replaces the value with `[redacted:api-key]`. This will not catch everything; it *will* catch the "someone printed the request body" class.
2. **Never log request/response bodies of outbound HTTP calls by default.** If you use an interceptor for observability, opt in to header/body logging per URL, not for everything. Provider URLs go on the deny list.
3. **Sample and audit your own log stream.** Once a week for a first feature: `grep` your log stream for `sk-`, `Bearer `, `@` (email), `password`, `token`. Anything that shows up that you did not intend is an incident.

## Common mistakes

- **Logging the full prompt "just for the first week to see how the feature is doing."** That first week always becomes six months, and the log store now has a data-retention problem. Set up the redacted trace on day one; put raw prompts in the separate short-retention store from the same day.
- **A `request_id` that changes between the handler and the log line.** Generate it once, at the top of the handler, and pass it explicitly. Do not rely on a middleware to inject the same one everywhere unless you have verified it does.
- **A trace record that is not JSON.** A formatted string is unparseable at scale. You cannot slice cost by model, latency by prompt version, or error rate by route unless the fields are separable.
- **Recording the client-side token count instead of the server's.** The server's `usage` is what you were charged for. Your local estimate is fine for a pre-flight budget check (chapter 1) and is wrong to record after the fact.
- **A trace for the outgoing call, but not for the endpoint's own outcome.** You need the *endpoint's* wall clock, not just the SDK's. The user cares about "how long the whole thing took," not just "how long the model took."

## Summary

- One trace record per request. JSON. Emit to stdout; let the log collector handle transport.
- Fields to keep: `request_id`, `user_id_hash`, `route`, `model`, `prompt_version`, `prompt_hash`, token counts, `stop_reason`, `latency_ms`, `time_to_first_token_ms`, `cost_usd_estimate`, `status`, `error_class`. Nothing else *by default*.
- Fields to keep out: raw prompts, raw replies, API keys, full emails, IPs, tracebacks with locals. Any raw prompt/reply goes to a separate, access-controlled, short-retention store — not to the log stream.
- `prompt_hash` fingerprints the template, not the values. That is what makes it useful for grouping.
- Redact secrets at the logging layer as a safety net. Do not log outbound HTTP headers by default.

The next chapter takes the `MODEL` and `PROMPT.version` fields you just emitted and makes them things you can change without a code deploy.
