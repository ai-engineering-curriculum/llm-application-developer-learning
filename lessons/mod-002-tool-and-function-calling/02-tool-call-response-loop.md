# Chapter 2 — The request → tool_call → tool_result → response loop

You declared a tool. The model wants to use it. This chapter walks through the loop that turns that intent into an executed function call and, ultimately, into an answer the user reads. Get this loop right and everything else in tool calling is refinement; get it wrong and every downstream feature will feel flaky for reasons you cannot pin down.

## Motivation

Programmers new to tool calling often expect the API to *call the function* for them. It does not. The API returns a **structured request** to call the function; your program executes it; your program hands the result back on the next turn; the model then produces the user-facing reply. The whole thing is a small state machine you drive from Python.

That means every step is your responsibility: schema, execution, error handling, sending the result back in the exact shape the provider expects. If you drop any step, the model will either loop forever, silently ignore the tool, or, worst of all, answer with fabricated data.

## The four states

Every tool-calling turn — from a single user message to a single user-visible reply — walks through these four states:

```
1. REQUEST         you send: user message + tool definitions
2. TOOL_CALL       server returns: assistant message with tool_use / tool_calls block(s)
                   stop_reason is "tool_use" (Anthropic) or "tool_calls" (OpenAI)
3. TOOL_RESULT     you execute the tool locally
                   you append the assistant turn + a tool_result to the transcript
                   you send the full transcript back
4. FINAL_RESPONSE  server returns: assistant text answering the user
                   stop_reason is "end_turn" / "stop"
```

State 3 is where beginners fumble. **You must send the assistant tool-call turn back too**, not just the tool_result. The provider is stateless (see mod-001, chapter 1): it needs the transcript that produced the tool call to know what the tool_result answers.

If the model wants to call more than one tool, or wants to call another tool after seeing the first result, the loop revisits states 2 and 3 more than once. Chapter 3 covers that; this chapter stays with the one-tool-per-turn case to keep the mechanics visible.

## Anthropic — end-to-end

Here is a complete, runnable loop that answers "What's the weather in Paris?" by calling a stub `get_current_weather` tool.

```python
import anthropic

client = anthropic.Anthropic()

TOOLS = [{
    "name": "get_current_weather",
    "description": "Return the current weather for a city.",
    "input_schema": {
        "type": "object",
        "required": ["city"],
        "properties": {
            "city": {"type": "string"},
            "units": {"type": "string", "enum": ["celsius", "fahrenheit"]},
        },
    },
}]

def get_current_weather(city: str, units: str = "celsius") -> dict:
    # Real implementation would call a weather API. Stubbed for this example.
    return {"city": city, "temperature": 18, "units": units, "conditions": "cloudy"}

def run_tool(name: str, arguments: dict) -> str:
    if name == "get_current_weather":
        result = get_current_weather(**arguments)
    else:
        raise ValueError(f"Unknown tool: {name}")
    # tool_result content must be a string (or a list of content blocks). JSON is fine.
    import json
    return json.dumps(result)

# --- state 1: REQUEST -----------------------------------------------------
messages = [{"role": "user", "content": "What's the weather in Paris right now?"}]

response = client.messages.create(
    model="claude-opus-4-7",
    max_tokens=1024,
    tools=TOOLS,
    messages=messages,
)

# --- state 2: TOOL_CALL ---------------------------------------------------
while response.stop_reason == "tool_use":
    # Append the full assistant turn (text + tool_use blocks) to the transcript.
    messages.append({"role": "assistant", "content": response.content})

    # Build a single user message containing one tool_result per tool_use block.
    tool_results = []
    for block in response.content:
        if block.type == "tool_use":
            output = run_tool(block.name, block.input)
            tool_results.append({
                "type": "tool_result",
                "tool_use_id": block.id,
                "content": output,
            })

    messages.append({"role": "user", "content": tool_results})

    # --- state 3: TOOL_RESULT sent back ----------------------------------
    response = client.messages.create(
        model="claude-opus-4-7",
        max_tokens=1024,
        tools=TOOLS,
        messages=messages,
    )

# --- state 4: FINAL_RESPONSE ---------------------------------------------
final_text = next(
    block.text for block in response.content if block.type == "text"
)
print(final_text)
```

A few things to notice, each of which is a common bug source:

