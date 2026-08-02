# Exercise 02 — `pgvector` top-*k* retrieval

Paired with [chapter 2 — storing and searching vectors with `pgvector`](../02-pgvector-top-k-retrieval.md).

**Estimated effort:** 90–150 minutes.

## Objective

Stand up a working `pgvector` store from scratch, load a small corpus of your own text, and pull the top-*k* passages for a handful of user-shaped queries. Every later exercise assumes you have this end of the pipeline running against a real database and real data.

## Problem statement

Build a small ingest-and-search tool (one script or a tiny module split across two: `ingest.py` and `search.py`) that:

1. Creates the `documents` table from chapter 2 in a Postgres you can reach.
2. Loads a **small local corpus** (see below), splits it into ~500-token chunks, embeds each chunk with the same model you used in exercise 01, and inserts the results.
3. Accepts a query string on the command line, embeds it, and prints the top-*k* passages by cosine distance.

### The corpus

Pick something you can commit to your local disk and read yourself. Two good options:

- **Recommended:** a real docs directory you have permission to use — an open-source project's `docs/` folder cloned locally, a set of public policy PDFs converted to plain text, or Wikipedia articles for a set of topics you know well. Aim for **50–500 chunks total** after splitting. Small enough to eyeball, big enough for retrieval to be non-trivial.
- **Fallback:** the Markdown files from mod-001 through mod-004 of *this* curriculum. `find lessons/mod-00{1,2,3,4} -name '*.md' -not -path '*/exercises/*'` will give you a defensible set that is small, English, and has real structure.

## Requirements

### Postgres and `pgvector`

- Use a Postgres you actually run. Local Docker (`docker run -p 5432:5432 -e POSTGRES_PASSWORD=postgres pgvector/pgvector:pg16`), a hosted managed Postgres you have access to, or Supabase / Neon on their free tiers all work.
- Enable `pgvector` (`CREATE EXTENSION IF NOT EXISTS vector;`) — if the extension is missing on your Postgres, install it before continuing. Do not skip and use another vector store; the exercise is about `pgvector` specifically.

### Schema

- Match the shape from chapter 2 — `id`, `source_uri`, `chunk_index`, `content`, `embedding VECTOR(N)`, `created_at` — with `N` matching your embedding model's dimension exactly.
- Add a unique constraint on `(source_uri, chunk_index)` so re-running ingest is idempotent (`ON CONFLICT DO NOTHING`).

### Chunking

- Use a **simple, defensible splitter.** Sentence-aware split (`nltk.sent_tokenize`, `spacy`, or a hand-rolled regex is fine) grouping into chunks of ~500 tokens with no overlap. Do not optimise chunking in this exercise; chapter 2 said chunking strategy is out of scope and lives in the RAG track.
- Count chunk length in **tokens using the provider's tokenizer** (mod-001 chapter 3), not characters.
- Preserve `source_uri` (the file path or URL) and a sequential `chunk_index` per source. Both are load-bearing for the citations in exercise 03.

### Ingest

- Embed in **batches** — one embedding call per source, or better, one call per configurable batch size (e.g. 100 chunks per request). Do not embed one chunk at a time.
- Log a one-line summary at the end: `ingested N chunks from M sources into <db>`.

### Search

- Command-line invocation like:
  ```
  python search.py --query "how do I request a refund?" --k 5
  ```
- Use the **`<=>` cosine-distance operator** in your SQL. Bind the query vector as a parameter; do not string-format it into the SQL.
- Print each hit in a stable, greppable format:
  ```
  rank  distance  source_uri#chunk_index    content_preview
  1     0.183     policies/refunds.md#12    We accept returns within 30 days...
  2     0.211     policies/refunds.md#14    Damaged items may be returned...
  ```
- No ANN index for this exercise — the corpus is small enough that a full scan is fine and gives you exact results. Adding an HNSW index is a stretch goal.

## Acceptance criteria

- `ingest.py` (or equivalent) runs end to end without manual intervention, embeds and inserts your corpus, and is idempotent — running it twice does not create duplicate rows.
- `search.py` (or equivalent) takes a `--query` and a `--k`, prints exactly `k` rows in the format above, and takes under a couple of seconds on a small corpus.
- Run three queries by hand — one whose answer is *definitely* in the corpus, one whose answer is *definitely not*, and one that is ambiguous or paraphrased. For each, look at the top-5 by hand. Note in a comment or short paragraph:
  - For the in-corpus query: at what rank did the correct passage appear?
  - For the not-in-corpus query: how close was the top-1 distance? Do you see a clear gap vs. the in-corpus queries?
  - For the ambiguous query: is the top-1 a good answer? Is the top-5 a good candidate pool?

You will use those observations directly in exercise 04.

## Starter guidance

- `pgvector` install and usage: <https://github.com/pgvector/pgvector>
- `pgvector-python` (adapters for `psycopg` and `SQLAlchemy`): <https://github.com/pgvector/pgvector-python>
- OpenAI embeddings quickstart: <https://platform.openai.com/docs/guides/embeddings>
- If your OpenAI SDK's `client.embeddings.create(...)` supports a `dimensions` parameter, do *not* use it here — use the model's native dimension so your schema and stretch-goal indexing behave normally.
- If you get a `dimension mismatch` error on insert, your `VECTOR(N)` does not match your embedding model. Fix the schema (drop the table, `CREATE` again with the right `N`), do not reshape the vector.

## Stretch goals

- Add an **HNSW index** (`CREATE INDEX ... USING hnsw (embedding vector_cosine_ops)`) after your bulk load. Time three queries with and without the index. On a small corpus you may find the unindexed version is faster — record the numbers.
- Add a **`WHERE source_uri LIKE 'policies/%'` (or equivalent) filter** and re-run one query. Confirm you get the same top-*k* structure, just scoped to a subset of the corpus.
- Add a **`--threshold` flag**: if the top-1 distance is above the threshold, print `no matches above threshold` and exit non-zero. Try values around 0.3, 0.5, 0.7 on your corpus and see which is the natural cliff between "there is an answer" and "there is not."
- **Time the ingest** — how many chunks / second? If a real corpus is 1,000,000 chunks, how long would this loader take? (This is a mod-004-style habit — the question you should always ask before a long-running job.)
