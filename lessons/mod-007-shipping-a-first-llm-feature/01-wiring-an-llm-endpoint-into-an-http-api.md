# Chapter 1 — Wiring an LLM call into an HTTP API

The previous six modules taught you how to shape prompts, drive tool loops, stream tokens, choose a model, ground answers in your documents, and evaluate the whole thing. This chapter is where those pieces stop being scripts on your laptop and start being an endpoint another program can call. The unit of work changes from "a call to `client.messages.create`" to "an HTTP request that validates its input, streams its response, and refuses to blow the budget."

## Motivation

A production LLM endpoint has three responsibilities that a script does not:

1. **It has to survive bad input.** A caller will send an empty string, a 4 MB document, a value with the wrong type, or a Unicode payload that stresses whatever assumption you made about "text." If your validation is wrong, the model provider's 400 becomes the caller's 500, and the caller is your customer.
2. **It has to be pleasant to stream.** The one thing users notice about an LLM feature is the perceived latency to first character. If your endpoint blocks for six seconds and then returns the whole response, you have thrown away most of the wall-clock win streaming already earned you in mod-003.
3. **It has to refuse to spend money you did not approve.** Every call has a worst-case cost. The endpoint should know its own budget and reject requests it cannot serve within it — *before* it opens a socket to the model provider.

Missing any of these three costs you differently. Bad input handling costs you an on-call page. Non-streamed responses cost you a bad review. Missing budget enforcement costs you an invoice. This chapter is about the smallest endpoint that gets all three right.

## The stack we standardise on

