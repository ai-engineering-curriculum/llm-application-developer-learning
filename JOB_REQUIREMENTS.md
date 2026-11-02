# LLM Application Developer — Job Requirements

_Refreshed 2026-11-02 (prior passes 2026-08-02, 2026-09-02, 2026-10-02). Sample: 31-posting 2026-08 baseline + 27-posting 2026-09 refresh + 31-posting 2026-10 refresh + 33-posting 2026-11 refresh = **122 unique postings** across 2026-07-04 through 2026-11-02. The last-90-day window (2026-08-04 through 2026-11-02) contains **89 unique postings** — the 2026-08 baseline is one day outside the strict window but its ratios are retained as the primary rank reference. Five posting URLs were observed as still-active across cycles and are not double-counted. Sources: Greenhouse, SmartRecruiters, Lever. Titles pooled: "LLM Application Developer", "Generative AI Developer", "AI Application Engineer", "LLM App Engineer", "GenAI Developer", "GenAI Application Engineer", "Applied AI Engineer", "Forward Deployed AI Engineer", "Agentic AI Engineer" (level-20 phrasing only), and entry-to-mid "AI Engineer" listings._

_Refresh ratios in each section (2026-09 / 2026-10 / 2026-11) are quote-verified fresh subsets (denominators 27, 31, 33 respectively). The 2026-08 baseline ratios (denominator 31) are preserved as the primary rank because the baseline methodology counted implicit mentions as well. The three refreshes have largely confirmed the rank order rather than displacing it; the one material 2026-11 shift is MCP crossing 30% in the fresh pool for the first time — see the watch-list below._

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

### Tool / function calling — 18/31 (58%) baseline → 71% 2026-10 refresh → 6/33 (18%) 2026-11 refresh (terminology shift)

- **Coverage:** partial — [`lessons/mod-001-prompt-engineering-foundations/lectures/03-structured-output.md`](lessons/mod-001-prompt-engineering-foundations/lectures/03-structured-output.md) introduces "tool-shaped responses" as a variant of schema-constrained output, and exercises 7–9 drill the underlying schema-constrained mechanism. The authored but unshipped `mod-002-tool-and-function-calling` is scoped exactly for the end-to-end `tools=[...]` → `tool_call` → `tool_result` loop.
- **2026-11 signal note:** Explicit standalone mentions fell to 18% because the 2026-11 fresh pool bundles tool-calling inside compound phrases — "modern AI stack: LLM APIs, MCP, agent frameworks ... prompt engineering, RAG, and evaluation methods" (Fanatics Inc.), "tool calling, RAG, structured outputs, agent frameworks, MCP" (Figma), "Agentic systems and MCP server and client integrations" (Fanatics Commerce). The underlying demand did not drop — it is now implicit under agent-framework / MCP / agentic-workflow phrasing. mod-002 remains the right landing spot and the next content-production priority; no scope change.
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

- **Coverage:** `gap-below-threshold`. Legitimate territory for this role but under the 30% bar every cycle sampled: 26% baseline → 7% (2026-09) → 10% (2026-10) → 0% explicit (2026-11). Likely implicit in chat/agent-UX production work rather than disappearing. mod-003 (planned) remains the right landing spot when authored.
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

