# Chapter 5 — Diagnosing prompt failures

The four previous chapters gave you a working prompt. This chapter is about what to do when it stops working. Every LLM feature you ship will misbehave — the question is whether you can recognise the failure quickly and go straight to the fix, or whether you spend two afternoons flailing. Three failure shapes account for most of what you will see: **hallucination**, **refusal**, and **format drift**. Learn to name them on sight.

## Motivation

Debugging LLM output is unlike debugging deterministic code in one important way: the same prompt does not necessarily produce the same output twice. That means log-and-repro workflows built for classical software need a different discipline here. Naming the failure shape narrows the search — a hallucination is not fixed the way a refusal is, and a refusal is not fixed the way format drift is. If you get the diagnosis wrong, you will "fix" the wrong thing and be surprised when the bug returns.

## The three shapes

### Hallucination

**What it looks like:** the model produced a confident, fluent, wrong answer. It cited a function that does not exist. It quoted a policy that was never written. It gave you a product SKU it invented on the spot. The output is well-formed and often internally consistent — that is why it is dangerous.

**Why it happens:** the model was asked to answer without enough grounding, or the grounding it did have was buried in a place the model did not attend to. Its underlying job is to produce the *most probable* next tokens given what it has seen, and "plausible" is not the same as "true."

**First debugging step: check what the model actually saw.** Log the full prompt (system + user + any retrieved context) that produced the bad answer, and read it top to bottom as if you were the model. Ask two questions:

1. Was the answer *derivable* from what was in the prompt? If not, the fix is upstream: add grounding, retrieve better context, or route to a workflow that has access to the truth. This is where mod-005 (retrieval) comes in.
2. Was the answer *contradicted* by what was in the prompt? If yes, the fix is prompt-level: tighten the instructions, move the source-of-truth closer to the answer field, or ask the model to cite which line of the input its answer comes from.

**What NOT to do first:** do not lower `temperature` and call it done. Lower temperature makes the same wrong answer more consistent, not more true. Address the grounding before you address the sampling.

### Refusal

**What it looks like:** the model returned "I cannot help with that", or a policy-shaped apology, on a request you know is benign. Or its structured-output response arrived with a `refusal` field instead of your schema. The model chose not to answer.

**Why it happens:** the request tripped a safety classifier, or the prompt was worded in a way the model reads as adversarial ("pretend you are…", "for this exercise…"), or you asked for content the provider's policies genuinely restrict.

**First debugging step: read the refusal message, then read your prompt.** The refusal usually names the category the model thought you were in. Two questions:

1. Is the refusal *correct*? If you asked for something the provider explicitly restricts (self-harm instructions, election-influence material, etc.), the answer is to change the product, not the prompt. Do not spend a day trying to trick a safety filter.
2. Is the refusal a *false positive*? If yes, the fix is usually to make the intent obvious. Add context about the setting ("you are a security engineer reviewing a codebase for vulnerabilities…"), remove trigger phrases from user-supplied text before you send it, or split the request into smaller pieces where each piece is unambiguous.

**What NOT to do first:** do not blindly retry with a slightly different phrasing until it "works." That path teaches the model nothing and teaches you nothing — you are just hoping the sampler flips. Fix the prompt so the correct behaviour is obvious.

### Format drift

