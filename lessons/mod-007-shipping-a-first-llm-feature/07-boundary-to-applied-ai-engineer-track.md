# Chapter 7 — The boundary to `applied-ai-engineer-learning`

The last chapter of this module, and the last chapter of the LLM Application Developer track. Its job is short: to tell you what this track is *not* — so you do not spend six months reinventing platform infrastructure that a peer track already covers as its main material.

## Motivation

The line matters because both directions of overreach are expensive.

- **Reinventing the applied-AI track's material inside a single feature.** Bringing up your own IaC-managed deployment on AWS Bedrock, wiring up your own CI/CD, standing up your own OpenTelemetry pipeline with LLM semantic conventions, running your own multi-tenant guardrail stack — every one of those is a serious project, and every one of them has a well-lit path in [`applied-ai-engineer-learning`](https://github.com/ai-engineering-curriculum/applied-ai-engineer-learning) (level 30). Trying to compress those into a "shipping-my-first-feature" module produces a shallow, brittle version of each.
- **Being an application developer who assumes platform is someone else's problem forever.** The applied-AI track is where you go when this module stops being enough. Knowing what lives on the other side of the boundary is what lets you *choose to graduate*, instead of getting there by accident because your feature outgrew what you learned here.

## What is in scope for this module

Everything chapters 1–6 covered:

- **A single HTTP endpoint** on a chosen framework (FastAPI or comparable) that validates its input, streams its response, and enforces a per-request token / cost budget.
- **Environment-based secret management** with a per-environment store and no keys in code, in logs, or in the repository.
- **Basic redacted tracing** — request id, prompt fingerprint, token counts, latency, cost, stop reason — emitted to stdout for a downstream collector.
- **Feature-flagged model / provider swaps** with canary → ramp → bake → cleanup, and a flag layer that fails open.
- **A one-page runbook** for the three most likely failure modes, with immediate steps and named escalations.
- **App-developer defences** against prompt injection and PII leakage: structural framing, output validation, boundary sanitisation.

That is a complete first shipping story for a single feature owned by a small team. It is deliberately narrow.

## What is *out* of scope and lives in `applied-ai-engineer-learning`

Not exhaustive, but the categories a first-feature shipper is most likely to reach for and misjudge:

