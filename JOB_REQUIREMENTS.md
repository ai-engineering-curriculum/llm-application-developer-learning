# LLM Application Developer — Job Requirements

_Refreshed 2026-09-02 (previous pass 2026-08-02). Sample: the original 31-posting 2026-08 baseline plus a 27-posting refresh observed 2026-09-02, deduplicated to **55 unique postings** across 2026-05-01 through 2026-09-02 (all within the last-90-day window per the research packet). Sources: Greenhouse and SmartRecruiters. Titles pooled: "LLM Application Developer", "Generative AI Developer", "AI Application Engineer", "LLM App Engineer", "GenAI Developer", "GenAI Application Engineer", and entry-to-mid "AI Engineer" listings._

_The 2026-09 refresh reported ratios below use the fresh 27-quote-verified subset (denominator 27) where the requirement is called out explicitly. The 2026-08 baseline ratios (denominator 31) are preserved as the primary rank because both samples fall inside the 90-day window and the 2026-09 pass largely confirmed the rank order rather than displacing it._

Machine-readable form: [`.aicg/job-requirements.json`](.aicg/job-requirements.json).

## How to read this document

- **Owner** — the curriculum track that primarily teaches the requirement, per the role hierarchy's lowest-relevant-level ownership rule.
- **Coverage** — `covered` (this repo teaches it), `partial` (touched but not fully drilled), `delegated` (owned by another track — link out), `prerequisite` (assumed entry skill), `gap-below-threshold` (real market signal but below the 30% evidence bar this cycle), or `out-of-scope`.
- **Frequency** — postings mentioning the requirement ÷ 31 sampled postings.

## Owned by this role (LLM Application Developer, level 20)

### Prompt engineering & schema-constrained structured output — 24/31 (77%)