**What it looks like:** the response used to be clean JSON and now is JSON wrapped in ```json fences. The `category` field used to be lowercase and now sometimes has a capital letter. A new field appears that was not in the schema. The reply drifted from what your parser expects, and it drifts more often on some inputs than others.

**Why it happens:** three common causes.

1. **Weak or missing structural constraints.** If you are relying on prose instructions to hold the format ("respond with JSON only"), any variation in input can perturb the output shape. Ask-nicely (chapter 4, technique 1) drifts. Schema-constrained mode (technique 3) does not.
2. **A model change.** The provider silently rolled the underlying model, or you upgraded to a new model version, and the new one interprets your prompt slightly differently.
3. **A prompt change you did not think mattered.** You reordered a paragraph, added a comma, removed a redundant sentence — and the format broke on 3% of inputs.

**First debugging step: check the structural guarantee, not the wording.** Two questions:

1. **Are you using the strongest structured-output mode your provider supports?** If not, that is your first fix. Do not spend an hour rewriting instructions when you could turn on `strict: true` in ten minutes.
2. **Is the strong mode actually enabled?** This is the single most common miss. A typo in the `response_format` object, a missing `strict: true`, extracting the wrong field from the response — any of these produce silent regression to ask-nicely mode.

**What NOT to do first:** do not add "Please respond with valid JSON only. This is very important." to the prompt. Prose reinforcement of a structural rule is a smell — the correct fix lives in the API call, not in the string.

## A quick triage flowchart

```
Response arrived → parses cleanly?
├─ No  → format drift → check strict mode is on and the right field is extracted
└─ Yes → answer is correct?
        ├─ Yes → done
        └─ No  → answer is a refusal?
                ├─ Yes → refusal → read the reason, decide "product change" vs "prompt change"
                └─ No  → hallucination → log full prompt, check grounding
```

Print this out. Tape it to your monitor for the first month. After that you will have internalised it and can throw it away.

## The most common debugging mistakes

- **Debugging on one bad example.** LLM output is a distribution. One bad reply may be a tail event. Before you change the prompt, run it on 10–20 inputs (real ones, from your logs) and see whether the failure is systemic or a one-off.
- **Debugging without logs.** If you do not have the full prompt that produced the bad response, you are guessing. Log the model, the system prompt, the messages, the sampling parameters, the usage block, and the stop reason on every call from day one. Redact PII before it hits the log store, but log everything else.
- **Changing more than one thing at a time.** You added a few-shot example *and* lowered the temperature *and* re-worded the system prompt. When the behaviour improves, you do not know which change did it. Change one thing. Re-measure. Then change the next.
- **Trusting a single-run comparison.** Because output is non-deterministic, "the new version worked once" is not evidence. Run each version on the same input set enough times to see the distribution. This is the shape of an evaluation, which is why mod-006 exists.

## Reproducing a bad response

For the runs where you *can* get a deterministic-ish reproduction:

- **Set `temperature=0`.** Not fully deterministic on most providers, but the closest you can get.
- **Pin the model version.** If the provider offers dated snapshots (e.g., `claude-opus-4-7-20260601`), pin the exact one for your repro. Otherwise a background model roll can move the ground under you.
- **Capture the full request.** All messages, the system prompt, the tool definitions if any, the sampling parameters, the `max_tokens` cap.
- **Capture the full response.** Every content block, the stop reason, the usage counts, the refusal field if any.

With those in hand, replaying the call locally is a one-liner. Without them, you are relying on the user to describe what happened.

## When you cannot get a clean repro

Not all failures are reproducible. A one-in-ten-thousand format break at `temperature > 0` may never happen again on the same input. Two habits help:

1. **Turn every "did not repro" into a monitored alert.** If format drift happens 0.01% of the time in production, you will not catch it locally, but a dashboard on your parse-failure rate will.
2. **Add a regression test the moment you understand the failure class.** Even if you cannot repro the exact string, you can usually construct an input that stresses the same weakness and pin it in your test set. Mod-006 turns this habit into a workflow.

## What to remember

- Three failure shapes cover almost all of what you will see: hallucination, refusal, format drift.
- Hallucination is a grounding problem — fix the prompt's inputs, not its wording.
- Refusal is a policy problem — read the reason, decide whether to change the product or clarify the intent.
- Format drift is a structural-guarantee problem — reach for schema-constrained mode before you rewrite prose instructions.
- Debug on a distribution, not a single example. Change one thing at a time. Log everything.
- A bug you cannot repro is a monitoring gap, not a mystery.

You have finished mod-001. The next module (`mod-002-tool-and-function-calling`) picks up the same message-list shape and adds tools — the model now not only writes text but also calls back into your program.
