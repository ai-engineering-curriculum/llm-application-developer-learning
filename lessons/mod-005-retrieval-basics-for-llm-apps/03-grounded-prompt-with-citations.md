# Chapter 3 — The grounded prompt with citations

Retrieval that never reaches the model is decoration. This chapter is the last mile: take the top-*k* passages from chapter 2, paste them into a prompt in a shape the model reads reliably, and instruct the model to *cite* which of them each part of its answer came from. Citations are not a nice-to-have — they are the single feature that turns "the LLM said something confident" into "the user can verify it."

## Motivation

Without grounding, the model answers from its training data — sometimes correct, sometimes fluently wrong (chapter 5 of mod-001 called this hallucination). Grounding it on retrieved passages *helps*, but only if two things are true:

1. **The passages are actually in the prompt** in a shape the model can attend to.
2. **The prompt tells the model to prefer the passages over its own knowledge** and to say so when a passage does not cover the question.

If the passages are pasted in but the instructions do not name them, the model will treat them as background reading and answer from wherever. If the passages are named but the instructions do not require citations, the answer will read like a summary but nobody can check which sentence came from which source. Both patterns look like retrieval working; neither is.

## The shape of a grounded prompt

The reliable pattern, across both major providers, is:

1. **System prompt.** Persona plus the "answer only from the passages" rule plus the citation format.
2. **User prompt with two delimited sections.** A `<sources>` block that lists each retrieved passage with an ID, and a `<question>` block containing what the user actually asked.

You already know most of the pieces — this is the "delimit and separate rules from data" pattern from mod-001 chapter 2, applied to retrieval.

```
System:
  You answer questions using only the passages provided inside <sources>.
  If the sources do not contain the answer, respond exactly "I don't know."
  Cite every claim with the source ID in brackets, e.g. [S2].
  Do not invent source IDs. Do not answer from outside the sources.

User:
  <sources>
  <source id="S1" uri="policies/refunds.md#12">
  We accept returns within 30 days for undamaged items.
  ...
  </source>
  <source id="S2" uri="policies/refunds.md#14">
  Damaged items may be returned within 90 days at our discretion.
  ...
  </source>
  </sources>

  <question>
  How long do I have to return a damaged item?
  </question>
```

A model answer that follows this shape looks like:

> You have up to 90 days to return a damaged item [S2].

The `[S2]` is the load-bearing character. Every claim gets one; every one refers to a source ID that appeared in the `<sources>` block; the ID maps back to the row you retrieved. Your application can now render "damaged items may be returned within 90 days at our discretion" as a link to the source URI — the user can click through and check.

## Assembling the prompt in code

The mechanical assembly is small. Given the `hits` list from chapter 2:

```python
def build_sources_block(hits):
    lines = ["<sources>"]
    for i, (source_uri, chunk_index, content, distance) in enumerate(hits, start=1):
        # S1, S2, ... — short, stable, easy for the model to reproduce.
        lines.append(
            f'<source id="S{i}" uri="{source_uri}#{chunk_index}">\n'
            f'{content.strip()}\n'
            f'</source>'
        )
    lines.append("</sources>")
    return "\n".join(lines)

SYSTEM = (
    "You answer questions using only the passages provided inside <sources>. "
    "If the sources do not contain the answer, respond exactly \"I don't know.\" "
    "Cite every claim with the source ID in brackets, e.g. [S2]. "
    "Do not invent source IDs. Do not answer from outside the sources."
)

def build_messages(user_question, hits):
    user = (
        f"{build_sources_block(hits)}\n\n"
        f"<question>\n{user_question}\n</question>"
    )
    return SYSTEM, [{"role": "user", "content": user}]
```

Then call the model exactly as you did in mod-001:

```python
system, messages = build_messages(user_question, hits)
response = client.messages.create(
    model="claude-opus-4-7",
    max_tokens=400,
    system=system,
    messages=messages,
)
answer = response.content[0].text
```

That is a working retrieval-grounded Q&A endpoint. The four pieces — embed the query, top-*k* from `pgvector`, assemble the sources block, call the model — are the entire loop. Exercise 03 wires the pieces together end-to-end.

## Structured citations (recommended over free-form)

The `[S2]` marker in prose is easy to write and easy to read. It is also fragile: the model might drop the brackets, might use `(S2)`, might attach two markers to the same claim, might occasionally invent `[S9]` where no `S9` exists. If you want to programmatically render each claim with a link to its source, prefer a structured-output shape (mod-001 chapter 4) whose schema names citations explicitly:

```python
schema = {
    "type": "object",
    "additionalProperties": False,
    "required": ["answer", "citations"],
    "properties": {
        "answer": {"type": "string"},
        "citations": {
            "type": "array",
            "items": {
                "type": "object",
                "additionalProperties": False,
                "required": ["source_id", "quote"],
                "properties": {
                    "source_id": {"type": "string"},   # "S1", "S2", ...
                    "quote": {"type": "string"},        # substring from the source
                },
            },
        },
    },
}
```

The `quote` field is the most useful part of the structure. Requiring the model to reproduce a *verbatim substring* from the source it is citing catches a large fraction of loose or invented citations — you can post-validate in code that every `quote` actually appears in the source it names, and reject or flag the answer if not.

