# Chapter 1 — Embeddings and similarity

Retrieval-augmented generation, at its base, is one very small idea repeated a thousand different ways: turn text into numbers so a computer can tell which pieces of text "mean the same thing," pull the relevant pieces out of a large pile, and hand them to the model as part of the prompt. This chapter is about the "turn text into numbers" step and the "which pieces mean the same thing" step. Neither is provider-specific, but every provider dresses the same idea up in slightly different SDK code.

## Motivation

Most first LLM features do not need retrieval. A classifier, a summariser, a translator, a JSON extractor — none of them require external documents. Retrieval enters the moment your product needs the model to *know things it was not trained on*: your company's docs, your customer's tickets, last week's meeting notes, this morning's PDFs. Fine-tuning is a heavy answer to that need. Retrieval is the light one: leave the model alone, give it the right passage of text in the prompt, and let it read.

The pinch point is "give it the *right* passage." You cannot paste every document into every prompt — you would blow the context window and the bill. You need a way to search over a large corpus by *meaning*, not by keyword. Embeddings are how you do that.

## What an embedding is

An **embedding** is a fixed-length list of floating-point numbers — a vector — that represents a piece of text. The embedding model has been trained so that two pieces of text with similar *meaning* end up as vectors that are close together in the vector space. Two pieces of text with unrelated meaning end up far apart.

Concretely, if you ask an embedding model to embed the strings `"refund never arrived"` and `"where is my money back"`, you get two vectors of, say, 1,536 floats each. They will have a small angle between them. If you also embed `"the mitochondria is the powerhouse of the cell"`, that third vector will be nearly perpendicular to the first two. You did not have to write a synonym dictionary; the model absorbed the "these mean the same" relationship from its training.

Three properties matter for this module:

- **Fixed dimension.** Every embedding from the same model has the same length. `text-embedding-3-small` returns 1,536-dim vectors; `text-embedding-3-large` returns 3,072-dim vectors; Voyage's and Cohere's models each have their own dimensions. Two vectors from *different* models are not comparable at all.
- **Deterministic.** Embedding the same text with the same model twice gives you the same vector (up to floating-point rounding). Unlike generation, there is no temperature and no sampling.
- **Dense.** Every dimension carries meaning. Unlike a keyword index — where 99.99% of positions are zero — an embedding is a solid block of ~1k–4k floats. This changes what storage and search look like.

## Cosine similarity: measuring "closeness"

Given two vectors, how do you decide whether they are close? For text embeddings, the near-universal answer is **cosine similarity**:

```
cosine(a, b) = (a · b) / (||a|| * ||b||)
```

That is: the dot product divided by the product of the norms. It ranges from `-1` (opposite meaning) to `1` (identical direction, i.e. the same meaning). In practice on real text embeddings the values cluster in a narrower band — two related sentences might land at `0.72`, two unrelated ones at `0.08`, and truly opposite meanings are rare.

Two practical shortcuts follow from the definition:

- **If your vectors are already normalised (length 1),** cosine similarity is just the dot product `a · b`. Every mainstream embedding provider either returns normalised vectors or lets you request them; do that once at ingest time and every subsequent similarity computation becomes one multiplication and one sum per pair.
- **Cosine distance = `1 - cosine(a, b)`**. Some libraries and databases report distance instead of similarity — smaller is closer. `pgvector` uses distance for its `<=>` operator (chapter 2), so know which one you are looking at.

You will also see **Euclidean distance** (`||a - b||`) and the **dot product** used directly. For normalised text embeddings the three produce the same ranking. Pick cosine unless you have a specific reason otherwise, and stay consistent — mixing metrics across the same corpus is a common self-inflicted bug.

## Calling the embeddings APIs

Every hosted provider exposes essentially the same shape: send a list of strings, get back a list of vectors.

### OpenAI

```python
from openai import OpenAI

client = OpenAI()

response = client.embeddings.create(
    model="text-embedding-3-small",
    input=[
        "refund never arrived",
        "where is my money back",
        "the mitochondria is the powerhouse of the cell",
    ],
)

vectors = [item.embedding for item in response.data]
# vectors[0] and vectors[1] should be close; vectors[2] should be far from both.
```

Reference: <https://platform.openai.com/docs/guides/embeddings>. Model reference (dimensions, pricing tier): <https://platform.openai.com/docs/models>.

<!-- needs-research: confirm the current recommended default OpenAI embedding model and its dimension as of 2026-08 — check https://platform.openai.com/docs/guides/embeddings. -->

Notes:

- OpenAI's `text-embedding-3-*` models accept a `dimensions` parameter to shrink the returned vector below the model's native size (Matryoshka embeddings). Do not use it unless you have measured that the smaller size still hits your retrieval quality — the whole point of a larger model is the extra capacity.
- Batch your calls. The API accepts up to a large number of inputs per request (check the reference for the current cap); a single request of 100 strings is dramatically cheaper in latency and roughly the same in dollars as 100 requests of one string.

