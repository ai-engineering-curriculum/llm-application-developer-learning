# mod-007-shipping-a-first-llm-feature: Shipping a First Production LLM Feature

> Scaffolded by `aicg org execute-plan`. Lecture chapters and exercise content are authored on subsequent autonomous cycles.

**Estimated effort:** 12 hours

## Learning objectives

- Wire an LLM call into an existing HTTP API (FastAPI or comparable) as an endpoint that validates its input, streams its response, and enforces a per-request token / cost budget
- Manage API keys and provider secrets safely with environment variables and a per-environment secret store — never commit keys, never log them in traces
- Emit basic traces (request id, prompt fingerprint / hash, input token count, output token count, latency, cost estimate) to stdout or a lightweight sink; know what to include and what to redact
- Add feature flags for provider / model swaps so a bad model rollout can be rolled back without a code deploy
- Write a runbook a colleague can use during an incident — the three most-likely failure modes for this feature, the immediate response, and the escalation contact
- Introduce prompt injection and PII handling from the *application-developer* perspective — sanitise user-supplied context before concatenating into a prompt, validate LLM output before it triggers a side-effect. Route to `applied-ai-engineer-learning` (level 30) for full production guardrails and to `ai-risk-engineer-learning` (level 25) for adversarial red-team engineering depth
- Draw the boundary to `applied-ai-engineer-learning` (level 30) for full cloud deployment (AWS Bedrock, GCP Vertex, Azure OpenAI), CI/CD, IaC, and platform-scale observability

## Structure

- `01-…md` … `0N-…md`: lecture chapters.
- `exercises/`: per-exercise prompts.
- `labs/`: long-form hands-on labs.
- `quizzes/`: knowledge checks.
- `resources.md`: external references.
