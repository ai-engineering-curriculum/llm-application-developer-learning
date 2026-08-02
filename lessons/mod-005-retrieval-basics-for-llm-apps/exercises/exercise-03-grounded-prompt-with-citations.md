# Exercise 03 — Grounded prompt with citations

Paired with [chapter 3 — the grounded prompt with citations](../03-grounded-prompt-with-citations.md).

**Estimated effort:** 90–150 minutes.

## Objective

Wire the retrieval store from exercise 02 into a full retrieval-grounded Q&A loop: user question in, embedded and searched, top-*k* assembled into a grounded prompt with structured citations, model called, citations validated against their source passages before the answer is returned.

This is the "minimal retrieval-grounded Q&A feature that a colleague can point at their own document set and see run" that the module was written to build. Get this working and you have hit the load-bearing learning objective for mod-005.

## Problem statement

Build an `ask.py` (or module) that:

1. Takes a user question as a command-line argument.
2. Embeds the question with your chapter-1 provider and asks `pgvector` for the top-*k* passages.
3. Assembles the grounded prompt shape from chapter 3:
   - A system prompt containing the "answer only from the sources" rule.
   - A user prompt with a `<sources>` block and a `<question>` block.
   - Best-match passage placed *last* in the sources block.
4. Calls the generation model (OpenAI or Anthropic) using **schema-constrained output** (mod-001 chapter 4) with the `{"answer": string, "citations": [{"source_id": string, "quote": string}]}` schema from chapter 3.
5. **Validates every citation** in code — each `source_id` must exist in the sources block, and each `quote` must be a substring of the source it names.
6. Prints the answer and, for each citation, the source URI and whether the quote validated.

## Requirements

### The prompt

- System prompt must contain, in behavioural (not aspirational) wording, the "if the sources do not contain the answer, respond exactly `I don't know.`" rule.
- Use the exact `<sources>` / `<question>` XML-tag delimiter shape from chapter 3, or a documented alternative you can defend.
- Assign source IDs `S1`, `S2`, … in the order they appear in the sources block. Best-match last.

### The model call

- Use **strict structured output** for the response:
  - OpenAI: `response_format={"type": "json_schema", "json_schema": {..., "strict": true}}`.
  - Anthropic: define a single tool with the citation schema as `input_schema` and force it with `tool_choice`.
- Ask the model to return the JSON directly — do not ask for `[S2]`-style inline markers in this exercise. Free-form markers are covered in the chapter as a fallback; the structured shape is what you build against.

### Citation validation

Implement `validate_citations(payload, sources_by_id)`:

- Every `citation.source_id` must appear in `sources_by_id`. Unknown ID → raise / return an error.
- Every `citation.quote` must be a **verbatim substring** of `sources_by_id[source_id].content`. Case-sensitive is fine for a first pass; whitespace-normalise if you want to be more forgiving.
- If validation fails, do **not** silently drop the citation. Either:
  - Reject the whole response and log the failure (default), or
  - Return the answer with an explicit `ungrounded_citations` list surfaced next to it.

Pick one behaviour and commit to it — the point is that ungrounded citations are handled, not that they are handled the same way as every reader.

### Output format

For a real query:

```
Q: How long do I have to return a damaged item?

Answer:
  You have up to 90 days to return a damaged item if the return is granted at our discretion.

Citations:
  [S2] policies/refunds.md#14  "Damaged items may be returned within 90 days at our discretion."  (validated)

Retrieved passages (top-5):
  rank 1  distance 0.183  policies/refunds.md#14  (placed last in prompt)
  rank 2  distance 0.211  policies/refunds.md#12
  rank 3  distance 0.271  policies/shipping.md#03
  rank 4  distance 0.298  policies/refunds.md#08
  rank 5  distance 0.301  policies/faq.md#02
```

That "retrieved passages" section is the load-bearing log for exercise 04. Do not skip it.

### The refusal path

Add a `--distance-threshold` flag (default something reasonable for your corpus — start with `0.5`, tune per your exercise-02 observations). If the top-1 hit's distance is above the threshold, **do not call the model.** Print `I don't know (no relevant passages retrieved).` and exit.

## Acceptance criteria

- Running `ask.py --query "..."` on an answerable question returns a short, correct answer, one or more citations, and the retrieved-passages log.
- Running `ask.py --query "..."` on a question your corpus obviously does not answer:
  - Either the retriever short-circuits on the threshold, or
  - The model returns the exact string `I don't know.` per the system prompt.
- Every citation returned to the user has passed `validate_citations`. You have manually verified this by picking one answer's citation, grepping the `quote` field in the original source file, and finding an exact match.
- You can point at at least **one** example question, from your corpus, where deliberately breaking the retrieval (e.g. `WHERE source_uri = 'a-file-that-does-not-answer.md'`) makes the answer either flip to `I don't know.` *or* start producing citations that fail validation. This is your first evidence that citations are actually load-bearing on the retrieved passages.

## Starter guidance

- Chapter 3 has the full `build_sources_block` / `build_messages` skeleton. Copy it and modify as needed.
- If you have not shipped strict structured output before, revisit mod-001 chapter 4 for the OpenAI `json_schema` and Anthropic `tool_use` patterns.
- For the "quote is a substring" check, whitespace normalisation (`re.sub(r'\s+', ' ', s).strip()`) on both the source and the quote catches most legitimate variants without opening the door to fully invented quotes.
- Keep `k` small — 3 to 5 is plenty for this exercise. Larger `k` will pass fine but you will be doing a lot of hand-eyeballing.

## Stretch goals

- **Enable prompt caching** on the system prompt (Anthropic `cache_control: {"type": "ephemeral"}` on the system block; OpenAI cached input is automatic if the prefix is stable enough — verify against the response `usage` block).
- **Add a `--k` flag** and, on a fixed test question, sweep `k = 1, 3, 5, 10`. Does the answer change? Does it get more accurate? Does the model start hedging?
- **Add a "verbatim quote required" check** — reject any answer where no citation's `quote` overlaps with any word of the `answer` string. This catches the "citation-as-fig-leaf" failure mode where the model cites a source that has nothing to do with what it just said.
- **Log everything to a JSONL file** — query, retrieved passages (with distances), full prompt, full response, citation-validation result, model, timing. This becomes the input to exercise 04's triage work.
