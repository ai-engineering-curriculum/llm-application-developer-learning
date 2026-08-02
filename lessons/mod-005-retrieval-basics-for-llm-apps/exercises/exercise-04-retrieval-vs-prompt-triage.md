# Exercise 04 — Retrieval vs. prompt triage

Paired with [chapter 4 — retrieval vs. prompt: the triage question](../04-retrieval-vs-prompt-triage.md).

**Estimated effort:** 90–120 minutes.

## Objective

Build the diagnostic muscle chapter 4 was written to teach: on a wrong answer, decide *quickly* whether retrieval was the bottleneck or the prompt was. Do it by running a small labelled evaluation on your own corpus, breaking the pipeline in two known ways, and confirming that the triage tells you which side of the split each failure is on.

## Problem statement

Starting from the `ask.py` from exercise 03:

1. Build a small **labelled question set** — 10 to 20 questions against your corpus. For each, record: the question, the *expected* answer in one sentence, and the `source_uri#chunk_index` of the *correct passage(s)* that answer it. Store as JSONL or a small YAML file.
2. Add a **`--eval`** mode that runs every question through the full pipeline and, for each, computes:
   - **Retrieval hit rate** — did the correct `source_uri#chunk_index` appear in the top-*k* the model saw?
   - **Answer correctness (rule-based first pass)** — a coarse check. Options: exact-match on a short expected string, keyword presence, or a fixed regex. Use the coarsest check that separates "clearly right" from "clearly wrong" for your corpus. You are *not* building a full eval (that is mod-006) — you are building a signal to triage against.
3. Print a small summary table — total questions, retrieval hit rate, answer correctness rate, and the gap between the two.
4. **Deliberately break the pipeline in two ways**, one at a time, and rerun the eval:
   - **Break A — retrieval-only.** Force the retriever to return `k` clearly-wrong passages (e.g. `ORDER BY random()` instead of the cosine operator, or `WHERE source_uri = 'known-irrelevant-file'`). Rerun. Answer correctness should collapse; retrieval hit rate should collapse first.
   - **Break B — prompt-only.** Restore correct retrieval, but replace the "answer only from the sources" instruction with "answer to the best of your ability, using the sources if they help." Rerun. Retrieval hit rate should be unchanged; answer correctness should degrade if any of your questions have plausible-but-wrong answers in the model's training data.
5. In a short write-up (in the script's output or a `TRIAGE.md`), describe what each break did to each number and what that tells you about how to read the two-number dashboard in a real incident.

## Requirements

### The labelled set

- 10 to 20 questions covering: some easy factoid lookups, some paraphrase-of-the-source questions, some that need synthesis across two passages, and at least one that your corpus does *not* answer. Aim for a spread — a set of ten trivial factoid lookups will not teach you anything.
- Every "answerable" question must have the source `source_uri#chunk_index` recorded. If you cannot point at the exact chunk, the question is not in your labelled set — remove it or improve the corpus.
- For unanswerable questions, the correct behaviour is either short-circuit-on-threshold or the model responding `I don't know.` The `--eval` mode should score both as correct.

### The eval loop

- One request per question. For each question, capture:
  - The top-*k* retrieved passages with their `source_uri#chunk_index` and distances.
  - The full prompt sent to the model.
  - The full response.
  - Whether the expected passage was in the top-*k*.
  - Whether the answer passed your coarse correctness check.
  - Whether every citation validated (from exercise 03).
- Emit a JSONL log per run. This makes side-by-side comparison across the "before break" and "after break" runs a `diff`, not a memory game.

### The two breaks

Encapsulate each break as a **single-flag toggle** on your script — `--break=retrieval-random`, `--break=prompt-loose`, `--break=none` (default). Do not fork the script; keep the pipeline identical apart from the intentional break so the comparison is clean.

### The write-up

Answer these questions in a `TRIAGE.md` (or in a printed report at the end of the script). Two to three sentences each is enough:

1. **Under Break A (retrieval-random),** what happened to the retrieval hit rate? What happened to answer correctness? Which one moved first?
2. **Under Break B (prompt-loose),** what happened to retrieval hit rate? What happened to answer correctness? Did any answer stay "correct" for the wrong reason (the model happened to know)?
3. If a user files a bug tomorrow, describe in one paragraph how you would use the retrieval hit rate and answer correctness numbers to decide whether to look at retrieval or the prompt first.

## Acceptance criteria

- `--eval` mode runs the full labelled set, prints the summary table, and writes a JSONL log without manual intervention.
- On the **baseline** run (no break): retrieval hit rate is meaningfully above zero (ideally >70% on a corpus you built the questions from). Answer correctness is at or below retrieval hit rate.
- On **Break A**: retrieval hit rate drops sharply; answer correctness drops with it.
- On **Break B**: retrieval hit rate is unchanged from baseline; answer correctness drops (perhaps modestly, depending on your questions).
- Your `TRIAGE.md` (or printed report) explicitly names, in one paragraph each, the failure mode under Break A vs. Break B and the fix category each would need.
- You have logged enough per-request detail that a colleague could, without your help, look at one wrong answer and answer chapter 4's core question: *was the correct passage in the top-*k* the model saw?*

## Starter guidance

- Keep the coarse correctness check very coarse. `expected_keyword.lower() in answer.lower()` is a reasonable start; you can tighten later. Mod-006 will teach you to build a real eval — this exercise deliberately stops at "coarse but honest."
- If you find yourself needing an LLM-as-judge to score answers, you have overshot this module — a plain-old string check on a well-chosen expected snippet is the mod-005 altitude.
- Do the labelling **before** you run the pipeline, not after. Labelling after seeing the model's answers is a well-documented way to bias yourself into scoring the model correct.
- For Break A, `ORDER BY random() LIMIT k` on your `documents` table is the shortest reliable saboteur. For Break B, keep the whole prompt identical apart from swapping in the "aspirational" instruction — this is the mod-001 chapter 5 "change one thing at a time" rule applied to a pipeline.

## Stretch goals

- **Add a "correct passage rank" column.** For every question where the correct passage is in the top-20 (not just top-*k*), record the exact rank. Bucket the counts: rank 1, 2–5, 6–10, 11–20, >20. You now have a "recall curve" you can look at when tuning `k`.
- **Add a Break C — filter-drop.** Add a `--tenant-filter=none-existent` flag that forces `WHERE tenant_id = 'no-such-tenant'`. Confirm that this looks *exactly* like Break A on the two-number summary but has a totally different fix (fix the filter, not the retrieval).
- **Add a Break D — chunking regression.** Re-ingest your corpus with a comically small chunk size (100 tokens) or a comically large one (5000 tokens) and rerun eval. Compare the numbers to baseline. This is your first evidence for why chunking strategy is a full topic in the RAG track.
- **Track cost per eval run.** Sum the `usage` blocks across every model call. When you tell someone "the retrieval eval takes N dollars," that number should come from your logs, not a guess.