The examples in this chapter use **FastAPI**. The concepts translate to any modern Python HTTP framework (Starlette directly, Litestar, Django's async views) or to Node equivalents (Fastify, Hono, Express with `async` handlers). Nothing in the shape is FastAPI-specific except the import lines. Reference: <https://fastapi.tiangolo.com/>.

Two reasons FastAPI is a good default here:

- **Pydantic-based validation.** You get typed request bodies, coerced query strings, and a 422 with a machine-readable error body without writing any of it yourself. Reference: <https://docs.pydantic.dev/latest/>.
- **First-class streaming.** `StreamingResponse` and async generators are the whole story for SSE and chunked responses. Reference: <https://fastapi.tiangolo.com/advanced/custom-response/#streamingresponse>.

If you are on another stack, keep the same three moves: schema-validated body in, streamed body out, budget check before the model call. The framework is a wrapper on those.

## Validating the request body

The body of an LLM request is a small object — a prompt, some parameters, maybe a conversation id. It is also the surface that a malicious or careless caller will use to break your service. Validate it strictly with a schema, not with `try: json.loads(...)`.

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field

app = FastAPI()

class SummariseRequest(BaseModel):
    # Constrain both ends of the length. An empty string wastes a model call;
    # a 100 KB blob will blow the input-token budget below.
    text: str = Field(min_length=1, max_length=20_000)
    # Optional, capped so a caller cannot override our max.
    max_output_tokens: int = Field(default=256, ge=1, le=1024)
```

Two things the schema is doing for you:

- **Type coercion and 422 responses for free.** A caller who sends `{"text": null}` gets a structured 422 with the field name and the reason. No exception in your handler. Reference: <https://fastapi.tiangolo.com/tutorial/body/>.
- **Documented contract.** FastAPI's `/docs` page renders the schema. When another team asks "what does your endpoint take?" you can point at a URL.

Validate the **input to the feature**, not the assembled prompt. The prompt is your program's internal concern; the schema is the interface. Keep them separate — this is the same discipline that made your golden-set inputs work in mod-006.

## Streaming the response

Every mod-003 concept applies. Two things change now that streaming lives behind an HTTP endpoint:

1. **You return an async generator wrapped in a `StreamingResponse`,** not `return {"text": full_string}`.
2. **You choose a wire format.** Two options for a small first feature:
   - **Plain chunked text.** Simplest. Suitable when the client is your own frontend and you can teach it to append chunks as they arrive.
   - **Server-Sent Events.** The right choice when you want a well-defined delimiter, a done event, and browser `EventSource` compatibility. See mod-003 chapter 1 for the SSE record format.

A minimal chunked-text streaming endpoint on top of the Anthropic SDK:

```python
from fastapi.responses import StreamingResponse
import anthropic

client = anthropic.AsyncAnthropic()

@app.post("/summarise")
async def summarise(body: SummariseRequest):
    async def gen():
        async with client.messages.stream(
            model="claude-opus-4-7",
            max_tokens=body.max_output_tokens,
            system="Summarise the input in three sentences.",
            messages=[{"role": "user", "content": body.text}],
        ) as stream:
            async for text_chunk in stream.text_stream:
                yield text_chunk

    return StreamingResponse(gen(), media_type="text/plain; charset=utf-8")
```

<!-- needs-research: confirm the current Anthropic Python SDK streaming context-manager API name (messages.stream vs. messages.create with stream=True) as of 2026-08 — check https://docs.anthropic.com/en/api/messages-streaming. -->

The equivalent for OpenAI's Chat Completions API uses the sync/async iterator form; the shape is the same generator-inside-`StreamingResponse` sandwich. Reference: <https://platform.openai.com/docs/api-reference/streaming>.

Three things to know that only become obvious the first time you deploy this:

- **Close the upstream stream on client disconnect.** If the browser closes the tab, your generator should stop, and the SDK's `async with` block should cancel the upstream call. Otherwise you keep generating (and paying for) tokens no one will ever see. FastAPI cancels the coroutine on client disconnect; make sure your SDK call is inside the async context manager so cancellation propagates.
- **Do not buffer.** If you `list(stream)` before yielding, you have written a non-streaming endpoint with extra steps. The whole point is that bytes arrive at the client as the model produces them.
- **Set `media_type` correctly.** `text/plain` for raw chunks. `text/event-stream` for SSE. Some clients (browsers, corporate proxies) will buffer if the media type suggests they may.

## Enforcing a per-request budget

The single most useful piece of code in this chapter. A budget check has two independent halves:

**Half 1: cap the input.** Before you send the request, count input tokens. If the total (system prompt + user text + any context you concatenate) exceeds a threshold you picked, reject the request with a 413 before you spend a cent.

**Half 2: cap the output.** Set `max_tokens` on the SDK call to a value you have decided you can afford at worst case. This is the only knob that bounds the response length; there is no other one.

Both halves are non-negotiable. Neither alone is enough. Together they define a `(input_tokens_max, output_tokens_max)` box, and the worst-case cost of any single request is the two corners of that box multiplied by the provider's per-token prices — the arithmetic from mod-001 chapter 3.

A concrete example. Suppose you have decided that no single request should cost more than $0.05 at Claude Opus 4.7 pricing.

<!-- needs-research: confirm claude-opus-4-7 input/output per-million token prices as of 2026-08 — cite from https://www.anthropic.com/pricing. -->

You look up the per-token prices, solve for the token budget, and land on something like `input_tokens_max = 8_000`, `output_tokens_max = 512`. Two places to enforce:

```python
from fastapi import HTTPException

INPUT_TOKEN_LIMIT = 8_000
OUTPUT_TOKEN_LIMIT = 512

async def check_input_budget(body: SummariseRequest) -> int:
    # Server-authoritative count. Anthropic's token-counting endpoint charges
    # against rate limits but not tokens; see mod-001 chapter 3.
    count = await client.messages.count_tokens(
        model="claude-opus-4-7",
        system="Summarise the input in three sentences.",
        messages=[{"role": "user", "content": body.text}],
    )
    if count.input_tokens > INPUT_TOKEN_LIMIT:
        raise HTTPException(
            status_code=413,
            detail={
                "error": "input_too_large",
                "input_tokens": count.input_tokens,
                "limit": INPUT_TOKEN_LIMIT,
            },
        )
    return count.input_tokens

@app.post("/summarise")
async def summarise(body: SummariseRequest):
    input_tokens = await check_input_budget(body)
    # ... streaming block below, with max_tokens=min(body.max_output_tokens, OUTPUT_TOKEN_LIMIT)
```

Three practical notes:

- **The token-count endpoint is authoritative because it counts what the server will count** — including role-framing overhead the client-side `tiktoken` misses. For OpenAI, count client-side with `tiktoken` and add a small safety margin (a few percent). See mod-001 chapter 3.
- **`max_tokens` is a hard cap the caller cannot override upwards.** The `SummariseRequest.max_output_tokens` field is `le=1024`, and the handler `min`s it against `OUTPUT_TOKEN_LIMIT`. The caller sets a preference; you set the ceiling.
- **413 is the right status.** Use `413 Payload Too Large` for oversize input, not a generic 400. It is machine-readable and it names the problem in the response body so the caller can shrink and retry.

## What "cost" means in this endpoint

Two costs to track, both derivable from what you already have:

- **Tokens.** Input count from your budget check, output count from the response's `usage` block. Log both. Chapter 3 wires them into the trace.
- **Dollars.** `input_tokens * P_in + output_tokens * P_out` at the provider's *current* prices. Do not memorise prices — read them from a small config, and put them next to the model name so a model swap updates both together.

You do not need a real billing pipeline for this module. Emitting the estimated dollar cost per request is enough — it lets you plot cost per feature per day, and it is the number that makes the argument for the model choice you defended in mod-004.

## Errors: what to return when the model fails

The model provider will sometimes 5xx you, sometimes rate-limit you, sometimes drop the connection mid-stream. Two rules keep the endpoint sane:

1. **Do not translate a provider 5xx into a 200.** If the upstream call failed, the client's response has to reflect that. Return the same class of error the provider gave you — a 503 for upstream unavailable, a 429 for rate limits, a 504 for a timeout.
2. **Set a hard request timeout on the provider client.** The default in most SDKs is generous. In an HTTP handler, "hang forever" is worse than "fail in six seconds and let the caller retry."

For rate limits (`429`) and transient upstream failures, the retry discipline from mod-003 chapter 4 applies — retry with jittered exponential backoff at the caller layer (or inside a retry wrapper you own), not inside the endpoint handler. If the endpoint retries silently, the caller has no visibility into slow requests.

## Auth, in one paragraph

Every LLM endpoint you ship needs *some* caller-authentication story — the model provider's key is your money, and an unauthenticated endpoint is an open door on your invoice. The simplest thing that works for a first feature is a shared internal token (`Authorization: Bearer …`), validated by a dependency, with the token stored in the same secret store you put the model key in (chapter 2). Do not roll your own auth. If the endpoint is customer-facing, put an API gateway, an OAuth provider, or your existing auth layer in front of it. This module does not go deep on auth — the graduate track (see chapter 7) does.

## Putting the shape together

The end state for a first LLM endpoint is small and disciplined:

```python
@app.post("/summarise")
async def summarise(body: SummariseRequest):
    input_tokens = await check_input_budget(body)
    output_cap = min(body.max_output_tokens, OUTPUT_TOKEN_LIMIT)

    async def gen():
        async with client.messages.stream(
            model=MODEL,               # from config, chapter 4 puts a flag here
            max_tokens=output_cap,
            system=SYSTEM_PROMPT,
            messages=[{"role": "user", "content": body.text}],
        ) as stream:
            async for text_chunk in stream.text_stream:
                yield text_chunk
            # Post-stream: read the final message for usage counts,
            # emit the trace record (chapter 3), update the cost gauge.

    return StreamingResponse(gen(), media_type="text/plain; charset=utf-8")
```

Everything else in this module — secrets, tracing, feature flags, runbooks, input sanitisation — hangs off this shape. Keep the handler small; push the moving parts (`MODEL`, `SYSTEM_PROMPT`, budget constants, provider client) into modules you can test in isolation.

## Summary

- Validate the request body with a schema. `min_length` / `max_length` / `ge` / `le` on the fields are load-bearing, not decorative.
- Return a `StreamingResponse` over an async generator. Do not buffer.
- Enforce the input-token budget with the provider's token-count endpoint (Anthropic) or `tiktoken` (OpenAI), and reject oversize input with a 413 *before* the model call.
- Cap the output with `max_tokens`. The caller can request less; they cannot request more than your ceiling.
- Set a hard request timeout on the SDK client. Propagate upstream failures with matching status codes.

The next chapter is where the `MODEL`, `SYSTEM_PROMPT`, and API key stop being module-level constants and start living in a per-environment secret store.
