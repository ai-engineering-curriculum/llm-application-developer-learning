# Chapter 2 — Storing and searching vectors with pgvector

Chapter 1 gave you a way to turn text into vectors. This chapter is about putting those vectors somewhere you can search, and pulling out the top-*k* most similar entries for a given query. There are half a dozen serious vector databases on the market (Pinecone, Weaviate, Qdrant, Chroma, Milvus, …) and each has its own SDK, its own indexing story, and its own operational trade-offs — the depth of that comparison lives in the `rag-engineer-learning` track. In this module you build against **Postgres with the `pgvector` extension**, because it is the shortest path from "I already run a Postgres" to "I have a working vector store," and every idea in the chapter transfers to the specialised stores when you graduate to them.

Reference for everything in this chapter: <https://github.com/pgvector/pgvector>.

## Motivation

Retrieval is a search problem. Search problems have three moving parts: the storage layer (where the data lives), the query (what you ask), and the index (what makes the query fast). For a hobby corpus of a few hundred documents, all three are trivial — a Python list and a for-loop is a valid vector store. For anything bigger you want:

- **Persistence** so a restart does not lose your embeddings.
- **A single query interface** that filters on ordinary SQL columns (customer ID, doc type, tenant) *and* on vector similarity at the same time.
- **An index** that scales sub-linearly with corpus size.

`pgvector` gives you all three inside a database you almost certainly already know how to operate. It is genuinely the right tool for a first retrieval feature, and it stays a defensible choice up into the millions of vectors — long enough that most teams graduate past this chapter before they outgrow the store.

## Install and enable

`pgvector` is a Postgres extension. On a modern managed Postgres (RDS, Cloud SQL, Supabase, Neon) it is pre-installed and you enable it per database with:

```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

For self-hosted Postgres, install the extension from source or your package manager, then run the same statement. The [pgvector README](https://github.com/pgvector/pgvector#installation) has current install instructions per platform.

<!-- needs-research: confirm current pgvector version and any breaking changes vs. what's in the README as of 2026-08. -->

## The table shape

A minimal retrieval table has three parts: the source text (so you can put it into the prompt later), the embedding (so you can search), and enough metadata to filter and to cite.

```sql
CREATE TABLE documents (
    id           BIGSERIAL PRIMARY KEY,
    source_uri   TEXT NOT NULL,           -- where this passage came from (file path, URL, doc ID)
    chunk_index  INT  NOT NULL,           -- position within the source
    content      TEXT NOT NULL,           -- the passage itself
    embedding    VECTOR(1536) NOT NULL,   -- match the dimension of your embedding model
    created_at   TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

Three things about that shape:

- **The `VECTOR(N)` dimension has to match your embedding model exactly.** `text-embedding-3-small` is 1536, `text-embedding-3-large` is 3072, `voyage-3` is 1024, `embed-english-v3.0` is 1024. Get this wrong and inserts will error at write time — a good failure to have.
- **Store the source text next to the vector.** Retrieval returns rows, not just distances. When you assemble the grounded prompt (chapter 3), you need the *text* to paste in; the vector is only useful for search.
- **Keep a `source_uri` and a `chunk_index`.** Citations in chapter 3 require both — you need to tell the user *which* document a passage came from, not just "some passage."

## Inserting a batch of vectors

The Python SDKs for `psycopg` and `SQLAlchemy` both work with `pgvector` — the extension ships adapters so `list[float]` marshals correctly. Using `psycopg` directly:

```python
import psycopg
from pgvector.psycopg import register_vector

conn = psycopg.connect(DATABASE_URL)
register_vector(conn)  # teaches psycopg how to bind a list[float] to VECTOR

rows = [
    (source_uri, chunk_index, content, embedding)
    for source_uri, chunk_index, content, embedding in prepared_batch
]

with conn.cursor() as cur:
    cur.executemany(
        "INSERT INTO documents (source_uri, chunk_index, content, embedding) "
        "VALUES (%s, %s, %s, %s)",
        rows,
    )
conn.commit()
```

Reference: <https://github.com/pgvector/pgvector-python>.

Two operational habits worth taking on early:

- **Batch inserts.** A single `executemany` on 500 rows is dramatically faster than 500 single-row inserts, and the throughput ceiling of a well-tuned insert path is what determines how long the initial embed-and-load run takes.
- **Idempotent ingest.** Add a unique constraint on `(source_uri, chunk_index)` and use `ON CONFLICT DO NOTHING` (or `DO UPDATE`) so re-running the loader after a partial failure does not duplicate rows. Every real ingest job crashes at least once.

## The top-*k* query

`pgvector` exposes three distance operators on the `VECTOR` type:

| Operator | Meaning | When to use |
|---|---|---|
| `<->` | Euclidean (L2) distance | Rare for text embeddings. |
| `<=>` | Cosine distance = `1 - cosine_similarity` | The default choice for text embeddings. |
| `<#>` | Negative inner product | If your vectors are normalised and you want the dot product. |

For cosine similarity on normalised text embeddings, `<=>` is what you want. The query for the top-5 most similar passages to a query vector `q`:

```sql
SELECT id, source_uri, chunk_index, content, embedding <=> %s AS distance
FROM documents
ORDER BY embedding <=> %s
LIMIT 5;
```

Note the same `%s` parameter appears twice — once in the `SELECT` to get the distance back for scoring, once in the `ORDER BY` to actually sort. Some drivers let you use a named parameter to avoid double-binding; `psycopg` does. Bind the query vector as a Python list; `pgvector-python` handles the conversion:

```python
from pgvector.psycopg import register_vector

register_vector(conn)

query_vector = embed_one(user_question)  # returns list[float] of the right dimension

with conn.cursor() as cur:
    cur.execute(
        """
        SELECT id, source_uri, chunk_index, content, embedding <=> %s AS distance
          FROM documents
         ORDER BY embedding <=> %s
         LIMIT %s
        """,
        (query_vector, query_vector, 5),
    )
    hits = cur.fetchall()
```

Reading the results:

- `distance` is cosine *distance*: `0.0` means the vectors point in the same direction, `1.0` means they are orthogonal (unrelated), values above `1.0` mean opposite directions (very rare for real text embeddings). Convert to similarity as `1 - distance` if that reads more naturally in your logs and dashboards — the ordering is the same either way.
- Rows come back in ascending distance order. `hits[0]` is your best match.

## Combining vector search with SQL filters

The point of using a real database as your vector store is that you can filter on ordinary columns in the same query. For a per-tenant retrieval feature:

```sql
SELECT id, source_uri, chunk_index, content, embedding <=> %s AS distance
FROM documents
WHERE tenant_id = %s
  AND deleted_at IS NULL
ORDER BY embedding <=> %s
LIMIT 5;
```

Two things about the interaction between the `WHERE` clause and the vector index (next section):

- If you have an ANN index (below) and a very selective `WHERE` (e.g. matches 0.1% of the table), the planner may choose to scan the filtered rows directly and skip the index. That is often correct — an exact scan over a small filtered set beats an approximate scan over the whole table.
- If your `WHERE` filters over a lot of rows and the ANN index kicks in, some ANN indexes can return fewer than `k` rows after post-filtering (they scan a fixed-size candidate pool and then apply your filter). That is one of the reasons "hybrid pre-filter vs post-filter" is a whole design surface in specialised vector DBs — for `pgvector` at the scale this module targets, over-fetch by a factor of 2–5x and let the LIMIT trim the extras.

## Indexing: exact vs. approximate

Without an index, a top-*k* query is a full scan: Postgres computes the distance to every row and picks the smallest *k*. That is fine at 10,000 rows on a laptop; at 10,000,000 rows it will hurt. `pgvector` gives you two approximate-nearest-neighbour index types.

### IVFFlat

```sql
CREATE INDEX ON documents USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
```

Splits the vector space into `lists` centroids and scans only the closest few. Faster than exact, but recall drops if `lists` is wrong for your corpus size. Rules of thumb from the `pgvector` README: `lists = rows / 1000` for < 1M rows, `lists = sqrt(rows)` above that.

### HNSW (recommended for most cases)

```sql
CREATE INDEX ON documents USING hnsw (embedding vector_cosine_ops)
  WITH (m = 16, ef_construction = 64);
```

A hierarchical graph index. Typically better recall / speed trade-off than IVFFlat, at the cost of higher build time and memory. Query-time knob is `hnsw.ef_search` (higher = slower but more accurate).

<!-- needs-research: confirm the current recommended default HNSW parameters (m, ef_construction) in the pgvector README as of 2026-08. -->

Three things to internalise before you tune indexes:

- **Both indexes are approximate.** They return "the top-*k* nearest points *probably*." For a retrieval feature that grounds a downstream LLM, that is almost always fine — you have a top-5 or top-10 that the model reads, and one wrong entry in position 5 rarely changes the answer. For workloads where the exact nearest neighbour matters (deduplication, near-duplicate detection), use an exact scan and pay the linear cost.
- **Build the index *after* the bulk load, not before.** Insert into an unindexed table, `CREATE INDEX` once at the end. Building an index while writes are happening is slower than doing them in sequence.
- **Do not add an ANN index if your corpus is small.** Below ~10,000 vectors on modern hardware, a full scan is usually faster than the ANN codepath. Add the index when you can measure that scan latency is the bottleneck, not on day one.

## An operationally complete example

Putting it all together — schema, ingest, query — end to end:

```python
import psycopg
from pgvector.psycopg import register_vector

# 1. Connect and register the vector type.
conn = psycopg.connect(DATABASE_URL)
register_vector(conn)

# 2. Make sure the extension and table are in place.
with conn.cursor() as cur:
    cur.execute("CREATE EXTENSION IF NOT EXISTS vector;")
    cur.execute(
        """
        CREATE TABLE IF NOT EXISTS documents (
            id           BIGSERIAL PRIMARY KEY,
            source_uri   TEXT NOT NULL,
            chunk_index  INT  NOT NULL,
            content      TEXT NOT NULL,
            embedding    VECTOR(1536) NOT NULL,
            created_at   TIMESTAMPTZ NOT NULL DEFAULT NOW(),
            UNIQUE (source_uri, chunk_index)
        );
        """
    )
conn.commit()

# 3. Ingest — embed once, insert in a batch.
passages = load_and_split_source_text()  # returns list[(source_uri, chunk_index, content)]
texts = [p[2] for p in passages]
vectors = embed_batch(texts, input_type="document")  # from chapter 1

with conn.cursor() as cur:
    cur.executemany(
        """
        INSERT INTO documents (source_uri, chunk_index, content, embedding)
        VALUES (%s, %s, %s, %s)
        ON CONFLICT (source_uri, chunk_index) DO NOTHING
        """,
        [(uri, idx, text, vec) for (uri, idx, text), vec in zip(passages, vectors)],
    )
conn.commit()

# 4. Query at retrieval time.
q_vec = embed_batch(["what is our refund policy for damaged items?"], input_type="query")[0]

with conn.cursor() as cur:
    cur.execute(
        """
        SELECT source_uri, chunk_index, content, embedding <=> %s AS distance
          FROM documents
         ORDER BY embedding <=> %s
         LIMIT 5
        """,
        (q_vec, q_vec),
    )
    hits = cur.fetchall()

for source_uri, chunk_index, content, distance in hits:
    print(f"{distance:.3f}  {source_uri}#{chunk_index}  {content[:80]!r}")
```

That is a full retrieval store in ~40 lines of Python and one Postgres extension. Every knob you might turn later — replacing the index type, adding a tenant filter, switching embedding models — is a small delta on that shape.

## What this chapter is *not* teaching

Three things that are genuinely important for retrieval-augmented systems but that live in `rag-engineer-learning`, not here:

- **Chunking strategy.** How you split a source document into passages before embedding it dramatically affects retrieval quality. Fixed-size character windows, sentence-aware splits, semantic chunking, overlap-and-stride — there is a whole design space, and picking the wrong one is often the reason retrieval "does not work." For this module, use a simple sentence-aware splitter with a modest chunk size (~500 tokens) and treat chunking as out of scope for now.
- **Embedding-model selection.** Trade-offs across dimension, cost, per-domain performance, multilingual coverage, MTEB scores, and re-embed cost when you switch models are their own topic. For this module, pick one recommended model per provider and stay with it.
- **ANN internals and vector-DB comparisons.** Pinecone vs. Weaviate vs. Qdrant vs. Chroma vs. Milvus — indexing algorithms, sharding, replication, hosted vs. self-hosted — belongs to the RAG track.

The one-line rule from mod-005's introduction still holds: this module gets you to "a colleague can point it at a document set and see it run." Anything past that graduates.

## Summary

- `pgvector` adds a `VECTOR(N)` column type, distance operators (`<=>` for cosine), and optional ANN indexes to Postgres.
- Store the source text and metadata (`source_uri`, `chunk_index`) next to the vector — the vector is only useful for search; the text is what you put in the prompt.
- Use `<=>` (cosine distance) for text embeddings. Order ascending, take the top-*k*, over-fetch by 2–5x when you post-filter.
- Add an HNSW index only once you can measure that the exact scan is too slow — usually not on day one.
- Chunking strategy, embedding-model selection, and vector-DB comparisons live in the `rag-engineer-learning` track, not here.

The next chapter takes the top-*k* rows you can now retrieve and turns them into a prompt the model reads — with citations that make the answer verifiable.
