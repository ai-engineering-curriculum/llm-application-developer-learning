# Chapter 4 — Feature-flagging model and provider swaps

The endpoint in chapter 1 has a `MODEL` constant. The trace in chapter 3 records which model handled every request. This chapter is about the space between: **how you change the model in production without a code deploy, how you roll a change out safely, and — the important half — how you roll it back inside two minutes when it turns out to be a bad idea.**

## Motivation

Two situations that will happen to you in the first year of shipping an LLM feature:

1. **The provider silently rolls the underlying model.** The model id you pinned still exists, but the snapshot behind it moved. Your golden set (mod-006) starts regressing. You want to move traffic to the *previous* snapshot within minutes, not after a code review and a deploy.
2. **You want to try a cheaper or newer model.** The eval says the cheaper model matches quality on 90% of your traffic. Rather than a big-bang swap, you want to route 5% of traffic to the new model, watch the trace fields from chapter 3, and ramp up over a week — with a kill switch if the new model misbehaves.

Both are cases where the *ability to change* is the load-bearing property. The specific mechanism (env vars, a flag service, a config file) is secondary — what matters is that the model id is data, not code, and that changing it is fast and reversible.

## The shape of a model flag

At the simplest, a model flag is one value in your config layer (chapter 2) that the endpoint reads on every request:

```python
# config.py
import os
MODEL = os.environ.get("SUMMARISE_MODEL", "claude-opus-4-7")
```

That works for the "roll back to the previous snapshot" case in a matter of seconds — change the env var in the secret store, restart (or hot-reload) the process, done.

It does *not* work for the "5% of traffic to a new model" case. For gradual rollouts you need one more level: a small function that returns the model for the current request based on something the request carries.

```python
# routing.py
import hashlib

def model_for_request(user_id: str, config: dict) -> str:
    # config = { "default_model": "claude-opus-4-7", "canary_model": "claude-sonnet-4-6", "canary_ratio": 0.05 }
    if not config["canary_ratio"]:
        return config["default_model"]
    # Stable-per-user routing so a single user does not flap between models
    # mid-conversation. sha256 for readability; any stable hash works.
    bucket = int(hashlib.sha256(user_id.encode()).hexdigest(), 16) % 1000
    return config["canary_model"] if bucket < int(config["canary_ratio"] * 1000) else config["default_model"]
```

Two properties this shape has that a naive `random.random() < canary_ratio` does not:

- **Stable-per-user routing.** A given user consistently lands on the same model across requests, so their conversation does not flap mid-thread. This also makes traces group cleanly by (model, user), which is what makes the ramp-up decision defensible.
- **Deterministic replay.** Given the same user id and same config, you get the same routing decision. That is what lets you write a golden-set test that "canary_ratio=0.05 routes 5% ± noise" of a synthetic user list to the new model.

## Where the flag lives

Any of these are fine for a first feature. Pick the one you already have.

