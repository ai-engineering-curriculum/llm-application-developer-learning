# Chapter 3 — Tokens and cost estimation

Every character you send to an LLM has a price and every character it sends back has a bigger one. If you cannot predict how many characters your program will send and receive before it runs, you cannot predict its bill, its latency, or whether it will fit in the context window. This chapter is about the numbers underneath every hosted-LLM call: tokens.

## Motivation

There are three common ways a first LLM feature gets a nasty surprise in the first month of production:

1. **The bill.** Someone ran a batch job that concatenated all of a user's history into every prompt. Nobody counted before shipping. The invoice arrived and the feature was pulled.
2. **The context-window error.** A user pasted a long document and the request 400-ed at the API. The prompt looked fine locally with small inputs.
3. **The truncated JSON.** `max_tokens` was set to a comfortable-looking round number. Real inputs sometimes produced replies that hit the cap and cut off mid-object, breaking the parser downstream.

All three are token-accounting problems. All three are cheap to prevent if you know how to count.

## What a token actually is

Models do not read your prompt as characters — they read it as **tokens**, which are chunks of text produced by the model's tokenizer. Roughly:

- English prose runs about **1 token per 4 characters**, or 3/4 of a word. OpenAI documents this rough ratio at <https://platform.openai.com/tokenizer>.
- Whitespace, punctuation, code, and non-Latin scripts tokenize very differently — do not rely on the 4:1 estimate outside English prose.
- **Every provider uses its own tokenizer.** An OpenAI token count is not an Anthropic token count. The same string may weigh 62 tokens for one and 71 for the other.

Two consequences you should internalise before writing any cost-planning code:

- **Never use one provider's tokenizer to estimate another provider's cost.** The error compounds badly on long prompts.
- **Do not estimate cost from `len(string)`.** For English prose the character-count heuristic is close enough for a back-of-envelope estimate, but for code, JSON, non-Latin scripts, or long token IDs it is wildly wrong.

## Counting tokens in code

Both providers give you a way to count tokens without spending them.

### OpenAI — `tiktoken`

`tiktoken` is a Python library that ships the same tokenizers OpenAI uses on the server. See <https://github.com/openai/tiktoken>.

```python
import tiktoken

# Pick the encoding your model uses; check the tiktoken README for the current mapping.
enc = tiktoken.encoding_for_model("gpt-4.1")

prompt = "Estimate the token cost of a prompt-and-response before you send it."
input_tokens = len(enc.encode(prompt))
print(input_tokens)
```

<!-- needs-research: confirm the current tiktoken encoding for the default OpenAI model as of 2026-08 — check https://github.com/openai/tiktoken. -->

Notes:

- `tiktoken` is client-side and free. You can run it on millions of strings per second on a laptop.
- The encoding differs per model family. `cl100k_base` covered the GPT-4 / GPT-3.5 era; newer models use newer encodings. `encoding_for_model()` looks up the right one for you.
- `tiktoken` counts *raw text*. It does not add the small overhead the server adds for role framing, tool-call scaffolding, or the system-prompt wrapper. For back-of-envelope cost planning that is fine; for budget enforcement, add a small margin (a few percent) or use the server-side count from the response's `usage` block.

### Anthropic — the token-counting endpoint

Anthropic exposes a **token-counting endpoint** that returns the input-token count for a given `messages`/`system`/`tools` payload without running the model. See <https://docs.anthropic.com/en/docs/build-with-claude/token-counting>.

```python
import anthropic

client = anthropic.Anthropic()

count = client.messages.count_tokens(
    model="claude-opus-4-7",
    system="You classify customer messages.",
    messages=[{"role": "user", "content": "The refund never arrived."}],
)
print(count.input_tokens)
```

Notes:

- Because this is a real API call (though a very cheap one), it counts against rate limits. Do not put it inside a tight loop over a corpus of a million strings — batch or estimate in bulk instead.
- It counts what the *server* will count, including the role framing overhead. That makes it the authoritative source when you are enforcing per-request budgets in production code.

## Estimating output tokens

Input tokens are known before you send. Output tokens are not — the model decides how long the reply will be. Two techniques:

1. **Cap explicitly with `max_tokens`.** Whatever number you pick is an upper bound on the reply length in tokens. The model may stop earlier, but never later.
2. **Estimate from your task shape.** A classifier that returns `{"label": "bug"}` will use ~10 output tokens; a summarizer that returns three paragraphs will use ~300; a "give me a full RFC" prompt will use thousands. Measure the p50 and p95 output length on a small sample of real inputs before shipping.

