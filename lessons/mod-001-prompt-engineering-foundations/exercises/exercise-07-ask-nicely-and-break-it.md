# Exercise 07 — Ask nicely and break it

Paired with [chapter 4 — schema-constrained JSON output](../04-schema-constrained-json-output.md).

**Estimated effort:** 20 minutes.

## Objective

Measure the failure rate of "ask nicely for JSON" so that you internalise how much stronger schema-constrained output is. This exercise establishes the baseline that exercise 08 improves.

## Problem statement

Write a prompt (system + one user turn) that asks the model to reply with a JSON object containing **exactly two keys**, e.g.:

- `label`: a string
- `confidence`: a number between 0 and 1

Do not use the provider's JSON mode or schema-constrained output. This is the "ask nicely" tier from chapter 4.

Run the same request **ten times** with `temperature=1.0`. Attempt to parse each reply with `json.loads` (or the TypeScript equivalent).

Count and record:

- How many parses succeeded.
- How many parses failed.
- For each failure, one line describing what went wrong (code fence, prefix, malformed key, truncation, extra field, …).

Save one of the malformed outputs to a file (`malformed_example.txt` is fine) — you will refer back to it in exercise 08.

## Requirements

- Temperature must be **1.0**. Low temperature will mask the point of the exercise.
- Use the same prompt on all ten runs. Do not randomise anything except the sampler.
- Do not "clean up" outputs (strip code fences, chop off preambles) before parsing. Parse the raw response text.

## Starter guidance

- If your prompt is very strict ("Reply with only a valid JSON object. Do not include any other text."), the failure rate may be lower than 10/10 — that is expected. You are measuring, not maximising, the failure rate.
- If you get zero failures across ten runs, add a subtle instability to the prompt: ask for the JSON to be inside a paragraph, or ask the model to explain its reasoning first. The point is to see failures, not to hide them.

## Acceptance criteria

- Your notes contain a tally: `X of 10 parsed cleanly, Y of 10 failed`.
- Your notes have at least one line per failure describing the failure shape.
- `malformed_example.txt` (or your equivalent) contains one raw malformed response you can point at.

## Stretch goals

- Redo the ten runs at `temperature=0.0`. Compare the failure rate. Then decide, from the numbers you just gathered, whether "lower temperature" is a reliable fix for format drift. (Preview: it helps, but it does not close the gap the way schema-constrained mode does.)
- Rewrite your prompt following every hygiene rule from chapter 2 (system separation, delimiters, one-shot example). Rerun at `temperature=1.0`. Did the failure rate drop? Enough?
