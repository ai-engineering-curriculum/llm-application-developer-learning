# Exercise 08 — Turn on schema-constrained output

Paired with [chapter 4 — schema-constrained JSON output](../04-schema-constrained-json-output.md).

**Estimated effort:** 25 minutes.

## Objective

Prove to yourself, by running the same experiment twice, that schema-constrained output is a categorical improvement over "ask nicely." This exercise is the second half of exercise 07 — do exercise 07 first.

## Problem statement

Take the same prompt from exercise 07 (two keys — `label` string, `confidence` number 0..1). Enable the strongest structured-output mode your chosen provider supports:

- **OpenAI**: `response_format={"type": "json_schema", "json_schema": {..., "strict": true}}`. See <https://platform.openai.com/docs/guides/structured-outputs>.
- **Anthropic**: define a tool whose `input_schema` is your JSON schema and force it with `tool_choice={"type": "tool", "name": "..."}`. See <https://docs.anthropic.com/en/docs/build-with-claude/tool-use>.

Rerun the request **ten times** at `temperature=1.0`. For each reply, extract the payload (from the tool-use block on Anthropic, or the parsed JSON on OpenAI) and confirm that:

- It parses as JSON.
- It has exactly the required keys.
- The types match.

Count and record the failure rate. It should be zero.

## Requirements

- Same model, same temperature, and same underlying prompt as exercise 07. This is a controlled comparison.
- **`strict: true`** (OpenAI) or the equivalent `tool_choice` forcing (Anthropic) must actually be enabled. Missing this flag is the single most common miss for this exercise.
- Extract the payload from the correct field. On Anthropic, the tool-use payload is `block.input`, not `block.text`. On OpenAI Structured Outputs, the parsed object is on the message — do not re-parse a stringified version of it.

## Starter guidance

- Copy the exact JSON schema example from [chapter 4](../04-schema-constrained-json-output.md) and adjust the keys to match exercise 07. You do not need to design a new schema.
- Do not add prose reinforcement ("please respond with valid JSON") to the system prompt for this run. The structural mode is doing that job now; keeping the prompt otherwise identical to exercise 07 makes the comparison honest.
- If your provider does not support schema-constrained output at all, fall back to JSON mode (OpenAI `response_format={"type": "json_object"}`) and note that the failure rate should still drop dramatically compared to exercise 07 — but may not hit zero.

## Acceptance criteria

- You have a tally comparable to exercise 07's: `X of 10 parsed and validated, Y of 10 failed`. `Y` should be 0 (or near 0 if you fell back to JSON mode without a schema).
- If any reply *failed* to parse or validate, you have investigated and can name the root cause. Most likely causes: strict mode not actually enabled, wrong field extracted from the response, `max_tokens` cap cut the response off before the closing brace.
- You can articulate, in one sentence, what schema-constrained output *does not* guarantee. (See "What 'strict' actually guarantees" in chapter 4.)

## Stretch goals

- Feed the run an input designed to be genuinely ambiguous (e.g., a message that is both a complaint and a bug report). Note what the model puts in `confidence` — this is your entry point to exercise 09.
- Time the ten calls in constrained mode vs the ten calls from exercise 07. Is one meaningfully slower? Log the numbers — this becomes relevant when you get to latency budgets in mod-003.
