# Exercise 02 — Count tokens two ways

Paired with [chapter 3 — tokens and cost estimation](../03-tokens-and-cost-estimation.md).

**Estimated effort:** 20 minutes.

## Objective

Get a felt sense for the fact that "a token" is provider-specific by counting the same string with two different tokenizers and comparing the numbers.

## Problem statement

Take a paragraph of English prose — about 200–400 characters. A convenient choice is the "Motivation" section of [chapter 3](../03-tokens-and-cost-estimation.md), but any real paragraph you have written works.

Count its tokens two ways:

1. **OpenAI**: use the [`tiktoken`](https://github.com/openai/tiktoken) library with the encoding for the OpenAI model you would call in production.
2. **Anthropic**: use the [Anthropic token-counting endpoint](https://docs.anthropic.com/en/docs/build-with-claude/token-counting) for the Anthropic model you would call in production.

Print both counts side by side. Then answer, in one sentence in the script's output or a comment, *why* the numbers differ.

## Requirements

- Use the **encoding that matches the model** for tiktoken (`encoding_for_model(...)`), not a generic default.
- For the Anthropic side, send the paragraph as a single user message — do not include a system prompt (it would inflate the count for reasons unrelated to the paragraph).
- Print the two numbers and their difference. The difference should not be zero.

## Starter guidance

- `tiktoken` installs with `pip install tiktoken`. The README has a three-line quickstart.
- The Anthropic SDK's `messages.count_tokens(...)` returns an object with an `input_tokens` field. Do not confuse it with `messages.create(...).usage.input_tokens`, which spends real money.

## Acceptance criteria

- Your script prints two integers and their difference.
- Your one-sentence explanation of the difference names the fact that the two providers use different tokenizers trained on different corpora — not "one of them is buggy" or "one counts wrong."

## Stretch goals

- Repeat the count for a paragraph of Python code, and for a paragraph of Japanese or Arabic text. Compare the ratios — the 4-characters-per-token rule of thumb collapses badly outside English prose.
- Estimate what the *bill* would be to send this paragraph 1,000,000 times through each provider's current cheapest model. Show your arithmetic. (You will need current pricing from each provider's pricing page.)
