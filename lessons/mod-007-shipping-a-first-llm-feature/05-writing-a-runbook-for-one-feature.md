# Chapter 5 — Writing a runbook a colleague can use during an incident

A runbook is a short document that describes, in plain language, how to detect and respond to the three failure modes most likely to happen to your feature — and who to call when the runbook stops working. It is the *cheapest* piece of operational insurance you can write. It is also the piece that gets skipped most often, because "we all know how it works." At 3 am, the person on call is not the person who built it, and "we all know" is not a phrase anyone can act on.

## Motivation

Two situations that make a runbook non-optional:

1. **Someone else is on call.** Your teammate — or an oncall engineer from another team — is paged for a feature they did not build. They have five minutes to understand what to do before every additional minute has a customer-impact cost. Without a runbook they read your code, guess, and often make the incident worse. With a runbook they open the page, do steps 1–3 for the matching failure mode, and buy time to escalate.
2. **You are on call and it is 3 am.** Fatigue makes you skip the diagnostic step and jump to "just roll back everything." Sometimes that is right; often it is not, and a specific rollback is faster and less disruptive than a global one. A runbook you wrote when you were awake is smarter than you at 3 am.

The specific goal for this module: **write the runbook that gets your feature through its first six months.** Not exhaustive; not covering every possible mode. The three modes that will actually happen, with the immediate response for each and the escalation contact for when they do not resolve in five minutes.

## What a good runbook is not

Three shapes people confuse with a runbook:

- **A design doc.** Explains *how the system works*. Useful; wrong artefact for an incident. Nobody reads a design doc at 2 am.
- **A postmortem.** Describes *what happened during a past incident*. Also useful; also wrong artefact — it names the specific fix, not the general shape.
- **A bulleted list of "things to try."** No priorities, no timings, no escalations. The engineer reads it, tries three things at random, and calls you.

A runbook is a **decision tree** at the top and a **fixed procedure** underneath each branch. When symptom X appears, do steps A, B, C in that order. If steps A, B, C do not resolve within N minutes, escalate to person P. That is the whole shape.

## The three most likely failure modes for a first LLM feature

Every feature has different failure modes. For a shipped LLM feature that follows the chapters in this module, the three that account for most of your first-year incidents are:

1. **Provider outage / hard rate-limit / connection failures.** Anthropic 500s, OpenAI 429s across the board, DNS failures, TLS handshake errors. Your endpoint's error rate goes vertical in a few minutes. Cause: provider-side; you cannot fix it, only route around it.
2. **A bad model rollout (or a silent model roll from the provider).** The eval regresses, or user-facing signals (thumbs-down, escalation, follow-up-question rate) spike, or the trace's `stop_reason` distribution shifts. Cause: your recent flag change, or the provider silently updated the snapshot behind your model id.
3. **Runaway cost / quota drain.** The bill is diverging from the trace's `cost_usd_estimate` sum, or the daily-cost gauge from chapter 3 is 5× yesterday's number. Cause: a caller loop, a very large single request, prompt-caching regression, a compromised key, or a prompt change that inflated output length.

Yours may differ. If you have a `mod-005` retrieval step, "retrieval index stale / empty" belongs in the top three. If you use tool calling, "tool loop that will not terminate" belongs. Pick the three most likely for *your* feature, not a generic list.

## Runbook template — one page per feature

The whole runbook fits on a single page. Copy this shape.

```markdown
# Runbook — /summarise

**Owner:** @josh (Josh Ferguson)  •  **Backup owner:** @maria (Maria Nakamura)
**On-call rotation:** ai-app-oncall (PagerDuty)
**Provider status pages:**
  - Anthropic: https://status.anthropic.com/
  - OpenAI: https://status.openai.com/
**Where to look first:**
  - Dashboard: <internal-url>/dashboards/summarise
  - Log search: `route:/summarise last 15m`
  - Feature flags: <flag-service-url>/flags/summarise
**Emergency kill switch:**
  - Set `SUMMARISE_ENABLED=false` in <secret-store-path>. Endpoint returns 503.

## Symptom triage

1. Error rate spike + upstream 5xx/429? → Section A (provider outage)
2. Same error rate but user complaints / eval drop? → Section B (bad model rollout)
3. Cost gauge or usage dashboard 3×+ baseline? → Section C (runaway cost)

## A. Provider outage or hard rate-limit

Symptom: `error_class` = `upstream_5xx` or `upstream_429`, elevated in the log store, spread across users.

Immediate (5 min):
  1. Confirm on provider status page — link above.
  2. If confirmed: flip `canary_ratio` to 0 to concentrate on the healthy fallback provider (see feature flag `summarise:fallback`).
  3. If no fallback exists: set `SUMMARISE_ENABLED=false` and let the endpoint 503. Better than pretending to work.

If not resolved in 15 min: page the backup owner.
Escalate to: provider account manager (Anthropic contact: <internal-contact>).

## B. Bad model rollout (or silent provider snapshot roll)

Symptom: error rate normal, but user-complaint volume up, or the golden-set CI check failed on trunk in the last hour, or `stop_reason` distribution has shifted.

Immediate (5 min):
  1. Check trace bucket by `model` field: which model version is elevated?
  2. If it's a recent canary: set `canary_ratio` to 0 in the flag service. Traffic returns to the previous model within 60s.
  3. If it's the default (no recent flag change) → likely a silent provider snapshot roll. Pin the previous snapshot (see `SUMMARISE_MODEL` in secret store).

If not resolved in 15 min: page the backup owner.
Escalate to: eval-team channel #eval-ops for a rapid golden-set re-run on the pinned snapshot.

## C. Runaway cost

Symptom: cost gauge 3×+ baseline, or daily cost estimate is on track to exceed budget.

Immediate (5 min):
  1. Group cost by `user_id_hash` and `prompt_hash` — is one user or one template responsible?
  2. If one user: apply per-user rate limit in the gateway (documented at <internal-runbook>).
  3. If one prompt hash: swap the flag to the previous `prompt_version`.
  4. If neither: **rotate the API key immediately** — assume compromise until proven otherwise.

If not resolved in 15 min: page the backup owner AND the security-on-call rotation.
Escalate to: finance-ops (for provider spend cap conversation) and security (for key rotation follow-up).
```

