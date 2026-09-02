# mod-007 — Shipping a First Production LLM Feature

Your seventh and final module on the LLM Application Developer track. Mod-001 taught you how to shape a prompt, mod-002 how to drive tool calls in a loop, mod-003 how to stream and orchestrate concurrent requests, mod-004 how to defend a model choice with numbers, mod-005 how to ground an answer in your own documents, and mod-006 how to keep the whole thing working with a golden-set gate. This module is where those pieces stop being scripts on your laptop and become a single feature another program can depend on: an HTTP endpoint with a validated schema, streamed output, a per-request budget, secrets that never touch the repo, redacted traces, a feature-flag-driven model swap, a one-page runbook, and the boundary defences that keep an unauthenticated caller from steering your production system.

By the end of the module you can wire an LLM call into a FastAPI (or comparable) endpoint that survives production traffic, manage keys and per-environment config on a defensible footing, emit a JSON trace record per request without leaking prompts or PII, ramp a model swap behind a flag with a two-minute rollback, write the one-page runbook a colleague can drive under time pressure, and add the three application-developer defences that stand between your endpoint and the two attack shapes it will actually see.

**Estimated effort:** ~12 hours (chapters ~2 hours; six exercises 8–10 hours; the rest is time reading provider docs and eyeballing your own trace stream).

## Prerequisites

- You have finished [`mod-001-prompt-engineering-foundations`](../mod-001-prompt-engineering-foundations/README.md) through [`mod-006-minimal-eval-and-regression-checks`](../mod-006-minimal-eval-and-regression-checks/README.md). Mod-003 chapter 1 (streaming) is what exercise 01's `StreamingResponse` is built on; mod-004 chapter 2 (cost estimation) is the arithmetic behind the per-request budget; mod-006's golden-set gate is what makes the exercise-04 rollout signal a real number, not a vibe.
- Comfort in Python at the level of running a FastAPI app locally, hitting it with `curl`, and reading the streamed response as it arrives. If the framework is new, the FastAPI first-steps guide is on the reading list — <https://fastapi.tiangolo.com/tutorial/first-steps/>.
- A working install of the OpenAI **or** Anthropic Python SDK. Pick one for the whole module and stay with it — the shape is the same, but straddling both while learning the shape is unnecessary work.
- A shell you can `curl -N` in and a text editor. No cloud account, no Kubernetes cluster, no observability platform — chapter 7 explains why those are deliberately out of scope.
- Roughly USD $2–5 of API credit across the whole module. Most exercises are a handful of calls; the exercise-04 synthetic-caller ramp is where the small amount of spend sits.
- One LLM-backed feature to ship. The chapters use a summariser as the running example; the JSON extractor from mod-001 exercise 08, the tool-loop agent from mod-002 exercise 03, or the grounded Q&A feature from mod-005 exercise 03 all work as substitutes.

If any of those are new to you, work through the FastAPI or provider quickstart before starting the chapters.

## Learning objectives

After finishing this module you will be able to:

1. **Wire an LLM call into an HTTP endpoint** on FastAPI (or comparable) that validates its request body with a Pydantic schema, streams its response through a `StreamingResponse`, and enforces a per-request token / cost budget with a proper 413 for oversize input and a hard `max_tokens` ceiling the caller cannot override upwards.
2. **Manage API keys and per-environment secrets safely** — no key in the repository, no key in a log, `.env` for local development with `.env.example` documenting the variable names, and a fail-loud startup path when a required secret is missing.
3. **Emit a redacted JSON trace per request** — request id, hashed user id, model, prompt fingerprint, input/output tokens, latency, cost estimate, stop reason, status — with a redaction filter at the logging boundary and a smoke test that proves the filter fires.
4. **Feature-flag a model or provider swap** so a bad rollout can be reverted in a config change, not a code deploy — with stable-per-user routing, a `bucket` field in the trace, and a flag layer that fails open when the flag source is unreachable.
5. **Write a one-page runbook** for the three most likely failure modes of your feature — symptom, three-to-five immediate steps with time budgets, a stop-condition, and an escalation rotation for each — and rehearse it once before you ship.
6. **Add the three application-developer defences** against prompt injection and PII leakage: structural framing of user text inside delimited blocks, strict validation of model output before any side effect, and boundary sanitisation (length cap, delimiter escape, Unicode normalisation). Know what is *not* your job — the deep material lives in `ai-risk-engineer-learning` (level 25) and `applied-ai-engineer-learning` (level 30).
7. **Draw the boundary to `applied-ai-engineer-learning`** (level 30): AWS Bedrock / GCP Vertex / Azure OpenAI, IaC, CI/CD beyond a golden-set gate, container orchestration, platform-scale observability, cost governance across many features, production guardrail infrastructure, PII redaction at scale, multi-region failover, and async job platforms all belong there. Do not try to reproduce that depth here; graduate to the applied track when the second feature ships.

## Chapters

Read in order. Each chapter maps to one learning objective and builds directly on the last — the exercises assume the running summariser example accumulates across chapters.

