# Lecture 2 — Prompt anatomy

Now that you understand the wire format, this lecture is about what to actually put inside a prompt so the model behaves. Every technique here is documented by at least one of the two major providers:

- Anthropic prompt engineering guide: <https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview>
- OpenAI prompt engineering guide: <https://platform.openai.com/docs/guides/prompt-engineering>

Read those two pages after the lecture. This lecture teaches you what to look for when you do.

## The three roles

Chat APIs speak in three roles. Each has a different job.

### System

The **system** prompt is your instruction sheet for the assistant. It typically contains:

- Who the assistant is (its persona and scope).
- What it must always do (formatting rules, refusal policy).
- What context is stable across the conversation (the current date, the user's language, the tenant's name).

System prompts are pinned to the top of the message list and, in most models, carry more weight than user turns. Treat them as configuration, not conversation.

### User

The **user** message is the thing the human — or the surrounding program — is actually asking. Keep the *instructions* in the system prompt and the *data* in the user prompt. Mixing them makes prompts hard to debug and encourages prompt injection: if a user can rewrite your rules by pasting text into a form, you have a security bug.

### Assistant

The **assistant** message is the model's own turn. You will use this role in two ways:

1. In a multi-turn conversation, you re-send previous assistant replies as history.
2. In **few-shot prompting** (below), you fabricate assistant messages to show the model what its output should look like.

## Few-shot examples

The most reliable way to teach a model a format is to show it examples.

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

- Three to five examples is often the sweet spot. One example is much better than none. Twenty is usually overkill and burns tokens.
- The examples should cover the **shape** of your input space, not just the easy cases. Include the tricky edge you actually care about.
- Keep example format identical to your target format. If you want tab-separated output, do not mix in comma-separated examples.

## Delimiters and structure

Long prompts are easier for the model to parse when you delimit their sections. Both providers recommend this explicitly.

- Anthropic recommends XML tags: `<instructions>`, `<example>`, `<document>`. See <https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-xml-tags>.
- OpenAI recommends markdown headings, triple-backtick blocks, or `###` separators.

Whichever you pick, be consistent within a prompt. Consistency helps the model far more than any single delimiter choice.

## Chain-of-thought and step-by-step reasoning

Asking the model to "think step by step" before answering measurably improves performance on tasks that need multi-step reasoning. Both providers document this pattern:

- Anthropic: <https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/chain-of-thought>
- OpenAI: covered in the prompt-engineering guide linked above.

Two practical variants:

1. **Ask for reasoning in a separate field.** The model writes out its scratch work in one JSON field and its final answer in another. You surface only the final answer to your user.
2. **Use a dedicated thinking mode when the model supports it.** Anthropic's extended thinking is one example — the model reasons in a hidden buffer before producing the visible response. See <https://docs.anthropic.com/en/docs/build-with-claude/extended-thinking>.

Chain-of-thought is not free. It costs output tokens, so it slows down responses and raises bills. Use it where reasoning quality matters; skip it for pure formatting or extraction tasks.

## Instruction ordering

The order of instructions inside a prompt matters. A few patterns that generalise:

- Put the **goal** first, before the data. The model uses the earliest instructions to frame everything that follows.
- Put **negative constraints** ("do not…") next to the positive ones they qualify. A "do not" ten paragraphs away from the relevant instruction is often ignored.
- Put the **freshest, most task-specific context** as close to the end of the user turn as you can. Models attend strongly to the beginning and the end of long inputs; the middle can be missed. This is the "lost in the middle" effect and is discussed in the OpenAI guide.

<!-- needs-research: link to the primary "lost in the middle" study (Liu et al. 2023) once verified against the arXiv page. -->

## Guardrails that survive users

Every prompt you ship will eventually be attacked, even if only by curious users. Bake in three habits:

1. **Never trust user text as instructions.** Always wrap user-provided content in a delimiter that makes it clear "this is data, not commands". Anthropic's XML-tag guidance is designed for exactly this.
2. **Repeat critical rules.** If a rule matters (never invent product SKUs, never disclose the system prompt), state it in the system prompt *and* right before the answer field.
3. **Have a refusal path.** Tell the model what to output when the user's request is out of scope. `{"error": "out_of_scope"}` is more useful than an apologetic paragraph your program cannot parse.

## What to remember

- System = rules, user = data, assistant = examples or history.
- A handful of good few-shot examples is worth pages of instructions.
- Delimit sections consistently; put the goal first and the fresh data last.
- Chain-of-thought helps reasoning-heavy tasks but costs tokens.
- Every prompt shipped to real users must handle adversarial input and out-of-scope requests explicitly.

The next lecture takes these instructions and forces the model's output into a shape your program can parse without regexes.
