# Resources for mod-005 — Retrieval Basics for LLM Applications

Prefer primary sources — provider documentation, extension maintainers, and standards — over blog posts. The list below is deliberately short. If a link goes stale, the fix is to look up the provider or maintainer's current documentation index, not to grab whatever a search engine returns first.

## Embeddings providers

### OpenAI

- **Embeddings guide** — request shape, model dimensions, the `dimensions` (Matryoshka) parameter, batch limits. <https://platform.openai.com/docs/guides/embeddings>
- **Model reference** — current embedding model names and per-model dimensions. <https://platform.openai.com/docs/models>
- **Pricing** — canonical per-million-token embedding prices. <https://openai.com/api/pricing/>

### Voyage AI

- **Embeddings reference** — model list, `input_type` (`"document"` vs `"query"`), batch limits. <https://docs.voyageai.com/docs/embeddings>
- **Pricing** — canonical per-million-token prices. <https://docs.voyageai.com/docs/pricing>
- **Anthropic's pointer to Voyage as the recommended embeddings companion for Claude.** <https://docs.anthropic.com/en/docs/build-with-claude/embeddings>

### Cohere

- **`embed` API reference** — model list, `input_type` (`"search_document"` vs `"search_query"`), `embedding_types`. <https://docs.cohere.com/reference/embed>
- **Pricing** — canonical per-million-token prices. <https://cohere.com/pricing>

## The vector store this module uses

### `pgvector`

- **Extension source, installation guide, and operator / index reference** — the source of truth for `<->`, `<=>`, `<#>`, IVFFlat, and HNSW. <https://github.com/pgvector/pgvector>
- **`pgvector-python`** — Python adapters for `psycopg`, `SQLAlchemy`, `Django`, and `asyncpg`. <https://github.com/pgvector/pgvector-python>
- **`psycopg` docs** — the modern Postgres driver used in the chapter 2 examples. <https://www.psycopg.org/psycopg3/docs/>

<!-- needs-research: confirm the current recommended pgvector version and any changes to the default HNSW parameters (m, ef_construction) as of 2026-08. -->

## Grounded generation and citations

- **Anthropic prompt engineering — "Use XML tags"** — the delimiter guidance chapter 3's `<sources>` / `<question>` shape uses. <https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-xml-tags>
- **OpenAI prompt engineering guide** — the OpenAI-side equivalent of delimiter guidance. <https://platform.openai.com/docs/guides/prompt-engineering>
- **OpenAI Structured Outputs** — the strict-JSON-Schema mode you use for structured citations in exercise 03. <https://platform.openai.com/docs/guides/structured-outputs>
- **Anthropic tool use** — the tool-schema forcing pattern you use as the Anthropic-side equivalent for structured citations. <https://docs.anthropic.com/en/docs/build-with-claude/tool-use>

## Prompt caching for grounded prompts

- **Anthropic prompt caching** — `cache_control` on system / tools / message blocks; usage-block fields for hit vs. miss. <https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching>
- **OpenAI cached input** — automatic prefix caching on Chat Completions / Responses; usage-block fields. <https://platform.openai.com/docs/guides/prompt-caching>

<!-- needs-research: confirm the current OpenAI cached-input docs URL and any usage-field name changes as of 2026-08. -->

## Where the rest of this material lives

The topics mod-005 deliberately does *not* teach in depth all live in the peer track:

- **`rag-engineer-learning` (level 30)** — chunking strategies, embedding-model benchmarking (MTEB), vector-DB internals and comparisons across specialist stores (Pinecone / Weaviate / Qdrant / Chroma / Milvus), re-ranking, hybrid search, query rewriting, retrieval-quality evaluation methodology. <https://github.com/ai-engineering-curriculum/rag-engineer-learning>

For eval methodology that graduates the exercise-04 muscle into a CI-gated regression check, see the next module in *this* track:

- **`mod-006-minimal-eval-and-regression-checks`** — [`../mod-006-minimal-eval-and-regression-checks/README.md`](../mod-006-minimal-eval-and-regression-checks/README.md)

## Specialist vector databases (for the next-track handoff)

These are here so you know where to look when you outgrow `pgvector` — none of them are on the mod-005 exercise path. Depth on their trade-offs lives in `rag-engineer-learning`.

- **Pinecone** — hosted, closed-source. <https://docs.pinecone.io/>
- **Weaviate** — hosted or self-hosted, open source. <https://weaviate.io/developers/weaviate>
- **Qdrant** — hosted or self-hosted, open source. <https://qdrant.tech/documentation/>
- **Chroma** — embeddable, open source. <https://docs.trychroma.com/>
- **Milvus** — self-hosted or hosted (Zilliz), open source. <https://milvus.io/docs>

## Background reading (optional)

- **"Lost in the Middle: How Language Models Use Long Contexts"** — Liu et al., 2023. The effect chapter 3 refers to when it says "best-match last." Skim, do not memorise. <https://arxiv.org/abs/2307.03172>

<!-- needs-research: verify the "Lost in the Middle" arXiv link and citation format before merge; confirm this is the primary source vs. a later revision. -->

## What is deliberately not on this list

- **"How to build a chatbot with RAG in 20 minutes" tutorials.** Almost all of them combine framework abstractions (LangChain / LlamaIndex), a specialist vector DB, and a chunking strategy in ways that make it hard to see which piece is doing which job. Learn the raw shape first (this module), pick your abstractions second.
- **Framework quickstarts (LangChain, LlamaIndex, Haystack).** Those are covered in [`agentic-ai-developer-learning`](https://github.com/ai-engineering-curriculum/agentic-ai-developer-learning) and, for the retrieval-specific pieces, `rag-engineer-learning`. As mod-001's resources page put it: learning the raw API surface first makes the framework abstractions later much easier to reason about.
- **Provider pricing screenshots.** Prices change; screenshots do not. Always link to the current pricing page.
