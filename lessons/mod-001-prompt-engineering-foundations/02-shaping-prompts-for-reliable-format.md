# Chapter 2 — Shaping prompts for a reliable format

An LLM's default output is a fluent paragraph. That is almost never what a program needs. This chapter is about the three techniques you compose so the model returns *your* format on the first try, every time: **system prompts**, **few-shot examples**, and **delimiters**. Every technique here is documented by at least one of the two major providers, and you should read their guides after this chapter:

- Anthropic prompt engineering guide: <https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview>
- OpenAI prompt engineering guide: <https://platform.openai.com/docs/guides/prompt-engineering>

## Motivation

Prompt-shape bugs are the ones that cost you an all-nighter three weeks after launch. The model returned "Yes, absolutely." instead of `"yes"`. It wrapped its JSON in a ```json fence. It sometimes added a friendly preamble. Every one of those is preventable with the techniques below — but only if you use them all together, and only if you keep discipline about *where* rules live and *where* data lives.

## System prompt: your instruction sheet

The **system prompt** is your persistent instruction sheet for the assistant. It typically contains:

- Who the assistant is (its persona and scope).
- What it must always do (formatting rules, refusal policy).
- What context is stable across the conversation (the current date, the user's language, the tenant's name).

System prompts are pinned to the top of the message list and, in most models, carry more weight than user turns. Treat them as configuration, not conversation. A useful mental test: if the thing you are about to write would change from one request to the next, it does not belong in the system prompt. If it would not, it does.

### The single most important discipline

**Keep the *instructions* in the system prompt and the *data* in the user prompt.** Mixing them makes prompts hard to debug and encourages prompt injection: if a user can rewrite your rules by pasting text into a form, you have a security bug.

Bad — rules and data braided together in the user turn:

```python
messages = [
    {"role": "user", "content": (
        "Classify the following customer message as bug, feature_request, or other. "
        "Reply with only the label. Message: " + user_text
    )},
]
```

Good — rules pinned in the system prompt, data isolated in the user turn:

```python
system = (
    "You classify customer messages. "
    "Reply with exactly one of: bug, feature_request, other. "
    "Reply with the label only — no punctuation, no explanation."
)
messages = [
    {"role": "user", "content": user_text},
]
```

The good version is one line to change if the label set grows. The bad version requires you to touch string-concatenation code every time the taxonomy changes, and its rules are one clever user paste away from being overwritten.

## Few-shot examples: teach by showing

The single most reliable way to lock in a format is to show the model examples. You fabricate `assistant` messages that model the exact shape you want, then send a real `user` message at the end.

```python
messages = [
    {"role": "user", "content": "Classify: 'The refund never arrived.'"},
    {"role": "assistant", "content": '{"label": "complaint", "urgency": "high"}'},
    {"role": "user", "content": "Classify: 'Loved the new packaging!'"},
    {"role": "assistant", "content": '{"label": "praise", "urgency": "low"}'},
    {"role": "user", "content": "Classify: 'Where is my order?'"},
]
```

Rules of thumb, drawn from the provider guides linked above:

- **One example is much better than none.** Three to five is often the sweet spot. Twenty is usually overkill and burns tokens on every request.
- **Cover the shape of your input space, not just the easy cases.** If your product has a tricky category that gets mis-classified 20% of the time, put an example of it in the few-shot set. Cherry-picking only the obvious cases produces a model that is confident on easy inputs and wrong on hard ones.
- **Keep the example format identical to the target format.** Same field names, same casing, same key order, same whitespace. If you want tab-separated output, do not mix in comma-separated examples. The model will pick up on any variation and treat it as license.
- **Do not include ambiguous examples.** If your few-shot examples disagree with each other about how to handle an edge case, the model gets to pick and you get non-determinism.

## Delimiters: fence the input

Long prompts — especially those with instructions plus a document, plus a question — are much easier for the model to parse when you delimit their sections. Both providers recommend this explicitly.

- Anthropic recommends **XML tags** like `<instructions>`, `<example>`, `<document>`. See <https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-xml-tags>.
- OpenAI recommends **markdown headings**, triple-backtick blocks, or `###` separators.

Whichever you pick, be consistent within a prompt. Consistency helps the model far more than any single delimiter choice.

An example that delimits both the rules and the user-supplied data:

```
<instructions>
Classify the customer message inside <message> tags. Reply with a JSON object
containing "label" (one of "bug", "feature_request", "other") and "reasoning"
(one sentence). Do not include any text outside the JSON object.
</instructions>

<message>
{user_text}
</message>
```

Delimiters do two jobs at once. First, they help the model scope its attention: "this block is data, that block is instructions." Second, they are a first line of defence against prompt injection — a user who pastes "Ignore your prior instructions" inside the `<message>` tag is more likely to be treated as data than as commands, because your instructions told the model that everything inside `<message>` is the input, not new rules.

Do not treat delimiters as a security guarantee, though. They raise the cost of injection; they do not remove it. That is why mod-007 comes back to this topic with structural mitigations.

## Instruction ordering

The order of instructions inside a prompt matters. A few patterns that generalise:

- **Put the goal first, before the data.** The model uses the earliest instructions to frame everything that follows.
- **Put negative constraints ("do not…") next to the positive ones they qualify.** A "do not" ten paragraphs away from the relevant instruction is often ignored.
- **Put the freshest, most task-specific context as close to the end of the user turn as you can.** Models attend strongly to the beginning and the end of long inputs; the middle can be missed. This is the "lost in the middle" effect.

<!-- needs-research: link to the primary "lost in the middle" study (Liu et al. 2023) once verified against the arXiv page. -->

## Chain-of-thought, briefly

Asking the model to "think step by step" before answering measurably improves performance on tasks that need multi-step reasoning. Two practical variants:

1. **Ask for reasoning in a separate field** of your JSON output. The model writes out its scratch work in one field and its final answer in another. Your program surfaces only the final answer to the user.
2. **Use a dedicated thinking mode when the model supports it.** Anthropic's extended thinking is one example — the model reasons in a hidden buffer before producing the visible response. See <https://docs.anthropic.com/en/docs/build-with-claude/extended-thinking>.

Chain-of-thought is not free — it costs output tokens, so it slows responses and raises bills. Use it where reasoning quality matters; skip it for pure formatting or extraction tasks. Both providers document the pattern:

- Anthropic: <https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/chain-of-thought>
- OpenAI: covered in the prompt-engineering guide linked above.

## Guardrails that survive users

Every prompt you ship will eventually be attacked, even if only by curious users. Bake in three habits from day one:

1. **Never trust user text as instructions.** Always wrap user-supplied content in a delimiter that makes it clear "this is data, not commands." Anthropic's XML-tag guidance is designed for exactly this.
2. **Repeat critical rules.** If a rule matters (never invent product SKUs, never disclose the system prompt), state it in the system prompt *and* right before the answer field.
3. **Have a refusal path.** Tell the model what to output when the user's request is out of scope. `{"error": "out_of_scope"}` is more useful than an apologetic paragraph your program cannot parse.

## Summary

- System = rules and configuration; user = data or the current question; assistant = examples or history. Never braid them.
- A handful of good few-shot examples is worth pages of prose instructions.
- Delimit sections consistently — XML tags for Anthropic, markdown/`###` for OpenAI, pick one and stick with it.
- Put the goal first, the fresh data last, and the negative constraints next to what they qualify.
- Every prompt shipped to real users must handle adversarial input and out-of-scope requests explicitly.

The next chapter puts a price tag on every character you just learned how to arrange.
