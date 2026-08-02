# Exercise 04 — Feature-flag a model swap and roll it out safely

Paired with [chapter 4 — feature-flagging model and provider swaps](../04-feature-flags-for-model-and-provider-swaps.md).

**Estimated effort:** 2 hours.

## Objective

Turn the hard-coded `MODEL` constant into a flag you can change without a code deploy, add stable-per-user canary routing, and run through one end-to-end rollout — canary 5% → 25% → 100% → cleanup — using the traces from exercise 03 to make the ramp decisions. By the end you should be able to *demonstrate* rolling back to the previous model in under two minutes on a running server.

## Problem statement

Extend `first-feature/` from exercise 03:

1. **Introduce a flag layer.** Any of these is acceptable for the exercise:
   - The simplest: a `flags.yaml` (or `flags.json`) file the endpoint watches for changes with `watchdog` (Python) / `chokidar` (Node), and reloads without a restart.
   - A hosted feature-flag SDK (LaunchDarkly, Statsig, ConfigCat, Flagsmith, Unleash, GrowthBook, PostHog). Free tiers are fine.
   - A tiny `flags` module that reads env vars and re-reads them on a background task tick every 5 seconds.
   Pick one; the shape below is the same for any of them.
2. **Model the flag payload** as `{ "default_model": "…", "canary_model": "…" | null, "canary_ratio": 0.0-1.0, "fallback_model": "…" | null }`. `canary_model` and `canary_ratio` together drive the ramp. `fallback_model` is the "if the primary 5xx-es, try this" model — you do not need to implement the fallback path for this exercise, but the field should be in the payload so the flag layer already supports it.
3. **Implement `model_for_request(user_id, flags)` with stable hashing** — the shape in chapter 4. A given `user_id` (or the request-attribute you pick when there is no auth — IP + `User-Agent` is fine) always maps to the same bucket for a given `canary_ratio`.
4. **Add a `bucket` field to the trace record** ("default" or "canary") so exercise 03's dashboard queries can filter by it. The `model` field already exists; the `bucket` field is what lets you *compare* the two.
5. **Make the flag layer fail open.** If the flag source is unreachable, the endpoint keeps serving on the last-known-good configuration (or a compiled-in default). Test this by pointing your flag file at a nonexistent path, restarting, and confirming the endpoint still serves — with a `flag.fetch.failed` warning in the log.
6. **Run the rollout end-to-end** against a synthetic caller (a script that fires 500 requests with 100 distinct fake user ids). At each step, use the trace records to compute the requested metric bucket-by-bucket:
   - `canary_ratio=0.05` → confirm ~25 of 500 requests land in the canary bucket, ± noise.
   - Ramp to 0.25 → confirm the ratio moves.
   - Ramp to 1.0 → confirm all requests use the new model.
   - Roll back to `canary_ratio=0.0`, then remove the canary from the flag payload — the flag is "cleaned up."
7. **Rehearse a rollback under time pressure.** Set a stopwatch. Ramp the canary to 50%, then reproduce this two-step: "the eval failed on trunk five minutes ago." Move `canary_ratio` to 0. Confirm the trace's next 30 records all show `bucket: default` and the previous model. Time the whole thing. It should complete in under two minutes end to end. If it does not, the flag layer's re-read latency is the bottleneck — tune it.

## Requirements

- **The model is not a constant.** `config.py`'s `MODEL` value is gone (or now points at the flag layer). Grep the codebase; the only string literal of a model id lives in the flag file and in a compiled-in default for fail-open.
- **Stable-per-user routing.** Fire the same synthetic user id 20 times with `canary_ratio=0.5` — 20 requests land in the same bucket, not roughly 10/10.
- **The trace's `bucket` field is present on every request.** Exercise 03's trace shape gains one field; every downstream query still parses.
- **Fail-open behaviour is tested.** With the flag source unreachable, the endpoint continues to serve; a `flag.fetch.failed` log line appears at most every N seconds (not on every request); the response uses the last-known-good config or the compiled-in default.
- **Rollout ramp works from an external control surface** — you change `canary_ratio` in the flag file / dashboard / env var and the running server picks up the change within your target window (5 seconds is a good default; sub-minute is acceptable).
- **The rollout is auditable.** Whatever flag layer you picked, the change to `canary_ratio` has some kind of `who / when / why` record — a git commit on the flag file, a change log entry in the vendor UI, or a `flags.log` you write yourself. "I changed it because I wanted to" is not an audit entry.
- **The stale flag is cleaned up.** After the ramp to 100%, the canary fields are removed from the flag payload and the code paths that reference them are simplified. Do not leave `canary_ratio: 0.0` sitting in the file forever — that is exactly the stale-flag anti-pattern chapter 4 warned about.

