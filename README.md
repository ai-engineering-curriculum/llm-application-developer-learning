# AI Engineering · LLM Application Developer — Learning Repository

<!-- aicg:site-banner -->
> 🎓 Part of the free, open-source **AI Career Curriculum** ecosystem — [Infrastructure](https://github.com/ai-infra-curriculum) · [ML Engineering](https://github.com/ml-engineering-curriculum) · [AI Engineering](https://github.com/ai-engineering-curriculum) · [Governance](https://github.com/ai-governance-curriculum). Live cohorts &amp; team programs: **[ai-infra-curriculum.github.io](https://ai-infra-curriculum.github.io/)**.
<!-- /aicg:site-banner -->

<!-- aicg:sponsor -->
> 💜 **[Sponsor this curriculum](https://github.com/sponsors/ai-engineering-curriculum)** — sponsorships keep the whole open-source AI Career Curriculum free and moving.
<!-- /aicg:sponsor -->

**Role level:** 20 (application-developer craft — AI Engineering family)

Ship LLM-backed features end-to-end as an application developer: call hosted LLM APIs, engineer prompts and force schema-constrained structured output, declare and consume tools, orchestrate async / streaming calls with honest tail-latency reporting, keep cost defensible with prompt caching and cascade routing, ground answers with a minimal retrieval path, gate a feature on a golden-set regression suite, and land the whole thing behind a small HTTP API with feature flags, safe secrets, basic traces, a runbook, and input / output validation.

> **Status**: `mod-001` is shipped. The rest of the plan (`mod-002` through `mod-007` + two projects) is authored from [`.aicg/curriculum-plan.json`](.aicg/curriculum-plan.json) and lands as autonomous content cycles run. See [`CURRICULUM.md`](CURRICULUM.md) for the full plan and [`JOB_REQUIREMENTS.md`](JOB_REQUIREMENTS.md) for the 31-posting sample that grounds the coverage decisions.

## What you get

Planned commitment: **70 hours across 7 modules** + **50 hours across 2 projects** = **~120 hours**.

- **[`mod-001` — Prompt Engineering Foundations](lessons/mod-001-prompt-engineering-foundations/) *(shipped)*** — chat-style LLM APIs, prompt anatomy, structured JSON output with schema validation, token estimation, and the three most common shapes of prompt failure. Nine short exercises, one lab (`text-classifier`), and a quiz.
- **`mod-002` — Tool and Function Calling *(planned)*** — declare typed tool schemas, run the tool_call → tool_result loop end-to-end, handle parallel tool calls and malformed arguments. Closes the "partial" coverage flagged in `JOB_REQUIREMENTS.md` (58% of postings).
- **`mod-003` — Streaming, Async, and Parallel LLM Orchestration *(planned)*** — SSE stream consumption, partial JSON streaming, async fan-out with bounded concurrency, retry with backoff + jitter, honest p50 / p95 / p99 latency reporting.
- **`mod-004` — Model Selection, Cost, and Prompt Caching *(planned)*** — cost-per-call estimation, small-vs-frontier A/B benchmarks, Anthropic and OpenAI prompt-caching with hit-ratio measurement, cascade routing, and graceful degradation for a provider outage.
- **`mod-005` — Retrieval Basics for LLM Applications *(planned)*** — a *minimal* retrieval intro: embeddings, `pgvector`, top-k retrieval, grounded prompts with citations. Full RAG depth is delegated to [`rag-engineer-learning`](https://github.com/ai-engineering-curriculum/rag-engineer-learning) (level 30).
- **`mod-006` — Minimal Evaluation and Regression Checks *(planned)*** — a *minimal* golden-set intro: 20 to 50 example fixtures, `pytest` CI gate, rule / similarity / LLM-as-a-judge scoring. Full evaluation engineering is delegated to [`ai-eval-engineer-learning`](https://github.com/ai-engineering-curriculum/ai-eval-engineer-learning) (level 30).
- **`mod-007` — Shipping a First Production LLM Feature *(planned)*** — FastAPI endpoint with streaming, safe secrets, safe traces, feature flag for provider / model swap, runbook, and input / output validation. Full production hardening (cloud deployment, CI/CD, production guardrails) is delegated to [`applied-ai-engineer-learning`](https://github.com/ai-engineering-curriculum/applied-ai-engineer-learning) (level 30).
- **`project-001` — Multi-Provider Comparison Report *(planned, 20h)*** — one feature, two providers, honest cost / latency / quality comparison.
- **`project-002` — Grounded, Tool-Calling, Streaming LLM Feature *(planned, 30h)*** — production-shaped LLM feature end-to-end with SSE, tools, retrieval, caching, golden-set CI, traces, feature flag, and runbook.

## Level ladder — where this track fits

- **level 10** — [`ai-infra-junior-engineer-learning`](https://github.com/ai-infra-curriculum/ai-infra-junior-engineer-learning) (assumed engineering-craft prerequisites)
- **level 20 (peers)** — **`llm-application-developer-learning`** (this track) and [`agentic-ai-developer-learning`](https://github.com/ai-engineering-curriculum/agentic-ai-developer-learning). The two split the level-20 LLM-application space: this track owns non-agentic patterns; the peer owns agent frameworks and multi-step agent orchestration.
- **level 25** — [`ai-risk-engineer-learning`](https://github.com/ai-governance-curriculum/ai-risk-engineer-learning) (AI-risk-engineering craft — harm modelling, red-team, guardrail effectiveness)
- **level 30 (next-up in this family)** — [`rag-engineer-learning`](https://github.com/ai-engineering-curriculum/rag-engineer-learning) (full RAG pipeline), [`ai-eval-engineer-learning`](https://github.com/ai-engineering-curriculum/ai-eval-engineer-learning) (application-layer evaluation + observability), [`applied-ai-engineer-learning`](https://github.com/ai-engineering-curriculum/applied-ai-engineer-learning) (production hardening + cloud deployment), [`agentic-ai-engineer-learning`](https://github.com/ai-engineering-curriculum/agentic-ai-engineer-learning) (production agent engineering)
- **level 40+** — [`senior-agentic-ai-engineer-learning`](https://github.com/ai-engineering-curriculum/senior-agentic-ai-engineer-learning) and higher AI Engineering / Governance / Infra tracks

See [`CURRICULUM.md`](CURRICULUM.md) for the ownership rule this track applies and [`PREREQUISITES.md`](PREREQUISITES.md) for the assumed lower-level curriculum.

## Layout

```
llm-application-developer-learning/
├── lessons/mod-XXX-*/        modules with lectures, exercises, labs, quizzes
├── projects/project-XXX-*/   multi-module capstones
├── CURRICULUM.md             role-level coverage map
├── PREREQUISITES.md          assumed entry skills
├── VERSIONS.md               release history
├── JOB_REQUIREMENTS.md       requirements catalog with cited postings evidence
├── .aicg/job-requirements.json    machine-readable requirements & posting sample
├── .aicg/curriculum-plan.json     machine-readable module / exercise / project plan
└── README.md                 this file
```

## Paired Solutions Repo

[`llm-application-developer-solutions`](https://github.com/ai-engineering-curriculum/llm-application-developer-solutions) carries the reference implementations.

---

<!-- aicg:maintained-by -->
Maintained by [VeriSwarm.ai](https://veriswarm.ai)
