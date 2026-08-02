# mod-005 — Retrieval Basics for LLM Applications

Your fifth module on the LLM Application Developer track. So far you have shaped prompts (mod-001), driven tool calls in a loop (mod-002), streamed and orchestrated concurrent requests (mod-003), and defended a model choice with numbers (mod-004). This module is where the model stops answering only from its training data: you teach it to answer from *your* documents by fetching the right passages at request time, pasting them into the prompt with citations, and diagnosing what went wrong when the answer is still wrong.

By the end of this module you can generate embeddings with a hosted API, store them in Postgres with `pgvector`, retrieve top-*k* passages by cosine similarity, assemble a grounded prompt with structured citations that validate against their source text, and — when the answer is wrong — triage whether the bottleneck is retrieval or the prompt.

**Estimated effort:** ~10 hours (chapters ~2 hours; four exercises 6–8 hours; the rest is time reading provider docs and eyeballing your own retrieval logs).

## Prerequisites

- You have finished [`mod-001-prompt-engineering-foundations`](../mod-001-prompt-engineering-foundations/README.md) and [`mod-004-model-selection-cost-and-prompt-caching`](../mod-004-model-selection-cost-and-prompt-caching/README.md). Mod-001 chapter 4 (schema-constrained JSON output) is load-bearing for exercise 03's structured citations; mod-004 chapter 3 (prompt caching) applies directly to the stable prefix of a grounded prompt.
- Comfort in Python at the level of connecting to Postgres, writing small SQL, and reading provider SDK response objects.
- A working install of the OpenAI Python SDK **or** Anthropic Python SDK for generation. For embeddings you need at least one of: the OpenAI SDK, the [Voyage AI SDK](https://docs.voyageai.com/), or the [Cohere SDK](https://docs.cohere.com/).
- A running Postgres you can `CREATE EXTENSION vector` on. Local Docker (`pgvector/pgvector:pg16` image), a hosted managed Postgres you have access to, or Supabase / Neon on their free tiers all work.
- Roughly USD $1–3 of API credit. Embeddings are cheap per call; you will make many of them across the exercises. Chapter 1 has current provider pricing pages.

If any of those are new to you, work through the provider quickstart or the `pgvector` README before starting the chapters.

## Learning objectives

After finishing this module you will be able to:

1. Generate embeddings with a hosted embeddings API (OpenAI, Voyage, Cohere) and store them in Postgres (`pgvector`) or a comparable minimal store.
2. Retrieve top-*k* passages by cosine similarity and assemble a grounded prompt that cites the retrieved sources.
3. Recognise when retrieval quality is the bottleneck vs. when the prompt is — the first triage question when a retrieval-grounded answer is wrong.
4. Ship a minimal retrieval-grounded Q&A feature that a colleague can point at their own document set and see run.
5. Draw the boundary to `rag-engineer-learning` (level 30): chunking strategies, embedding-model benchmarking, vector-DB internals (Pinecone / Weaviate / Qdrant / Chroma / Milvus), retrieval-quality evaluation methodology, hybrid search, and re-ranking all belong there. Do not try to reproduce that depth here; graduate to the RAG track when the corpus is real.

## Chapters

Read in order. Each chapter maps to one learning objective.

| # | Chapter | Objective |
|---|---|---|
| 1 | [Embeddings and similarity](01-embeddings-and-similarity.md) | Objective 1 |
| 2 | [Storing and searching vectors with `pgvector`](02-pgvector-top-k-retrieval.md) | Objective 1 & 2 |
| 3 | [The grounded prompt with citations](03-grounded-prompt-with-citations.md) | Objective 2 & 4 |
| 4 | [Retrieval vs. prompt: the triage question](04-retrieval-vs-prompt-triage.md) | Objective 3 |
| 5 | [The boundary to `rag-engineer-learning`](05-scope-and-boundary-to-rag-track.md) | Objective 5 |

## Exercises

Hands-on drills paced to the chapters. Do the matching exercise **after** its chapter and **before** starting the next.

| # | Exercise | Chapter |
|---|---|---|
| 01 | [First embeddings and similarity](exercises/exercise-01-first-embeddings-and-similarity.md) | 1 |
| 02 | [`pgvector` top-*k* retrieval](exercises/exercise-02-pgvector-top-k-retrieval.md) | 2 |
| 03 | [Grounded prompt with citations](exercises/exercise-03-grounded-prompt-with-citations.md) | 3 |
| 04 | [Retrieval vs. prompt triage](exercises/exercise-04-retrieval-vs-prompt-triage.md) | 4 |

## Labs and quizzes

`labs/` and `quizzes/` are reserved for long-form hands-on work and knowledge checks authored in later cycles. If they are still empty when you get here, the exercises above are enough to cement the objectives.

## Resources

Real, citable references for the topics in this module — provider embeddings docs, the `pgvector` extension, Voyage and Cohere references, and the vector-DB and RAG-track pointers you graduate to. See [`resources.md`](resources.md).

## How to work through this module

1. Read chapter 1, then do exercise 01. The cosine-similarity matrix on strings you picked is where "embeddings represent meaning" stops being an abstraction.
2. Read chapter 2 and do exercise 02. Standing up your own `pgvector` store on your own corpus is the load-bearing skill — every other exercise assumes it.
3. Read chapter 3 and do exercise 03. This is where you have a working retrieval-grounded Q&A feature end to end. Get here and you have hit the module's core objective.
4. Read chapter 4 and do exercise 04. Deliberately breaking retrieval and then the prompt, on a labelled question set, is what builds the diagnostic muscle you actually need on the day something goes wrong in production.
5. Read chapter 5. It is short. It tells you what mod-005 is *not* trying to teach and where to go when you outgrow it — save yourself three months of accidentally reinventing the RAG track.

## What comes next

`mod-006-minimal-eval-and-regression-checks` is next. Exercise 04 is a preview of what an eval looks like — a labelled input/output pair, a coarse correctness check, a summary table. Mod-006 turns that muscle into a CI-gated regression check so a prompt or model change cannot silently regress your feature.

The peer track [`rag-engineer-learning`](https://github.com/ai-engineering-curriculum/rag-engineer-learning) picks up every topic this module deliberately does not go deep on — chunking, embedding-model benchmarking, vector-DB internals, re-ranking, hybrid search, retrieval-quality evaluation. When the corpus gets real, that is where you go.