- **Environment variables + a process restart.** The "roll back the model in one minute" story. Simple, no dependency. The downside is that gradual rollouts require a config change *per* value of `canary_ratio`, and there is no audit trail. Fine for the emergency-rollback story alone.
- **A committed JSON/YAML file, hot-reloaded on change.** A `config/prompts.yaml` the endpoint watches with `watchdog` or similar. Version-controlled, reviewable in PRs, but a config change is still a deploy (in the "someone pushed a commit" sense). Good for teams where "prompt/model change" is a real code review.
- **A hosted feature-flag service.** LaunchDarkly (<https://launchdarkly.com/docs/home>), Statsig (<https://docs.statsig.com/>), ConfigCat (<https://configcat.com/docs/>), Flagsmith (<https://docs.flagsmith.com/>), Unleash (<https://docs.getunleash.io/>), Split (<https://help.split.io/>), PostHog Feature Flags (<https://posthog.com/docs/feature-flags>), or GrowthBook (<https://docs.growthbook.io/>). Sub-minute rollout changes, an audit trail, a UI a PM can operate, and — the underrated feature — a *reason* field on every change so the "why did we swap the model" question has an answer six weeks later.
- **A homegrown flag table** in your primary datastore. A row with `feature_key`, `variants`, `ratios`, `updated_by`, `updated_at`. Sub-optimal ergonomics; auditable and simple. Reasonable if you already have durable storage and do not want a new SaaS dependency.

<!-- needs-research: verify current feature-flag vendor list and their canonical docs URLs as of 2026-08 — this space consolidates and re-brands frequently. -->

The wrong answer is: "we will restart the whole service to change the model." That works exactly once, at 4 pm on a Tuesday; it does not work at 3 am when you are the one paged.

## What is *worth* flagging in this feature

Not everything. Every flag is state you have to reason about later; over-flagging is its own footgun.

- **Model id.** Yes. The whole point of this chapter.
- **Provider.** Yes, one level up — sometimes you want to swap not just Claude-Opus for Claude-Sonnet but Claude entirely for GPT. This is more work than a model swap (different response shapes; see mod-001 chapter 1) but the flag is the same shape. In practice most swaps stay within one provider unless you are running redundancy.
- **System prompt version.** Yes. Prompt regressions look like model regressions, and swapping the prompt to a previous version is the same rollback shape as swapping the model. This is where the `prompt_version` field from chapter 3 earns its keep.
- **`max_tokens` and other sampling knobs.** Maybe. Flagging these adds surface area; you usually want changes here to go through a code review. Keep them in config, not behind a rollout flag.
- **`canary_ratio`.** Yes — this *is* the rollout knob.
- **Fallback behaviour.** Yes. "If the primary model 5xx-es, try the fallback model" is a flag you might want to disable if the fallback starts misbehaving.

What is not worth flagging: the feature's *existence*. If you need to turn the whole endpoint off, that is a load-balancer / API-gateway concern, not a flag. Flags are for behaviour changes; kill switches for the endpoint itself live at a different layer.

## The rollout shape

A safe model-swap rollout for a first feature is boring:

1. **Prep.** Run the new model against your mod-006 golden set. If the eval score does not drop by more than the tolerance you agreed on with the team, the model is a candidate. If it does, go back and either tune or reject.
2. **Canary at 1–5%.** Flip `canary_ratio` to 0.01 or 0.05 in the flag layer. Watch the trace fields from chapter 3 for the canary bucket only: p50/p95 latency, output-token drift, `stop_reason` distribution, error rate, cost per call, user-side signals (thumbs, follow-up rate) if you have them.
3. **Ramp.** 5% → 25% → 50% → 100% over a few days, watching the same signals at each step. Do not ramp during on-call handoffs; do not ramp on a Friday afternoon.
4. **Bake at 100%.** Hold for at least one full traffic cycle (a week is safe) before deleting the old-model code path.
5. **Delete the flag.** Once the new model is 100% and has been for a week, remove the flag from the codebase. **Stale flags are a bug.** They add branches to reason about and rot into "we cannot remember which value is correct."

Rollback is the inverse: `canary_ratio` back to 0, `default_model` back to the previous value, done. One config change; no code deploy.

## Signals that mean "roll back"

Numbers you set thresholds on *before* the rollout, and act on when they trip:

- **`stop_reason` drift.** A jump in `max_tokens` stop reasons for the new model means your outputs are being truncated more often. Usually the new model is more verbose; either raise the cap or roll back.
- **Latency jump.** p95 latency on the canary bucket doubling versus the default is worth investigation. If it is a new model that is genuinely slower, that is a product decision — cheaper models are sometimes slower.
- **Cost per call jump.** If the new model was supposed to be cheaper and the trace's `cost_usd_estimate` for the canary bucket is *higher*, the new model is producing more output tokens for the same input. Check `output_tokens` distribution; roll back or tune before ramping.
- **Golden-set drop.** The eval from mod-006, run continuously against the canary configuration, is the highest-signal number. A drop past your tolerance is a rollback with no further debate.
- **User-signal shift.** Thumbs-down rate, follow-up-question rate, escalation-to-human rate. These lag by hours-to-days depending on traffic; do not wait for them at the 5% step, but do wait for them before going to 100%.

The trace fields from chapter 3 make every one of these queries a small dashboard. If your log store cannot answer "p95 latency on `/summarise` where `model='claude-sonnet-4-6'` and `ts` last hour," fix the log-store side before the rollout, not during it.

## A one-line dependency: your flag layer must fail open

If the flag service is down and your endpoint cannot reach it, what happens?

**The endpoint keeps serving traffic on the last-known-good configuration** (or a compiled-in default). Never the other way around. An LLM endpoint that 500s because the flag service is unavailable has coupled two unrelated dependencies for no reason.

The pattern in practice:

```python
# routing.py
_last_good_config: dict | None = None

def get_config() -> dict:
    global _last_good_config
    try:
        _last_good_config = flag_client.fetch("summarise")
    except Exception as exc:
        if _last_good_config is None:
            # First fetch failed at boot — fall through to compiled-in default.
            _last_good_config = {"default_model": "claude-opus-4-7", "canary_ratio": 0.0}
        # Log the fetch failure so an alert fires, but keep serving.
        log.warning("flag.fetch.failed", extra={"error_class": type(exc).__name__})
    return _last_good_config
```

Every hosted flag SDK offers this pattern (LaunchDarkly, Statsig, ConfigCat) with cached values and configurable timeouts. Turn the cache on. Set an aggressive timeout — a flag fetch that takes 300 ms is unacceptable in the request path.

## The prompt-flag corollary

A prompt change is a model change from your endpoint's perspective: same inputs, potentially different outputs. Treat it the same way.

- Keep the prompt text version-controlled in the repo, indexed by `prompt_version`.
- The flag selects the *version*, not the prompt text.
- Roll it out the same way — canary → ramp → bake → delete.
- The `prompt_hash` field in the trace (chapter 3) is what tells you the change actually took effect.

This is what mod-006 chapter 4 assumed when it talked about "route the regression back to the prompt version that caused it."

## Common mistakes

- **Rolling a model change to 100% and *then* looking at metrics.** Any regression is now maximally painful. Ramp is boring for a reason.
- **Sampling the canary bucket without stable-per-user routing.** A single conversation flapping between models produces contradictory replies to the same follow-up questions. Users notice.
- **Leaving stale flags in the code.** Every flag has an owner and a lifecycle. If a flag has been at 100% for a month, delete it. Track this — some flag services have a stale-flag report; if yours does not, write a quarterly cleanup PR.
- **Flagging things that are not risky.** Every flag is a branch in your code. If you would never actually change the value, do not put it behind a flag; put it in config.
- **Not making the flag change auditable.** "Who set canary_ratio to 0.5 last night?" is a question you will need to answer. Any hosted service records this; homegrown flag tables should record `updated_by` and `reason`.
- **A flag layer that fails closed.** Your endpoint should keep serving on the last-known-good config when the flag service is unreachable, not 500. Every flag SDK ships this behaviour; you have to opt in.

## Summary

- The model id is data, not code. Change it without a deploy.
- Flags that earn their keep: model id, provider, prompt version, canary ratio, fallback toggle. Keep the list short.
- Rollout shape: eval → canary 1–5% → ramp → bake → delete the flag. Rollback is the inverse, in one config change.
- Watch traces from chapter 3, bucketed by the canary flag: `stop_reason`, latency, output tokens, cost, error rate, and the mod-006 eval score.
- Route users to buckets with a stable hash so a single user does not flap between models mid-conversation.
- Flag layer must fail open. An unreachable flag service does not take the endpoint down.
- Delete stale flags. They rot into bugs.

The next chapter is about what you do when — despite the eval, the trace, and the safe rollout — something breaks anyway.
