# Prerequisites — LLM Application Developer track

**Role level:** 20 (application-developer craft, AI Engineering family)
**Track:** `llm-application-developer-learning`

This is an entry-rung AI Engineering track. It assumes the learner arrives with general software-engineering fundamentals already muscle-memoried — Python fluency, HTTP APIs, JSON, and basic testing. Those fundamentals are **not** re-taught here; the modules step directly onto LLM API usage, prompt engineering, tool calling, streaming, cost optimisation, and shipping a first LLM-backed feature.

The 2026-08 posting sample summarised in [`JOB_REQUIREMENTS.md`](JOB_REQUIREMENTS.md) shows the same skills sitting under the "assumed" line in real hiring: 84% of postings call for Python, 52% for backend / REST / FastAPI / GraphQL, 26% for SQL. This track adopts that split — those skills are prerequisites, not the teaching load.

## Assumed lower-level curriculum

Complete or be practically fluent in the equivalent of this lower-level track before starting:

- **[`ai-infra-junior-engineer-learning`](https://github.com/ai-infra-curriculum/ai-infra-junior-engineer-learning) — level 10** — the engineering-craft prerequisites this track builds on: Linux basics, Git, Python packaging (`pyproject.toml`, `uv` / `pip` / `venv`), HTTP APIs, JSON / YAML, unit testing (`pytest`), logging, Docker basics. If you cannot ship a small Python service, call an HTTP API from it, and print JSON, start there.

The peer level-20 track [`agentic-ai-developer-learning`](https://github.com/ai-engineering-curriculum/agentic-ai-developer-learning) is not a prerequisite — the two tracks can be taken in either order or in parallel, and each is a valid on-ramp to the level-30 AI Engineering tracks.

## Assumed working knowledge

Even without holding the equivalent role formally, you should be practically fluent in:

- **Python** — enough to write a script that reads a file, calls an HTTP API, and prints the result. Comfortable with virtualenvs, `pip` / `uv`, and `pytest`. Familiar with `pydantic` for typed data at the boundary (the postings sample flags this at 84%).
- **HTTP APIs** — comfortable making REST calls, reading JSON responses, handling 4xx / 5xx status codes, and setting a request timeout. Comfortable with headers, authentication, and rate-limit behaviour.
- **TypeScript / Node.js is optional** — mod-001 accepts either Python or TypeScript. The rest of the modules assume Python, matching where the majority of the 2026-08 posting sample is anchored (Python at 84% vs. TypeScript / Node at 29%).
- **SQL basics** — reading a `SELECT`, understanding a `JOIN`, and knowing what an index is. Postgres is the working assumption in the mod-005 retrieval intro (via `pgvector`), but nothing here requires DBA depth.
- **JSON, JSON Schema, and typed data modelling** — mod-001 exercises 7–9 and mod-002 exercise 1 both assume you know what a JSON Schema is and can hand-author a small one.
- **Git and pull requests** — every module ends with content that lands in a repo; comfort with `git commit`, branches, and PR review is assumed.

## Not assumed, taught here

You do **not** need prior experience with:

- Any specific LLM provider — mod-001 covers OpenAI and Anthropic API basics side-by-side, and later modules stay SDK-agnostic where the concept is provider-independent.
- Prompt engineering as a discipline — mod-001 covers the message-list model, system / user / assistant roles, few-shot patterns, and delimiters.
- Structured / JSON-schema-constrained output — mod-001 lecture 03 covers the strict-mode toggle and the semantic-vs-syntactic distinction.
- Tool / function calling as an API surface — mod-002 covers schema declaration, the tool_call → tool_result loop, and parallel tool calls in one turn.
- Streaming responses (SSE) or async LLM orchestration — mod-003 covers both.
- Prompt caching, cascade routing, or provider cost / latency trade-offs — mod-004 covers them.
- Retrieval-augmented prompts at the intro tier — mod-005 covers embeddings, `pgvector`, and grounded prompts with citations. Full RAG pipeline depth lives at [`rag-engineer-learning`](https://github.com/ai-engineering-curriculum/rag-engineer-learning) (level 30).
- Golden-set regression checks — mod-006 covers `pytest`-driven eval in CI at the intro tier. Full evaluation engineering lives at [`ai-eval-engineer-learning`](https://github.com/ai-engineering-curriculum/ai-eval-engineer-learning) (level 30).
- Shipping an LLM feature behind a FastAPI endpoint with secrets, traces, feature flags, and a runbook — mod-007 covers the application-developer slice. Full production hardening lives at [`applied-ai-engineer-learning`](https://github.com/ai-engineering-curriculum/applied-ai-engineer-learning) (level 30).

## Recommended reading before starting

- [OpenAI Platform Docs — Quickstart](https://platform.openai.com/docs/quickstart) — do the quickstart against your own API key before mod-001.
- [Anthropic API — Getting Started](https://docs.anthropic.com/en/api/getting-started) — do the quickstart against your own API key before mod-001.
- [`tiktoken`](https://github.com/openai/tiktoken) — install it and count tokens on a paragraph of your own writing.
- [Pydantic docs](https://docs.pydantic.dev/) — skim once if `pydantic` is new to you.
- [FastAPI docs — first steps](https://fastapi.tiangolo.com/tutorial/first-steps/) — mod-007 assumes you have written a `GET /hello` endpoint before.

You do **not** need to have read anything about LangChain, LlamaIndex, LangGraph, CrewAI, AutoGen, or Semantic Kernel — those are the peer [`agentic-ai-developer-learning`](https://github.com/ai-engineering-curriculum/agentic-ai-developer-learning) track's teaching load.

## Not required, useful to have

- **Prior work with a hosted LLM API** — even a weekend script that calls OpenAI or Anthropic and prints the reply gives you a running start on mod-001.
- **A cheap Postgres instance** — mod-005 uses `pgvector`; a local Docker Postgres is fine.
- **A cheap secret store** — mod-007 covers env-var-based secret handling but references AWS Secrets Manager / GCP Secret Manager / 1Password as production alternatives.
- **AI coding tools (Claude Code, Cursor, GitHub Copilot)** — 19% of the 2026-08 posting sample calls this out. Not a hard prereq, but you will move faster on the labs if you have one wired in.

## Not in scope for this track (linked out)

- **Agent frameworks (LangChain / LangGraph / LlamaIndex / CrewAI / AutoGen / Semantic Kernel) and multi-step agent design patterns** — owned by peer [`agentic-ai-developer-learning`](https://github.com/ai-engineering-curriculum/agentic-ai-developer-learning) (level 20).
- **Full RAG pipeline engineering** — chunking strategies, embedding-model selection, vector-DB internals across providers, retrieval-quality evaluation, re-ranking — owned by [`rag-engineer-learning`](https://github.com/ai-engineering-curriculum/rag-engineer-learning) (level 30).
- **Application-layer evaluation engineering** — LLM-as-a-judge calibration, RAG eval, online eval, observability / tracing (Langfuse, OpenTelemetry, Datadog), cost / latency / quality dashboards — owned by [`ai-eval-engineer-learning`](https://github.com/ai-engineering-curriculum/ai-eval-engineer-learning) (level 30).
- **Production-hardened LLM apps** — cloud deployment (AWS Bedrock, GCP Vertex, Azure OpenAI), containerisation / CI/CD / IaC, production guardrails (input validation, output filtering, prompt-injection defence, PII handling), fine-tuning / LoRA basics — owned by [`applied-ai-engineer-learning`](https://github.com/ai-engineering-curriculum/applied-ai-engineer-learning) (level 30).
- **AI-risk-engineering craft** — harm modelling, adversarial red-team programs, guardrail effectiveness measurement, EU AI Act Article 72 post-market surveillance — owned by [`ai-risk-engineer-learning`](https://github.com/ai-governance-curriculum/ai-risk-engineer-learning) (level 25).
- **Production agent engineering** — owned by [`agentic-ai-engineer-learning`](https://github.com/ai-engineering-curriculum/agentic-ai-engineer-learning) (level 30) and [`senior-agentic-ai-engineer-learning`](https://github.com/ai-engineering-curriculum/senior-agentic-ai-engineer-learning) (level 40).
- **Fine-tuning depth (SFT, RLHF, LoRA)** — flagged in 29% of postings but delegated to Applied AI Engineer at level 30.
- **Enterprise-system integration (Salesforce, ServiceNow, SAP, Zendesk, Marketo) and domain compliance (HIPAA, SOC 2, FedRAMP)** — employer-specific; out of scope for a level-20 learning track. See [`JOB_REQUIREMENTS.md`](JOB_REQUIREMENTS.md) for the below-threshold evidence.
