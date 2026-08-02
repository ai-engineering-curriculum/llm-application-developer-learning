# Resources for mod-002 — Tool and Function Calling

Prefer primary sources — provider documentation and specification authors — over blog posts. Tool-calling APIs move quickly; a link to the provider's current tool-use guide will outlive a screenshot of a specific SDK version.

## Provider references

### Anthropic

- **Tool use overview** — the canonical reference for tool declarations, the tool_use / tool_result loop, and `tool_choice`. <https://docs.anthropic.com/en/docs/build-with-claude/tool-use>
- **How to implement tool use** — step-by-step build guide with request/response examples. <https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use>
- **Parallel tool use** — how the model fans out multiple tool_use blocks per turn and how to disable it. <https://docs.anthropic.com/en/docs/build-with-claude/tool-use/parallel-tool-use>
- **Handling tool-use errors** — the `is_error` flag and recommended patterns for surfacing failures to the model. <https://docs.anthropic.com/en/docs/build-with-claude/tool-use/handling-tool-use-errors>
- **Messages API reference** — the underlying request shape used by every tool-calling example in this module. <https://docs.anthropic.com/en/api/messages>
- **Client SDKs** — official Python and TypeScript libraries. <https://docs.anthropic.com/en/api/client-sdks>

### OpenAI

- **Function calling guide** — declarations, `tool_choice`, strict mode, and the tool_calls / tool-result loop. <https://platform.openai.com/docs/guides/function-calling>
- **Structured Outputs** — the strict-mode subset restrictions that also apply to strict function tools. <https://platform.openai.com/docs/guides/structured-outputs>
- **Parallel function calling** — the `parallel_tool_calls` request parameter and its default. <https://platform.openai.com/docs/guides/function-calling#parallel-function-calling>
- **Chat Completions API reference** — request/response fields, including `tool_calls` and the `role: "tool"` message. <https://platform.openai.com/docs/api-reference/chat>
- **Model reference** — which models support function calling, and their per-model limits. <https://platform.openai.com/docs/models>
- **Client SDKs** — official Python and TypeScript libraries. <https://platform.openai.com/docs/libraries>

## Standards worth knowing

- **JSON Schema** — the specification behind every `input_schema` / `parameters` block. Know at least `type`, `required`, `additionalProperties`, `enum`, `minimum` / `maximum`, and `pattern`. <https://json-schema.org/>
- **JSON Schema draft 2020-12** — the current draft targeted by both providers' strict modes. Skim the vocabularies index once. <https://json-schema.org/draft/2020-12/schema>
- **IANA time zone database** — the canonical source for the timezone strings used in exercises 02 and 04. <https://www.iana.org/time-zones>

## Libraries

- **pydantic** — the standard Python library for validating tool arguments before you execute the tool. <https://docs.pydantic.dev/>
- **zod** — the TypeScript equivalent, if you are working across languages. <https://zod.dev/>
- **`concurrent.futures.ThreadPoolExecutor`** — the standard-library primitive for running independent tool calls concurrently. <https://docs.python.org/3/library/concurrent.futures.html>
- **`asyncio`** — the async alternative when your tools are I/O-bound and already `async def`. <https://docs.python.org/3/library/asyncio.html>

## Related modules and tracks

- **mod-001-prompt-engineering-foundations** (this track) — required prerequisite. Chapter 4 (schema-constrained JSON output) is the single most load-bearing precursor.
- **mod-003-streaming-async-and-orchestration** (this track, next) — streaming the model's output as tokens are generated, and running many tool calls concurrently at scale.
- **mod-004-model-selection-cost-and-prompt-caching** (this track, later) — how prompt caching softens the round-trip cost of the tool-call loop.
- **mod-006-minimal-eval-and-regression-checks** (this track, later) — how to catch the *semantic* failure mode chapter 4 explicitly leaves out (the model calling the wrong tool, or calling it for the wrong reason).
- **`agentic-ai-developer-learning`** (peer track, level 20) — multi-step planning, ReAct-style loops, and multi-agent coordination. Chapter 5 explains why that boundary is where it is. <https://github.com/ai-engineering-curriculum/agentic-ai-developer-learning>

## What is deliberately not on this list

- Framework quickstarts (LangChain agents, LlamaIndex tool use, and so on). Those live in the peer `agentic-ai-developer-learning` track. Learning the raw tool-call API surface first — which this module does — makes the framework abstractions later much easier to debug.
- "Awesome-tools" lists and blog posts. Tool-calling patterns evolve every few months; the provider docs above are updated in step, the blog posts are not.
- Provider pricing screenshots. See mod-001 resources — link to the current pricing page, do not cite yesterday's numbers.