## Starter guidance

- Pick the simplest flag layer that meets the requirements. A YAML file + `watchdog` is 40 lines of code and demonstrates everything. Go hosted if you want the UI experience and the audit trail comes for free.
- Both providers have a cheap-tier model that produces different-enough outputs from the expensive tier for a canary demo. For Anthropic, Claude Opus vs. Claude Sonnet or Claude Haiku is the natural pair. For OpenAI, GPT-5 tier vs. GPT-5-mini (or whichever pairing is current — check <https://platform.openai.com/docs/models>).

<!-- needs-research: confirm current default / cheap-tier / frontier model IDs for Anthropic and OpenAI as of 2026-08. -->

- The synthetic caller can be a `for` loop in a script with `httpx.AsyncClient` — the mod-003 async-fanout patterns you already have. Do not launch 500 requests with no concurrency limit; a semaphore of ~20 is polite to the provider.
- The rollout playbook in chapter 4 (canary → ramp → bake → delete) is the shape you are practising. In a real environment you would spread the ramp over days, not minutes. The exercise compresses it — but the *steps* are the same.
- LaunchDarkly quickstart: <https://launchdarkly.com/docs/home/getting-started>. Statsig quickstart: <https://docs.statsig.com/client/introduction>. GrowthBook quickstart: <https://docs.growthbook.io/quick-start>.

## Acceptance criteria

- With `canary_ratio=0.05` and 500 synthetic requests from 100 distinct user ids, the trace's `bucket` field distribution is approximately 25 canary / 475 default — repeat with the same user ids and the distribution is identical (stable routing).
- Ramping `canary_ratio` from 0.05 → 0.25 → 1.0 in the flag layer is observed in the trace within your target window (≤ 5 s) without a code deploy or process restart.
- With the flag source unreachable at process start, the endpoint serves on the compiled-in default and logs a `flag.fetch.failed` warning at most once every N seconds. The request rate does not degrade.
- The two-minute rollback drill: with a stopwatch, set `canary_ratio` to 0 and confirm the next 30 trace records all show `bucket: default` and the previous model, in under two minutes. If it takes longer, tune the re-read cadence.
- Changing the `canary_model` in the flag layer is auditable — you can point at exactly *when* the change happened and *who* made it.
- The final cleanup PR removes the canary fields from the flag payload and the branching code paths in `routing.py`. No dead flag remains after ramp-to-100%.

## Stretch goals

- **Implement the fallback path.** When the primary model returns a 5xx, the handler retries once against `fallback_model` and records `used_fallback: true` in the trace. Test it by making the primary point at a bogus model id (guaranteed 400) and confirming the fallback fires. Delete the test flag when done — do not leave a "bogus model id" default lying around.
- **Add prompt-version flagging.** The `SYSTEM_PROMPT_VERSION` env var from exercise 02 becomes a flag with the same canary/ramp shape. Roll out a `summarise-v2.md` prompt using the exact same procedure. The trace's `prompt_hash` and `prompt_version` fields make the ramp visible.
- **Wire the ramp signals into a dashboard.** Grafana + Loki, Vector + a local dashboard, or your log collector's UI. Bucket by `bucket` and `model`; graph latency, `stop_reason` distribution, and `cost_usd_estimate` for each. This is what an operator would look at during a real ramp.
- **Compute a golden-set score bucketed by model.** Take the mod-006 eval you built and run it against both `default_model` and `canary_model`, produce a two-column table. This is the mod-004 "defend the model swap with numbers" habit compounding with mod-006 into the shape a PM will actually approve a rollout against.
