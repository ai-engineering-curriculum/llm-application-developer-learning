# Chapter 1 — The message-list model and roles

Before you can shape a prompt, you need a concrete mental picture of what a hosted LLM API is doing on your behalf. This chapter stays at the wire level: what you send, what you get back, and what the "roles" in a chat request actually mean.

## Motivation

Almost every skill later in this track — few-shot prompting, tool calling, streaming, retrieval — is a variation on one API shape: send a list of messages, receive one new assistant message. If you understand that shape and its two provider-specific dialects, the rest of the module is small deltas on top. If you don't, later chapters will feel like magic incantations.

## The chat-completion shape

Modern hosted LLMs expose a *chat-completion* endpoint. You send a **list of messages** and the server returns a new **assistant message**. Both OpenAI and Anthropic use this shape, though with slightly different field names.

- OpenAI's **Chat Completions API** takes a `messages` array where each entry has a `role` (`"system"`, `"user"`, `"assistant"`, `"tool"`, or `"developer"`) and `content`. See <https://platform.openai.com/docs/guides/text-generation>.
- Anthropic's **Messages API** takes a `messages` array of `user` and `assistant` turns, plus a **top-level `system` field** for the system prompt. See <https://docs.anthropic.com/en/api/messages>.

Both APIs are **stateless** — the server does not remember your previous call. If you want a conversation, *you* re-send the transcript on every request. Your program is the memory, not the model. Internalising this one fact prevents an entire category of bugs later.

## The roles, and what each is for

Every message in the list has a role. Both providers use largely the same names, with small differences. The roles are not decorative — the model weights them differently.

| Role | OpenAI | Anthropic | What it carries |
|---|---|---|---|
| `system` | in `messages`, or top-level `system` | top-level `system` field | Rules, persona, stable context. |
| `user` | in `messages` | in `messages` | The human's question or the input data. |
| `assistant` | in `messages` | in `messages` | Prior model turns, or fabricated few-shot outputs. |
| `tool` (OpenAI) / `tool_result` block (Anthropic) | in `messages` | inside a `user` message | The output of a tool the model called. Covered in mod-002. |
| `developer` (OpenAI, newer models) | in `messages` | — | Instructions that outrank the user turn but are separate from the persona. |

A few practical notes:

- **Anthropic has no `system` role inside `messages`.** The system prompt is a separate top-level string. Trying to send `{"role": "system", "content": "..."}` in the Anthropic array is an error, not a silent no-op.
- **OpenAI accepts either a `system` message at the top of the array or a top-level `instructions` field on newer endpoints.** The Chat Completions API uses the message; the Responses API uses the field. Read your endpoint's reference before you hard-code one.
- The **`assistant` role** is used for two very different things: (1) real prior turns in a conversation you are replaying, and (2) fabricated example outputs you use for few-shot prompting. The model does not know or care which is which — it just sees "an example of what an assistant reply looks like." That symmetry is what makes few-shot work.

## A minimal request

Every LLM API call you make in this curriculum is a variation on this shape.

### Anthropic

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

### OpenAI

```python
from openai import OpenAI

client = OpenAI()

response = client.chat.completions.create(
    model="gpt-4.1",
    max_tokens=256,
    messages=[
        {"role": "system", "content": "You are a concise assistant. Answer in one sentence."},
        {"role": "user", "content": "What is a token?"},
    ],
)

print(response.choices[0].message.content)
```

<!-- needs-research: confirm the recommended default OpenAI chat model as of 2026-08 — check https://platform.openai.com/docs/models. -->

Note the two structural differences: Anthropic's `system` lives outside `messages`; OpenAI's `system` is a message in the array. If you switch providers, this is one of the two or three lines that always has to change.

## Multi-turn: you re-send the whole transcript

To hold a conversation, append each new turn — user *and* assistant — to your local list and send the entire list on every request.

```python
messages = [
    {"role": "user", "content": "What is a token?"},
    {"role": "assistant", "content": "A token is a chunk of text a model reads or writes."},
    {"role": "user", "content": "How many tokens is 'hello world'?"},
]
```

The server does not join these turns for you across calls. If you forget to append the previous assistant reply, the next turn will feel amnesic. If you *do* append it but truncate incorrectly (dropping the system prompt, dropping the last user turn), you will introduce a subtle behavioural bug that only shows up on turn 3 or turn 4. Multi-turn state is your program's responsibility from the first line.

## What the response looks like

Both providers return an object with:

- A **content field** — the text (or list of content blocks) the model produced.
- A **usage block** — how many input tokens you sent and how many output tokens the model generated. Save this. You need it for cost accounting, and later for prompt caching (mod-004).
- A **stop reason** — why generation halted. The important values are `"end_turn"` / `"stop"` (the model finished), `"max_tokens"` (you hit your cap and the reply is truncated), and `"tool_use"` (the model wants to call a tool — covered in mod-002).

Always check the stop reason before you trust the reply. A response cut off by `max_tokens` may still parse as text, but if you are asking for JSON it will parse to invalid syntax. Do not learn this lesson in production.

## The two dialects, side by side

If you take one thing away from this chapter, take this table. It is the entire cross-provider portability story for the message-list layer.

| Concern | OpenAI (Chat Completions) | Anthropic (Messages) |
|---|---|---|
| System prompt lives | as a message with `role: "system"` | in a top-level `system` string |
| Model field | `model` | `model` |
| Token cap | `max_tokens` | `max_tokens` (required) |
| Response text | `response.choices[0].message.content` | `message.content[0].text` |
| Token counts | `response.usage.prompt_tokens` / `.completion_tokens` | `message.usage.input_tokens` / `.output_tokens` |
| Stop signal | `response.choices[0].finish_reason` | `message.stop_reason` |

## Summary

- Chat-completion APIs are stateless: you send the whole transcript every time.
- OpenAI keeps `system` in the `messages` array; Anthropic keeps it in a top-level `system` field.
- The `assistant` role does double duty — real prior turns and fabricated few-shot examples look identical to the model.
- Always check the stop reason and the token usage on every response; both are load-bearing for the chapters that follow.

The next chapter takes this shape and starts filling the messages with content that reliably produces the format your program needs.