- **Coverage:** covered — [`lessons/mod-001-prompt-engineering-foundations/`](lessons/mod-001-prompt-engineering-foundations/) (lectures 02–03, exercises 4–9).
- Evidence:
  - GitLab, "AI Engineer": "Prompt engineering as a core discipline." [posting](https://job-boards.greenhouse.io/gitlab/jobs/8565469002)
  - Future, "Applied AI Engineer": "Hands-on experience with LLMs in production: prompt engineering, tool/function calling, structured output, evaluation." [posting](https://job-boards.greenhouse.io/future/jobs/4683133005)
  - WITHIN, "AI Engineer": "structured outputs (JSON schema)... prompt design and failure modes." [posting](https://job-boards.greenhouse.io/agencywithin/jobs/5056863007)

### Hosted-LLM-API usage (OpenAI, Anthropic, comparable) — 25/31 (81%)

- **Coverage:** covered — [`lessons/mod-001-prompt-engineering-foundations/lectures/01-llm-api-basics.md`](lessons/mod-001-prompt-engineering-foundations/lectures/01-llm-api-basics.md) plus exercises 1–3.
- Evidence:
  - Cadence Solutions, "AI Engineer": "Hands-on experience with LLM APIs (OpenAI, Anthropic, open-source models)." [posting](https://job-boards.greenhouse.io/solutions/jobs/4680769006)
  - GitLab, "AI Engineer": "Practical fluency across the LLM ecosystem: hands-on experience with models from Anthropic, OpenAI, open-source alternatives." [posting](https://job-boards.greenhouse.io/gitlab/jobs/8565469002)
  - Neo Security, "AI Engineer": "LLM providers (Anthropic Claude, OpenAI, Gemini)." [posting](https://job-boards.greenhouse.io/neosecurityinc/jobs/4323673009)

### Tool / function calling — 18/31 (58%)

- **Coverage:** partial — [`lessons/mod-001-prompt-engineering-foundations/lectures/03-structured-output.md`](lessons/mod-001-prompt-engineering-foundations/lectures/03-structured-output.md) introduces "tool-shaped responses" as a variant of schema-constrained output, and exercises 7–9 drill the underlying schema-constrained mechanism.
- **Gap note:** No exercise walks a learner through declaring a `tools=[...]` schema, receiving a `tool_call` block, executing it, and returning a `tool_result`. Considered adding one this cycle but left as-is per the continuity bias — the API surface is already exercised in a different framing, and the next cycle can revisit if the signal strengthens.
- Evidence:
  - WITHIN, "AI Engineer": "tool/function calling." [posting](https://job-boards.greenhouse.io/agencywithin/jobs/5056863007)
  - Cadence Solutions, "AI Engineer": "Experience with prompt engineering, tool use / function calling, and structured outputs." [posting](https://job-boards.greenhouse.io/solutions/jobs/4680769006)
  - Neo Security, "AI Engineer": "Fluency with structured outputs, function/tool calling, and multi-agent orchestration." [posting](https://job-boards.greenhouse.io/neosecurityinc/jobs/4323673009)

### Token cost, latency, and model-selection trade-offs — 10/31 (32%)

- **Coverage:** covered — mod-001 learning objective #3 ("Estimate the token cost of a prompt-and-response before you send it") and [exercise 2](lessons/mod-001-prompt-engineering-foundations/exercises/README.md) ("Count tokens two ways").
- Evidence:
  - GitLab, "AI Engineer": "Model selection and cost-performance trade-offs." [posting](https://job-boards.greenhouse.io/gitlab/jobs/8565469002)
  - Neo Security, "AI Engineer": "Judgment about cost, latency, and provider trade-offs at scale." [posting](https://job-boards.greenhouse.io/neosecurityinc/jobs/4323673009)
  - Future, "Applied AI Engineer": "prompt caching strategies, token budgets, retry logic, and observability." [posting](https://job-boards.greenhouse.io/future/jobs/4683133005)

### Streaming responses and async orchestration — 8/31 (26%) — below threshold

- **Coverage:** `gap-below-threshold`. Legitimate territory for this role but under the 30% bar. 2026-09 refresh: 2/27 explicit mentions (Future, Anthropic Voice) — no upward drift, still tracked for next cycle.
- Evidence:
  - Future, "Applied AI Engineer": "tool-calling LLM systems with structured output, parallel API orchestration, and streaming responses." [posting](https://job-boards.greenhouse.io/future/jobs/4683133005)
  - Future, "Applied AI Engineer": "async Python, HTTP APIs, and streaming protocols (SSE, webhooks)." [posting](https://job-boards.greenhouse.io/future/jobs/4683133005)

## Delegated to other tracks (link out — do not duplicate)

### RAG pipelines — 27/31 (87%) → owned by [RAG & Retrieval Engineer](https://github.com/ai-engineering-curriculum/rag-engineer-learning) (level 30)

The single most-cited requirement in the sample. Chunking, embeddings, vector DBs (Pinecone, Weaviate, Qdrant, Chroma), retrieval evaluation, re-ranking. Per the ownership rule, level-30 RAG Engineer owns the depth; this track links out.

- OPAQUE Systems, "Forward Deployed Engineer (AI)": "Deep hands-on experience with RAG pipeline design: chunking strategies, embedding models, vector databases, retrieval quality evaluation, re-ranking." [posting](https://job-boards.greenhouse.io/opaquesystems/jobs/4235505009)
- Bright.AI, "Senior AI Engineer — LLM, RAG": "Experience building RAG pipelines with tools like LangChain, LlamaIndex, or custom vector database integrations." [posting](https://job-boards.greenhouse.io/brightai/jobs/5616545004)
- WelbeHealth, "AI Engineer I": "Familiarity with retrieval-augmented generation (RAG) concepts and vector database such as Pinecone, Weaviate, Chroma, or similar tools." [posting](https://job-boards.greenhouse.io/jobboard/jobs/8599687002)

### Agent frameworks (LangGraph / LangChain / LlamaIndex / CrewAI / AutoGen / Semantic Kernel) — 24/31 (77%) → owned by [Agentic AI Developer](https://github.com/ai-engineering-curriculum/agentic-ai-developer-learning) (level 20)

- Snorkel AI, "Applied AI Engineer — AI Solutions": "LLM orchestration, workflow, agent authoring tools (e.g., LlamaIndex, LangGraph, CrewAI)." [posting](https://job-boards.greenhouse.io/snorkelai/jobs/5709067004)
- Lynx Analytics, "AI Engineer (US)": "Deep hands-on experience with agentic frameworks (LangChain, LlamaIndex, AutoGen, CrewAI, or similar)." [posting](https://job-boards.greenhouse.io/lynxanalytics/jobs/8647264002)
- LTS, "Senior Applied AI Engineer": "Experience with AI frameworks such as LangGraph, LangChain, LlamaIndex, Semantic Kernel, CrewAI, AutoGen, or similar." [posting](https://job-boards.greenhouse.io/lts/jobs/4340498009)

### Multi-step agent design patterns (ReAct, planning loops, memory, multi-agent coordination) — 22/31 (71%) → owned by [Agentic AI Developer](https://github.com/ai-engineering-curriculum/agentic-ai-developer-learning) (level 20)

- Lynx Analytics: "Strong understanding of agent design patterns: ReAct, planning loops, tool use, multi-agent coordination, and memory architectures." [posting](https://job-boards.greenhouse.io/lynxanalytics/jobs/8647264002)
- Cadence Solutions: "Design multi-step agent orchestration: planning, memory, tool use, error recovery." [posting](https://job-boards.greenhouse.io/solutions/jobs/4680769006)
- Mixpanel, "Software Engineer, AI Platform": "Expertise in building scalable frameworks for multi-step agent workflows." [posting](https://job-boards.greenhouse.io/mixpanel/jobs/8064216)

### Evaluation frameworks (golden sets, LLM-as-a-judge, regression suites) — 20/31 (65%) → owned by [AI Evaluation Engineer](https://github.com/ai-engineering-curriculum/ai-eval-engineer-learning) (level 30)

- Future: "Exposure to evaluation frameworks: LLM-as-a-judge, automated scoring, dataset management." [posting](https://job-boards.greenhouse.io/future/jobs/4683133005)
- Octus, "AI Engineer": "Hands-on experience designing and implementing automated evaluation frameworks for LLM systems." [posting](https://job-boards.greenhouse.io/octus/jobs/5155045007)
- Cadence Solutions: "Develop evaluation frameworks: offline benchmarks, safety tests, regression suites." [posting](https://job-boards.greenhouse.io/solutions/jobs/4680769006)

### Observability / tracing (Langfuse, OpenTelemetry, Datadog) — 15/31 (48%) → owned by [AI Evaluation Engineer](https://github.com/ai-engineering-curriculum/ai-eval-engineer-learning) (level 30)

- Future: "Observability and tracing tools (Langfuse, OpenTelemetry, Datadog)." [posting](https://job-boards.greenhouse.io/future/jobs/4683133005)
- Neo Security: "Deep observability into the pipeline — spans, traces, and metrics for every model call." [posting](https://job-boards.greenhouse.io/neosecurityinc/jobs/4323673009)
- Awin, "Senior AI Engineer": "Familiarity with tracing, auditability, and logging standards for AI systems." [posting](https://job-boards.greenhouse.io/awin/jobs/7720634003)

### Cloud deployment (AWS Bedrock, GCP Vertex, Azure OpenAI) — 21/31 (68%) → owned by [Applied AI Engineer](https://github.com/ai-engineering-curriculum/applied-ai-engineer-learning) (level 30)

- Future: "Familiarity with AWS (Bedrock, ECR, CloudFront, S3, Cognito) or other cloud agent hosting." [posting](https://job-boards.greenhouse.io/future/jobs/4683133005)
- SMG Swiss Marketplace Group, "Senior AI Engineer": "Experience with cloud infrastructure, especially AWS and/or GCP." [posting](https://jobs.smartrecruiters.com/SMGSwissMarketplaceGroup/744000136476919-senior-ai-engineer)
- TensorOps: "Experience deploying and scaling ML systems on AWS, GCP, or Azure." [posting](https://job-boards.greenhouse.io/tensorops/jobs/4930699101)

### Containerisation, CI/CD, and IaC — 16/31 (52%) → owned by [Applied AI Engineer](https://github.com/ai-engineering-curriculum/applied-ai-engineer-learning) (level 30)

- Lynx Analytics: "APIs, testing, CI/CD, containerisation (Docker/Kubernetes)." [posting](https://job-boards.greenhouse.io/lynxanalytics/jobs/8647264002)
- Future: "Infrastructure-as-code (Terraform, CDK)." [posting](https://job-boards.greenhouse.io/future/jobs/4683133005)
- SMG Swiss Marketplace Group: "Comfortable working with CI/CD, containerization, infrastructure-as-code, observability, and production incident handling." [posting](https://jobs.smartrecruiters.com/SMGSwissMarketplaceGroup/744000136476919-senior-ai-engineer)

### Guardrails, PII handling, prompt-injection defence — 9/31 (29%) → owned by [Applied AI Engineer](https://github.com/ai-engineering-curriculum/applied-ai-engineer-learning) (level 30) — below threshold

mod-001 exercise 6 (jailbreak) introduces the topic conceptually; production guardrails live at level 30.

- GitLab: "Design appropriate guardrails (input validation, output filtering, access controls, prompt injection defences)." [posting](https://job-boards.greenhouse.io/gitlab/jobs/8565469002)
- Awin: "Strong understanding of AI-specific risks, including privacy exposure, PII handling, unsafe outputs." [posting](https://job-boards.greenhouse.io/awin/jobs/7720634003)
- Awin: "Experience designing or implementing guardrails for AI or LLM-powered systems." [posting](https://job-boards.greenhouse.io/awin/jobs/7720634003)

### Hallucination detection & grounding metrics — 7/31 (23%) → owned by [AI Evaluation Engineer](https://github.com/ai-engineering-curriculum/ai-eval-engineer-learning) (level 30) — below threshold

- Neo Security: "eval-driven instinct... detect and reduce hallucination." [posting](https://job-boards.greenhouse.io/neosecurityinc/jobs/4323673009)
- Mixpanel: "Proficiency with evaluation systems for measuring reasoning quality and hallucination rates." [posting](https://job-boards.greenhouse.io/mixpanel/jobs/8064216)
- Rackner: "Evaluate AI outputs for grounding, reliability, accuracy, relevance, and mission usefulness." [posting](https://job-boards.greenhouse.io/rackner/jobs/4717776005)

### Fine-tuning / instruction-tuning / LoRA basics — 9/31 (29%) → owned by [Applied AI Engineer](https://github.com/ai-engineering-curriculum/applied-ai-engineer-learning) (level 30) — below threshold

Not an LLM-application-developer concern at this tier.

- Cadence Solutions: "Fine-tuning experience (SFT, RLHF, LoRA) on domain-specific tasks." [posting](https://job-boards.greenhouse.io/solutions/jobs/4680769006)
- Bright.AI: "Fluency with prompt engineering, instruction tuning, or fine-tuning open-source models." [posting](https://job-boards.greenhouse.io/brightai/jobs/5616545004)
- Eclipse: "fine-tuning Large Language Models (LLMs)." [posting](https://job-boards.greenhouse.io/eclipse/jobs/4981191008)

## Prerequisites (assumed entry skills)

These show up frequently in postings but are covered by general software-engineering study, not by this track. Referenced in [`PREREQUISITES.md`](PREREQUISITES.md).

| Requirement | Frequency | External resources |
| --- | --- | --- |
| Python fluency (async, pydantic, HTTP) | 26/31 (84%) | [Python docs — asyncio](https://docs.python.org/3/library/asyncio.html), [Pydantic docs](https://docs.pydantic.dev) |
| Backend / REST / FastAPI / GraphQL | 16/31 (52%) | [FastAPI docs](https://fastapi.tiangolo.com), [GraphQL learn](https://graphql.org/learn/) |
| TypeScript / Node.js | 9/31 (29%) | [TypeScript handbook](https://www.typescriptlang.org/docs/handbook/intro.html) |
| SQL / relational databases | 8/31 (26%) | [PostgreSQL tutorial](https://www.postgresql.org/docs/current/tutorial.html) |
| AI coding tools (Claude Code, Cursor, Copilot) | 6/31 (19%) | Provider docs for the tool of your choice |

## Below-threshold / niche requirements (tracked, not adopted this cycle)

| Requirement | Frequency (baseline / 2026-09 refresh) | Notes |
| --- | --- | --- |
| MCP servers, MCP tool-server development, agent-SDK familiarity | 4/31 (13%) → 4/27 (15%) | **Signal hardening:** in the 2026-09 refresh MCP graduated from "familiar with the modern AI stack" phrasing to concrete build-and-operate asks (Indicium AI: "Architect and deploy custom data interfaces and tool servers using the Model Context Protocol (MCP)"; Vericast: "Design and build MCP Server integrations"; Anthropic: "MCP development"). Still below the 30% bar and ownership would sit with [Agentic AI Developer](https://github.com/ai-engineering-curriculum/agentic-ai-developer-learning) per the ownership rule, so no adoption here. Elevated to top of watch list. |
| Computer use / browser automation (Playwright, CUA) | 3/31 (10%) → 4/27 (15%) | **Signal hardening:** Snorkel AI now hires explicitly for a "CUA (Computer Use Agents)" specialty and Kargo asks for Playwright-based agent workflows. Still niche, still owned by Agentic AI Developer if adopted. Watch list. |
| LLM observability / tracing as a distinct sub-specialty (Langfuse, Braintrust, LangSmith, OpenTelemetry for LLMs) | 15/31 (48%) → 4/27 (15%) explicit + one dedicated title (Honeycomb "LLM Observability") | 2026-09 refresh confirms this has bifurcated into (a) generic observability expectations in most senior GenAI roles and (b) a dedicated hiring track at observability vendors (Honeycomb, Glean, ID.me). Depth remains delegated to [AI Evaluation Engineer](https://github.com/ai-engineering-curriculum/ai-eval-engineer-learning). |
| Voice / multimodal agents (real-time voice, ASR/TTS in the agent loop) | not sampled in 2026-08 → 1/27 (4%) | **New emerging signal:** dedicated voice-agent postings (Anthropic Voice Platform, Cresta, AssemblyAI, Inflection) surfaced in 2026-09 but the volume is still very low in the LLM-App-Dev title pool. If it clears the bar, ownership belongs to [Applied AI Engineer](https://github.com/ai-engineering-curriculum/applied-ai-engineer-learning) or the Agentic AI family. Watch. |
| Fluency with agentic coding tools (Claude Code, Cursor, Codex, Gemini CLI) as a stated requirement | 6/31 (19%) → 2/27 (7%) explicit but now phrased as "own every line" (Sourcegraph, Artefact) | Remains a prerequisite framing, not a curriculum target. |
| Enterprise-system integration (Salesforce/SAP/ServiceNow) | 5/31 (16%) → 1/27 (4%) | Employer-specific; out of scope for a level-20 learning track. |
| Domain compliance (HIPAA / SOC 2 / FedRAMP) | 7/31 (23%) → 0/27 (0%) | Employer-specific; belongs to Applied AI Engineer or the governance curriculum. |

## This-cycle decision summary

- **Additions proposed (2026-09):** none. See [`.aicg/curriculum-plan-delta.json`](.aicg/curriculum-plan-delta.json).
- **Rationale (2026-09 refresh):** The fresh 27-posting pass (2026-09-02) confirms the rank order of the 2026-08 baseline. Every above-threshold requirement in this role's ownership is already covered by mod-001, and the planned mod-002 through mod-007 already reserve capacity for the partial-coverage gaps (tool/function calling in mod-002, streaming/async in mod-003) — no scope adjustment is warranted in a 30-day cycle when the market rank didn't move. Emerging signals (MCP server dev, computer-use / browser automation, voice agents, LLM-observability-as-a-specialty) all remain below the 30% bar in this title pool, and — critically — all four belong to peer or higher tracks under the ownership rule (Agentic AI Developer for MCP/computer-use, Applied AI Engineer for voice deployment, AI Evaluation Engineer for observability). Duplicating them here would violate the ownership rule and displace continuity for content that is already funded elsewhere.
- **Watch list for next cycle (updated 2026-09):**
  - MCP server / tool-server development — hardened from "familiar with" to concrete build asks; if it clears 30% in the LLM-App-Dev pool, ownership belongs to Agentic AI Developer (raise a cross-track proposal there, not here).
  - Computer-use / browser-automation agents — new dedicated postings at Snorkel AI and Kargo; same peer-ownership routing.
  - Voice / multimodal agents — surfacing as a distinct sub-specialty at Anthropic, Cresta, AssemblyAI.
  - Streaming / async — unchanged at ~26%; continues to sit as `gap-below-threshold` and is already the planned scope of mod-003.
