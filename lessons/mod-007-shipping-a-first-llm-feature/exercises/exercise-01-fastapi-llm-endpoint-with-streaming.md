# Exercise 01 — A FastAPI LLM endpoint with streaming and a budget

Paired with [chapter 1 — wiring an LLM call into an HTTP API](../01-wiring-an-llm-endpoint-into-an-http-api.md).

**Estimated effort:** 2 hours.

## Objective

Build the smallest reasonable production shape of an LLM endpoint. By the end of the exercise you have a single HTTP route on FastAPI (or a comparable framework) that (a) validates its request body with a Pydantic schema, (b) streams the model's response back to the caller without buffering, (c) enforces a per-request input-token budget with a proper 413 for oversize input, and (d) caps the output with a `max_tokens` value the caller cannot override upwards. Every later exercise in this module extends this endpoint.

## Problem statement

Pick one feature you want to ship. Concrete options if you do not have one:

- A **summariser** endpoint (`POST /summarise`) — the running example in the chapters.
- A **JSON extractor** endpoint (`POST /extract`) — building on the mod-001 exercise-08 extractor.
- A **grounded Q&A** endpoint (`POST /answer`) — building on mod-005 exercise 03, with the retrieved passages assembled server-side.

Then, in a small repo (call it `first-feature/`), build:

1. A **FastAPI application** with one route. The route accepts a JSON body with at least a `text` (or equivalent) field. The response streams the model's reply as `text/plain` chunks (or SSE, if you already prefer it).
2. **Pydantic request-body validation** with explicit `min_length` / `max_length` on the text field and an `le`-capped `max_output_tokens` field.
3. A **budget-check step** that runs *before* the model call. Use Anthropic's `messages.count_tokens` endpoint (or `tiktoken` for OpenAI) to count input tokens for the full prompt-including-system-and-user-text; if the total exceeds a threshold you pick (`INPUT_TOKEN_LIMIT`), return `413 Payload Too Large` with a JSON body naming the count and the limit.
4. A **streaming call** to the model provider (Anthropic `messages.stream` or OpenAI `chat.completions.create(stream=True)`), yielded through a FastAPI `StreamingResponse`. The generator must yield chunks as they arrive; no `list(stream)` or full-buffer accumulation.
5. **A hard cap on `max_tokens`** — the SDK call uses `min(body.max_output_tokens, OUTPUT_TOKEN_LIMIT)`, so the caller can request less but not more.

Do not add secrets management, tracing, feature flags, or input sanitisation yet — those are the next four exercises. Keep this one focused.

## Requirements

- **`main.py`** with the FastAPI app and one route.
- **Explicit request timeout** on the SDK client (`AsyncAnthropic(timeout=15.0)` or the OpenAI equivalent). Do not rely on the SDK default.
- **`INPUT_TOKEN_LIMIT`** and **`OUTPUT_TOKEN_LIMIT`** as module-level constants. Both must be numbers you can defend — a one-line comment explaining how you picked them (from a cost calculation) is enough. This is chapter 1's per-request budget arithmetic.
- **The 413 response body** is JSON with at least `error`, `input_tokens`, and `limit` fields, so the caller can shrink and retry programmatically.
- **The streaming generator must handle client disconnect.** FastAPI cancels the coroutine on client disconnect; verify that the upstream SDK call is inside an `async with` block that will propagate the cancellation. A short comment in the code claiming "cancellation propagates" is not enough; test it (see acceptance criteria).
- **The `X-Request-Id` response header** is set to a UUID or ULID your handler generates at the top. Exercise 03 will feed this into the trace.
- **No secrets in code.** The SDK reads `ANTHROPIC_API_KEY` / `OPENAI_API_KEY` from the environment. The endpoint fails loudly at startup if the key is missing — a `RuntimeError` from a lazy client getter, not a 500 on the first request.
- The endpoint runs locally with `uvicorn main:app --reload` and answers `curl -N -X POST` with a streamed response.

