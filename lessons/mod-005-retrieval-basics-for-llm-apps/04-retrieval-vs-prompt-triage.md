# Chapter 4 — Retrieval vs. prompt: the triage question

Every retrieval-grounded feature you ship will eventually produce a wrong answer. When that happens, the most valuable thing you can do — before any code change, any prompt tweak, any embedding-model shopping trip — is answer one question:

> **Was the correct passage in the top-*k* the model saw?**

That question splits the entire debugging space in two. If the answer is *no*, retrieval is your bottleneck and no amount of prompt work will fix it. If the answer is *yes*, retrieval did its job and the prompt (or the model) failed to use what it had. Different fixes; different chapters; different teams often. This chapter teaches the triage.

## Motivation

The single most common failure mode on a young RAG feature is *misdiagnosis*. A team ships a grounded Q&A feature, a user files a bug ("it told me the wrong policy"), and the team spends two weeks rewriting the prompt. The prompt was fine. The correct passage was never retrieved — the user's question used different vocabulary than the source, or the chunk was split at an unlucky place, or the tenant filter dropped the right row. No amount of prompt engineering could have recovered from that. Meanwhile the *other* team ships a similar feature, sees a wrong answer, and immediately starts A/B-testing embedding models — when the passages were there in the top-3 and a five-word tightening of the system prompt would have fixed it.

You do not want to be either team. The triage below is the fastest way to know which side of the split you are on.

## The triage, in one flow

For every wrong answer worth investigating:

1. **Log the retrieval result and the model response together.** The passages the retriever returned, the distances, the source URIs, the exact prompt sent to the model, the exact reply. Chapter 3 called this the load-bearing instrumentation. If you do not have it, stop and add it before debugging further — you cannot triage on guesses.
2. **Read the top-*k* passages by hand.** Is the correct answer *anywhere* in them?
   - **No** → **retrieval is the bottleneck.** Section: *Fixing retrieval*.
   - **Yes** → **the prompt (or model) is the bottleneck.** Section: *Fixing the prompt*.
3. **Do not skip step 2.** "The model got it wrong so retrieval must be broken" is exactly the misdiagnosis this chapter is written to prevent.

Write this three-step check on the same index card as the mod-001 chapter 5 failure-shape triage. In practice you will do them together: parse-clean, factually-correct, and grounded-in-the-source are the three checks every retrieval-grounded answer has to pass.

## Fixing retrieval (when the passage was not in the top-*k*)

If the correct passage was not among the retrieved candidates, one of these things happened:

- **The passage exists in the corpus but not in the top-*k*.** The vector for the user's question was farther from the correct passage's vector than from *k* other passages. Look at where the correct passage *did* rank — check the top-20 or top-50. If it is at rank 8 when *k* was 5, over-fetching to 20 and re-ranking (a topic for the RAG track) may be the fastest win. If it is at rank 300, you have a real semantic gap, usually because the question uses different vocabulary than the source.
- **The passage does not exist in the corpus.** The user asked something the corpus does not answer, and retrieval correctly reported "nothing very close." This is not a bug in your system; it is a bug in what the user was told the system knows. Address it at the product boundary — "this assistant only knows about policies X, Y, Z" — not by tightening the prompt.
- **The passage was filtered out by SQL.** A `WHERE tenant_id = ?` clause, a `deleted_at IS NULL`, a soft-delete you forgot about. Rerun the query without the filter and see if the passage appears. This is almost always a data / permissions bug that a prompt change cannot fix.
- **The passage was split badly at chunking time.** The answer is spread across two chunks, neither of which is a great match on its own. This is the strongest signal that chunking strategy matters — but for this module, that fix graduates to the RAG track. In the short term, over-fetch, or reduce chunk overlap changes, or accept the miss and log it.

The fixes at *this* module's altitude are usually one of:

- **Over-fetch.** Ask `pgvector` for top-20, keep top-5. Cheapest change; often catches "the passage was rank 6."
- **Fix the SQL filter.** Look at your `WHERE` clause. Confirm it is not stripping the row you wanted.
- **Fix the source data.** Reindex after adding the missing document; correct wrong metadata; delete the duplicate that is winning the retrieval.
- **Fix the query.** Sometimes rephrasing the user's question — expanding "refund" into "refund OR return OR money back" — helps. Do this in code, not by asking the user to try again.

If you have tried the four above and retrieval quality is still your bottleneck, that is the signal in mod-005's closing objective: your problem is now the RAG track, not this module. Graduating is the right call; don't try to reproduce that track's depth from scratch here.

## Fixing the prompt (when the passage *was* in the top-*k*)

