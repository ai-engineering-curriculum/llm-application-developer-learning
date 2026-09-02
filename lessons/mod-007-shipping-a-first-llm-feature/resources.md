# Resources for mod-007 — Shipping a First Production LLM Feature

Prefer primary sources — provider documentation, tool maintainers, standards bodies — over blog posts. The list below is deliberately short. If a link goes stale, the fix is to look up the maintainer's current documentation index, not to grab whatever a search engine returns first.

## Chapter 1 — the HTTP endpoint, streaming, and budgeting

### The web framework

- **FastAPI documentation index.** <https://fastapi.tiangolo.com/>
- **FastAPI tutorial: request body.** How Pydantic-typed bodies produce 422 responses for free. <https://fastapi.tiangolo.com/tutorial/body/>
- **FastAPI `StreamingResponse`.** The async-generator sandwich the streaming endpoint returns. <https://fastapi.tiangolo.com/advanced/custom-response/#streamingresponse>
- **Pydantic (v2) documentation.** The validator behind `Field(min_length=..., max_length=..., le=..., ge=...)`. <https://docs.pydantic.dev/latest/>
- **Starlette (FastAPI's ASGI foundation).** Named in case you want to read the streaming and cancellation primitives one layer down. <https://www.starlette.io/>

### Provider SDKs and streaming

- **Anthropic — Messages API streaming.** The `messages.stream` context-manager form used in the chapter. <https://docs.anthropic.com/en/api/messages-streaming>
- **Anthropic — token counting endpoint.** Server-authoritative input-token count for the budget check. <https://docs.anthropic.com/en/docs/build-with-claude/token-counting>
- **OpenAI — streaming.** The `stream=True` iterator form. <https://platform.openai.com/docs/api-reference/streaming>
- **OpenAI — Chat Completions reference.** <https://platform.openai.com/docs/api-reference/chat>
- **`tiktoken`.** The client-side tokeniser you use to pre-flight-budget OpenAI requests (with a small safety margin, as the chapter notes). <https://github.com/openai/tiktoken>

<!-- needs-research: confirm the current Anthropic Python SDK streaming context-manager API name (messages.stream vs. messages.create with stream=True) as of 2026-08. -->

### HTTP status semantics

- **RFC 9110 (HTTP semantics).** The authoritative source for the 413 / 429 / 503 / 504 status codes the chapter uses. <https://www.rfc-editor.org/rfc/rfc9110.html>

## Chapter 2 — secrets, environment, and per-environment config

### Local development

- **`python-dotenv`.** Loads `.env` into `os.environ` at process start. <https://github.com/theskumar/python-dotenv>
- **`dotenv` (Node).** The Node equivalent. <https://github.com/motdotla/dotenv>
- **`pydantic-settings`.** Typed settings model with env-var and `.env` loading built in. Useful once your config grows past ~10 knobs. <https://docs.pydantic.dev/latest/concepts/pydantic_settings/>
- **`direnv`.** Auto-load `.env`-shaped files on `cd`. Optional; nice across many projects. <https://direnv.net/>

### Secret scanning

- **`gitleaks`.** Pre-commit and repo-scan tool for detecting committed secrets. <https://github.com/gitleaks/gitleaks>
- **`detect-secrets` (Yelp).** Baseline-oriented alternative. <https://github.com/Yelp/detect-secrets>
- **`pre-commit`.** The framework the hooks above plug into. <https://pre-commit.com/>
- **GitHub secret scanning.** GitHub's built-in scanner for public and Advanced-Security repos. <https://docs.github.com/en/code-security/secret-scanning/about-secret-scanning>

### Cloud-provider secret managers

- **AWS Secrets Manager.** <https://docs.aws.amazon.com/secretsmanager/>
- **GCP Secret Manager.** <https://cloud.google.com/secret-manager/docs>
- **Azure Key Vault (secrets).** <https://learn.microsoft.com/en-us/azure/key-vault/secrets/>
- **Kubernetes Secrets.** <https://kubernetes.io/docs/concepts/configuration/secret/>
- **Kubernetes encryption-at-rest.** Named because the default Kubernetes Secret is not encrypted at rest in etcd. <https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/>
- **HashiCorp Vault.** <https://developer.hashicorp.com/vault/docs>

### Developer-secrets sync layers

- **Doppler.** <https://docs.doppler.com/>
- **Infisical.** <https://infisical.com/docs>
- **1Password Secrets Automation.** <https://developer.1password.com/docs/secrets-automation/>

### Provider key management

- **Anthropic — getting started (API keys).** <https://docs.anthropic.com/en/api/getting-started>
- **OpenAI — API keys and authentication.** <https://platform.openai.com/docs/api-reference/authentication>

## Chapter 3 — traces, redaction, and the twelve-factor log stream

### Standards and shape

- **The Twelve-Factor App — logs.** The "log to stdout, let the platform handle transport" pattern the chapter follows. <https://12factor.net/logs>
- **OpenTelemetry semantic conventions for GenAI.** The vendor-neutral shape the chapter-3 record foreshadows; named so the graduation to platform-scale observability is not a fresh vocabulary. <https://opentelemetry.io/docs/specs/semconv/gen-ai/>

### Logging libraries

- **Python `logging` (stdlib).** The foundation. <https://docs.python.org/3/library/logging.html>
- **`python-json-logger`.** JSON-formatter for stdlib `logging` — the cheapest way to emit one JSON object per record. <https://github.com/madzak/python-json-logger>
- **`structlog`.** Structured-logging library with context-vars, processors, and a JSON renderer. Nicer once you want request-scoped context threaded automatically. <https://www.structlog.org/>
- **`time.perf_counter` (Python).** The monotonic clock the chapter uses for `latency_ms` and `time_to_first_token_ms`. <https://docs.python.org/3/library/time.html#time.perf_counter>

### Log collectors named for the "ship to stdout" story

- **Vector.** <https://vector.dev/docs/>
- **Fluent Bit.** <https://docs.fluentbit.io/manual/>
- **Datadog Agent.** <https://docs.datadoghq.com/agent/>
- **AWS CloudWatch Logs agent.** <https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AgentReference.html>

### Hosted LLM-observability platforms named for the graduation

Not for use in this module — for use after chapter 7's boundary is crossed. Named so the terminology is familiar.

- **Langfuse.** <https://langfuse.com/docs>
- **LangSmith.** <https://docs.smith.langchain.com/>
- **Arize Phoenix.** <https://docs.arize.com/phoenix>
- **Braintrust.** <https://www.braintrust.dev/docs>
- **Datadog LLM Observability.** <https://docs.datadoghq.com/llm_observability/>

## Chapter 4 — feature flags and safe model rollouts

### Provider pricing (for the exercise-03 price table and the exercise-04 canary comparison)

- **Anthropic pricing.** <https://www.anthropic.com/pricing>
- **OpenAI pricing.** <https://openai.com/api/pricing/>
- **Anthropic model reference.** <https://docs.anthropic.com/en/docs/about-claude/models>
- **OpenAI model reference.** <https://platform.openai.com/docs/models>
- **OpenAI deprecations.** The page to check for silent-snapshot-roll risk. <https://platform.openai.com/docs/deprecations>

<!-- needs-research: confirm current default / cheap-tier / frontier model IDs for Anthropic and OpenAI, and their per-token prices, as of 2026-08 — cite from the pricing pages above. -->

### Feature-flag services

- **LaunchDarkly.** <https://launchdarkly.com/docs/home>
- **Statsig.** <https://docs.statsig.com/>
- **ConfigCat.** <https://configcat.com/docs/>
- **Flagsmith.** <https://docs.flagsmith.com/>
- **Unleash.** <https://docs.getunleash.io/>
- **Split.** <https://help.split.io/>
- **PostHog Feature Flags.** <https://posthog.com/docs/feature-flags>
- **GrowthBook.** <https://docs.growthbook.io/>

<!-- needs-research: verify current feature-flag vendor list and their canonical docs URLs as of 2026-08 — this space consolidates and re-brands frequently. -->

### Progressive-delivery tooling (referenced in chapter 7)

- **Argo Rollouts.** <https://argo-rollouts.readthedocs.io/en/stable/>
- **Flagger.** <https://docs.flagger.app/>

### File-watching for a homegrown YAML/JSON flag file

- **`watchdog` (Python).** <https://python-watchdog.readthedocs.io/>
- **`chokidar` (Node).** <https://github.com/paulmillr/chokidar>

## Chapter 5 — runbooks and incident response

### Practice and philosophy

- **Google SRE Book.** The foundational text; the "Testing for Reliability" and "Practical Alerting" chapters are the most directly relevant. <https://sre.google/sre-book/table-of-contents/>
- **Google SRE Workbook.** The practical companion volume. <https://sre.google/workbook/table-of-contents/>

### Paging and runbook-automation platforms

- **PagerDuty.** <https://www.pagerduty.com/docs/>
- **PagerDuty runbook automation.** <https://support.pagerduty.com/docs/runbook-automation>
- **Opsgenie.** <https://support.atlassian.com/opsgenie/>
- **Grafana OnCall.** <https://grafana.com/docs/oncall/latest/>
- **Rootly.** <https://docs.rootly.com/>

### Provider status pages named in the chapter's template

- **Anthropic status.** <https://status.anthropic.com/>
- **OpenAI status.** <https://status.openai.com/>
- **AWS status.** <https://health.aws.amazon.com/health/status>
- **GCP status.** <https://status.cloud.google.com/>
- **Azure status.** <https://azure.status.microsoft/en-us/status>

## Chapter 6 — prompt injection, PII, and app-developer defences

### Provider prompt-engineering and guardrail guidance

- **Anthropic — prompt engineering overview.** <https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview>
- **Anthropic — mitigate jailbreaks.** The direct reference for the structural-framing pattern the chapter uses. <https://docs.anthropic.com/en/docs/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks>
- **OpenAI — prompt engineering guide.** <https://platform.openai.com/docs/guides/prompt-engineering>
- **OpenAI — moderation guide.** The moderation-API endpoint chapter 6 names as the layer above your defences. <https://platform.openai.com/docs/guides/moderation>

<!-- needs-research: confirm the current Anthropic jailbreak-mitigation and prompt-engineering doc URLs as of 2026-08 — Anthropic has reorganised these pages more than once. -->

### Standards and community threat catalogues

- **OWASP Top 10 for Large Language Model Applications.** The reference threat catalogue for LLM systems. Skim, do not memorise. <https://owasp.org/www-project-top-10-for-large-language-model-applications/>
- **Trojan Source paper.** The rationale for rejecting bidirectional and zero-width control characters at the boundary. <https://trojansource.codes/>
- **Unicode Normalization (UAX #15).** Reference for NFKC normalisation. <https://unicode.org/reports/tr15/>
- **Python `unicodedata`.** The stdlib module the boundary-sanitisation exercise uses. <https://docs.python.org/3/library/unicodedata.html>

### Classifier and guardrail primitives named as *signals, not answers*

- **Meta Prompt Guard.** A prompt-injection classifier. <https://huggingface.co/meta-llama/Prompt-Guard-86M>
- **Llama Guard 3 (8B).** A content-safety classifier. <https://huggingface.co/meta-llama/Llama-Guard-3-8B>
- **NVIDIA NeMo Guardrails.** Guardrails framework for LLM applications. <https://docs.nvidia.com/nemo/guardrails/>
- **Lakera Guard.** Hosted prompt-injection and content-safety service. <https://www.lakera.ai/guard>
- **Microsoft Presidio.** PII detection and redaction library. <https://microsoft.github.io/presidio/>

### Provider data-handling policy pages

- **OpenAI — enterprise privacy.** <https://openai.com/enterprise-privacy/>
- **Anthropic — commercial terms.** <https://www.anthropic.com/legal/commercial-terms>

<!-- needs-research: verify current OpenAI and Anthropic data-handling policy URLs as of 2026-08 — vendors update these frequently. -->

## Chapter 7 — the boundary to the applied-AI-engineer track

### Managed LLM inference

- **AWS Bedrock.** <https://aws.amazon.com/bedrock/>
- **Google Vertex AI.** <https://cloud.google.com/vertex-ai>
- **Azure OpenAI Service.** <https://azure.microsoft.com/en-us/products/ai-services/openai-service>

### Infrastructure-as-Code

- **Terraform.** <https://developer.hashicorp.com/terraform/docs>
- **Pulumi.** <https://www.pulumi.com/docs/>
- **AWS CloudFormation.** <https://docs.aws.amazon.com/cloudformation/>
- **AWS CDK.** <https://docs.aws.amazon.com/cdk/>

### Container orchestration

- **Kubernetes.** <https://kubernetes.io/docs/>
- **Amazon ECS.** <https://docs.aws.amazon.com/ecs/>
- **Google Cloud Run.** <https://cloud.google.com/run/docs>
- **HashiCorp Nomad.** <https://developer.hashicorp.com/nomad/docs>

### Async and batch job platforms

- **OpenAI Batch API.** <https://platform.openai.com/docs/guides/batch>
- **Anthropic Message Batches.** <https://docs.anthropic.com/en/docs/build-with-claude/batch-processing>

### The graduation destination

- **`applied-ai-engineer-learning` (level 30).** Every "out of scope for mod-007" topic lands here as its main material — cloud-hosted managed inference, IaC, CI/CD beyond the eval gate, container orchestration, platform-scale observability, cost governance, production guardrails, PII redaction at scale, multi-region failover, async job orchestration. <https://github.com/ai-engineering-curriculum/applied-ai-engineer-learning>

## Cross-references into other modules on this track

The chapters reach back into earlier modules more often than they reach out to external sources. If any of these feel shaky, re-read them before you finish the module.

- **mod-001 chapter 3** — token counting and per-request cost arithmetic. Chapter 1's `INPUT_TOKEN_LIMIT` / `OUTPUT_TOKEN_LIMIT` numbers come from this. `../mod-001-prompt-engineering-foundations/`
- **mod-001 chapter 4** — schema-constrained JSON output. Chapter 6's output validation is a strict extension of this discipline. `../mod-001-prompt-engineering-foundations/`
- **mod-003 chapter 1** — streaming primitives. Chapter 1's `StreamingResponse` is the HTTP wrapper around what mod-003 already taught. `../mod-003-streaming-async-and-orchestration/`
- **mod-003 chapter 4** — retry with jittered exponential backoff. Chapter 1's "do not retry silently inside the handler" note is the mirror of this — the retry belongs at the caller layer. `../mod-003-streaming-async-and-orchestration/`
- **mod-004 chapter 2** — cost estimation. Chapter 3's `cost_usd_estimate` field and the exercise-04 canary comparison both trace back here. `../mod-004-model-selection-cost-and-prompt-caching/`
- **mod-004 chapter 3** — prompt caching. Chapter 3's `cached_input_tokens` field is only meaningful once caching is on. `../mod-004-model-selection-cost-and-prompt-caching/`
- **mod-006 chapter 4** — regression tracking across model / prompt changes. Chapter 4's "eval as a rollout signal" is this eval, run continuously against the canary configuration. `../mod-006-minimal-eval-and-regression-checks/`

## Peer tracks named across the module

- **`applied-ai-engineer-learning` (level 30).** The graduation track. <https://github.com/ai-engineering-curriculum/applied-ai-engineer-learning>
- **`ai-risk-engineer-learning` (level 25).** The red-team-engineering track named in chapter 6. <https://github.com/ai-engineering-curriculum/ai-risk-engineer-learning>
- **`ai-eval-engineer-learning` (level 30).** The eval-depth track named in the mod-006 boundary; still the right pointer once your eval story outgrows a single golden-set gate. <https://github.com/ai-engineering-curriculum/ai-eval-engineer-learning>

## What is deliberately not on this list

- **"How to build a production LLM app" blog posts.** Almost all of them either sell one vendor's platform or compress everything from chapter 1 to chapter 7 into a checklist that assumes the reader already knows the shape. Learn the shape first (this module), evaluate the vendor pitch second.
- **Screenshots of provider pricing pages.** Prices change; screenshots do not. Always link to the current pricing page and put the number you used in the code with the date you copied it.
- **Curated "prompt-injection example" lists.** Attackers do not read curated lists. Structural framing, output validation, and boundary sanitisation are the defences that hold up regardless of the wording of the specific attack.
- **Kubernetes / IaC / CI/CD deep material.** Those are the applied track's main content; the pointers above are enough for this module. Learning them here is what chapter 7 exists to talk you out of.