The template is *deliberately mundane*. It is a checklist, not a design document. Every step is a specific action a colleague can take without knowing how the feature works internally.

## What each section must contain

- **A symptom sentence.** How the failure looks from outside — from the dashboard, from user reports, from the log stream. This is the "how do I know I'm in this section?" question.
- **Immediate steps with time budgets.** Three to five numbered actions, each doable in under a minute. If a step needs a URL, put the URL in. If it needs a specific env var or flag key, name it.
- **A stop-condition and an escalation contact.** After N minutes, if the steps have not worked, page who? Do not leave this open-ended. "Escalate" without a name is not an escalation.

## What "escalation contact" means

Two roles the runbook should always name:

- **The backup owner.** The teammate who can take over debugging when the primary owner is unavailable. Not the manager. Someone who can actually push a change.
- **The subject-matter fallback.** The team or channel that owns the *deeper* fix. For provider outages, the account manager. For eval regressions, the eval team. For security incidents, the security-on-call rotation. These do not fix the endpoint; they own the next layer.

Escalations should be **automated where possible.** PagerDuty (<https://www.pagerduty.com/docs/>), Opsgenie (<https://support.atlassian.com/opsgenie/>), Grafana OnCall, or your equivalent already knows how to page the right person for a shift. Do not maintain a hand-updated "who is on call" list in the runbook itself — link to the rotation.

## Where the runbook lives

Two things that make a runbook actually usable:

1. **In the same repo as the feature, in a well-known location.** `docs/runbooks/summarise.md`, next to the code that implements it. When someone finds the endpoint, they find the runbook. This is where "runbook is a piece of the feature" comes from — you do not merge the feature without the runbook.
2. **Linked from the on-call tooling.** The paging alert's description should include the runbook URL. The dashboard for the feature should link to the runbook. If someone has to search for it during an incident, it might as well not exist.

## Keeping the runbook honest

Two habits that keep a runbook from rotting:

- **Update it after every real incident.** If the runbook's section for the failure you just handled was not right, fix it in the postmortem PR. This is the single most reliable way to keep the runbook accurate — new failure modes appear, links go stale, tools change.
- **Run a game day once a quarter.** Pick a failure mode, simulate the symptom in a staging environment (a bad prompt version behind the flag, a fake 5xx from the provider via a middleware, a canary rolled to a broken model), and have someone *not* the owner drive through the runbook. Time it. Anything that went wrong is a runbook bug. Reference for the practice: <https://sre.google/sre-book/testing-reliability/>.

## The one-page rule

If your runbook is longer than one page — genuinely one printed page, not one scrollable browser window — it is too long. The person reading it under pressure is going to skim. Prune. If the "detailed background" is important, link to a design doc; do not paste it in.

## Common mistakes

- **A runbook that describes how the system *works* instead of what to *do*.** The design doc is a separate artefact. The runbook is a set of actions.
- **A runbook that lists twelve possible causes without prioritising.** No time to work through twelve; use the top three.
- **Escalation contacts by name only, without a paging route.** People change teams. Roles and rotations are more durable than names — link the rotation, not the individual.
- **A "check the logs" step with no query template.** If the log-search query is not written down, everyone under pressure writes a different one, and half of them miss the interesting field.
- **A runbook that says "restart the service."** Sometimes that is right; usually it is a shortcut that hides the underlying problem. Prefer specific actions over blunt ones.
- **The runbook is right at the point it was written and never revised.** After a real incident, if the runbook did not describe the actual response, update it. This is the whole reason the "update after every incident" habit exists.

## Summary

- A runbook is a decision-tree-plus-fixed-procedure for the three most likely failure modes of one feature. Not exhaustive; not a design doc.
- Each section has: symptom, three to five immediate steps with time budgets, a stop-condition, and an escalation contact.
- Escalation contacts are rotations, not individuals. Automate the paging.
- One page. If it does not fit on a page, it will not be read.
- Lives in the same repo as the feature, linked from every dashboard and every paging alert.
- Update it after every real incident. Game-day it once a quarter.

The next chapter is about the failure mode this runbook does not cover well because it is not always visible in metrics: someone actively trying to make the endpoint do something it should not.
