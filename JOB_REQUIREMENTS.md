# LLM Application Developer — Job Requirements

_Generated 2026-08-02. Sample: 31 postings from 2026-05 through 2026-08 on Greenhouse and SmartRecruiters. Titles pooled: "LLM Application Developer", "Generative AI Developer", "AI Application Engineer", "LLM App Engineer", "GenAI Developer", "GenAI Application Engineer", and entry-to-mid "AI Engineer" listings._

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

- **Coverage:** `gap-below-threshold`. Legitimate territory for this role but under the 30% bar. Track next cycle.
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

| Requirement | Frequency | Notes |
| --- | --- | --- |
| MCP / agent-SDK familiarity | 4/31 (13%) | Track for next cycle; ownership would sit with [Agentic AI Developer](https://github.com/ai-engineering-curriculum/agentic-ai-developer-learning). |
| Computer use / browser automation | 3/31 (10%) | Niche; owned by Agentic AI Developer if adopted. |
| Enterprise-system integration (Salesforce/SAP/ServiceNow) | 5/31 (16%) | Employer-specific; out of scope for a level-20 learning track. |
| Domain compliance (HIPAA / SOC 2 / FedRAMP) | 7/31 (23%) | Employer-specific; belongs to Applied AI Engineer or the governance curriculum. |

## This-cycle decision summary

- **Additions proposed:** none. See [`.aicg/curriculum-plan-delta.json`](.aicg/curriculum-plan-delta.json).
- **Rationale:** Every above-threshold requirement in this role's ownership is already covered by mod-001. The two candidate gaps — dedicated tool/function-calling exercises (58%) and dedicated streaming exercises (26%) — are either exercised in a different framing (tool calling ⊂ existing structured-output drills) or below the 30% evidence bar (streaming). Higher-frequency items (RAG at 87%, agent frameworks at 77%, cloud deployment at 68%, evals at 65%) all belong to lower-siblings or higher-tier tracks per the ownership rule; this document links out to them rather than duplicating.
- **Watch list for next cycle:** streaming/async (26%, growing), MCP / agent SDKs (13%, industry momentum), computer use (10%, provider-driven).
