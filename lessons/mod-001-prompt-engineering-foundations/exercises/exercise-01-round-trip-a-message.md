# Exercise 01 — Round-trip a message

Paired with [chapter 1 — the message-list model and roles](../01-message-list-and-roles.md).

**Estimated effort:** 15 minutes.

## Objective

Confirm you can send one user message to a hosted LLM API and print its reply, using an explicit timeout and a small `max_tokens` cap. Every later exercise assumes this works.

## Problem statement

Write a small script (Python or TypeScript) that:

1. Instantiates an OpenAI **or** Anthropic client — pick whichever you have credit for.
2. Sends a single-turn request containing one user message: "Explain what a token is in one sentence."
3. Prints the model's reply to stdout.

Do not use a library that wraps the SDK for you. You want to see the raw request shape once with your own eyes.

## Requirements

- Set an **explicit request timeout** on the SDK client (e.g., `Anthropic(timeout=10.0)` or the OpenAI equivalent). Do not rely on the SDK default.
- Set `max_tokens` to a small value — 60 is plenty.
- Read the API key from an environment variable (`ANTHROPIC_API_KEY` or `OPENAI_API_KEY`). Do not paste it into the code.
- Print only the reply text, not the full response object.

## Starter guidance

Both providers publish minimal quickstarts that you can copy line-for-line for step 1:

- OpenAI: <https://platform.openai.com/docs/quickstart>
- Anthropic: <https://docs.anthropic.com/en/api/client-sdks>

Refer to the code snippet in [chapter 1](../01-message-list-and-roles.md) for the exact request shape once your client is set up.

## Acceptance criteria

- Running your script prints one to three sentences of English text and exits 0.
- The reply is short — clearly capped by your `max_tokens` setting rather than trailing off naturally at 500 tokens.
- Killing your Wi-Fi mid-run causes the script to fail with a timeout inside 15 seconds, not hang forever.

## Stretch goals

- Add a `--model` CLI flag and confirm the reply text changes when you switch between two models offered by your provider.
- Print the `usage` block from the response alongside the text. Compare `input_tokens` to the character count of your prompt divided by 4 — how close is the 4-chars-per-token heuristic on this string?
