# Lecture 3 — Structured output

Free-form prose is fine for a chatbot. It is a disaster for an application. If your program has to `if "yes" in response.lower()` on a model's reply, your product will break the day the model returns "Yes, that's correct" instead of "yes". This lecture is about forcing the model to produce machine-parseable output *by construction*.

## Three ways to get structured output

There are three techniques, in increasing order of guarantee:

1. **Ask nicely and parse defensively.** Tell the model "respond with JSON only" and wrap `json.loads` in a try/except.
2. **Use the provider's JSON mode.** The API guarantees the response is valid JSON — but not that it matches your schema.
3. **Use schema-constrained output.** The API guarantees the response matches a JSON Schema you supply.

You will use all three in your career, sometimes in the same project. Pick the strongest guarantee your model and endpoint support.

## Technique 1 — Ask nicely

Put your target format in the system prompt, give one or two few-shot examples, and parse defensively:

```python
import json

response_text = client.messages.create(
    model="claude-opus-4-7",
    max_tokens=200,
    system=(
        "Reply with a single JSON object and no other text. "
        'The object must have keys "label" (one of "bug", "feature", "question") '
        'and "confidence" (a number between 0 and 1).'
    ),
    messages=[{"role": "user", "content": "The app crashes when I rotate my phone."}],
).content[0].text

payload = json.loads(response_text)  # will raise on non-JSON output — that is a bug, not a normal case
```

This is the weakest guarantee. It usually works, but you will occasionally see:

- The JSON wrapped in a ```json code fence.
- A leading "Sure, here is the JSON:" that breaks the parser.
- Perfect JSON with the wrong keys.

For production code, always validate the parsed object against your schema before using it. `pydantic` in Python and `zod` in TypeScript are the standard tools.

## Technique 2 — Provider JSON mode

Both providers support a mode that guarantees the raw output is *syntactically* valid JSON.

- OpenAI: pass `response_format={"type": "json_object"}`. See <https://platform.openai.com/docs/guides/structured-outputs#json-mode>.
- Anthropic does not expose a plain "JSON mode" flag, but its **tool use** feature covers the same ground more strictly — see Technique 3.

JSON mode does not guarantee the object matches your schema. It only guarantees `json.loads` will not throw. You still need to validate the parsed result.

## Technique 3 — Schema-constrained output

The strongest guarantee: the API refuses to emit tokens that would violate a JSON Schema you supply.

- OpenAI **Structured Outputs**: pass a `response_format` with `type: "json_schema"` and a schema with `strict: true`. Detailed reference: <https://platform.openai.com/docs/guides/structured-outputs>.
- Anthropic **tool use**: define a tool whose `input_schema` is your JSON Schema, then either force the model to call it (`tool_choice`) or extract the tool call from the response. Reference: <https://docs.anthropic.com/en/docs/build-with-claude/tool-use>.

Example — OpenAI Structured Outputs:

```python
from openai import OpenAI

client = OpenAI()

response = client.chat.completions.create(
    model="gpt-4.1",  # replace with the current recommended model
    messages=[
        {"role": "system", "content": "Classify the ticket."},
        {"role": "user", "content": "The app crashes when I rotate my phone."},
    ],
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "ticket_classification",
            "strict": True,
            "schema": {
                "type": "object",
                "additionalProperties": False,
                "required": ["label", "confidence"],
                "properties": {
                    "label": {"type": "string", "enum": ["bug", "feature", "question"]},
                    "confidence": {"type": "number", "minimum": 0, "maximum": 1},
                },
            },
        },
    },
)
```

<!-- needs-research: confirm the currently recommended default OpenAI model for structured outputs — check https://platform.openai.com/docs/models before merge. -->

Example — Anthropic tool use as a structured-output workaround:

```python
tools = [{
    "name": "record_classification",
    "description": "Record the ticket classification.",
    "input_schema": {
        "type": "object",
        "required": ["label", "confidence"],
        "properties": {
            "label": {"type": "string", "enum": ["bug", "feature", "question"]},
            "confidence": {"type": "number", "minimum": 0, "maximum": 1},
        },
    },
}]

response = client.messages.create(
    model="claude-opus-4-7",
    max_tokens=200,
    tools=tools,
    tool_choice={"type": "tool", "name": "record_classification"},
    messages=[{"role": "user", "content": "The app crashes when I rotate my phone."}],
)

for block in response.content:
    if block.type == "tool_use":
        payload = block.input  # already a dict conforming to the schema
```

## What "strict" actually guarantees

Schema-constrained modes generally guarantee:

- The output parses as JSON.
- Required fields are present.
- Values match their declared types.
- Enum fields contain only allowed values.

They do **not** guarantee:

- The values are semantically correct. The model can still classify a bug as a feature.
- The output is factually true. Constrained JSON hallucinates just as readily as free-form prose does.

Structured output is a *format* guarantee, not a *content* guarantee. Your evaluation still has to check the *values*, not just that JSON came back.

## Choosing a technique

| Situation | Use |
|---|---|
| Quick prototype, cheap task | Ask nicely + validate. |
| The model or endpoint doesn't support schemas | JSON mode + validate. |
| Production feature, values must match a shape | Schema-constrained output. |

If your provider supports schema-constrained output, prefer it. The cost of turning it on is small; the cost of a downstream parser exploding at 3am is not.

## What to remember

- Never write a regex over free-form model output when a structured-output mode is available.
- Structured output guarantees *shape*, not *truth*. Validate values against your business rules separately.
- Even with strict mode on, catch parse errors — they should be alarms, not routine.

You now have the pieces to ship the lab: a small text classifier that takes free-form customer messages and returns typed JSON your program can act on.
