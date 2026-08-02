# LLM Application Developer Curriculum

**Role level:** 20 (application-developer craft — AI Engineering family)
**Family:** AI Engineering
**Status:** partly shipped, partly planned. `mod-001` is authored and released; `mod-002` through `mod-007` and the two projects are the planned scope, authored from [`.aicg/curriculum-plan.json`](.aicg/curriculum-plan.json). Lessons will be drafted by subsequent autonomous content cycles.

## Overview

This track teaches the *application-developer craft* of talking to hosted LLMs end-to-end for a level-20 practitioner: calling the API and reasoning about tokens / cost / model-selection → engineering prompts and forcing schema-constrained structured output → declaring tools / functions and handling the tool-call → tool-result loop → orchestrating async / streaming calls with retries and honest tail-latency reporting → using prompt caching and cascade routing to keep cost defensible → grounding an LLM answer with a minimal retrieval path → gating a feature on a golden-set regression suite → shipping the whole thing behind a small HTTP API with feature flags, safe secrets, basic traces, a runbook, and input / output validation.

It is a **level-20 application-developer track in the AI Engineering family**, and a **peer** to `agentic-ai-developer-learning` (also level 20). The two tracks split the level-20 LLM-application space: this track owns *non-agentic LLM-app patterns* (single-turn to short-context multi-turn, tool calls at the API-surface level, streaming, cost) and the peer owns *agent frameworks and multi-step agent orchestration* (LangChain / LangGraph / LlamaIndex / CrewAI / AutoGen / Semantic Kernel, ReAct-style loops, memory, multi-agent coordination). Depth beyond the level-20 tier is delegated to the level-30 peers: RAG pipeline engineering to `rag-engineer-learning`, application-layer evaluation and observability to `ai-eval-engineer-learning`, cloud deployment / CI/CD / IaC / production guardrails / fine-tuning to `applied-ai-engineer-learning`.

Total planned commitment: **70 hours across 7 modules** + **50 hours across 2 projects** = **~120 hours**.

## Ownership rule

Following the project-wide ownership rule — assign primary coverage to the lowest-level role that genuinely requires the skill; higher-level tracks link to that owner unless additional depth, architectural context, or leadership scope is required — this curriculum:

- **Owns** the LLM-application-developer craft: hosted LLM API usage (OpenAI / Anthropic / comparable); prompt engineering and schema-constrained structured output; tool / function calling at the API surface; token cost, latency, and model-selection trade-offs including provider prompt caching; streaming responses and async orchestration; a *minimal* retrieval-grounded prompt path; a *minimal* golden-set regression check; and shipping a first production LLM feature behind an HTTP API with feature flags, secrets, traces, a runbook, and input / output validation.
- **Defers down** to [`ai-infra-junior-engineer-learning`](https://github.com/ai-infra-curriculum/ai-infra-junior-engineer-learning) (level 10) for Python packaging / HTTP APIs / unit testing / logging / Docker basics; and to the standard software-engineering prerequisites captured in [`PREREQUISITES.md`](PREREQUISITES.md) for Python fluency (async, pydantic, typed data), backend service frameworks (FastAPI, GraphQL, webhooks), and SQL — see [`JOB_REQUIREMENTS.md`](JOB_REQUIREMENTS.md) for the postings evidence that these are entry skills, not this track's teaching load.
- **Defers sideways** to [`agentic-ai-developer-learning`](https://github.com/ai-engineering-curriculum/agentic-ai-developer-learning) (peer, level 20) for agent frameworks (LangChain / LangGraph / LlamaIndex / CrewAI / AutoGen / Semantic Kernel) and multi-step agent design patterns (ReAct, planning loops, memory, multi-agent coordination).
- **Defers up** to [`rag-engineer-learning`](https://github.com/ai-engineering-curriculum/rag-engineer-learning) (level 30) for full RAG pipeline depth — chunking strategies, embedding-model selection, vector-DB internals (Pinecone / Weaviate / Qdrant / Chroma), retrieval-quality evaluation, re-ranking; to [`ai-eval-engineer-learning`](https://github.com/ai-engineering-curriculum/ai-eval-engineer-learning) (level 30) for application-layer evaluation depth (LLM-as-a-judge calibration, RAG eval, online eval), observability / tracing stacks (Langfuse, OpenTelemetry, Datadog), and cost / latency / quality dashboards; to [`applied-ai-engineer-learning`](https://github.com/ai-engineering-curriculum/applied-ai-engineer-learning) (level 30) for production-hardened LLM apps — cloud deployment (AWS Bedrock, GCP Vertex, Azure OpenAI), containerisation / CI/CD / IaC, production guardrails (input validation, output filtering, prompt-injection defence, PII handling), and fine-tuning / LoRA basics; to [`ai-risk-engineer-learning`](https://github.com/ai-governance-curriculum/ai-risk-engineer-learning) (level 25) for the AI-risk-engineering craft — harm modelling, adversarial red-team programs, guardrail effectiveness measurement, EU AI Act Article 72 post-market surveillance; and to [`agentic-ai-engineer-learning`](https://github.com/ai-engineering-curriculum/agentic-ai-engineer-learning) (level 30) and [`senior-agentic-ai-engineer-learning`](https://github.com/ai-engineering-curriculum/senior-agentic-ai-engineer-learning) (level 40) for production agent engineering.
- **Out of scope** to fine-tuning depth (delegated to Applied AI Engineer) and to organisation-wide governance and program leadership (delegated to the governance family).

See [`JOB_REQUIREMENTS.md`](JOB_REQUIREMENTS.md) for the 31-posting sample and the requirement-to-coverage map behind the split.

## Module Plan

| Module | Title | Hours | Status |
|---|---|---|---|
| mod-001-prompt-engineering-foundations | Prompt Engineering Foundations | 6 | shipped |
| mod-002-tool-and-function-calling | Tool and Function Calling for LLM Applications | 12 | planned |
| mod-003-streaming-async-and-orchestration | Streaming, Async, and Parallel LLM Orchestration | 12 | planned |
| mod-004-model-selection-cost-and-prompt-caching | Model Selection, Cost, and Prompt Caching | 10 | planned |
| mod-005-retrieval-basics-for-llm-apps | Retrieval Basics for LLM Applications | 10 | planned |
| mod-006-minimal-eval-and-regression-checks | Minimal Evaluation and Regression Checks for LLM Features | 8 | planned |
| mod-007-shipping-a-first-llm-feature | Shipping a First Production LLM Feature | 12 | planned |

## Project Plan

| Project | Title | Hours | Status |
|---|---|---|---|
| project-001-multi-provider-comparison-report | Multi-Provider Comparison Report for One LLM Feature | 20 | planned |
| project-002-grounded-tool-calling-llm-feature | Grounded, Tool-Calling, Streaming LLM Feature Behind an HTTP API | 30 | planned |

## Module summaries

### mod-001 — Prompt Engineering Foundations *(shipped)*
Ship your first small text-classifier that calls a real hosted LLM API, returns typed JSON, and behaves predictably. Covers the chat-completion message-list model shared by OpenAI and Anthropic, system-prompt / few-shot / delimiter patterns, token estimation with `tiktoken` and the Anthropic token-counting endpoint, JSON-schema-constrained output modes, and the three most common shapes of prompt failure (hallucination, refusal, format drift). Nine short warm-up exercises paced to the three lectures, one lab (`text-classifier`), and a ten-question quiz.

### mod-002 — Tool and Function Calling *(planned)*
Close the "partial" coverage flagged in [`JOB_REQUIREMENTS.md`](JOB_REQUIREMENTS.md) — 18 of 31 postings (58%) call for tool / function calling. Declare typed tool schemas for OpenAI and Anthropic, run the request → tool_call → tool_result → response loop end-to-end, handle parallel tool calls in a single turn, defend the boundary between "the model chose to call the tool" and "the tool got malformed arguments". Draws the peer-boundary to `agentic-ai-developer-learning` for multi-step agent planning and ReAct-style loops — this module teaches the API surface, the peer teaches the orchestration.

### mod-003 — Streaming, Async, and Parallel LLM Orchestration *(planned)*
Adopt the streaming / async gap that sat at 26% in the 2026-08 posting sample (below the delta's 30% adoption threshold, but level-20 territory this role owns). Consume SSE streams from OpenAI and Anthropic without buffering to completion, stream partial JSON safely, fan out async calls with a bounded concurrency limit, retry with backoff + jitter while distinguishing transient from terminal errors, and report p50 / p95 / p99 latency and time-to-first-token honestly.

### mod-004 — Model Selection, Cost, and Prompt Caching *(planned)*
Extend the token-cost basics that live in mod-001 into a production-shaped skill. Compare small vs. frontier models on a real feature with an A/B benchmark, estimate cost per call and enforce per-request / per-user token budgets, use Anthropic prompt caching and OpenAI cached-input mechanisms and measure the hit ratio in a real workload, cascade-route between cheap and expensive models, and design a graceful degradation for a provider outage. 10 of 31 postings (32%) explicitly ask for this skill.

### mod-005 — Retrieval Basics for LLM Applications *(planned)*
A deliberately *minimal* retrieval intro — 27 of 31 postings (87%) ask for RAG, but the depth is delegated to [`rag-engineer-learning`](https://github.com/ai-engineering-curriculum/rag-engineer-learning) (level 30). This module teaches enough to ship a corpus-grounded Q&A prototype: generating embeddings with a hosted API, storing them in `pgvector`, retrieving top-k passages by cosine similarity, assembling a grounded prompt with citations, and triaging retrieval-quality vs. prompt failure. Explicitly stops short of chunking-strategy design, embedding-model selection, vector-DB internals across providers, retrieval-quality evaluation, and re-ranking — those are the RAG track.

### mod-006 — Minimal Evaluation and Regression Checks *(planned)*
Another deliberately *minimal* intro — 20 of 31 postings (65%) ask for evaluation frameworks, but the depth is delegated to [`ai-eval-engineer-learning`](https://github.com/ai-engineering-curriculum/ai-eval-engineer-learning) (level 30). This module teaches enough to gate a single LLM feature on regressions: authoring a 20 to 50 example golden set with defended sampling, running it as a pytest CI check, and scoring with rule-based / similarity / one LLM-as-a-judge check while knowing the calibration trade-off of each. Explicitly stops short of judge calibration methodology, RAG eval, online eval, observability / tracing stacks, and cost / latency / quality dashboards — those are the AI Eval Engineer track.

### mod-007 — Shipping a First Production LLM Feature *(planned)*
The connective-tissue capstone module. Wire an LLM call into a FastAPI (or comparable) endpoint that validates input, streams response, enforces a per-request token / cost budget, manages provider secrets safely, emits basic traces (request id, prompt hash, token counts, latency, cost estimate) with safe redaction, hides behind a feature flag so a bad model rollout is a config-only rollback, and comes with a runbook covering the three most-likely failure modes. Introduces prompt injection and PII handling at the *application-developer* boundary — sanitising user-supplied context before concatenating into a prompt and validating LLM output before it triggers a side-effect — while explicitly routing full production guardrails to [`applied-ai-engineer-learning`](https://github.com/ai-engineering-curriculum/applied-ai-engineer-learning) (level 30) and adversarial red-team engineering to [`ai-risk-engineer-learning`](https://github.com/ai-governance-curriculum/ai-risk-engineer-learning) (level 25).

## Project summaries

### project-001 — Multi-Provider Comparison Report *(planned, 20h)*
Implement one concrete LLM-backed feature against at least two providers (OpenAI + Anthropic, or one of those + an open-source hosted-inference model). Ship a shared interface both providers implement, a per-provider cost / latency measurement broken down by input / output tokens with p50 / p95 / p99 latency, a small golden set (≥20 examples) with rule-based + similarity scoring only (no LLM-as-a-judge to keep the compare provider-neutral), and a one-page recommendation the sponsoring engineer can act on. Integrates mod-001, mod-002 (if the feature uses tools), mod-004, and mod-006.

### project-002 — Grounded, Tool-Calling, Streaming LLM Feature *(planned, 30h)*
Ship a small production-shaped LLM feature end-to-end behind an HTTP API. Requirements include an SSE-streaming FastAPI endpoint, at least two tool / function-calling integrations against real APIs or a real database, a retrieval-grounded prompt path with source citations, provider prompt-caching enabled and measured, a pytest golden-set regression suite (≥30 examples) with rule + similarity + one LLM-as-a-judge score wired into CI, traces with request id / model / token counts / latency / cost estimate and no user PII, a feature flag for provider + model swap, and a runbook covering the three most-likely failure modes. Deliverable includes a routing note that names the level-30 tracks (`rag-engineer`, `ai-eval-engineer`, `applied-ai-engineer`) and the level-20 peer (`agentic-ai-developer`) for the depth this feature deliberately stops short of. Integrates mod-001 through mod-007.