| Requirement | Frequency (baseline / 2026-09 / 2026-10 / 2026-11 refresh / full 90-day corpus) | Notes |
| --- | --- | --- |
| MCP servers, MCP tool-server development, agent-SDK familiarity | 4/31 (13%) → 4/27 (15%) → 7/31 (23%) → **12/33 (36%)** / **23/89 (26%) full 90-day corpus** | **FOUR-CYCLE HARDENING — threshold crossed in the 2026-11 fresh pool for the first time.** Fresh evidence (6sense: "Redesign and scale the MCP layer for production workloads, including concurrency, per-tenant quotas, latency budgets, caching, and graceful degradation"; Fanatics Commerce: "Agentic systems and MCP server and client integrations. Agent runtimes such as AWS Bedrock AgentCore matter here"; Fanatics Inc.: "Fluency with the modern AI stack: LLM APIs, MCP, agent frameworks (Claude Agent SDK, OpenAI Agents SDK, LangGraph, or similar)"; Figma, Cymulate, Dscout, Aspenview, CodeRoad, Nimble Gravity, Anthropic Public Sector, Xsolla, Particle41) treats MCP as production tool-registry hygiene. The full-90-day-corpus ratio (23/89 = 26%) still sits below 30%, and per the ownership rule MCP server/tool-server development belongs to the peer [Agentic AI Developer](https://github.com/ai-engineering-curriculum/agentic-ai-developer-learning) track. **This track continues to link out and does NOT adopt.** The 2026-11 threshold crossing escalates the cross-track proposal urgency: raise it in `agentic-ai-developer-learning` this cycle. |
| Computer use / browser automation (Playwright, CUA) | 3/31 (10%) → 4/27 (15%) → 2/31 (6%) → **0/33 (0%)** | Signal flat-to-softening. No fresh 2026-11 mentions. Still owned by Agentic AI Developer if adopted. Watch, but downgraded. |
| LLM observability / tracing as a distinct sub-specialty (Langfuse, Braintrust, LangSmith, OTel-for-LLMs) | 15/31 (48%) → 4/27 (15%) → 8/31 (26%) → **6/33 (18%)** | Below threshold in 2026-11 fresh pool. Named platforms broaden further (Braintrust, LangSmith, Datadog LLM Observability, Arize Phoenix cited at Dscout, Fanatics Commerce, Further). Still bifurcated between generic ops and dedicated LLM-observability specialty. Depth remains delegated to [AI Evaluation Engineer](https://github.com/ai-engineering-curriculum/ai-eval-engineer-learning). |
| Voice / multimodal agents (real-time voice, ASR/TTS in the agent loop) | not sampled in 2026-08 → 1/27 (4%) → 1/31 (3%) → **3/33 (9%)** | Signal soft-broadens — Material Bank (multimodal images), Sia (text/audio/image), Nimble Gravity (multimodal AI). Still thin and still bundled rather than a dedicated role. Ownership belongs to [Applied AI Engineer](https://github.com/ai-engineering-curriculum/applied-ai-engineer-learning) or the Agentic AI family if adopted. Watch. |
| Fluency with agentic coding tools (Claude Code, Cursor, Codex, Gemini CLI) as a stated requirement | 6/31 (19%) → 2/27 (7%) → 3/31 (10%) → **2/33 (6%)** | Phrasing stayed sharp (Xsolla: "building custom Lovable or Claude Code"; Mercury: "actively incorporates AI tools") but the explicit-count stayed low. Remains a prerequisite framing, not a curriculum target. |
| Enterprise-system integration (Salesforce/SAP/ServiceNow/CAD/industrial) | 5/31 (16%) → 1/27 (4%) → 0/31 (0%) → **2/33 (6%)** | Fresh 2026-11 mentions at Anthropic Startups (Physical AI, CAD/industrial) and Fanatics Inc. (corporate functions). Employer-specific; out of scope for a level-20 learning track. |
| Domain compliance (HIPAA / SOC 2 / FedRAMP / air-gapped) | 7/31 (23%) → 0/27 (0%) → 1/31 (3%) → **2/33 (6%)** | Fresh at Anthropic Public Sector ("air-gapped or classified environments") and Accenture Federal ("Cleared GenAI Engineer"). Employer-specific; belongs to Applied AI Engineer or the governance curriculum. |
| Typed-agent frameworks (Pydantic AI explicitly cited) | not sampled → 0/27 → 4/31 (13%) → ~0/33 in 2026-11 explicit | 2026-10 Pydantic AI naming spike (Flagship, Axle, CoLab, Firmus) did not repeat in the 2026-11 fresh pool — reads as a flavour of agent framework and remains delegated to Agentic AI Developer. |
| **"Context engineering" as a named discipline alongside prompt engineering** | not sampled → not sampled → 2/31 (6%) → **8/33 (24%)** phrasing hardening | 2026-11 phrasing watch — appears at AECOM, M3 USA, Cymulate, Fanatics Commerce, Dscout, Anthropic (×3), Fanatics Inc., Mitratech. Reads as a naming variant of prompt engineering (what context to put in the window + how to compose system prompts, retrieval, tool definitions). Already covered by mod-001 (prompt engineering + schema-constrained output) and planned mod-005 (retrieval basics). Naming update only; no new requirement. |
| **LLM gateway / multi-provider routing / token-cost optimization as a dedicated bundle** | not separately sampled → **4/33 (12%)** in 2026-11 | New compound phrasing — Fanatics Commerce: "Build the enterprise LLM gateway: multi-provider routing, failover, caching, rate limiting, token optimization, and key management"; SimplifyNext (vLLM/Ollama inference efficiency); Sia (cost-performance tradeoffs); BPD (cost monitoring). Fits the planned `mod-004` scope precisely; no new content needed. |
| **AWS Bedrock AgentCore / Strands as a named production agent runtime** | not separately sampled → **4/33 (12%)** in 2026-11 | New compound phrasing — Fanatics Commerce, Taxbit, CodeRoad name AgentCore/Strands as a production agent runtime; PayPay Card (2026-10 dup) does the same. Agent-runtime-as-a-service is a level-30 production concern; delegation to Applied AI Engineer (cloud deployment) or Agentic AI Developer (agent runtime) stands. |
| Semantic / response caching + model routing (cost engineering as a named skill) | not separately sampled — see "LLM gateway" row above | Subsumed into the new LLM-gateway compound signal; same fit, same mod-004 scope. |

## This-cycle decision summary

- **Additions proposed (2026-11):** none. See [`.aicg/curriculum-plan-delta.json`](.aicg/curriculum-plan-delta.json).
- **Rationale (2026-11 refresh):** The fresh 33-posting pass (2026-11-02) confirms the rank order of the 2026-08 baseline for a third consecutive refresh on every requirement this track owns. The one material shift is the MCP signal finally crossing 30% in the fresh pool (12/33 = 36%) after four cycles of hardening (13% → 15% → 23% → 36%); the full-90-day-corpus ratio sits at 26% (23/89), still below threshold. Per the ownership rule — primary coverage to the lowest-level role that genuinely requires the skill — MCP server/tool-server development is owned by the peer [Agentic AI Developer](https://github.com/ai-engineering-curriculum/agentic-ai-developer-learning) track (also level 20), not here. This track continues to link out rather than duplicate content funded in a peer track. The clearing escalates the urgency of raising a cross-track adoption proposal in `agentic-ai-developer-learning` this cycle. Separately, the fresh pool revealed a terminology shift: explicit "tool/function calling" mentions dropped (18%) because the phrase is now bundled inside "modern AI stack: LLM APIs, MCP, agent frameworks" compound requirements (Fanatics Inc., Figma, Fanatics Commerce) — demand did not fall, it moved. mod-002 (planned) remains the right landing spot and the next content-production priority. "Context engineering" surfaced as a 2026-11 naming variant for prompt engineering (24% of fresh postings), already covered by mod-001 + planned mod-005. "LLM gateway" + "multi-provider routing" + "AgentCore agent runtime" are compound phrasings that fit `mod-004` (planned) and the delegated Applied AI Engineer scope — no re-scoping needed. Continuity bias ("prefer citing existing content over proposing new content") remains correct for a cohort still mid-flight through shipped mod-001 and queued mod-002 through mod-007.
- **Watch list for next cycle (updated 2026-11):**
  - **MCP server / tool-server development** — four cycles of hardening; threshold-crossed in fresh pool at 36% (90-day corpus 26%). Cross-track proposal in `agentic-ai-developer-learning` is now overdue; verify whether that track adopted before the next LLM-App-Dev cycle.
  - **Tool/function calling** — explicit mentions dropped to 18% in 2026-11 fresh pool due to compound-phrase bundling; underlying demand likely stable or growing when counted implicitly through MCP + agent-framework mentions. mod-002 content production remains the next cycle's priority.
  - **Context engineering as a naming variant** — new 24% phrasing signal in 2026-11; cover inside existing mod-001 (prompt engineering) + mod-005 (retrieval basics) narration without a new module.
  - **LLM gateway / multi-provider routing / AgentCore runtime** — new 2026-11 compound phrasing at 12% each; fits mod-004 planned scope and the Applied AI Engineer delegation. No scope change.
  - **Voice / multimodal agents** — soft-broadens to 9% in 2026-11; still bundled, still thin. Delegated.
  - **Streaming / async** — 0% explicit in 2026-11 fresh pool; still gap-below-threshold cumulatively. Already the planned scope of mod-003.
  - **Computer-use / browser automation** — 0% in 2026-11; downgraded from the active watch list.
