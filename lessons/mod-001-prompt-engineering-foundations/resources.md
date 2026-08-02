# Resources for mod-001 — Prompt Engineering Foundations

Prefer primary sources — provider documentation and library maintainers — over blog posts. The list below is deliberately short. If a link goes stale, the fix is to look up the provider's current documentation index, not to grab whatever a search engine returns first.

## Provider references

### OpenAI

- **Text generation guide** — the current message-list request shape for Chat Completions and Responses. <https://platform.openai.com/docs/guides/text-generation>
- **Prompt engineering guide** — the current recommended patterns for OpenAI models. <https://platform.openai.com/docs/guides/prompt-engineering>
- **Structured Outputs** — JSON mode and JSON Schema (`strict: true`) reference. <https://platform.openai.com/docs/guides/structured-outputs>
- **Model reference** — context windows, deprecation timelines, and per-model quirks. <https://platform.openai.com/docs/models>
- **Pricing** — canonical per-token input and output prices per model. <https://openai.com/api/pricing/>
- **Tokenizer playground** — paste text, see how it tokenises. <https://platform.openai.com/tokenizer>
- **Client SDKs** — official Python and TypeScript libraries. <https://platform.openai.com/docs/libraries>

### Anthropic

- **Messages API reference** — request/response shape, `system` field, content blocks. <https://docs.anthropic.com/en/api/messages>
- **Prompt engineering overview** — index into the technique-by-technique pages. <https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview>
- **Use XML tags** — the delimiter guidance chapter 2 refers to. <https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-xml-tags>
- **Chain-of-thought prompting** — pattern reference. <https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/chain-of-thought>
- **Extended thinking** — dedicated hidden-buffer reasoning mode. <https://docs.anthropic.com/en/docs/build-with-claude/extended-thinking>
- **Tool use** — the mechanism this module uses as a schema-constrained-output workaround, and the whole subject of mod-002. <https://docs.anthropic.com/en/docs/build-with-claude/tool-use>
- **Token counting endpoint** — count input tokens for a payload without spending them on generation. <https://docs.anthropic.com/en/docs/build-with-claude/token-counting>
- **Model reference** — context windows and current model names. <https://docs.anthropic.com/en/docs/about-claude/models>
- **Pricing** — canonical per-token input and output prices per model. <https://www.anthropic.com/pricing>
- **Client SDKs** — official Python and TypeScript libraries. <https://docs.anthropic.com/en/api/client-sdks>

## Tokenizers and libraries

- **tiktoken** — OpenAI's official tokenizer as a Python library. <https://github.com/openai/tiktoken>
- **pydantic** — the standard library for validating parsed model output in Python. <https://docs.pydantic.dev/>
- **zod** — the standard library for validating parsed model output in TypeScript. <https://zod.dev/>

## Standards worth knowing

- **JSON Schema** — the specification behind every strict-mode structured-output config. Know at least "type", "required", "additionalProperties", "enum", and pattern constraints. <https://json-schema.org/>
- **Server-Sent Events (SSE)** — the transport used by streaming LLM APIs. Not needed for mod-001, referenced heavily in mod-003. <https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events>

## Background reading (optional but useful)

- **"Attention Is All You Need"** — Vaswani et al., 2017. The transformer paper that underlies every model you will call in this module. Skim, do not memorise. <https://arxiv.org/abs/1706.03762>

<!-- needs-research: verify a currently-canonical primary link for the "lost in the middle" study (Liu et al. 2023) referenced in chapter 2 before merge. -->

## What is deliberately not on this list

- Prompt-engineering "guides" published by non-provider companies. The good patterns are the ones the providers document; the rest is often stale within months.
- Framework quickstarts (LangChain, LlamaIndex, and so on). Those are covered in the peer track [`agentic-ai-developer-learning`](https://github.com/ai-engineering-curriculum/agentic-ai-developer-learning). Learning the raw API surface first — which this module does — makes the framework abstractions later much easier to reason about.
- Provider pricing screenshots. Prices change; screenshots do not. Always link to the current pricing page.