| # | Chapter | Objective |
|---|---|---|
| 1 | [Wiring an LLM call into an HTTP API](01-wiring-an-llm-endpoint-into-an-http-api.md) | Objective 1 |
| 2 | [Managing keys and per-environment secrets](02-secrets-and-per-environment-config.md) | Objective 2 |
| 3 | [Emitting traces you can actually search, without leaking](03-traces-with-safe-redaction.md) | Objective 3 |
| 4 | [Feature-flagging model and provider swaps](04-feature-flags-for-model-and-provider-swaps.md) | Objective 4 |
| 5 | [Writing a runbook a colleague can use during an incident](05-writing-a-runbook-for-one-feature.md) | Objective 5 |
| 6 | [Prompt injection and PII from the app-developer perspective](06-prompt-injection-and-pii-for-app-developers.md) | Objective 6 |
| 7 | [The boundary to `applied-ai-engineer-learning`](07-boundary-to-applied-ai-engineer-track.md) | Objective 7 |

## Exercises

Hands-on drills paced to the chapters. Do the matching exercise **after** its chapter and **before** starting the next. Every exercise extends the same `first-feature/` repo — by the end you have one accumulated project, not six separate ones.

| # | Exercise | Chapter |
|---|---|---|
| 01 | [FastAPI LLM endpoint with streaming](exercises/exercise-01-fastapi-llm-endpoint-with-streaming.md) | 1 |
| 02 | [Secrets and env config](exercises/exercise-02-secrets-and-env-config.md) | 2 |
| 03 | [Trace emission with safe redaction](exercises/exercise-03-trace-emission-with-safe-redaction.md) | 3 |
| 04 | [Feature-flag a model swap](exercises/exercise-04-feature-flag-a-model-swap.md) | 4 |
| 05 | [Runbook for one feature](exercises/exercise-05-runbook-for-one-feature.md) | 5 |
| 06 | [User input sanitisation and output validation](exercises/exercise-06-user-input-sanitisation-and-output-validation.md) | 6 |

Chapter 7 does not have a paired exercise — it is the graduation chapter, and the exercise it maps to is "read the applied-track README once you are ready."

## Labs and quizzes

`labs/` and `quizzes/` are reserved for long-form hands-on work and knowledge checks authored in later cycles. If they are still empty when you get here, the exercises above are enough to cement the objectives — the six exercises together are a single shipped feature.

## Resources

Real, citable references for the topics in this module — FastAPI and Pydantic docs, provider SDK streaming and token-counting endpoints, secret-scanning tools, structured-logging libraries, feature-flag vendors, the twelve-factor logs standard, the OWASP LLM Top 10, and the applied-track handoff pointers. See [`resources.md`](resources.md).

## How to work through this module

1. Read chapter 1 and do exercise 01. The endpoint's shape — schema-in, stream-out, budget-checked — is the artefact every other chapter attaches to. If you skimp here, the later chapters have nothing to modify.
2. Read chapter 2 and do exercise 02. This is the moment `ANTHROPIC_API_KEY` stops being a line in your shell and becomes a real per-environment secret with a pre-commit hook standing between it and the repo.
3. Read chapter 3 and do exercise 03. Adding the JSON trace record turns the endpoint from a black box into something you can debug at 3 am. The redaction smoke test is the load-bearing test — do not skip it.
4. Read chapter 4 and do exercise 04. Rehearse the two-minute rollback. This is the drill you are practising for the day a silent model roll costs you your golden-set score at 4 pm on a Friday.
5. Read chapter 5 and do exercise 05. The one-page runbook is the *cheapest* piece of operational insurance you can write — and the one that gets skipped most often. Rehearse it against one simulated failure mode.
6. Read chapter 6 and do exercise 06. Structural framing, output validation, and boundary sanitisation are the *always-on* foundation under any guardrail service you eventually add. The three-test smoke suite in CI catches the failure modes worth catching without a full red-team.
7. Read chapter 7. It is short. It tells you what mod-007 is *not* trying to teach and where to go when you outgrow it — save yourself three months of accidentally reinventing platform infrastructure the applied track covers as its main material.

## What comes next

This is the last module of the LLM Application Developer track. If you have finished the exercises honestly, you can now *own* an LLM feature end to end for a small team — choose the model on evidence, ground it when the model needs facts it does not know, wire it into an endpoint that survives production traffic, keep it working with a gated eval, and respond to incidents when it does not.

The peer tracks pick up each of the specialised depths this track deliberately stopped short of:

- **[`applied-ai-engineer-learning`](https://github.com/ai-engineering-curriculum/applied-ai-engineer-learning) (level 30)** — the production platform track. Cloud-hosted managed inference (Bedrock, Vertex, Azure OpenAI), IaC, CI/CD, container orchestration, platform-scale observability, cost governance, guardrail infrastructure, PII redaction at scale, multi-region failover, async job platforms. Go here when the second or third feature ships, when you cross more than one host, or when a product decision hinges on multi-region latency or a compliance certification. Chapter 7 lists the specific signals.
- **[`ai-risk-engineer-learning`](https://github.com/ai-engineering-curriculum/ai-risk-engineer-learning) (level 25)** — the red-team engineering track. Adversarial suites, injection benchmarks, jailbreak taxonomies, indirect-injection defence at depth, threat modelling for agentic systems. Go here when your feature graduates from "public-facing summariser" to "agentic assistant with tools" — the risk surface changes shape, and the three defences in chapter 6 stop being enough.
- **[`ai-eval-engineer-learning`](https://github.com/ai-engineering-curriculum/ai-eval-engineer-learning) (level 30)** — the eval-depth track. Named in the mod-006 boundary chapter; still the right pointer when your eval story outgrows a single golden set with a pytest gate.

The right time to graduate to any of them is the moment their absence starts costing you more than their presence would.
