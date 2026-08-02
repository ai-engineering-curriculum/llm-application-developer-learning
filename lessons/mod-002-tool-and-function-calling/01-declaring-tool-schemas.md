# Chapter 1 — Declaring tool schemas

Tool calling is the moment your LLM stops being a text box and starts being a component in a larger program. This chapter is about the very first step: describing to the model *what* it is allowed to call, with *what arguments*, and *for what purpose*.

## Motivation

Every tool call begins with a schema you supply on the request. The schema is the entire contract between the model and your program:

- What the tool is called.
- What each argument is named, typed, and constrained to.
- What the tool is for — the description is a prompt, not documentation.

Get the schema right and the model calls your tool with the arguments you expect. Get it wrong — vague description, loose types, missing enum — and you will spend the rest of this module debugging symptoms of a bad contract instead of writing product code.

## What a "tool" actually is

A tool (OpenAI historically calls them "functions", Anthropic calls them "tools") is a **declaration**: name, description, and an input schema. Sending a tool definition to the model does not run any code. It only tells the model that this callable exists.

When the model decides to use the tool, it returns a **request** to call it — a structured object containing the tool name and the arguments it chose. Your program is the thing that actually executes the function. The chapter 2 loop is what closes that gap; this chapter is about the declaration only.

Two things follow from that:

1. The model has never seen your source code. Everything it knows about the tool lives in the schema you send. Descriptions carry weight equal to the system prompt.
2. Nothing about the tool is enforced server-side beyond the input schema. If you declare a tool named `delete_user` with a `user_id` argument, the model will happily emit a call with a plausible-looking user_id it invented. Guardrails are your job, not the model's.

## The three fields every tool schema has

Both providers agree on the essentials:

| Field | Purpose |
|---|---|
| `name` | A short identifier your code will switch on. Snake_case. No spaces. |
| `description` | A prose explanation of *when* the model should call this tool and *what it does*. This is the most important field. |
| Input schema | A [JSON Schema](https://json-schema.org/) describing the arguments. Types, required fields, enums, ranges. |

Providers differ on the wrapping shape (see below), but the interior is JSON Schema in both cases. Learn to write JSON Schema well and you will move between providers with the change of a wrapper.

## Anthropic tool declaration

Anthropic's shape is the flattest of the two. The `tools` argument is a list of objects with three keys:

```python
import anthropic

client = anthropic.Anthropic()

weather_tool = {
    "name": "get_current_weather",
    "description": (
        "Return the current weather for a city. "
        "Use this when the user asks about weather, temperature, or conditions "
        "for a specific place. Do not call this for weather forecasts more than "
        "one hour in the future — this tool does not support forecasting."
    ),
    "input_schema": {
        "type": "object",
        "required": ["city"],
        "properties": {
            "city": {
                "type": "string",
                "description": "City name, optionally with country. e.g. 'Paris' or 'Paris, France'.",
            },
            "units": {
                "type": "string",
                "enum": ["celsius", "fahrenheit"],
                "description": "Temperature units. Default: celsius.",
            },
        },
    },
}

response = client.messages.create(
    model="claude-opus-4-7",
    max_tokens=1024,
    tools=[weather_tool],
    messages=[{"role": "user", "content": "What's the weather in Paris right now?"}],
)
```

The `input_schema` field is literal JSON Schema — the same specification you already know from Chapter 4 of mod-001. Reference: <https://docs.anthropic.com/en/docs/build-with-claude/tool-use>.

## OpenAI tool declaration

OpenAI wraps the same information in one extra layer: each tool has a `type` (currently always `"function"`) and the schema lives under a `function` key.

```python
from openai import OpenAI

client = OpenAI()

weather_tool = {
    "type": "function",
    "function": {
        "name": "get_current_weather",
        "description": (
            "Return the current weather for a city. "
            "Use this when the user asks about weather, temperature, or conditions "
            "for a specific place. Do not call this for weather forecasts more than "
            "one hour in the future — this tool does not support forecasting."
        ),
        "parameters": {
            "type": "object",
            "additionalProperties": False,
            "required": ["city", "units"],
            "properties": {
                "city": {
                    "type": "string",
                    "description": "City name, optionally with country. e.g. 'Paris' or 'Paris, France'.",
                },
                "units": {
                    "type": "string",
                    "enum": ["celsius", "fahrenheit"],
                    "description": "Temperature units.",
                },
            },
        },
        "strict": True,
    },
}

response = client.chat.completions.create(
    model="gpt-4.1",
    tools=[weather_tool],
    messages=[{"role": "user", "content": "What's the weather in Paris right now?"}],
)
```

<!-- needs-research: confirm the currently recommended default OpenAI model that supports strict function tools as of 2026-08 — check https://platform.openai.com/docs/models. -->

Two OpenAI-specific things to notice:

- The schema field is called `parameters`, not `input_schema`.
- Setting `strict: true` opts the tool into schema-constrained argument generation — the same mechanism you used for Structured Outputs in mod-001. It comes with the same restrictions: every property must be listed in `required`, and `additionalProperties` must be `false`. Reference: <https://platform.openai.com/docs/guides/function-calling>.

Prefer strict tools on OpenAI whenever the schema fits the strict subset. Without `strict: true` you drop back to best-effort argument generation and get the format-drift failure mode from chapter 5 of mod-001.

## JSON Schema vs "provider-specific" tool definitions

Both providers accept **plain JSON Schema** for the input/parameters block. So in practice the only provider-specific parts are:

- Where the schema lives (`input_schema` for Anthropic, `parameters` inside `function` for OpenAI).
- OpenAI's `strict: true` flag and its accompanying subset restrictions.
- A handful of provider-owned "built-in" tools (Anthropic exposes computer-use and file-editor tools; OpenAI exposes web search, file search, code interpreter). Those built-ins have provider-defined schemas you use as-is — do not try to redefine them locally.

For your own tools, the practical playbook is:

1. Write the input schema once as ordinary JSON Schema.
2. Wrap it in the two provider shapes at call time.
3. Never hand-write two divergent copies. A small helper (`to_openai_tool(schema)`, `to_anthropic_tool(schema)`) keeps them aligned.

If you find yourself reaching for a provider-specific feature outside the shared JSON Schema surface, pause and ask whether it is really required. Provider-specific features lock you in; JSON Schema does not.

## Writing schemas the model will actually use well

The schema is not just a type constraint. It is a hint to the model about *when* and *how* to call the tool. Four practices matter more than the rest:

### 1. Descriptions are prompts, not comments

The `description` field is fed to the model verbatim. It is the primary signal the model uses to decide whether a given user request should trigger this tool. A description of "gets weather" is worse than useless — the model has to guess when to use it, and will call it too often on borderline cases (city names in unrelated questions) or not often enough (a user asking "should I bring an umbrella?").

A useful description names the trigger and the boundary:

- **Trigger:** "Use when the user asks about current weather, temperature, or conditions for a place."
- **Boundary:** "Do not call for forecasts more than one hour in the future."

Every argument's `description` field also gets read. Use it to explain the units, the format, and what happens on ambiguity.

### 2. Constrain aggressively

Every string that has a fixed set of valid values should be an `enum`. Every number with a bounded range should have `minimum` and `maximum`. Every pattern-shaped string (dates, IDs, currency codes) should have a `pattern` or a `format`.

Loose schemas ("just a string") turn every argument into an opportunity for the model to invent something. Tight schemas turn the model's creativity into structured input, which is what you want.

### 3. Mark required fields explicitly

Any field the model *must* supply belongs in `required`. Optional fields are for genuine defaults. If a field is functionally required but not in `required`, the model will occasionally omit it — and your tool implementation now has to handle a `None` case that should never have happened.

Under OpenAI strict mode, **every property must be in `required`**; use `["null", "string"]` (a union type) for genuinely optional inputs. Reference: <https://platform.openai.com/docs/guides/structured-outputs#supported-schemas>.

### 4. Prefer a small number of well-designed tools

A dozen narrow tools (`get_user_email`, `get_user_phone`, `get_user_address`) crowd the model's attention and make selection unreliable. One tool with a well-typed enum field (`get_user_field` with `field: enum["email", "phone", "address"]`) is usually cleaner. Both providers document tool-count practical limits in their tool-use guides; treat two dozen as the rough ceiling for reliable selection and revisit your grouping before you go past it.

## A minimal cross-provider helper

Here is the pattern most codebases converge on: keep one canonical definition and produce both provider shapes from it.

```python
def to_anthropic_tool(name, description, input_schema):
    return {"name": name, "description": description, "input_schema": input_schema}

def to_openai_tool(name, description, input_schema, strict=True):
    return {
        "type": "function",
        "function": {
            "name": name,
            "description": description,
            "parameters": input_schema,
            "strict": strict,
        },
    }
```

If you have a `strict=True` OpenAI tool, remember to also add every property to `required` and set `additionalProperties: False` on the schema — otherwise the API will reject it at call time.

## What the response looks like when the model wants to call a tool

You will see the full loop in chapter 2, but for orientation: when the model decides to call a tool, the response's stop reason changes and the content includes a structured tool-call block.

- **Anthropic:** `message.stop_reason == "tool_use"` and one or more content blocks with `type: "tool_use"`, each carrying `id`, `name`, and `input` (a dict already shaped like your schema).
- **OpenAI:** `response.choices[0].finish_reason == "tool_calls"` and a `message.tool_calls` list, each with `id`, `function.name`, and `function.arguments` (a JSON string you have to parse).

Notice the small asymmetry: Anthropic hands you a parsed dict; OpenAI hands you a string. Do not forget the `json.loads` on the OpenAI side.

## Summary

- A tool declaration is three fields: `name`, `description`, and a JSON Schema for the arguments.
- The provider difference is a thin wrapper. JSON Schema is the portable interior.
- Prefer OpenAI's `strict: true` mode when the schema fits its subset; it is the same class of guarantee as Structured Outputs.
- Descriptions and argument constraints do most of the work. Vague descriptions and loose types are the two biggest causes of bad tool-calling behaviour.
- Keep one canonical schema per tool and wrap it into both provider shapes with a helper — do not duplicate.

The next chapter takes these declarations and shows what happens when you actually run the loop: request → tool_use → tool_result → final answer.