```python
def validate_citations(payload, sources_by_id):
    for c in payload["citations"]:
        source = sources_by_id.get(c["source_id"])
        if source is None:
            raise UngroundedCitationError(f"unknown source_id {c['source_id']!r}")
        if c["quote"] not in source["content"]:
            raise UngroundedCitationError(
                f"quote not found in {c['source_id']}: {c['quote']!r}"
            )
```

Depending on your risk tolerance you can:

- **Reject** the response and either re-prompt or return "I don't know" to the user.
- **Log** and pass through, with a metric you can chart over time.
- **Downgrade** — display the answer but mark uncited claims visually so the user knows to double-check.

## What "answer only from the sources" really means

Two failure shapes show up on almost every first grounded-prompt build:

- **The model refuses to answer even when the source clearly contains it.** Usually because the passage answers the question in different words than the question uses ("returns" vs "refunds"), or because the instructions are so tight that the model plays it safe. Fix: soften the "only" without removing it — "answer the question using the passages provided; if the passages do not contain the answer say 'I don't know'" is often more accurate than "answer only from the sources." Also check chapter 4 of *this* module: refusal-when-source-was-there is one of the retrieval-vs-prompt symptoms.
- **The model answers from outside the sources but still cites one.** The citation is a fig leaf: the answer contains information the source does not, and the source ID is attached to give the impression of grounding. This is why the `quote` field above matters — if the model has to reproduce a substring of the source, it becomes much harder to launder ungrounded content through a bogus citation. Structured citations are not a full fix but they raise the cost of the failure mode dramatically.

## Ordering of passages inside the prompt

Two things about the order:

- **Best-match last.** Models attend most strongly to the beginning and end of the input. If you put your top-1 passage at the end of the `<sources>` block (just before the `<question>`), it is most likely to be the one the model uses. This runs against the natural instinct to "sort by relevance descending" — invert it. This is the "lost in the middle" effect at work.
- **Deduplicate.** If your top-5 has two near-duplicate chunks from the same source, they eat context-window budget and give the model nothing new. Cheap defence: skip a retrieved chunk if its content is a near-duplicate (by simple string overlap or by cosine distance to already-included chunks) of one you have already added.

<!-- needs-research: link to the primary "lost in the middle" study (Liu et al. 2023) once verified against the arXiv page. -->

## How many passages to include

Top-*k* on retrieval is a real knob. Practical starting points:

- **k = 3 to 8** is the usual band for a first retrieval-grounded feature.
- **Small k, better model, verbose passages** > **large k, weaker model, shorter passages** in most first builds. You are almost always trading context-window budget for reasoning budget; the model can only usefully attend to so many passages at once.
- **Over-fetch and post-rank.** Ask `pgvector` for top-20, then trim to the top-5 or top-8 you want to send. That leaves you a knob you can turn later — insert a re-ranker (a topic the RAG track owns), drop passages that fail a metadata filter, deduplicate — without touching the SQL.

The single most useful piece of instrumentation: log both the retrieved passages *and* the citations the model produced, per request. You cannot debug retrieval-grounded output at all without that pair.

## The "no relevant passages" case

Every retrieval feature will, on some queries, retrieve nothing useful. Two policies handle that gracefully:

1. **Threshold on distance.** If the top-1 hit has a cosine distance above some threshold (calibrate against your corpus — commonly around `0.4`–`0.6` for text embeddings, but measure), treat the retrieval as "no match" and skip the model call entirely. Return an "I could not find anything relevant" response.
2. **Let the model refuse.** Rely on the system-prompt rule ("if the sources do not contain the answer, respond exactly 'I don't know.'"). Easier to build; harder to trust — see chapter 4.

For a first version, both. Threshold-out obvious misses, and instruct the model to refuse on the rest.

## Prompt caching applies here

The `<sources>` block is per-request — it changes on every call — but the system prompt, the citation format instructions, and the enclosing structure do not. If you have finished mod-004, mark everything up to the *start* of the `<sources>` block as a cacheable prefix. On a warm cache, the input cost per request drops to just the sources block plus the question, which is often 3–5x cheaper than a fresh call.

- Anthropic prompt caching: <https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching>
- OpenAI cached input: <https://platform.openai.com/docs/guides/prompt-caching>

<!-- needs-research: confirm the current OpenAI docs URL for cached input as of 2026-08. -->

## Summary

- A grounded prompt has three parts: a system prompt that names the "answer only from the sources" rule, a delimited `<sources>` block with per-passage IDs, and a delimited `<question>` block.
- Free-form `[S2]` citations are easy to render but easy for the model to invent — prefer a structured-output shape with a `quote` field so citations can be validated in code.
- Best-match passage goes *last* in the sources block — the model attends most strongly to the ends of the input.
- Threshold on distance to short-circuit the "no relevant passages" case; instruct the model to say "I don't know" on the rest.
- Everything up to the `<sources>` block is cacheable — turn caching on the moment you have a stable system prompt.

The next chapter is about the diagnostic you will need on the day the answer is wrong: was retrieval the bottleneck, or was the prompt?
