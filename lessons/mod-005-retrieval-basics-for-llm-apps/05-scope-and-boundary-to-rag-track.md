# Chapter 5 — The boundary to `rag-engineer-learning`

This is a deliberately short chapter about a deliberately short module. Its whole job is to tell you what mod-005 does not cover and where to go when you need what it does not cover. If you skip it, the risk is not that you fail an exercise — it is that you spend three months building a retrieval pipeline this module is not preparing you for, learn the RAG track's material accidentally and slowly, and ship something the RAG track would have taught you to ship well.

## Motivation

Retrieval-augmented generation is a large, deep, fast-moving practice. It has its own reading list, its own conferences, its own vocabulary, and — critically — its own dedicated learning track in this curriculum: [`rag-engineer-learning`](https://github.com/ai-engineering-curriculum/rag-engineer-learning) (level 30). Trying to compress that whole practice into one module of a broader "LLM Application Developer" track would produce a bad copy of both. mod-005 is deliberately narrow: get you to the point where you can ship a first retrieval-grounded feature that a colleague can point at their own document set and see run, and hand off any of the real depth to the specialist track.

Two failure modes this chapter is written to prevent:

1. **Building a serious RAG system out of just the tools in this module.** Everything in mod-005 works for a corpus of a few hundred to a few tens of thousands of chunks answering a well-scoped question with a well-behaved user base. Push it into a real production corpus with adversarial users, dense domain jargon, and quality SLOs — without the RAG track's material — and you will end up learning the missing pieces the expensive way.
2. **Reproducing the RAG track's material inside this module.** Chunking strategies, embedding-model benchmarking, ANN internals across Pinecone / Weaviate / Qdrant / Chroma / Milvus, re-ranking, retrieval quality evaluation, hybrid search, query rewriting — every one of those has a full chapter in the RAG track. If you find yourself writing chunking heuristics into your Q&A feature, the sign is on the wall: promote the work to the RAG track, or bring in someone from that track.

## What is in scope for this module

The material is exactly what the five chapters have covered:

- **Generate embeddings** with a hosted API (OpenAI, Voyage, Cohere). Batch and cache.
- **Store vectors** in a minimal store (`pgvector` on Postgres), alongside enough metadata to filter and cite.
- **Retrieve top-*k*** by cosine similarity with a single SQL query.
- **Assemble a grounded prompt** with delimited sources, structured citations, and a per-request budget on how much retrieved content you paste.
- **Triage the "wrong answer" case** by first asking whether the correct passage was in the top-*k* the model saw.
- **Ship a small end-to-end feature** — a Q&A endpoint against your own docs corpus that a colleague could point at their own document set.

Anything outside that list, but that is retrieval-adjacent — see the next section.

## What is *out* of scope and lives in `rag-engineer-learning`

Not exhaustive, but the biggest categories:

- **Chunking strategy.** How to split a document before embedding it. Fixed-size vs. sentence-aware vs. semantic vs. hierarchical, overlap vs. stride, section-aware splitting for structured documents (Markdown, HTML, PDFs with headings). This is the single biggest lever on retrieval quality and is a full topic on its own.
- **Embedding-model selection and evaluation.** Benchmarks (MTEB and its per-domain slices), cost / dimension / quality trade-offs, per-language and per-domain model choice, re-embed cost management when you switch.
- **Vector-DB comparison and internals.** Pinecone vs. Weaviate vs. Qdrant vs. Chroma vs. Milvus vs. staying on `pgvector`. Sharding, replication, hosted vs. self-hosted, HNSW / IVF / PQ / scalar-quantisation internals, memory footprints, warm-start behaviour.
- **Hybrid search.** Combining vector similarity with a keyword / BM25 index. Reciprocal-rank fusion. When to prefer keyword search (exact IDs, entity names, rare tokens) and how to detect when to route to it.
- **Re-ranking.** Cross-encoder or LLM re-rankers applied to an over-fetched candidate pool from the ANN index. Cost / latency trade-offs vs. just fetching more.
- **Query understanding and rewriting.** Multi-query expansion, HyDE (hypothetical-document embedding), query decomposition, question type classification (factoid vs. summarisation vs. instruction).
- **Retrieval-quality evaluation methodology.** Building a proper test set. Precision@k, recall@k, MRR, nDCG. LLM-as-judge for RAG-specific rubrics — separating "retrieval was good" from "answer was good." Continuous evaluation in CI.
- **Chunking-aware ingest pipelines.** Deduplication, incremental re-index on document update, PII scrubbing during ingest, per-document access control, multi-tenant isolation at scale.
- **Advanced grounding patterns.** Multi-hop retrieval, tool-augmented retrieval, retrieval over structured data (tables, SQL databases, knowledge graphs), citation calibration and refusal calibration.

Every one of the above is a *good* problem. Every one of the above is a problem the RAG track will teach you to think about carefully. Every one of the above is a bad problem to invent from scratch inside a mod-005-shaped feature.

## When to graduate

Rough signals — none are strict, all together they are unambiguous:

- **Your corpus is real.** Not the docs directory you cloned for the lab; the actual thousands or millions of chunks your product needs to answer over.
- **You have measurable retrieval-quality failures** that you cannot fix with the techniques in chapter 4 alone. The recall-at-5 chart from chapter 4 is your best evidence.
- **The users are adversarial or the domain is jargon-heavy.** People will find the corners; the corners will be full of the RAG track's material.
- **You are considering swapping embedding models "to fix retrieval."** That is a RAG-track decision, not a mod-005 decision. Do it under the RAG track's methodology.
- **A colleague is spending most of their week on the retrieval pipeline.** Congratulations — that colleague is now a RAG engineer. Give them the RAG track and hire (or grow) accordingly.

## What mod-005 does prepare you for

Even after graduating to the RAG track, everything you built here still holds:

- The **grounded prompt + citations** pattern from chapter 3 is exactly the same at scale — the RAG track improves the *content* of the sources block, not its shape.
- The **retrieval-vs-prompt triage** from chapter 4 is the same question at every corpus size — the RAG track just turns "read the top-*k* by hand" into a real dashboard.
- The **`pgvector` + SQL filter + top-*k*** shape is the shape you will see under every specialist vector DB. Weaviate calls it different things, Pinecone calls it different things, the API changes — the pipeline does not.
- The **mod-004 disciplines** — prompt caching on the stable prefix, cost per call, small-vs-frontier decisions — become *more* important the moment your prompts grow to include retrieved chunks. Nothing you learned there gets undone.

If you have done this module honestly, the RAG track's job is to add depth — not to un-teach anything.

## Summary

- mod-005 is the *intro* to retrieval-augmented apps. It teaches enough to ship a first grounded Q&A feature end to end.
- Everything past that — chunking, embedding-model selection, vector-DB internals, re-ranking, retrieval-quality evaluation, hybrid search, query rewriting — belongs to `rag-engineer-learning`. Do not try to build that depth from scratch here.
- When the corpus gets real, when quality failures resist the chapter-4 fixes, or when a colleague is spending most of their time on the retrieval pipeline, that is your signal to graduate.
- The patterns you learned here — grounded prompt shape, structured citations, retrieval-vs-prompt triage — carry forward. The RAG track adds depth to them; it does not replace them.

You have finished mod-005. The next module (`mod-006-minimal-eval-and-regression-checks`) is what turns "the answer was wrong once" into a measurable, gated signal you can catch before it reaches a user.
