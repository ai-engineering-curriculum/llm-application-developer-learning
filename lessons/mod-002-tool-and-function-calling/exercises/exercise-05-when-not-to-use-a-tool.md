# Exercise 05 — When NOT to use a tool

Paired with [chapter 5 — when to reach for a tool call (and when not to)](../05-when-to-reach-for-a-tool-call.md).

**Estimated effort:** 45–75 minutes.

## Objective

Practise the design judgment that chapter 5 introduces. For a set of feature briefs, decide whether the task warrants a tool call, structured output, or plain generation — and back the decision with a small measurement, not just a preference. The point is that "always add a tool" is a real anti-pattern and one you should catch yourself falling into.

## Problem statement

You are given five feature briefs. For each, produce two things:

1. A **decision**: `plain-generation`, `structured-output`, `single-tool-call`, `multi-tool-loop`, or `out-of-scope-for-this-module` (i.e. an agent).
2. A **short justification** — three or four sentences — grounding your choice in the trade-offs from chapter 5: cost, latency, reliability, and whether the model needs data or side effects it cannot produce on its own.

Then implement two of the five briefs (your choice) and run a small side-by-side measurement — plain generation vs. tool call — on the same input. Compare wall-clock latency, token counts, and a rough correctness rate on a handful of inputs. Confirm your decision holds up empirically, or update it.

## The briefs

1. **Ticket classifier.** Given a support email, produce a label (`bug`, `feature`, `question`) and a priority (`low`, `med`, `high`). Historical data suggests plain classification with a few-shot prompt already hits ~90% agreement with human labels.
2. **Booking-status lookup.** The user asks: *"What's the status of booking ABC-123?"* Your product owns a booking service that exposes `get_booking(booking_id) -> dict` behind an internal auth boundary.
3. **Email summariser.** Given the body of a long email, produce a three-sentence summary plus a bulleted list of action items.
4. **Currency converter.** The user asks: *"How much is $123.45 in euros?"* You have access to a live FX API (`get_fx_rate(from, to) -> float`).
5. **Multi-step incident triage.** The user reports a service outage. The assistant must (a) look up recent deploys, (b) fetch the current on-call, (c) check a couple of dashboards, (d) decide whether to page the on-call, and (e) draft an incident report. Each step's action depends on the previous step's output.

## Requirements

1. For each of the five briefs, write your decision + justification in a `decisions.md` (or equivalent) file. Keep each justification to three or four sentences.
2. For **two** of the briefs — one where your decision is "no tool" and one where it is "yes tool" — implement two variants of the feature and measure them side by side:
   - **Variant A:** the "no tool" (or lower-tier) approach.
   - **Variant B:** the "yes tool" (or higher-tier) approach.
   - Run each variant on 5–10 inputs. Record: wall-clock latency, total tokens billed (input + output), and a rough correctness rate (a human can eyeball this — you are not building an eval suite yet).
3. Compare the numbers. If your original decision is inconsistent with the measurement, update the justification — and note *why* the measurement changed your mind.

## Starter guidance

- Chapter 5 gives you the framework. Do not skip step 3 (marginal-cost check).
- If you implement brief 5 (multi-step incident triage), stop after noting the design; do not build it. It is out of scope for this module and belongs in the peer `agentic-ai-developer-learning` track.
- Prompt caching (mod-004) will lower some of the tool-call cost you observe here. It is fine to note "this looks expensive today but will be cheaper once caching is on" in your justification.
- The "correctness rate" measurement is deliberately loose. Mod-006 turns this into a proper evaluation workflow. For this exercise, human eyeballing on ~10 inputs is enough.

## Acceptance criteria

- `decisions.md` (or equivalent) contains one decision + one three-to-four-sentence justification for each of the five briefs.
- Two of the five have paired implementations (Variant A + Variant B).
- Your measurement table is populated for both variants of both briefs and records at least: latency (median and p95), tokens billed, and correctness rate on 5–10 inputs.
- Your write-up includes at least one sentence per implemented brief comparing the two variants: which one you would ship, and why.
- Brief 5 is marked "out of scope for this module" and the write-up names the peer track that would handle it.

## Reference decisions (do not read until after you have written yours)

Once you have committed your own decisions, compare them against these reference answers:

1. **Ticket classifier** → `structured-output`. It is a single-turn typed output, no external data, no side effects. Adding a tool is pure overhead.
2. **Booking-status lookup** → `single-tool-call`. The booking data is behind an auth boundary the model does not have.
3. **Email summariser** → `plain-generation` (with structured output if you need parseable action items). Summarisation is core model competence; no data or side effect required.
4. **Currency converter** → `single-tool-call`. Rates are volatile and the model should not guess them; deterministic lookup is exactly what tools are for.
5. **Multi-step incident triage** → `out-of-scope`. Sequential, dependent, plan-revising work — this is the peer track's domain.

If your decisions matched all five, you have internalised the framework. If any diverged, re-read chapter 5 and note *which* rule you missed — the framework is useful only if you can recall it in the moment, not just recognise it after the fact.

## Stretch goals

- For brief 2 (booking-status lookup), build a version that puts the booking data *directly into the system prompt* on the first turn (a lookup you did in code before ever calling the model) instead of a tool call. Compare latency, cost, and how you would keep the prompt fresh in a real system. When would each design win?
- For brief 4 (currency converter), build a version that uses structured output to have the model *state* the conversion formula, then compute the result in Python without ever calling the FX API. What breaks when the exchange rate is stale? What breaks when the model gets the formula subtly wrong?
- Sketch (do not implement) an agent-style version of brief 5 in the peer `agentic-ai-developer-learning` track's vocabulary — plan, act, observe. What makes this feature genuinely agentic rather than a long tool-call loop?