If the correct passage was in the retrieved set and the model still got the answer wrong, the failure lives in one of four places:

- **The model did not attend to the right passage.** The correct one was there but buried in the middle. Chapter 3's "best-match last" habit is the first defence — put your top-1 immediately before the question, not five passages up the block. Also check whether the correct passage was one of the last things in the sources block or was crowded out by near-duplicates.
- **The prompt did not lock the model to the sources.** The model answered from its own knowledge, and the retrieved passages were background noise. The instruction "answer only from the passages provided" is stronger when phrased as *behaviour* ("If the sources do not contain the answer, respond exactly 'I don't know.'") than as *aspiration* ("try to use only the sources").
- **The citation shape is loose.** The model produced a plausible-sounding answer with `[S3]` attached, but S3 does not actually contain the claim. This is where the `quote` field from chapter 3 pays for itself — validate every citation against a substring of the source it names, and reject or flag ungrounded ones.
- **The model itself is the wrong tool.** A small model may not synthesise across two passages well; a frontier model may. If your retrieval is right and your prompt is tight and answers are still wrong, run the same call on a stronger model and see if the answer moves. This is a mod-004 question — you already know how to do the A/B.

Fixes at this module's altitude, in rough order of what to try first:

1. **Re-order passages so best-match is last.**
2. **Tighten the "I don't know" fallback in the system prompt.**
3. **Move to structured citations with a `quote` field**, and reject responses where the quote is not in the named source.
4. **Try a stronger model** on the same prompt to see if the failure is model-side.

If those do not move the needle and retrieval is genuinely returning the right passages, the failure has moved out of mod-005 scope entirely — into evaluation methodology (mod-006) and re-ranking / prompt-optimisation (the RAG track).

## An instrumentation checklist

You cannot do any of the above without the right logs. From day one, log the following on every retrieval-grounded call:

- The **user question** (with PII redaction as your policy requires).
- The **query vector** — not usually necessary to keep, but occasionally useful; a hash or short summary at least.
- The **top-*k* results**, with `source_uri`, `chunk_index`, distance, and the first ~200 chars of content per row. Enough to see which passages the model was shown without loading the whole content column.
- **The exact prompt sent to the model** — system, messages, model name, sampling parameters, `max_tokens`.
- **The exact response** — full text, stop reason, `usage` block, refusal field if any.
- **Any citations extracted from the response** (if you used the structured shape) *and* whether each citation validated against a substring of its named source.

Two dashboards you will want, whether you build them yourself or wire up a hosted tool later:

- **"Correct passage in top-*k*" rate**, on a held-out set of questions with known answers. This is the ceiling on how well the model can possibly do — no prompt fix can push above it.
- **"Answer factually correct" rate**, on the same set. The gap between the two is your prompt-side headroom.

The RAG track (`rag-engineer-learning`) turns those two numbers into a full evaluation methodology and adds re-ranking, chunking experiments, and continuous monitoring. For this module, having the two numbers at all — even from a hand-audit — is a large step ahead of the median first-time RAG deployment.

## What not to do first

- **Do not swap embedding models on a single bad example.** Embedding models are broadly comparable at the "does the right passage make top-*k*?" question. Swap only if you have a measured recall gap across a real test set, not a vibe.
- **Do not raise `k` without measurement.** A larger *k* eats context budget on every request, degrades attention on the best passages, and does not fix the "the passage was at rank 300" problem. Over-fetch and *trim*; don't just send more.
- **Do not lower temperature to `0` and declare the bug fixed.** A wrong-but-grounded answer is a prompt bug; a fluent-but-ungrounded answer is a grounding bug. Both survive `temperature=0`.
- **Do not add a re-ranker without evaluating it.** A re-ranker is the RAG track's answer to "the passage is in top-20 but not top-5." It is a serious tool; it deserves an evaluation, not a copy-paste from an example.

## Summary

- The first triage question when a retrieval-grounded answer is wrong is: **was the correct passage in the top-*k* the model saw?** Everything else branches from that.
- If **no**, retrieval is the bottleneck — check rank, filters, chunk splits, and query rephrasing. If nothing simple helps, you have graduated to the RAG track.
- If **yes**, the prompt or the model is the bottleneck — reorder passages, tighten the "I don't know" rule, use structured citations with quote validation, try a stronger model.
- Log the retrieved passages, the exact prompt, the response, and the citation-validation result on every request from day one. You cannot triage without them.
- Two numbers on a dashboard — "correct passage in top-*k*" and "answer factually correct" — separate what retrieval can do from what the prompt is doing with it.

The next chapter draws the line between this module and the RAG track — what belongs here, and what to hand off when the corpus gets real.
