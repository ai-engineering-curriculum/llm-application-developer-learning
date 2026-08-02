# Exercise 01 — First embeddings and similarity

Paired with [chapter 1 — embeddings and similarity](../01-embeddings-and-similarity.md).

**Estimated effort:** 45–90 minutes.

## Objective

Prove to yourself, on strings you picked, that cosine similarity between hosted-model embeddings behaves the way chapter 1 says it does — semantically similar text is close, unrelated text is far, and identifiers / negation are *not* well captured. Every later exercise in this module rests on the reflex you build here.

## Problem statement

Write a small script (Python) that:

1. Calls a hosted embeddings API — OpenAI's `text-embedding-3-small`, Voyage's `voyage-3`, or Cohere's `embed-english-v3.0` — pick one and stay with it for the whole exercise.
2. Embeds a small set of strings *in a single batched request*.
3. Computes the full pairwise cosine-similarity matrix.
4. Prints the matrix in a readable form.
5. Answers, in comments or printed output, the two questions below.

Use this exact set of strings so your numbers are comparable across attempts:

```python
STRINGS = [
    # A. Two paraphrases of the same idea.
    "I never received the refund for my last order.",
    "Where is the money I was supposed to be refunded?",

    # B. A different topic, similar sentence shape.
    "The mitochondria is the powerhouse of the cell.",

    # C. Two strings that differ only in an exact identifier.
    "Please cancel order A47-991 immediately.",
    "Please cancel order A47-992 immediately.",

    # D. Two strings that differ only in negation.
    "The bug in the checkout flow is fixed.",
    "The bug in the checkout flow is not fixed.",
]
```

## Requirements

- One **batched** request. Do not call the API once per string — the whole point of the endpoint is that it takes a list.
- Compute cosine similarity yourself in code — do not import a "vector similarity" helper. This is a one-liner: `dot(a, b) / (norm(a) * norm(b))`. Use `numpy` if you like.
- Print a 7×7 matrix (or the upper triangle) with values rounded to 3 decimals. Row/column labels can be the first ~20 chars of each string, or letters `A1, A2, B, C1, C2, D1, D2` from the groupings above.
- Read the API key from an environment variable. Do not paste it into the code.
- Print the **dimension** of the embeddings you got back — this is a fact you should look at once so you know what number to put in your `pgvector` schema in exercise 02.

## Questions to answer (in the script's printed output or a top-of-file comment)

1. **Which pair from A, C, or D has the *highest* cosine similarity?** Is it the one you would expect from the meaning? Explain in one sentence.
2. **Compare the similarity of the two "cancel order A47-991 vs. A47-992" strings to the similarity of the two "the bug is fixed vs. is not fixed" strings.** What does this tell you about what embedding models capture and what they miss?

## Starter guidance

- Chapter 1 has minimal request code for OpenAI, Voyage, and Cohere — copy the one for the provider you picked.
- Provider quickstarts if you have not called an embeddings API before:
  - OpenAI: <https://platform.openai.com/docs/guides/embeddings>
  - Voyage: <https://docs.voyageai.com/docs/embeddings>
  - Cohere: <https://docs.cohere.com/reference/embed>
- For Voyage or Cohere, pass `input_type="document"` for this exercise — the asymmetry only matters once you have documents *and* queries (exercise 03).

## Acceptance criteria

- Your script prints a 7×7 (or 7×7 upper-triangle) cosine similarity matrix.
- The **A pair** (paraphrases) is meaningfully closer than any A-to-B or C-to-D cross-pair.
- The **C pair** (identifier-only difference) is *very* close — usually above 0.98. You can state, in one sentence, why this is a warning sign for anyone using semantic search on exact IDs.
- The **D pair** (negation-only difference) is also very close. You can state, in one sentence, why this is a warning sign for anyone using semantic search on questions where negation matters.
- The printed embedding dimension matches what your provider's docs say for the model you chose.

## Stretch goals

- Repeat the whole exercise with a *different* embedding model from the same provider (e.g. `text-embedding-3-small` → `text-embedding-3-large`, or Voyage's small vs. large). How do the numbers change? Is the C-pair gap smaller with the bigger model? Do not assume — measure.
- Add three of your own strings from your work or personal domain (a support ticket you wrote, two lines from a policy doc, a Slack message). Extend the matrix and see whether the neighbourhood structure makes intuitive sense to you.
- Time both the single-batched call and an equivalent set of one-per-string calls (careful with rate limits). Report the wall-clock ratio. This is why chapter 1 said "batch your calls" — the ratio is usually dramatic.
