# Exercise 05 — Write a runbook a colleague can use during an incident

Paired with [chapter 5 — writing a runbook a colleague can use during an incident](../05-writing-a-runbook-for-one-feature.md).

**Estimated effort:** 1.5 hours.

## Objective

Produce the one-page runbook for the endpoint you have been building since exercise 01. By the end of the exercise you have `docs/runbooks/<feature>.md` that names the three most likely failure modes for *your specific feature*, gives three to five immediate steps per mode with time budgets, points at the dashboard / log queries / flag URLs that make those steps concrete, and names an escalation contact for each. You then rehearse the runbook by simulating one failure mode and driving through the steps as if you had never seen the code before.

## Problem statement

Extend `first-feature/` from exercise 04:

1. **Pick the three most likely failure modes for *your* feature.** For most first LLM features, the chapter's default trio is the right starting point: **provider outage / hard rate-limit**, **bad model rollout (or silent provider snapshot roll)**, **runaway cost / quota drain**. If your feature adds retrieval (mod-005), swap one in for "retrieval index stale/empty." If it adds tools (mod-002), swap one in for "tool loop that will not terminate." Write down *why* you picked the three you did — one sentence each. This defence is more important than the specific choice.
2. **Write the runbook in `docs/runbooks/<feature>.md`** to the template shape in chapter 5. All of: owner, backup owner, on-call rotation link, provider status pages, dashboard link, log-search query template, feature-flag URL, emergency kill switch, and one section per failure mode with symptom / immediate steps / escalation.
3. **Every step names a specific action.** "Check the flag service" is not a step; "Set `canary_ratio` to 0 at `<url>/flags/summarise`" is. Every log query is written down, not left as "search the logs." Every link is a real URL, not a placeholder.
4. **Every failure mode has an escalation contact** and a stop-condition. "Escalate to <person>" without a paging route is not an escalation. Use a rotation (PagerDuty schedule name, Opsgenie team, Grafana OnCall rotation), not an individual — people change teams.
5. **Rehearse.** Pick *one* of your three failure modes and simulate it in your local environment for real. Examples:
   - **Bad model rollout** — set `canary_model` to a broken model id in the flag file; drive through the runbook to roll it back. Time it end to end.
   - **Runaway cost** — send a burst of large-input requests with a single user id; drive through the runbook to detect (which user? which prompt hash?) and mitigate (rate limit? kill switch?).
   - **Provider outage** — kill your machine's network to the provider (`iptables` DROP or an equivalent), watch the endpoint's error rate go up, drive through the runbook to identify and mitigate.
   Take notes on anything the runbook did *not* answer well. Update the runbook with those fixes. This is chapter 5's game-day discipline compressed into an exercise.

## Requirements

- **`docs/runbooks/<feature>.md` exists** and fits on one printed page (a browser window on a laptop screen — you should not have to scroll more than a full page height).
- **Each of the three failure modes** has: symptom sentence, three to five numbered immediate steps with time budgets, a stop-condition (`If not resolved in N minutes...`), and an escalation route.
- **Every URL in the runbook resolves.** You can click the flag-service link, the dashboard link, the log-search link, the provider status page. If a link is aspirational ("your team's dashboard, once you have one"), replace it with the fallback shape ("in the meantime: `jq '. | select(.route==\"/summarise\")'` on `server.log`").
- **The runbook is linked from the trace source.** Concretely: exercise 03's trace shape (or a small `README.md` next to it) links to the runbook. The path from "I got paged" → "I have the runbook open" is one click.
- **A `docs/runbooks/README.md`** lists all runbooks in the repo (currently one). This is where a colleague looks when they do not know the feature name.
- **The rehearsal produces a numbered write-up** of what you found — either as a section in the runbook itself ("Last rehearsal: 2026-08-02, notes below") or as a linked postmortem-shaped doc. At least one runbook update comes out of the rehearsal. If nothing needed updating, either the runbook was already perfect (unlikely) or the rehearsal was too soft (much more likely).
- **The `X-Request-Id` from the trace** is called out in the runbook as the key by which the log store / archive / support ticket are joined. This is the field that turns "the user's complaint" into "the row in the log."

## Starter guidance

- Read chapter 5's template first. It is deliberately mundane. Copy the structure — do not reinvent it.
- Time budgets on steps come from experience — pick numbers you think you could hit fresh; the rehearsal will correct you.
- Escalation contacts should be **rotations, not people**. Solo project? Point at your own PagerDuty schedule or, for a hobby project, at yourself with an explicit "there is no rotation; if this happens you should build one" note. The point is that the field is real, not that it names three names.
- If you do not have a hosted feature-flag service and your flag is a YAML file in the repo, the "emergency rollback" step is `git commit -m "roll back canary" && deploy`. That is still a specific action; write it down that specifically.
- For the runaway-cost failure mode: your trace's `cost_usd_estimate` field is what you group by `user_id_hash` and `prompt_hash` to find the culprit. If you did not add these fields in exercise 03, go back and add them before you finish the runbook — the runbook cites them.
- Google SRE Workbook on runbooks (short and free): <https://sre.google/workbook/table-of-contents/> — chapter on "Prevention of Outages" and the "Practical Alerting from Time Series Data" chapter are the most relevant.
- PagerDuty runbook automation reference (for context on the shape hosted providers standardise): <https://support.pagerduty.com/docs/runbook-automation>.

## Acceptance criteria

- `docs/runbooks/<feature>.md` is one page, contains the top-of-page fields from chapter 5's template, and has three failure-mode sections each with symptom / steps / escalation.
- A colleague (or you-tomorrow, self-blinded) can start from the runbook and land on the right immediate action for each of the three failure modes in under 30 seconds — no reading of the feature's code required.
- The rehearsal actually ran — you have a timestamped note of the simulation, the timings you measured, and at least one runbook update that came from it.
- Every URL in the runbook resolves to a real page or a real search query.
- Every escalation is a **rotation** (or a documented "there is no rotation, and here is what to do instead").
- The runbook is discoverable from the trace source (README next to `main.py`, or a docstring in the log-formatter module, or your dashboard).
- If you added retrieval or tools to your feature, one of the three modes reflects that — the runbook is not the chapter's generic default trio but the trio for *your* feature.

## Stretch goals

- **Automate one step from the runbook.** The "emergency kill switch" is a good candidate — a small script `scripts/kill_switch.sh` that flips the flag and reads back the change. Or a PagerDuty runbook automation action. This turns a manual step into a one-command action.
- **Add a `docs/runbooks/GAME_DAY.md`** template that describes how to run a game day for this feature. Prompts and mitigations for the three failure modes, a scoring rubric ("did we hit the time budget?"), a spot for the next-game-day date. Quarterly cadence is the sweet spot.
- **Write a "did-not-repro" section.** For each failure mode, name what the *symptom* would have looked like if it had happened at 0.01% frequency instead of the visible burst — and what monitor would have caught it. This is chapter 5's "turn every did-not-repro into an alert" pattern for the runbook layer.
- **Cross-reference from the chapter-6 smoke tests.** A fourth failure mode you *did not* put in the runbook is "user is running a prompt-injection attack." Add a short section — even one that just points at the exercise-06 smoke tests and says "if these start firing in prod, the mitigation is..." — so the runbook has some coverage for the security-adjacent failure the top three miss.
