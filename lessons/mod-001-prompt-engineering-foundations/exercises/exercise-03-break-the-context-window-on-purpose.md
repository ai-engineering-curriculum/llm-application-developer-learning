# Exercise 03 — Break the context window on purpose

Paired with [chapter 3 — tokens and cost estimation](../03-tokens-and-cost-estimation.md).

**Estimated effort:** 20 minutes.

## Objective

See, in your own console, what happens when you exceed the model's context window. This class of error will come back several times in this curriculum — you should recognise its shape on sight.

## Problem statement

Deliberately construct a prompt that is larger than the context window of the model you are calling. Send it. Catch the error the API returns and print **the error's class name and its message** — not the whole traceback.

## Requirements

- Look up your model's context window in the provider's model reference:
  - OpenAI: <https://platform.openai.com/docs/models>
  - Anthropic: <https://docs.anthropic.com/en/docs/about-claude/models>
- Build a prompt that is safely larger than the window. Do not try to hit it exactly — pad by 10× so there is no ambiguity. (Repeating a short string is fine; you are testing error handling, not the model's response.)
- **Catch the specific SDK exception**, not a bare `Exception`. Print `type(e).__name__` and `str(e)`.
- Do not swallow the error silently. If the request unexpectedly succeeds, that is a signal that your prompt was not actually oversized — investigate before moving on.

## Starter guidance

- The OpenAI SDK raises `openai.BadRequestError` (or the equivalent subclass) for this class of failure. The Anthropic SDK raises `anthropic.BadRequestError`. Both surface a message that mentions the token count and the model's limit.
- You do not need to send a valid document. A prompt of `"a " * 1_000_000` characters is fine for the purposes of the test.
- If your provider has multiple models with different context windows, use one with a *smaller* window for the test — you will hit the limit faster and pay less.

## Acceptance criteria

- Your script prints one line of output: the exception class name and its message.
- The error is a client-side error (HTTP 400 in the message, or a `BadRequestError` subclass), not a server error. If you see a 5xx, you did not exceed the window — you probably hit a rate limit or a network issue.
- You can articulate, in one sentence, why this specific error should *not* be retried.

## Stretch goals

- Modify your script to detect this error *before* sending the request. Use the token-counting tool from exercise 02, compare to the model's window, and refuse the request locally with a clear error message. This is the pattern real production code uses.
- Confirm that retrying this exact error does not help. Wrap the call in a retry loop with three attempts and note that every attempt fails identically.
