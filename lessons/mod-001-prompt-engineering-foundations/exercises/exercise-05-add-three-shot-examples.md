# Exercise 05 — Add three-shot examples

Paired with [chapter 2 — shaping prompts for a reliable format](../02-shaping-prompts-for-reliable-format.md).

**Estimated effort:** 25 minutes.

## Objective

Measure — with your own eyes and a small tally — how few-shot examples affect format reliability. This is the exercise most people finish believing few-shot prompting is unreasonably effective.

## Problem statement

Pick a small binary classification task. Any of these are fine:

- spam vs not-spam
- positive vs negative sentiment
- on-topic vs off-topic

For the same task, write three versions of the prompt:

1. **Zero-shot.** System prompt states the rules and the required output format. No examples.
2. **One-shot.** Same as zero-shot, plus one `user`/`assistant` example pair before the real user turn.
3. **Three-shot.** Same as one-shot, but with three example pairs — chosen to cover different shapes of input, not three easy cases.

Take five inputs. Run each input through each version. Record, in a small table, whether the output was **exactly** the format you asked for.

## Requirements

- Use the same model, the same temperature, and the same system prompt across the three versions. Only the number of example pairs changes.
- "Exactly the format" means byte-for-byte parseable by your intended parser. A stray `"Sure! "` prefix does not count as correct.
- The three-shot examples must not be identical to any of your five test inputs. That is cheating; you are measuring generalisation.

## Starter guidance

- If you cannot think of five test inputs, five real messages from a project you own (redacted if needed) are better than five invented ones. Real inputs surface real edge cases.
- If your five inputs are all easy for zero-shot, add a harder one. The exercise is only informative if some version of the prompt fails.

## Acceptance criteria

- Your notes contain a 3-row × 5-column tally of hit/miss for the three prompt versions across the five inputs.
- You can articulate one *shape* of failure the zero-shot version had that the three-shot version did not. Common answers: format drift (wrapping in code fences), label leakage ("This is spam because…"), label vocabulary drift ("negative" vs "Negative" vs "neg").
- You did not confuse "the answer was more accurate" with "the format was more consistent." This exercise is about format.

## Stretch goals

- Run each cell of the tally five times (not just once) and record the format-hit rate as a fraction. This is a first hands-on look at what a real evaluation loop feels like — and why mod-006 exists.
- Try a version with *five* examples. Does the format-hit rate improve over three-shot enough to justify the extra input tokens? Compute the cost delta.
