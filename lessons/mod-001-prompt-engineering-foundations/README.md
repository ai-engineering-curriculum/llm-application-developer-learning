# mod-001 — Prompt Engineering Foundations

Your first module on the LLM Application Developer track. You will learn what actually happens when your program talks to a hosted LLM, how to shape prompts so the model produces the response your product needs, and how to force the model to return machine-parseable structured output.

By the end of this module you can build a small text classifier that calls a real LLM API, returns typed JSON, and behaves predictably enough to embed in a larger application.

## Prerequisites

- Comfort in Python **or** TypeScript at the level of writing a script that reads a file, calls an HTTP API, and prints the result.
- A working install of the OpenAI Python SDK **or** the Anthropic Python SDK. Instructions:
  - OpenAI: <https://platform.openai.com/docs/libraries>
  - Anthropic: <https://docs.anthropic.com/en/api/client-sdks>
- An API key for at least one hosted model provider. You do not need both.
- Roughly USD $1 of API credit — the labs are cheap by design.

If any of those are new to you, work through the provider's quickstart before starting the lectures.

## Learning objectives

After finishing this module you will be able to:

1. Explain the message-list model behind chat-style LLM APIs, and identify the roles used by OpenAI and Anthropic.
2. Write a prompt that reliably produces a specific format, using system prompts, few-shot examples, and delimiters.
3. Estimate the token cost of a prompt-and-response before you send it.
4. Force the model to return JSON that conforms to a schema you define, and handle the failure cases.
5. Recognise the common shapes of prompt failure (hallucination, refusal, format drift) and describe the first debugging step for each.

## Contents

| Path | What it is |
|---|---|
| `lectures/01-llm-api-basics.md` | How a chat-completion API call actually works. |
| `lectures/02-prompt-anatomy.md` | System, user, assistant, and few-shot examples. |
| `lectures/03-structured-output.md` | JSON mode, JSON Schema, and tool-shaped responses. |
| `exercises/README.md` | Short warm-ups for each lecture. |
| `labs/lab-01-text-classifier.md` | Ship a small classifier that returns structured JSON. |
| `quizzes/quiz-01.md` | Ten-question self-check with answers. |

## Time budget

Roughly 4–6 hours if you do every exercise and the lab end to end. The lectures alone are about 90 minutes of reading.

## How to work through this module

1. Read the lectures in order. Do the matching exercises before you move on.
2. Do the lab. Commit your solution to your own working directory — the paired [`llm-application-developer-solutions`](https://github.com/ai-engineering-curriculum/llm-application-developer-solutions) repo holds a reference implementation you can compare against once you finish.
3. Take the quiz without looking at your notes. Anything you miss, re-read the relevant lecture section.

## What comes next

Later modules will build on this foundation with retrieval, tool use, evaluation, and a first production LLM feature. Do not skip ahead — retrieval-augmented systems are dramatically harder to debug if your prompt hygiene is weak.