### Anthropic

Anthropic does not currently offer a first-party embeddings API. Its docs point you at Voyage AI (`voyage-*` models) as the recommended companion for embeddings when the generation side is Claude. See <https://docs.anthropic.com/en/docs/build-with-claude/embeddings>.

<!-- needs-research: verify Anthropic still points to Voyage as the recommended embeddings partner and confirm the current recommended Voyage model as of 2026-08 — check https://docs.anthropic.com/en/docs/build-with-claude/embeddings and https://docs.voyageai.com/docs/embeddings. -->

### Voyage

```python
import voyageai

vo = voyageai.Client()  # reads VOYAGE_API_KEY

result = vo.embed(
    texts=[
        "refund never arrived",
        "where is my money back",
    ],
    model="voyage-3",       # confirm current recommended model
    input_type="document",  # or "query" — see below
)

vectors = result.embeddings
```

Reference: <https://docs.voyageai.com/docs/embeddings>.

### Cohere

```python
import cohere

co = cohere.ClientV2()

response = co.embed(
    texts=[
        "refund never arrived",
        "where is my money back",
    ],
    model="embed-english-v3.0",   # confirm current recommended model
    input_type="search_document", # or "search_query"
    embedding_types=["float"],
)

vectors = response.embeddings.float
```

Reference: <https://docs.cohere.com/reference/embed>.

## The document / query asymmetry

Voyage, Cohere, and some other providers ask you to tell the API whether you are embedding a **document** (the passage being stored and searched over) or a **query** (the user's question at retrieval time). The vectors it produces are subtly different — the model has been trained to align "what a query looks like" with "what a document that answers it looks like," which is not the same as "two strings that are grammatically similar."

Practical rules:

- If your provider exposes an `input_type` / `input_type="search_query"` / `input_type="search_document"` distinction, use it. Embed passages at ingest time with the document type; embed the user's question at query time with the query type. Compare across the two.
- OpenAI's `text-embedding-3-*` models do not currently expose this distinction — one embedding shape, used symmetrically. That is fine.
- Never mix. Embedding a query with the document type (or vice versa) usually gives measurably worse retrieval and the failure mode is silent — nothing errors, results are just quietly worse.

## What embeddings do *not* know

Two failure modes to internalise before chapter 2, because both look like "retrieval is broken" and are actually "embeddings do not know that":

- **Exact identifiers.** "Order #A47-991" and "Order #A47-992" have basically identical embeddings; the meaning-level model does not care about the last digit. If your users search by exact SKU / ticket ID / invoice number, semantic search is the wrong tool — use a keyword or exact-match index in parallel. Combining the two ("hybrid search") is a mod-005-scope-adjacent topic that the `rag-engineer-learning` track owns; for this module, know the limitation and route exact-lookup queries around retrieval entirely.
- **Freshness and negation.** "The refund policy changed in 2025" and "The refund policy changed in 2024" have very close embeddings. "The bug is fixed" and "the bug is not fixed" have close embeddings. If your product needs to answer negation-sensitive or date-sensitive questions, embedding-based retrieval alone is not enough — the generation-side prompt has to do the work of resolving those distinctions on the retrieved passages, or you need structured metadata alongside the vector (chapter 2 covers that shape).

## Cost shape

Embedding calls are dramatically cheaper per token than generation calls — typically an order of magnitude or more. That does not make them free. Two habits from day one:

- **Embed once, use many.** Store the vector in your database next to the source text. Never re-embed the same passage on every request; that is a bug, not an optimisation opportunity.
- **Re-embed on model change.** The moment you change embedding models, every stored vector is worthless — vectors from `text-embedding-3-small` cannot be compared to vectors from `voyage-3`. Plan the re-embed as a background job; do not attempt a live cutover.

Provider pricing pages are the source of truth:

- OpenAI: <https://openai.com/api/pricing/> (embedding models are listed alongside generation models)
- Voyage: <https://docs.voyageai.com/docs/pricing>
- Cohere: <https://cohere.com/pricing>

<!-- needs-research: confirm current per-million-token embedding prices for text-embedding-3-small, voyage-3, and embed-english-v3.0 as of 2026-08 — cite from the provider pricing pages. -->

## Summary

- An embedding is a fixed-length vector of floats produced by a model that has been trained so semantically similar text lands close together.
- Vectors from different models are not comparable — pick one embedding model per corpus and re-embed whenever you change it.
- Cosine similarity (or, for normalised vectors, the plain dot product) is the near-universal "how close" measure for text embeddings.
- Some providers require you to distinguish "document" vs "query" at embed time; if yours does, respect the distinction.
- Embeddings do not know exact identifiers, dates, or negation — do not use them for those queries.

The next chapter takes the vectors you can now produce and puts them somewhere you can search over — Postgres with `pgvector`.