## Starter guidance

- Read the chapter first. The whole endpoint's shape is in the "Putting the shape together" section — you are filling in one route.
- For the summariser choice, the system prompt is a one-liner: "Summarise the input in three sentences." Do not over-tune the prompt for this exercise; the point is the wire shape, not the prompt.
- FastAPI Pydantic body reference: <https://fastapi.tiangolo.com/tutorial/body/>. `StreamingResponse` reference: <https://fastapi.tiangolo.com/advanced/custom-response/#streamingresponse>.
- Anthropic streaming reference: <https://docs.anthropic.com/en/api/messages-streaming>. OpenAI streaming reference: <https://platform.openai.com/docs/api-reference/streaming>.
- Anthropic token counting reference: <https://docs.anthropic.com/en/docs/build-with-claude/token-counting>. `tiktoken` reference: <https://github.com/openai/tiktoken>.
- If you get a `text/event-stream`-buffered-by-proxy problem locally, `--reload` + `curl -N` (no-buffer) is enough to see chunks arrive. If they still coalesce, check that you are not `.join`-ing them somewhere inside `gen()`.
- A minimal `pyproject.toml` or `requirements.txt` — you only need `fastapi`, `uvicorn`, and the one provider SDK. Do not add anything else in this exercise.

## Acceptance criteria

- `curl -N -X POST http://localhost:8000/summarise -H 'Content-Type: application/json' -d '{"text": "…paragraph…"}'` streams a reply that arrives incrementally (you can watch it type). If you disable your network mid-stream, the connection ends within your configured timeout, not "eventually."
- Sending `{"text": ""}` returns a 422 with a field-level error from Pydantic. Sending `{"text": null}` returns a 422. Sending `{"max_output_tokens": 5000}` (above the schema's `le`) returns a 422.
- Sending a text long enough to exceed `INPUT_TOKEN_LIMIT` returns a 413 with a JSON body naming `input_tokens` and `limit`. Confirm the model was **not** called — the counter did the check *before* the streaming block ran.
- Sending a `max_output_tokens` request value below the ceiling produces a shorter reply; sending one above is capped by `OUTPUT_TOKEN_LIMIT` (the response's own token count is at or below the ceiling, not the request value).
- The response includes an `X-Request-Id` header with a distinct value per request.
- Closing the client mid-stream (kill the `curl` after two chunks) stops your process from generating more tokens against the provider — confirm this by watching the provider's dashboard for the request, or by adding a temporary `print` in the generator's `except CancelledError` branch. The upstream call closes; you are not paying for tokens the user never saw.
- Unsetting `ANTHROPIC_API_KEY` (or `OPENAI_API_KEY`) and re-running the server produces a startup error (or a `RuntimeError` on the first request), not a stack trace containing "invalid key" from the SDK.

## Stretch goals

- **Switch the wire format to SSE.** Change `media_type` to `text/event-stream`, wrap each chunk in `data: <chunk>\n\n`, and add a `event: done\ndata: {}\n\n` terminator when the model finishes. The frontend equivalent is `new EventSource("/summarise/stream")`.
- **Log the final `usage` block from the response.** Prints `input_tokens`, `output_tokens`, and `stop_reason` to stdout at the end of the stream. Exercise 03 turns this into a full JSON trace.
- **Add a hard request timeout to the handler.** Wrap the streaming block in `asyncio.wait_for(...)` with a ceiling higher than your SDK timeout. If the whole thing exceeds the ceiling (retries + streaming), fail the request rather than let it hang.
- **Compare token counts to `tiktoken` estimates.** For the same input, compute the client-side estimate with `tiktoken` (if you are on Anthropic, do it with the Anthropic count endpoint; if on OpenAI, do the reverse using an Anthropic count for comparison). Note the delta — this is the safety margin chapter 1 mentioned.