- **`while response.stop_reason == "tool_use"`.** The loop is not "if tool_use then handle once". The model may take the tool_result and decide it needs another tool call. Guard on the stop reason and iterate.
- **`messages.append({"role": "assistant", "content": response.content})`.** You pass the raw content list back. Anthropic content blocks are JSON-serialisable objects the SDK accepts as-is on subsequent calls.
- **`tool_result` lives inside a user message.** This trips up almost everyone the first time. There is no `role: "tool"` on Anthropic. It is a *content block type* inside a user message.
- **`tool_use_id` must match the `id` from the tool_use block.** If you swap IDs across parallel calls the API will error. Chapter 3 goes deeper.
- **You still send `tools=TOOLS` on the second call.** Tools are declared per request, not per session. Drop them from the second call and the model cannot call them again if it wants to.

Anthropic tool use reference: <https://docs.anthropic.com/en/docs/build-with-claude/tool-use>.

## OpenAI — end-to-end

The same loop, expressed in the Chat Completions dialect. Structurally identical; the wrapping is different.

```python
import json
from openai import OpenAI

client = OpenAI()

TOOLS = [{
    "type": "function",
    "function": {
        "name": "get_current_weather",
        "description": "Return the current weather for a city.",
        "parameters": {
            "type": "object",
            "additionalProperties": False,
            "required": ["city", "units"],
            "properties": {
                "city": {"type": "string"},
                "units": {"type": "string", "enum": ["celsius", "fahrenheit"]},
            },
        },
        "strict": True,
    },
}]

def get_current_weather(city: str, units: str = "celsius") -> dict:
    return {"city": city, "temperature": 18, "units": units, "conditions": "cloudy"}

messages = [{"role": "user", "content": "What's the weather in Paris right now?"}]

response = client.chat.completions.create(
    model="gpt-4.1",
    tools=TOOLS,
    messages=messages,
)

while response.choices[0].finish_reason == "tool_calls":
    assistant_msg = response.choices[0].message
    # Append the assistant turn verbatim (includes tool_calls).
    messages.append(assistant_msg.model_dump(exclude_none=True))

    for tool_call in assistant_msg.tool_calls:
        # OpenAI hands you a JSON string, not a dict. Parse it.
        args = json.loads(tool_call.function.arguments)
        if tool_call.function.name == "get_current_weather":
            output = get_current_weather(**args)
        else:
            raise ValueError(f"Unknown tool: {tool_call.function.name}")

        messages.append({
            "role": "tool",
            "tool_call_id": tool_call.id,
            "content": json.dumps(output),
        })

    response = client.chat.completions.create(
        model="gpt-4.1",
        tools=TOOLS,
        messages=messages,
    )

print(response.choices[0].message.content)
```

<!-- needs-research: confirm the currently recommended default OpenAI model for function calling as of 2026-08 — check https://platform.openai.com/docs/models. -->

Key differences from Anthropic to keep straight:

- **`finish_reason == "tool_calls"` (plural).** Even for a single tool call.
- **Tool arguments come as a JSON *string*.** `json.loads` it. Do not `eval`.
- **The tool result goes in a message with `role: "tool"`** — an actual role in the messages array, not a content block. Match it to the assistant turn with `tool_call_id`.
- **You append the whole assistant message back**, including the `tool_calls` list. Skipping it (only appending the tool result) will produce an `invalid_request_error`.

OpenAI function calling reference: <https://platform.openai.com/docs/guides/function-calling>.

## Side-by-side: the same loop in both providers

| Step | Anthropic | OpenAI |
|---|---|---|
| Stop-reason for tool call | `stop_reason == "tool_use"` | `finish_reason == "tool_calls"` |
| Tool-call block location | Content block with `type: "tool_use"` | `message.tool_calls[i]` |
| Argument shape | Already a dict (`block.input`) | JSON string (`tool_call.function.arguments`) — parse it |
| Tool-result location | Content block with `type: "tool_result"` inside a `user` message | Message with `role: "tool"` |
| ID-matching field | `tool_use_id` on the result matches `id` on the block | `tool_call_id` on the result matches `id` on the call |
| Tools re-declared each turn | Yes | Yes |

The provider dialect is superficial. The loop is not.

## Turning the loop into a helper

Copying that loop into every feature is a mistake — it drifts, and each copy accumulates its own subtle bugs. A single well-tested helper is much better:

```python
def run_tool_conversation(client, model, tools, tool_impls, messages, max_turns=6):
    """Drive the tool-call loop to termination and return the final assistant text.

    tool_impls: dict mapping tool name -> callable(**arguments) -> JSON-serialisable result.
    max_turns: hard cap on assistant turns. See discussion below.
    """
    ...
```

Two things this helper must do that a naive loop typically forgets:

- **Cap the loop.** The model can, in principle, keep asking for tools forever. A cap on assistant turns (say, 6) turns an infinite loop into a bounded failure. When you hit the cap, return an explicit error — never silently give up.
- **Log every state transition.** Log the tool name, the arguments, the result (redacted if needed), and the elapsed time. Debugging a tool loop without logs is torture. mod-001 chapter 5 warned you: log everything.

The helper is a good target for the exercises in this module. Building your own once, from scratch, will teach you more than reading an SDK's built-in loop.

## `tool_choice` — how you steer the model

Both providers expose a knob controlling whether the model may, must, or must-not call a tool this turn. You will reach for it more often than you expect.

- **Anthropic `tool_choice`:** `{"type": "auto"}` (default), `{"type": "any"}` (must call some tool), `{"type": "tool", "name": "..."}` (must call this exact tool), `{"type": "none"}` (may not call any tool).
- **OpenAI `tool_choice`:** `"auto"`, `"required"` (must call some tool), `{"type": "function", "function": {"name": "..."}}`, `"none"`.

Practical uses:

- **Force a specific tool** on turn 1 when your workflow requires it regardless of the user's phrasing (this is the same pattern used in mod-001 chapter 4 for schema-constrained output).
- **Force `none`** on the final turn of a fixed workflow so the model has no choice but to reply in text — useful when you are done using tools and want to close the conversation.

Do not use `tool_choice="required"` as a workaround for a vague description. If the model is not calling your tool when it should, the schema is the fix.

## Stop reasons you should know

The stop reason on every response tells you which state you are in. Handle each explicitly:

| Anthropic `stop_reason` | OpenAI `finish_reason` | Meaning | You should… |
|---|---|---|---|
| `tool_use` | `tool_calls` | Model wants to call one or more tools. | Execute, send results, loop. |
| `end_turn` | `stop` | Model produced a final text reply. | Return it to the user. |
| `max_tokens` | `length` | You ran out of output budget. Reply may be truncated mid-tool-call. | Raise the cap or narrow the task; do not proceed as if the reply is complete. |
| `stop_sequence` | (n/a) | You configured a stop string and the model hit it. | Design choice — you know why. |

The `max_tokens` case in particular is a trap: if generation is cut off mid-tool-call, the arguments may be malformed JSON. Do not silently retry — surface the error.

## Common bugs to prepare for

- **Dropping the assistant turn from the transcript.** Sending only the tool_result back — without the assistant's tool_use turn preceding it — produces an invalid-request error on both providers. Always append the assistant turn first.
- **Forgetting to re-send `tools=...` on subsequent calls.** The provider does not remember the tool definitions from the previous request.
- **Not iterating.** One tool_use turn is not enough. The model can chain calls. Guard on the stop reason inside a `while` loop.
- **`json.loads` on `None` or an empty string.** Only OpenAI hands you `arguments` as a string, and if `max_tokens` truncated it, the string can be invalid or empty. Wrap the parse and treat parse failures as tool-call failures (chapter 4).
- **Silent no-op on unknown tool names.** If the model calls a tool your dispatcher does not recognise, do NOT return an empty string as if nothing happened. Return an explicit error (chapter 4 covers the shape).

## Summary

- The tool-call loop is a small state machine you drive from your program: REQUEST → TOOL_CALL → TOOL_RESULT → FINAL_RESPONSE, iterated until the model produces a plain text reply.
- Anthropic returns tool_use as content blocks and expects tool_result content blocks inside a user message. OpenAI returns tool_calls on the assistant message and expects results as messages with `role: "tool"`.
- You must always append the assistant tool-call turn to the transcript before appending its result. Skipping this step is the single most common tool-calling bug.
- Cap the loop, log every transition, and never silently ignore an unknown tool name or a `max_tokens` truncation.
- Wrap the loop in one well-tested helper. Do not copy the state machine into every feature.

The next chapter handles the case where the model wants to call several tools in a single turn — parallel tool use, and the extra rules the providers add around it.
