# mod-001 — Prompt Engineering Foundations

Your first module on the LLM Application Developer track. You will learn what actually happens when your program talks to a hosted LLM, how to shape prompts so the model produces the response your product needs, how to force schema-conforming JSON, and how to recognise the failure shapes you will meet in production.

By the end of this module you can build a small text classifier that calls a real LLM API, returns typed JSON, and behaves predictably enough to embed in a larger application.

## Prerequisites

- Comfort in Python **or** TypeScript at the level of writing a script that reads a file, calls an HTTP API, and prints the result.
- A working install of the OpenAI Python SDK **or** the Anthropic Python SDK. Instructions:
  - OpenAI: <https://platform.openai.com/docs/libraries>
  - Anthropic: <https://docs.anthropic.com/en/api/client-sdks>
- An API key for at least one hosted model provider. You do not need both.
- Roughly USD $1 of API credit — the exercises and lab are cheap by design.

If any of those are new to you, work through the provider's quickstart before starting the chapters.

## Learning objectives

After finishing this module you will be able to:

1. Explain the message-list model behind chat-style LLM APIs, and identify the roles used by OpenAI and Anthropic.
2. Write a prompt that reliably produces a specific format, using system prompts, few-shot examples, and delimiters.
3. Estimate the token cost of a prompt-and-response before you send it.
4. Force the model to return JSON that conforms to a schema you define, and handle the failure cases.
5. Recognise the common shapes of prompt failure (hallucination, refusal, format drift) and describe the first debugging step for each.

## Chapters

Read in order. Each chapter maps to one learning objective.

| # | Chapter | Objective |
|---|---|---|
| 1 | [The message-list model and roles](01-message-list-and-roles.md) | Objective 1 |
| 2 | [Shaping prompts for a reliable format](02-shaping-prompts-for-reliable-format.md) | Objective 2 |
| 3 | [Tokens and cost estimation](03-tokens-and-cost-estimation.md) | Objective 3 |
| 4 | [Schema-constrained JSON output](04-schema-constrained-json-output.md) | Objective 4 |
| 5 | [Diagnosing prompt failures](05-diagnosing-prompt-failures.md) | Objective 5 |

## Exercises

Short warm-ups paced to the chapters. Do them **after** reading the matching chapter but **before** moving on. Each takes 15–30 minutes.

| # | Exercise | Chapter |
|---|---|---|
| 01 | [Round-trip a message](exercises/exercise-01-round-trip-a-message.md) | 1 |
| 02 | [Count tokens two ways](exercises/exercise-02-count-tokens-two-ways.md) | 3 |
| 03 | [Break the context window on purpose](exercises/exercise-03-break-the-context-window-on-purpose.md) | 3 |
| 04 | [Separate rules from data](exercises/exercise-04-separate-rules-from-data.md) | 2 |
| 05 | [Add three-shot examples](exercises/exercise-05-add-three-shot-examples.md) | 2 |
| 06 | [Test a jailbreak](exercises/exercise-06-test-a-jailbreak.md) | 2 & 5 |
| 07 | [Ask nicely and break it](exercises/exercise-07-ask-nicely-and-break-it.md) | 4 |
| 08 | [Turn on schema-constrained output](exercises/exercise-08-turn-on-schema-constrained-output.md) | 4 |
| 09 | [Semantic vs syntactic correctness](exercises/exercise-09-semantic-vs-syntactic-correctness.md) | 4 & 5 |

## Lab and quiz

- **Lab:** [`labs/lab-01-text-classifier.md`](labs/lab-01-text-classifier.md) — ship a small classifier that returns structured JSON.
- **Quiz:** [`quizzes/quiz-01.md`](quizzes/quiz-01.md) — ten-question self-check with answers.

## Resources

Real, citable references for the topics in this module — provider docs, tokenizers, standards. See [`resources.md`](resources.md).

## Time budget

Roughly 4–6 hours if you do every exercise and the lab end to end. The chapters alone are about 90 minutes of reading.

## How to work through this module

1. Read the chapters in order. Do the matching exercises before you move on.
2. Do the lab. Commit your solution to your own working directory — the paired [`llm-application-developer-solutions`](https://github.com/ai-engineering-curriculum/llm-application-developer-solutions) repo holds a reference implementation you can compare against once you finish.
3. Take the quiz without looking at your notes. Anything you miss, re-read the relevant chapter.

## What comes next

Later modules build on this foundation with tool / function calling, streaming and async orchestration, model selection and prompt caching, minimal retrieval, minimal evaluation, and a first shipped LLM feature. Do not skip ahead — retrieval-augmented systems and tool-calling loops are dramatically harder to debug if your prompt hygiene is weak.
