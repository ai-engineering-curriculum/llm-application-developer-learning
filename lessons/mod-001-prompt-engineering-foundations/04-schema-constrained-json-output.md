# Chapter 4 — Schema-constrained JSON output

Free-form prose is fine for a chatbot. It is a disaster for an application. If your program has to `if "yes" in response.lower()` on a model's reply, your product will break the day the model returns "Yes, that's correct" instead of "yes". This chapter is about forcing the model to produce machine-parseable output *by construction*, and about handling the failures you cannot prevent.

## Motivation

The gap between "the demo worked" and "this is a shipped feature" is almost always a parsing gap. In the demo, you eyeballed the string. In production, your program has to consume it. Any technique that raises the probability of clean JSON from 95% to 99.99% is worth every minute you spend on it, because the difference between those numbers, over a million calls, is 40,000 broken user experiences.

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

payload = json.loads(response_text)  # raises on non-JSON output — that is a bug, not a normal case
```

This is the weakest guarantee. It usually works, but you will occasionally see:

- The JSON wrapped in a ```json code fence.
- A leading "Sure, here is the JSON:" that breaks the parser.
- Perfect JSON with the wrong keys.
- Perfect JSON that hit `max_tokens` and cut off mid-object.

For production code, always **validate the parsed object against your schema** before using it. `pydantic` in Python and `zod` in TypeScript are the standard tools. Parsing checks the syntax; validation checks the shape.

## Technique 2 — Provider JSON mode

Both providers support a mode that guarantees the raw output is *syntactically* valid JSON.

- **OpenAI**: pass `response_format={"type": "json_object"}`. See <https://platform.openai.com/docs/guides/structured-outputs#json-mode>.
- **Anthropic** does not expose a plain "JSON mode" flag, but its **tool use** feature covers the same ground more strictly — see Technique 3.

JSON mode does not guarantee the object matches your schema. It only guarantees `json.loads` will not throw. You still need to validate the parsed result. You also still need to tell the model, in the prompt, what shape you want — JSON mode does not read your mind, it only prevents the model from producing non-JSON tokens.

## Technique 3 — Schema-constrained output

The strongest guarantee: the API refuses to emit tokens that would violate a JSON Schema you supply. This is what you should reach for in production whenever it is available.

- **OpenAI Structured Outputs**: pass a `response_format` with `type: "json_schema"` and a schema with `strict: true`. Detailed reference: <https://platform.openai.com/docs/guides/structured-outputs>.
- **Anthropic tool use**: define a tool whose `input_schema` is your JSON Schema, then either force the model to call it (`tool_choice`) or extract the tool call from the response. Reference: <https://docs.anthropic.com/en/docs/build-with-claude/tool-use>.

### Example — OpenAI Structured Outputs

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

Two things to watch for with OpenAI Structured Outputs:

- The `strict: true` flag is the whole point — without it you drop back to a best-effort mode. Every schema you ship should set it.
- Strict mode has schema-subset restrictions (for example, `additionalProperties` must be `false` and every property must be listed in `required`). Read the reference before you paste in a schema you generated somewhere else.

### Example — Anthropic tool use as a structured-output workaround

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

The `tool_choice={"type": "tool", "name": ...}` forces the model to call your tool rather than reply in prose. Extract the tool-use block from the response — its `input` field is your validated payload. This same shape becomes the foundation of tool calling in mod-002, so getting comfortable with it now pays dividends later.

## What "strict" actually guarantees

Schema-constrained modes generally guarantee:

- The output parses as JSON.
- Required fields are present.
- Values match their declared types.
- Enum fields contain only allowed values.

They do **not** guarantee:

- The values are semantically correct. The model can still classify a bug as a feature.
- The output is factually true. Constrained JSON hallucinates just as readily as free-form prose does — it just hallucinates in the right shape.
- The output is present. If the model refuses on safety grounds, or if the request errors out at the transport layer, you get no JSON at all — you get an exception.

Structured output is a *format* guarantee, not a *content* guarantee. Your evaluation still has to check the *values*, not just that JSON came back. Chapter 5 and mod-006 both come back to this.

## Failure cases you still have to handle

Even with strict schema mode on, a few failure shapes remain. Handle each explicitly.

### 1. The model was cut off by `max_tokens`

The stop reason is `"max_tokens"` (Anthropic) or `finish_reason == "length"` (OpenAI). The reply may be *shaped* like JSON but truncated at the byte level. In strict mode, providers try to keep the output valid, but if the schema is too large for the cap, you can still get a partial response or an outright refusal. Fix: raise the cap, or shrink the schema.

### 2. The model refused

The response arrives with a refusal message instead of a schema-conforming object. OpenAI Structured Outputs returns a `refusal` field on the message; Anthropic will produce a `text` content block explaining why it will not call the tool. Do not treat this as a parse error — treat it as a policy signal. Log it, surface a graceful message to the user, and consider whether your prompt is asking the model to produce something it should not.

### 3. The transport failed

Network error, timeout, HTTP 429, HTTP 5xx. There is no JSON to parse because there is no response. This is the case for retries with exponential backoff on **idempotent** failures (429 and 5xx), and for a fast error return on non-retryable ones. Never blindly retry a 4xx that is your fault — invalid model name, bad request body — those are bugs, not weather.

### 4. The schema was too permissive

The JSON parsed, all fields are present, all types match — and the values are still nonsense. `confidence: 0` on a case the model clearly should have been sure about. `label: "other"` on a case that plainly fits an existing category. This is not a bug in the API. This is a signal that your schema needs tighter constraints (enums, minimum/maximum, regex patterns) or that your prompt needs sharper category definitions.

## A production-ready template

Combine everything from this chapter into a defensive call pattern:

```python
try:
    response = call_model_with_strict_schema(user_input)
    payload = extract_structured_payload(response)  # tool_use block or json_schema field
    validated = MyPydanticModel.model_validate(payload)  # defence-in-depth
except SchemaRefusalError as e:
    logger.warning("Model refused", extra={"reason": e.reason})
    return graceful_refusal_response()
except RateLimitError:
    raise  # let the outer retry policy handle it
except ValidationError as e:
    logger.error("Strict mode returned invalid payload", extra={"error": str(e)})
    raise  # alarm, not routine
```

The `logger.error` on validation failure is important: with strict mode on, validation failures should be an alarm, not a normal case. If you are seeing them regularly, either strict mode is not really on (a common config typo) or your parser is looking at the wrong field of the response.

## Choosing a technique

| Situation | Use |
|---|---|
| Quick prototype, cheap task | Ask nicely + validate. |
| The model or endpoint does not support schemas | JSON mode + validate. |
| Production feature, values must match a shape | Schema-constrained output. |

If your provider supports schema-constrained output, prefer it. The cost of turning it on is small; the cost of a downstream parser exploding at 3 a.m. is not.

## Summary

- Never write a regex over free-form model output when a structured-output mode is available.
- Ask-nicely → JSON mode → schema-constrained is the sequence of increasing guarantees. Use the strongest one your endpoint supports.
- Structured output guarantees *shape*, not *truth*. Validate values against your business rules separately.
- Even with strict mode on, expect four failure shapes — token cap hit, refusal, transport error, semantically-wrong values — and handle each explicitly.
- A parse failure in strict mode is an alarm, not routine. If it is routine, something is misconfigured.

The next chapter is about the failures that get past all of this: hallucination, refusal, and format drift — and the first debugging step for each.
