# LLM Application Developer — Job Requirements

_Refreshed 2026-10-02 (prior passes 2026-08-02 and 2026-09-02). Sample: 31-posting 2026-08 baseline + 27-posting 2026-09 refresh + 31-posting 2026-10 refresh = **89 unique postings** across 2026-07-04 through 2026-10-02 (all within the last-90-day window per the research packet). Three posting URLs were observed as still-active across cycles and are not double-counted. Sources: Greenhouse, SmartRecruiters, Lever. Titles pooled: "LLM Application Developer", "Generative AI Developer", "AI Application Engineer", "LLM App Engineer", "GenAI Developer", "GenAI Application Engineer", and entry-to-mid "AI Engineer" listings._

_Refresh ratios in each section (2026-09 and 2026-10) are quote-verified fresh subsets (denominators 27 and 31 respectively). The 2026-08 baseline ratios (denominator 31) are preserved as the primary rank because the baseline methodology counted implicit mentions as well. Both refreshes largely confirmed the rank order rather than displacing it._

Machine-readable form: [`.aicg/job-requirements.json`](.aicg/job-requirements.json).

## How to read this document

- **Owner** — the curriculum track that primarily teaches the requirement, per the role hierarchy's lowest-relevant-level ownership rule.
- **Coverage** — `covered` (this repo teaches it), `partial` (touched but not fully drilled), `delegated` (owned by another track — link out), `prerequisite` (assumed entry skill), `gap-below-threshold` (real market signal but below the 30% evidence bar this cycle), or `out-of-scope`.
- **Frequency** — postings mentioning the requirement ÷ 31 sampled 2026-08 baseline postings (primary rank). Refresh ratios from 2026-09 (denominator 27) and 2026-10 (denominator 31) appear in-line where the signal is actively shifting.

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

### Tool / function calling — 18/31 (58%) baseline → 22/31 (71%) 2026-10 refresh (hardening)

- **Coverage:** partial — [`lessons/mod-001-prompt-engineering-foundations/lectures/03-structured-output.md`](lessons/mod-001-prompt-engineering-foundations/lectures/03-structured-output.md) introduces "tool-shaped responses" as a variant of schema-constrained output, and exercises 7–9 drill the underlying schema-constrained mechanism. The authored but unshipped `mod-002-tool-and-function-calling` is scoped exactly for the end-to-end `tools=[...]` → `tool_call` → `tool_result` loop.
- **2026-10 signal note:** Explicit mentions climbed 58% (2026-08) → 63% (2026-09 refresh) → 71% (2026-10 refresh). This strengthens the case for prioritising mod-002 content production, but no scope change is warranted — mod-002 as planned already covers the end-to-end loop. Multi-provider tool-schema portability (Baselayer, Firmus, Machinify Agentic) and typed-tool-registry patterns are consistent with the planned module scope.
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

- **Coverage:** `gap-below-threshold`. Legitimate territory for this role but under the 30% bar. 2026-09 refresh: 2/27 (7%). 2026-10 refresh: 3/31 (10%, Axle + Machinify Agentic + Future). No upward drift. mod-003 (planned) remains the right landing spot when authored.
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