The important invariant: **your total cost per call is `input_price * input_tokens + output_price * capped_output_tokens`, in the worst case.** If you can bound both terms, you can bound the bill.

## Putting it together — a cost formula

You should be able to answer this question, in dollars, before you deploy a feature:

> "If our expected usage is *N* calls per day at *T_in* mean input tokens and *T_out* mean output tokens, and the worst-case call hits our `max_tokens` cap of *T_cap*, what does the bill look like at p50 and worst-case?"

The arithmetic:

```
per_call_p50  = P_in * T_in       + P_out * T_out
per_call_max  = P_in * T_in_p99   + P_out * T_cap
daily_p50     = N * per_call_p50
daily_max     = N * per_call_max
```

Where `P_in` and `P_out` are the provider's per-token input and output prices — look them up on the provider pricing page, do not memorise them:

- OpenAI pricing: <https://openai.com/api/pricing/>
- Anthropic pricing: <https://www.anthropic.com/pricing>

<!-- needs-research: exact input/output token pricing for claude-opus-4-7 and the current default OpenAI model as of 2026-08 — cite from the provider pricing pages once confirmed. -->

If `daily_max` is a number you would be uncomfortable seeing on an invoice, either lower the `max_tokens` cap, tighten the prompt, or introduce a per-user quota before you ship. This is what mod-004 formalises.

## Context windows

Every model has a **context window** — the maximum number of tokens it can consider in one request, **including both your input and the model's output**. When you exceed it, the API returns an error; it does not silently truncate.

- Look up the current context window for the model you are calling in the provider's model reference:
  - OpenAI: <https://platform.openai.com/docs/models>
  - Anthropic: <https://docs.anthropic.com/en/docs/about-claude/models>
- Bigger is not free. A long prompt costs more and, in most models, is processed more slowly.
- The context window is measured in the provider's own tokens (see above). A prompt that fits in one provider's context may not fit in the other's.

The most useful rule of thumb: leave headroom equal to your `max_tokens` cap plus a small safety margin. If the model's window is 200,000 tokens and your cap is 4,096, budget at most 195,000 tokens for input.

## The `usage` block: measure what you sent, not what you meant to send

Every response object carries a `usage` block:

- OpenAI: `response.usage.prompt_tokens` and `response.usage.completion_tokens`.
- Anthropic: `message.usage.input_tokens` and `message.usage.output_tokens`.

Log this on every request. Three reasons:

1. **Cost accounting.** The invoice from your provider is your source of truth, but your logs are how you attribute cost to a feature, a customer, or a bad prompt version.
2. **Drift detection.** If today's mean prompt length is 30% higher than last week's for the same feature, something changed — a template grew, a caller started concatenating history, a user found a way to pad input.
3. **Prompt caching (mod-004).** Anthropic and OpenAI both return a *cached* input-token count separately. You cannot measure your cache hit ratio without reading these fields.

## Common counting mistakes

- **Counting the prompt template but not the substituted values.** If your template has an `{article}` placeholder, count after substitution, not before.
- **Forgetting few-shot examples.** Every few-shot pair you added in chapter 2 is input tokens you pay for on *every* request. If your example list is 800 tokens and you make 100,000 calls a day, that is 80M tokens of examples per day.
- **Forgetting the assistant's history.** In a multi-turn conversation you re-send the whole transcript (chapter 1). Turn 10 is nine turns' worth of input tokens on top of the current one.
- **Assuming the response fits in `max_tokens - 1`.** If the model wants to write more than the cap, it stops mid-token or mid-word. If the reply was supposed to be JSON, it now is not. Always check the stop reason.

## Summary

- Tokens are provider-specific chunks of text, roughly 1 per 4 English characters — but stop guessing outside that regime.
- OpenAI: count with `tiktoken`, client-side, cheap. Anthropic: count with the token-counting endpoint, server-side, authoritative.
- Bound the cost per call by capping `max_tokens` and knowing your input's p95 length.
- Log the `usage` block on every response; it is your only defence against silent bill drift.
- The context window includes both directions — leave headroom for the output cap.

The next chapter takes those output tokens and forces them into a JSON shape your program can parse without hoping.
