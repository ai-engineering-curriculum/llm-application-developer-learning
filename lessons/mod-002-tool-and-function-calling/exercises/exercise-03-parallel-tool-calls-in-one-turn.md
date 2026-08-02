# Exercise 03 — Parallel tool calls in one turn

Paired with [chapter 3 — parallel tool calls in one turn](../03-parallel-tool-calls-in-one-turn.md).

**Estimated effort:** 90–120 minutes.

## Objective

Extend the loop from exercise 02 to handle multiple tool_use / tool_calls blocks in one assistant turn, execute them concurrently, and match results back to calls by ID. Deliberately construct a broken variant that swaps IDs and confirm the model produces a wrong answer — that failure teaches the invariant.

## Problem statement

Build a two-tool assistant with the following tools:

- `get_current_time(timezone: str) -> str` — same as exercise 02.
- `get_country_capital(country: str) -> str` — returns the capital city of the given country. Implement it against a small local dict (`{"France": "Paris", "Japan": "Tokyo", "Argentina": "Buenos Aires", ...}`); no external API call needed.

Then send prompts that should trigger the model to fan out both tools in a single turn:

1. `"What time is it right now in the capitals of France, Japan, and Argentina?"`
2. `"Give me the current time in Paris, London, and Sydney."` — a single-tool fan-out (three calls to `get_current_time`).

Your loop must (a) execute all requested tool calls concurrently, (b) send exactly one tool_result per tool_use back in the next turn, and (c) match by ID, never by list position.

## Requirements

1. Start from your `run_tool_conversation` helper in exercise 02. Extend it, do not rewrite from scratch.
2. Use `concurrent.futures.ThreadPoolExecutor` (or `asyncio.gather` if your tools are async) with a `max_workers` cap. Do not spawn an unbounded pool.
3. Build a `results_by_id` dict keyed by the tool_use / tool_call ID. Assemble the tool_result block from that dict. Do not use list indices.
4. Cap the per-tool timeout at 10 seconds. On timeout, return an explicit error content the model can read (chapter 4 shows the exact shape — for this exercise, an ad-hoc `"ERROR: timeout"` string is fine).
5. Log the wall-clock time of each concurrent tool call and the overall turn latency. Confirm the turn latency is close to the *slowest* tool, not the *sum* of the tools.

## Requirements — the ID-swap experiment

After the happy path works, add a `--swap-ids` mode that deliberately shuffles the tool_result IDs before sending them back. For prompt 1 above, run once with `--swap-ids off` and once with `--swap-ids on`, and capture both final answers.

Do not silently commit the swap. Print a warning to stderr when the flag is on. You are demonstrating the failure mode, not shipping it.

## Starter guidance

- Anthropic parallel tool use: <https://docs.anthropic.com/en/docs/build-with-claude/tool-use/parallel-tool-use>
- OpenAI parallel function calling: <https://platform.openai.com/docs/guides/function-calling#parallel-function-calling>
- `concurrent.futures.ThreadPoolExecutor` API: <https://docs.python.org/3/library/concurrent.futures.html>

On providers where the model rarely fans out on its own for simple prompts, you can nudge it in the system prompt: *"If the user asks about multiple items, request the tool for all of them in one turn."* Do not force it with `tool_choice` — the exercise is about handling parallelism, not forcing it.

## Acceptance criteria

- On prompt 1 with the happy-path loop, your assistant returns a coherent answer that names the correct capital and a plausible current time for each of the three countries.
- Your log shows one assistant turn with multiple tool_use blocks, and one follow-up user turn containing exactly the same number of tool_result blocks.
- The per-turn wall-clock latency is within ~150 ms of the *slowest* individual tool, not close to the sum of all three. If it is close to the sum, your executor is serialising — fix it before you submit.
- With `--swap-ids on`, the final answer is visibly wrong (e.g. Paris has Tokyo's time, or the capitals get mislabelled). Screenshot or paste the wrong answer into your notes. This is the invariant you are supposed to internalise.
- Missing one tool_result on purpose (do this once, then remove the change) causes the next API call to fail with a provider error. Note the exact error text — it will save you an hour someday.

## Stretch goals

- Add `parallel_tool_calls=False` (OpenAI) or `disable_parallel_tool_use=True` in `tool_choice` (Anthropic). Rerun prompt 1 and compare (a) total wall-clock latency, (b) total tokens billed. Which is bigger, and by how much?
- Replace the ThreadPoolExecutor with `asyncio.gather` and async tool implementations. Confirm the loop still works and that the log shows correct per-tool timings.
- Add a metric that reports the average fan-out width per user request over a small batch of prompts. If the average is 1.0, you are not really testing parallelism — try prompts that require more independent lookups.
