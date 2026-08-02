# Exercise 09 — Semantic vs syntactic correctness

Paired with [chapter 4 — schema-constrained JSON output](../04-schema-constrained-json-output.md) and [chapter 5 — diagnosing prompt failures](../05-diagnosing-prompt-failures.md).

**Estimated effort:** 15 minutes.

## Objective

See, in one line of code, the difference between "the shape is guaranteed" and "the content is right." This is the exercise that sets up why mod-006 exists.

## Problem statement

Take the schema-constrained call from exercise 08 (two keys — `label` string, `confidence` number 0..1).

Send it the empty string as input:

```python
messages = [{"role": "user", "content": ""}]
```

Run it three to five times. Look at the values the model puts in your required fields.

Then answer, in two or three sentences: what did the model do? What does that tell you about the guarantee schema-constrained output gives you?

## Requirements

- The user turn's content must be the empty string (`""`), not a whitespace string or an "N/A" sentinel. You are testing what happens with genuinely no input.
- Do **not** add prompt logic to reject empty input. The point of the exercise is to see what strict mode does when the input gives it nothing to work with.
- Log the raw payload — do not filter or interpret before writing it down.

## Starter guidance

- Different models will do different things. Some will emit a plausible-looking default (`{"label": "other", "confidence": 0.5}`). Some will produce a confident-sounding but arbitrary label. Some will refuse. All three are informative.
- If your provider returned a refusal instead of a schema-conforming payload, note the refusal reason and treat it as a signal that "strict mode + policy + empty input" is a corner your production code has to handle explicitly.

## Acceptance criteria

- Your notes contain three to five raw payloads produced from an empty user turn.
- You can articulate, in one sentence, why "the JSON parsed and the schema validated" is not enough to call this call *successful* from a product perspective.
- You have identified at least one input-validation check your program should do *before* calling the model, so that this failure mode never reaches the API.

## Stretch goals

- Repeat the exercise with three other adversarial inputs:
  - A very long string of `"a"` characters (well within the context window).
  - A single word in a language the model probably does not speak well.
  - A prompt-injection attempt like the one from exercise 06.
  
  For each, note whether the failure is caught by the schema (probably not) or requires a value-level check (probably yes).

- Draft the shape of a `validate_payload(payload) -> bool` function that would catch the semantic failures your notes identified. You will build something like this for real in mod-006.
