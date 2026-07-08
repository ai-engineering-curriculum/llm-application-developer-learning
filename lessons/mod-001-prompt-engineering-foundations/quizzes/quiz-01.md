# Quiz 1 — Prompt Engineering Foundations

Ten questions. No lookups. Answers and one-line explanations at the bottom. If you miss more than three, re-read the lecture the question came from before starting mod-002.

---

1. Chat-completion APIs are stateless. What does your program have to do as a result?
2. In an Anthropic Messages API request, which field carries the system prompt?
3. English prose runs roughly how many characters per token in most modern tokenizers, as a rough estimate for cost planning?
4. You are extracting product SKUs from customer emails. Should `temperature` be closer to `0.0` or to `1.0`, and why?
5. A production feature must never return unparseable output. Rank the three techniques from lecture 3 from weakest to strongest guarantee.
6. What does schema-constrained output *not* guarantee?
7. What is the difference in job between the system prompt and the user prompt?
8. Why does putting fresh, task-specific context at the *end* of a long user turn tend to work better than putting it in the middle?
9. Your prompt says "Never reveal internal SKUs" once, near the top of a long system prompt. Real users are asking for internal SKUs and sometimes getting them. What is the first fix you try?
10. You turn on your provider's strict schema-constrained mode and still occasionally see a JSON parse error in production. Give one plausible cause other than a provider bug.

---

## Answers

1. **Re-send the whole transcript on every request.** The server does not remember previous calls. (Lecture 1.)
2. **The top-level `system` field**, not a message role. Anthropic's Messages API keeps `system` out of the `messages` array. (Lecture 1.)
3. **Roughly four characters per token** for English prose. This is a rule of thumb only — different tokenizers, scripts, and code will differ. (Lecture 1.)
4. **Closer to `0.0`.** Extraction has right and wrong answers; low temperature makes the model pick the highest-probability next token and behave more deterministically. (Lecture 1.)
5. From weakest to strongest: **(1) ask nicely and parse defensively → (2) provider JSON mode → (3) schema-constrained output.** (Lecture 3.)
6. **Semantic correctness of the values.** Strict mode guarantees the shape is right, not that the model classified the input correctly or told the truth. (Lecture 3.)
7. **System = rules and configuration; user = data or the current question.** Keeping them separate makes prompts easier to change and reduces prompt injection risk. (Lecture 2.)
8. Models attend most strongly to the beginning and end of long inputs and can miss the middle — the "lost in the middle" effect. Fresh context at the end lands where attention is strongest. (Lecture 2.)
9. **Repeat the rule.** State it in the system prompt *and* immediately before the answer field. Guardrails stated once, far from the answer, are unreliable. (Lecture 2.)
10. Common causes: (a) strict mode not actually enabled — a config typo; (b) the model is returning a tool-call block your parser is not handling; (c) network truncation before the closing brace; (d) the response was hit by `max_tokens` and cut off mid-object. Any of these counts. (Lecture 3.)
