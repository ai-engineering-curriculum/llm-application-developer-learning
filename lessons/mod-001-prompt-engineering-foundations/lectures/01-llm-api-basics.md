# Lecture 1 — How an LLM API call actually works

Before you can engineer a prompt, you need a concrete mental model of what a hosted LLM API is doing on your behalf. This lecture stays at the wire level: request in, response out, tokens counted.

## The chat-completion shape

Modern hosted LLMs expose a *chat-completion* endpoint. You send a **list of messages** and the server returns a new **assistant message**. Both OpenAI and Anthropic use this shape, though with slightly different field names.

OpenAI's Chat Completions API takes a `messages` array where each entry has a `role` (`"system"`, `"user"`, `"assistant"`, or `"tool"`) and `content`. See <https://platform.openai.com/docs/guides/text-generation>.

Anthropic's Messages API takes a `messages` array of `user` and `assistant` turns, plus a top-level `system` field for the system prompt. See <https://docs.anthropic.com/en/api/messages>.

Both APIs are **stateless** — the server does not remember your previous call. If you want a conversation, *you* re-send the transcript on every request. This is the single most important thing to internalise: your program is the memory, not the model.

## A minimal request

Here is a minimal Anthropic call in Python. Every LLM API call you make in this curriculum is a variation on this shape.

```python
import anthropic

client = anthropic.Anthropic()

message = client.messages.create(
    model="claude-opus-4-7",
    max_tokens=256,
    system="You are a concise assistant. Answer in one sentence.",
    messages=[
        {"role": "user", "content": "What is a token?"},
    ],
)

print(message.content[0].text)
```

The OpenAI equivalent is structurally identical: a client, a model name, a message list, and a token cap.

## Tokens, not characters

Models do not read your prompt as characters — they read it as **tokens**, which are chunks of text produced by the model's tokenizer. Roughly:

- English prose runs about 1 token per 4 characters, or 3/4 of a word. OpenAI documents this rough ratio at <https://platform.openai.com/tokenizer>.
- Whitespace, punctuation, code, and non-Latin scripts tokenize very differently — do not rely on the 4:1 estimate outside English prose.
- Every provider uses its own tokenizer. An OpenAI token count is not an Anthropic token count.

You pay for tokens in **both** directions: the input tokens you send and the output tokens the model generates. Output tokens are typically more expensive per token than input tokens. Current per-token prices for each model are on the provider pricing page — do not memorise them, look them up.

<!-- needs-research: exact input/output token pricing for claude-opus-4-7 as of 2026-07 — cite from https://www.anthropic.com/pricing once confirmed. -->

### Estimating tokens in code

- OpenAI: use the `tiktoken` library. <https://github.com/openai/tiktoken>
- Anthropic: use the SDK's token-counting endpoint. <https://docs.anthropic.com/en/docs/build-with-claude/token-counting>

Learn to count *before* you send a prompt in production code. A prompt whose size you cannot predict is a prompt whose cost you cannot predict.

## The context window

Every model has a **context window** — the maximum number of tokens it can consider in one request, including both your input and the model's output. When you exceed it, the API returns an error; it does not silently truncate.

- Look up the current context window for the model you are calling in the provider's model reference:
  - OpenAI: <https://platform.openai.com/docs/models>
  - Anthropic: <https://docs.anthropic.com/en/docs/about-claude/models>
- Bigger is not free. A long prompt costs more and, in most models, is processed more slowly.

## Sampling parameters that actually matter

The API accepts several knobs that affect how the model picks the next token. In practice, three matter:

- **`temperature`** — higher values produce more varied output. `0` is not fully deterministic on most providers, but it is the closest you can get. Use low values (0–0.3) for classification, extraction, and any task with a right answer. Use higher values (0.7–1.0) for creative generation.
- **`max_tokens`** — the hard cap on the response length. If you set it too low, the model will be truncated mid-sentence. If you set it too high, you pay for tokens you did not need.
- **`stop`** sequences — strings that end generation early. Useful when the model's output shape has a natural terminator.

Other knobs (`top_p`, penalties, and so on) exist and occasionally matter, but you can ship a lot of software before you need to tune them.

## Streaming versus non-streaming

Both providers support **streaming** responses, where tokens are delivered as they are generated. Streaming does not change *what* the model produces, only *when* your program sees it. Use streaming when a human is waiting for the output; skip it for background jobs and evaluations. Provider references:

- OpenAI: <https://platform.openai.com/docs/api-reference/streaming>
- Anthropic: <https://docs.anthropic.com/en/docs/build-with-claude/streaming>

## Retries, timeouts, and rate limits

Hosted LLM APIs fail. Network hiccups, rate limits, and transient server errors are all normal. Two rules from the start:

1. Set an explicit timeout on every request. The SDK defaults are long.
2. Retry only **idempotent** failures — HTTP 429 (rate limit) and 5xx. Do **not** blindly retry 4xx errors that are your fault (invalid model name, bad request body). Use exponential backoff.

Both SDKs implement retry logic; read your provider's guide before rolling your own.

## What to remember

- A chat-completion API is stateless — you send the whole transcript every time.
- You pay for tokens, both directions, using the provider's own tokenizer.
- Low temperature for tasks with a right answer, higher for creative ones.
- Every request must have a timeout and a retry policy.

The next lecture drops the API scaffolding and looks at what actually goes *inside* the prompt.