| Requirement | Frequency (baseline / 2026-09 refresh / 2026-10 refresh / full 90-day corpus) | Notes |
| --- | --- | --- |
| MCP servers, MCP tool-server development, agent-SDK familiarity | 4/31 (13%) → 4/27 (15%) → **7/31 (23%)** / **15/89 (16.9%) full corpus** | **Signal hardening — now three cycles running.** 2026-10 adds MediaAlpha, Machinify Staff ("Experience with agent frameworks and MCP"), PayPay Card ("MCP-based integrations"), Axle ("MCP for tool integration"), Florence Healthcare, Utility Profit, and Axios ("MCP servers, agent tools"). The language has moved from "familiar with the modern AI stack" (2026-08) to "architect and deploy" (2026-09) to production tool-registry hygiene (2026-10). Still below the 30% bar in this title pool AND the full-90-day corpus; ownership belongs to [Agentic AI Developer](https://github.com/ai-engineering-curriculum/agentic-ai-developer-learning) per the ownership rule, so no adoption here. **Top of the watch list** — if it clears 30% in the Agentic AI Developer title pool first, that track should adopt and this track should link out. |
| Computer use / browser automation (Playwright, CUA) | 3/31 (10%) → 4/27 (15%) → **2/31 (6%)** | Pulled back slightly in the 2026-10 fresh pool. Utility Profit explicitly cites Playwright; Axios bundles with MCP and agentic coding tools. Still niche, still owned by Agentic AI Developer if adopted. Watch list. |
| LLM observability / tracing as a distinct sub-specialty (Langfuse, Braintrust, LangSmith, OpenTelemetry for LLMs) | 15/31 (48%) → 4/27 (15%) → **8/31 (26%)** | 2026-10 refresh adds Smartsheet (Langfuse / LangSmith / MLflow), Firmus ("agent observability using tracing, metrics, logs"), LaunchDarkly (DeepEval + Datadog + real-time observability), Impiricus (dedicated eval/observability role). Still bifurcated between generic ops and dedicated LLM-observability specialty (Honeycomb, Glean, ID.me). Depth remains delegated to [AI Evaluation Engineer](https://github.com/ai-engineering-curriculum/ai-eval-engineer-learning). |
| Voice / multimodal agents (real-time voice, ASR/TTS in the agent loop) | not sampled in 2026-08 → 1/27 (4%) → **1/31 (3%)** | Still thin. 2026-10 adds Aircall (dedicated TTS/ASR + voice-eval engineering role) — the clearest signal yet, but as a vendor-specialty rather than a general LLM-App-Dev expectation. Ownership belongs to [Applied AI Engineer](https://github.com/ai-engineering-curriculum/applied-ai-engineer-learning) or the Agentic AI family. Watch. |
| Fluency with agentic coding tools (Claude Code, Cursor, Codex, Gemini CLI) as a stated requirement | 6/31 (19%) → 2/27 (7%) → **3/31 (10%) with phrasing hardening** | 2026-10 phrasing hardens further — ServiceNow: "Build Agent live inside Cursor, Claude Code, and GitHub Copilot"; Axios: "using agentic coding tools — Claude Code, Codex, Cursor, GitHub Copilot"; Utility Profit frames daily agent-coding-tool fluency as a hiring filter. Remains a prerequisite framing, not a curriculum target. |
| Enterprise-system integration (Salesforce/SAP/ServiceNow) | 5/31 (16%) → 1/27 (4%) → 0/31 (0%) | Employer-specific; out of scope for a level-20 learning track. |
| Domain compliance (HIPAA / SOC 2 / FedRAMP) | 7/31 (23%) → 0/27 (0%) → 1/31 (3%, Mitratech) | Employer-specific; belongs to Applied AI Engineer or the governance curriculum. |
| Typed-agent frameworks (Pydantic AI explicitly cited) | not sampled → 0/27 → **4/31 (13%)** | Fresh 2026-10 signal: Flagship Pioneering, Axle, CoLab, Firmus explicitly cite Pydantic AI alongside LangGraph. Reads as a flavour of agent framework rather than a new requirement; still owned by Agentic AI Developer. Watch. |
| Semantic / response caching + model routing (cost engineering as a named skill) | not separately sampled | Baselayer: "response caching, semantic caching, model routing (cheap-first then escalate), thinking-budget tuning, prompt-cache hit-rate work"; Firmus: "model routing and fallback strategies"; Machinify Staff: multi-provider portability. Fits planned mod-004 scope; no new content needed. |

## This-cycle decision summary

- **Additions proposed (2026-10):** none. See [`.aicg/curriculum-plan-delta.json`](.aicg/curriculum-plan-delta.json).
- **Rationale (2026-10 refresh):** The fresh 31-posting pass (2026-10-02) confirms — once again — the rank order of the 2026-08 baseline. The single owned requirement that visibly shifted is tool/function calling (58% baseline → 71% in the 2026-10 fresh pool); this is already the scope of the authored but unshipped `mod-002`, so no new content is warranted — the hardening instead reinforces that mod-002 should be the next production priority. Cost/model-selection specifics (semantic caching, model routing, prompt-cache hit-rate work) that showed up explicitly at Baselayer, Firmus, and Machinify also fit within the planned `mod-004` scope with no re-scoping needed. The most visible emerging signal — MCP tool-server development — hardened for a third consecutive cycle (13% → 15% → 23% in refresh samples; 17.4% across the full 90-day corpus), but it still sits below the 30% adoption bar AND belongs to the peer Agentic AI Developer track per the ownership rule, so this track continues to link out and does not duplicate. Voice/multimodal, computer-use, and LLM-observability-as-a-specialty remain below threshold and are owned by peer or higher tracks. Continuity bias ("prefer citing existing content over proposing new content") is the correct outcome for a cohort still mid-flight through the shipped mod-001 and the queued mod-002 through mod-007.
- **Watch list for next cycle (updated 2026-10):**
  - **MCP server / tool-server development** — three cycles of hardening (13% → 15% → 23%). If it clears 30% in the LLM-App-Dev pool next cycle, ownership still belongs to Agentic AI Developer (raise the cross-track proposal there, not here). Watch whether the Agentic AI Developer track adopts it first.
  - **Tool/function calling as a planned-but-unshipped gap** — now at 71% in the fresh pool. mod-002 content production should be the next cycle's priority (not a new module).
  - **Typed-agent frameworks (Pydantic AI)** — new 13% signal in 2026-10; a flavour of agent framework, still delegated.
  - **Voice / multimodal agents** — flat at 3-4% in the LLM-App-Dev title pool; concentrated at voice vendors.
  - **Computer-use / browser automation** — pulled back to 6% in 2026-10; still owned by Agentic AI Developer if adopted.
  - **Streaming / async** — flat at ~10% in refresh pools, 26% baseline; continues to sit as `gap-below-threshold` and is already the planned scope of mod-003.
