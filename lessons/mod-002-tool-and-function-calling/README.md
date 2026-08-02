# mod-002 — Tool and Function Calling for LLM Applications

Your second module on the LLM Application Developer track. Here the LLM stops being a text box: you teach it to call functions in your program, hand it the results, and let it produce answers grounded in real data or side effects.

By the end of this module you can drive the full request → tool_call → tool_result → response loop end-to-end in Python, handle parallel tool calls in a single turn, recover cleanly from bad arguments and tool failures, and decide when a tool call is genuinely the right shape for a feature.

**Estimated effort:** ~12 hours (chapters ~2 hours; exercises 8 hours; the rest is time reading the provider docs).

## Prerequisites

- You have finished [`mod-001-prompt-engineering-foundations`](../mod-001-prompt-engineering-foundations/README.md). In particular, chapter 4 (schema-constrained JSON output) is a load-bearing prerequisite — a tool schema *is* a JSON Schema.
- Comfort in Python at the level of writing small dispatch tables, using `try` / `except`, and running functions concurrently with `concurrent.futures.ThreadPoolExecutor` or `asyncio.gather`.
- A working install of the OpenAI Python SDK **or** the Anthropic Python SDK, and an API key for one provider. You do not need both — every chapter shows the same loop in both dialects, so you can follow whichever you have credit for.
- Roughly USD $1–2 of API credit. Tool-calling exercises make more round trips than mod-001 did, but individual calls remain cheap.

If any of those are new to you, work through mod-001 or the provider's tool-use quickstart before starting the chapters.

## Learning objectives

After finishing this module you will be able to:

1. Declare typed tool / function schemas that OpenAI and Anthropic models can invoke, and choose between plain JSON Schema and provider-specific wrappings.
2. Handle the request → tool_call → tool_result → response loop end-to-end in Python, including turns where the model chains multiple tool calls.
3. Detect and handle malformed tool arguments, missing tools, and tool-side exceptions without silently mis-answering the user.
4. Reason about when a tool call is appropriate versus plain generation or structured output — the cost, latency, and reliability trade-off.
5. Draw the boundary to the peer track `agentic-ai-developer-learning` — this module covers the tool-call API surface; multi-step planning, ReAct loops, and multi-agent coordination live in that peer track.

## Chapters

Read in order. Each chapter maps to one learning objective.

| # | Chapter | Objective |
|---|---|---|
| 1 | [Declaring tool schemas](01-declaring-tool-schemas.md) | Objective 1 |
| 2 | [The request → tool_call → tool_result → response loop](02-tool-call-response-loop.md) | Objective 2 |
| 3 | [Parallel tool calls in one turn](03-parallel-tool-calls-in-one-turn.md) | Objective 2 |
| 4 | [Handling tool-call failures](04-handling-tool-call-failures.md) | Objective 3 |
| 5 | [When to reach for a tool call (and when not to)](05-when-to-reach-for-a-tool-call.md) | Objectives 4 & 5 |

## Exercises

Short hands-on drills paced to the chapters. Do the matching exercise **after** its chapter but **before** starting the next.

| # | Exercise | Chapter |
|---|---|---|
| 01 | [First tool schema](exercises/exercise-01-first-tool-schema.md) | 1 |
| 02 | [Tool call response loop](exercises/exercise-02-tool-call-response-loop.md) | 2 |
| 03 | [Parallel tool calls in one turn](exercises/exercise-03-parallel-tool-calls-in-one-turn.md) | 3 |
| 04 | [Malformed arguments and missing tools](exercises/exercise-04-malformed-arguments-and-missing-tools.md) | 4 |
| 05 | [When not to use a tool](exercises/exercise-05-when-not-to-use-a-tool.md) | 5 |

## Labs and quizzes

`labs/` and `quizzes/` are reserved for long-form hands-on work and knowledge checks authored in later cycles. If they are still empty when you get here, the exercises above are enough to cement the objectives.

## Resources

Real, citable references for the topics in this module — provider tool-use docs, JSON Schema, standards. See [`resources.md`](resources.md).

## How to work through this module

1. Read chapters 1 and 2 in one sitting. Do exercises 01 and 02 before moving on — the loop mechanics only stick after you have written them.
2. Read chapter 3 and do exercise 03. Parallelism is easy to get almost-right; the exercise is designed to break the almost-right implementations.
3. Read chapter 4 and do exercise 04. Chapter 4 is the shortest chapter and the one that separates ship-ready features from demos.
4. Read chapter 5 and do exercise 05. Exercise 05 is a design exercise, not a coding one — pick the right tool for a set of scenarios and defend the choice.
5. Commit your solutions to your own working directory. A reference implementation lives in the paired [`llm-application-developer-solutions`](https://github.com/ai-engineering-curriculum/llm-application-developer-solutions) repo once you finish.

## What comes next

`mod-003-streaming-async-and-orchestration` picks up the response wire itself: streaming tokens as they are generated, and running many calls concurrently without your process becoming its own bottleneck. Later modules add model selection and prompt caching, minimal retrieval, minimal evaluation, and a first shipped LLM feature. If you want to build systems where the model plans over many turns — not just calls one tool per turn — see the peer track [`agentic-ai-developer-learning`](https://github.com/ai-engineering-curriculum/agentic-ai-developer-learning). Chapter 5 explains why that boundary is where it is.