- **Cloud-hosted managed LLM inference — Bedrock, Vertex, Azure OpenAI.**
  AWS Bedrock (<https://aws.amazon.com/bedrock/>), Google Vertex AI (<https://cloud.google.com/vertex-ai>), Azure OpenAI Service (<https://azure.microsoft.com/en-us/products/ai-services/openai-service>). Each has its own auth model (IAM roles, service accounts, managed identities), its own pricing surface, its own model availability, its own quota-management story. The trade-offs among the three, and the reasons to run in one vs. bring-your-own-key with a direct-to-provider client, are a chapter in themselves in the applied track. This module deliberately stays on the direct-to-provider API — one auth model, one pricing page, one endpoint URL.
- **Infrastructure-as-Code and reproducible environments.**
  Terraform (<https://developer.hashicorp.com/terraform/docs>), Pulumi (<https://www.pulumi.com/docs/>), CloudFormation, CDK. The "you can rebuild the environment from source" story. Every production-scale LLM app runs in an IaC-managed environment; a first feature can survive with hand-provisioned resources and a documented setup. The applied track owns the "your infra is a repo" transition.
- **CI/CD pipelines beyond a golden-set gate.**
  GitHub Actions / GitLab CI / Buildkite pipelines that build container images, push them to registries, run integration tests against ephemeral environments, and roll to staging then production via progressive delivery (Argo Rollouts, Flagger). The mod-006 golden-set-in-CI is a *component* of this pipeline; the pipeline itself — with promotion gates, image-signing, and provenance — is a chapter in the applied track.
- **Container orchestration and horizontal scaling.**
  Kubernetes (<https://kubernetes.io/docs/>), ECS, Cloud Run, Nomad. The "handle 1000 QPS across N replicas with auto-scaling" story. This module's endpoint fits on one host. The applied track owns everything about running it on many.
- **Platform-scale observability.**
  OpenTelemetry with the LLM semantic conventions (<https://opentelemetry.io/docs/specs/semconv/gen-ai/>), Datadog LLM Observability, Grafana + Loki + Tempo, hosted platforms like Langfuse (<https://langfuse.com/docs>) and LangSmith (<https://docs.smith.langchain.com/>). The chapter-3 trace record is the *primitive* every one of those consumes; the *platform* — trace search across services, per-user spans, tool-call trees, per-tenant dashboards — is applied-track material.
- **Cost governance across many features.**
  Per-tenant billing, showback / chargeback, budget alerting, quota enforcement at the org level. Chapter 3's `cost_usd_estimate` is a *number*; a real cost-governance story is a *system*.
- **Production guardrail infrastructure.**
  NVIDIA NeMo Guardrails (<https://docs.nvidia.com/nemo/guardrails/>), Llama Guard (<https://huggingface.co/meta-llama/Llama-Guard-3-8B>), OpenAI's moderation API integrated at the platform layer. Chapter 6 named these as the natural next layer above the app-developer defences. The integration, calibration, and per-tenant policy stack all live in the applied track.
- **PII detection and redaction at scale.**
  Microsoft Presidio (<https://microsoft.github.io/presidio/>), Amazon Macie, GCP DLP. A first-feature endpoint can survive without an automated PII pipeline; a multi-feature platform cannot.
- **Multi-region deployment and failover.**
  Provider-side failover, cross-region redundancy, latency-aware routing. Every one of these is a distinct chapter in the applied track. This module's "roll back the flag" story is not the multi-region story.
- **Async job orchestration for long generations.**
  Batch APIs (<https://platform.openai.com/docs/guides/batch>, <https://docs.anthropic.com/en/docs/build-with-claude/batch-processing>), background workers, queue-based fan-out, long-running generation with progress webhooks. Mod-003 taught the primitives; the applied track owns the platform shape.

Every one of the above is a *good* problem. Every one of the above is a bad problem to invent from scratch inside a mod-007-shaped setup.

## When to graduate

Rough signals — no single one is decisive; together they are unambiguous:

- **You are shipping the second and third LLM feature.** Two features you can run in parallel with two runbooks and two flag configurations. Three or more, and you start wanting shared platform: a common trace shape, a common flag model, a common cost dashboard, a shared secrets story. That is the applied track's opening act.
- **The feature runs on more than one host.** Once you are behind a load balancer with N replicas, the "restart the service" step from your runbook has to become "roll the deployment," and every operational habit changes shape. That is where IaC, container orchestration, and progressive delivery all start earning their keep.
- **A different team owns the platform than the feature.** The moment you say "the platform team is asking us to migrate to the shared observability pipeline," the platform team is doing applied-AI-engineering work — and knowing what they know is how you have a productive conversation with them.
- **A product decision hinges on multi-region latency or availability.** Serving from three regions with automatic failover is not a chapter in this module; it is the whole opening of the applied track.
- **You need SOC 2 / HIPAA / ISO 27001 for the LLM feature specifically.** Every one of those is a checklist that lives in the applied track — audit-ready logging, tenant isolation, data-residency controls, signed provenance. Do not try to retrofit it under time pressure; ramp into it deliberately.
- **The bill has become someone's problem.** A first feature at low volume is a rounding error; a platform of features at high volume is a P&L line. Once finance is asking questions, the applied track's cost governance chapter is where the answer lives.

## What mod-007 does prepare you for

Even after graduating to the applied track, the shape you learned here holds:

- **The endpoint shape from chapter 1** (schema-in, stream-out, budget-check) is the same shape you deploy at scale. The differences are the *deployment* and the *orchestration*, not the endpoint.
- **The secrets discipline from chapter 2** does not change when you move to Bedrock / Vertex / Azure OpenAI. The auth model changes (IAM roles instead of an API key), but "no secret in the repo, no secret in the log" is invariant.
- **The trace record from chapter 3** is the exact shape OpenTelemetry LLM conventions formalise. Understanding it in raw form makes the semantic conventions make sense.
- **The flag-ramp shape from chapter 4** is the same shape a progressive-delivery tool (Argo Rollouts, Flagger) automates for you. The tool is doing what you already know how to do by hand — automating the ramp, watching the signals, rolling back on threshold breach.
- **The runbook from chapter 5** is the primitive for every incident-management platform (PagerDuty runbook automation, Grafana OnCall, Rootly). The platforms wrap the discipline; they do not replace it.
- **The app-developer defences from chapter 6** are the always-on foundation under any guardrail infrastructure you eventually add. NeMo Guardrails does not un-need your structural framing; it does not un-need your output validation.

If you have done this module honestly, the applied track's job is to add scale and depth — not to un-teach anything.

## The end of the LLM Application Developer track

You have finished the seven modules:

1. Prompt engineering foundations.
2. Tool / function calling.
3. Streaming, async, orchestration.
4. Model selection, cost, prompt caching.
5. Retrieval basics.
6. Minimal eval and regression checks.
7. Shipping a first production LLM feature — this one.

Between them, they are the working knowledge to *own* an LLM feature end to end for a small team: choose the model on evidence, ground it when the model needs facts it does not have, wire it into an endpoint that survives production traffic, keep it working with a gated eval, and respond to incidents when it does not.

The peer tracks — [`ai-eval-engineer-learning`](https://github.com/ai-engineering-curriculum/ai-eval-engineer-learning), [`ai-risk-engineer-learning`](https://github.com/ai-engineering-curriculum/ai-risk-engineer-learning), and [`applied-ai-engineer-learning`](https://github.com/ai-engineering-curriculum/applied-ai-engineer-learning) — pick up each of the specialised depths this track deliberately stopped short of. The right time to graduate to any of them is the moment their absence starts costing you more than their presence would.

## Summary

- mod-007 is the *first-feature shipping* module. Its job is a single endpoint, well operated, by a small team.
- Everything past that — Bedrock / Vertex / Azure inference, IaC, CI/CD beyond the eval gate, container orchestration, platform-scale observability, cost governance, production guardrails, PII redaction at scale, multi-region failover, async job platforms — lives in [`applied-ai-engineer-learning`](https://github.com/ai-engineering-curriculum/applied-ai-engineer-learning) (level 30).
- Signals to graduate: the second and third feature, more than one host, a platform team, multi-region latency, compliance certification, a real bill.
- The patterns you learned here — schema-validated endpoint, per-environment secrets, redacted traces, flag-gated rollouts, one-page runbooks, boundary defences against injection and PII — carry forward. The applied track adds scale and depth; it does not replace them.

You have finished the LLM Application Developer track. The next feature you ship is where this all becomes muscle memory.
