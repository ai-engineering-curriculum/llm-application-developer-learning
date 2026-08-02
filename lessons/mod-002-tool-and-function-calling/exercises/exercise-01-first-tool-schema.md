# Exercise 01 — First tool schema

Paired with [chapter 1 — declaring tool schemas](../01-declaring-tool-schemas.md).

**Estimated effort:** 45–60 minutes.

## Objective

Design a tool schema — name, description, and JSON Schema — that the model uses correctly on ambiguous inputs. Confirm you can render the same canonical schema into both provider dialects. The point of the exercise is that a good description and a tight input schema do most of the work.

## Problem statement

Design a `search_products` tool for a fictional online store. On a given user message the model must decide whether to call the tool at all and, if so, with what arguments.

Requirements for the tool:

- Name: `search_products`.
- Purpose: return a list of products matching a query, optionally filtered by category and by maximum price.
- Categories are a fixed enum: `"electronics"`, `"clothing"`, `"home"`, `"toys"`, `"books"`.
- Price is in USD, capped at $10,000. Negative prices are invalid.

Test messages the model must handle:

1. `"Find me some cheap headphones under $50"` — should call the tool with `category="electronics"` (or none), `max_price=50`, and a reasonable query string.
2. `"What's the weather in London?"` — should NOT call the tool.
3. `"Show me kids' toys"` — should call the tool with `category="toys"`.
4. `"I want a book about Roman history that costs less than a coffee"` — a borderline case. Watch how the model interprets "less than a coffee" — a good schema description helps here.

## Requirements

1. Write a **canonical JSON Schema** for the tool's input arguments as a plain Python dict. Do not hand-write two copies for the two providers.
2. Write two adapter functions: `to_anthropic_tool(name, description, input_schema)` and `to_openai_tool(name, description, input_schema, strict=True)`. Chapter 1 shows the shape of each.
3. Send the tool to both providers (or one, if you only have credit for one) on each of the four test messages. Print the tool_use / tool_calls block (or the plain text reply if the model did not call the tool).
4. On the OpenAI side, use `strict: true` and satisfy its subset restrictions (`additionalProperties: false`; every property listed in `required`; use nullable types for genuinely optional fields).
5. Use `enum` for the category. Use `minimum` and `maximum` for the price.
6. Every argument's `description` must explain the semantics — what units, what defaults, what happens on ambiguity.

## Starter guidance

Skim these before you start:

- Anthropic tool use: <https://docs.anthropic.com/en/docs/build-with-claude/tool-use>
- OpenAI function calling: <https://platform.openai.com/docs/guides/function-calling>
- Strict-mode subset restrictions: <https://platform.openai.com/docs/guides/structured-outputs#supported-schemas>

You do not need to *execute* any tool for this exercise — the goal is only to observe *what* the model asks to call and with *what* arguments. Print the tool_use block (Anthropic) or the `tool_calls` list (OpenAI) and stop there.

## Acceptance criteria

- Your script defines the schema exactly once and renders it into both provider shapes via the adapter functions.
- On test message 1, the model produces a tool call whose `max_price` is `50` and whose `query` is a plausible search phrase for headphones.
- On test message 2, the model does **not** produce a tool call. It produces a text reply. (If it does call the tool, revisit your description — the boundary is not clear enough.)
- On test message 3, the model produces a tool call with `category: "toys"`.
- On test message 4, whatever the model does, you can point to the specific sentence in your description or the specific field in your schema that produced its behaviour.
- The OpenAI request is accepted with `strict: true`. If the provider returns a schema-validation error, that is a bug to fix, not a reason to disable strict.

## Stretch goals

- Add a `sort_by` argument with enum values `"relevance"`, `"price_asc"`, `"price_desc"`. Try test message 1 again — does the model set it, or leave it as the default?
- Deliberately weaken the description (delete the trigger/boundary sentences). Rerun. Note how the model's behaviour on the ambiguous test messages changes. Restore the description and note the delta.
- Read the [JSON Schema draft 2020-12 spec](https://json-schema.org/draft/2020-12/schema) index and add a `pattern` constraint (e.g., a two-letter locale code the model must pick from `["en", "fr", "de", ...]`). Confirm your provider accepts it.
