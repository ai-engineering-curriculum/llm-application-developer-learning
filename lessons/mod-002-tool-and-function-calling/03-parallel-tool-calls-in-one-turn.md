# Chapter 3 — Parallel tool calls in one turn

Chapter 2 handled one tool call per turn. Real applications rarely stop there. The user asks about weather in three cities; the model wants to fetch a user record *and* a permission check in the same turn; a summarisation flow needs to look up five documents at once. This chapter is about what happens when a single assistant response contains multiple tool_use blocks — and the small rules you must follow so the next turn does not error.

## Motivation

Parallel tool calls are the difference between a chatty, slow assistant that fires off one round-trip per lookup and a snappy one that gets everything it needs in a single fan-out. That difference shows up in latency and in cost — one extra round trip is one more full prompt re-processed by the model. But the parallel case adds two rules the single-call case did not care about, and violating either one gives you a hard-to-diagnose failure:

1. You must return **exactly one tool_result per tool_use** in the following turn — all of them, together, before the model runs again.
2. The IDs must match. Order does not matter; identity does.

Get those two rules right and parallelism is nearly free. Get them wrong and you get 400 errors, hangs, or wrong answers.

## How the model fans out

When you enable parallel tool use (both providers do so by default on capable models), the model may emit *multiple* tool_use / tool_call blocks in the same assistant turn instead of picking one and waiting for its result. It typically does this when the tool calls are independent — no output feeds into another's input — because it saves a round trip.

- **Anthropic parallel tool use:** part of the standard tool-use API on current Claude models. Reference: <https://docs.anthropic.com/en/docs/build-with-claude/tool-use/parallel-tool-use>.
- **OpenAI parallel function calling:** enabled by default; controlled with the `parallel_tool_calls` boolean on the request. Reference: <https://platform.openai.com/docs/guides/function-calling#parallel-function-calling>.

The stop reason on the response is still `tool_use` / `tool_calls`. What changed is the *count* of tool-request blocks in the assistant message.

## Handling a parallel Anthropic turn

The single-call loop from chapter 2 already handles this correctly if you wrote it well — the key is to iterate over *every* tool_use block in the response, not just the first.

```python
def collect_tool_results(response, tool_impls):
    results = []
    for block in response.content:
        if block.type != "tool_use":
            continue
        output = tool_impls[block.name](**block.input)
        results.append({
            "type": "tool_result",
            "tool_use_id": block.id,
            "content": json.dumps(output),
        })
    return results

messages.append({"role": "assistant", "content": response.content})
messages.append({"role": "user", "content": collect_tool_results(response, TOOL_IMPLS)})
```

Two things that will bite you if you skip them:

- **Include every tool_use in one user message.** Do *not* send a separate turn per tool_result. The next `messages.create` call is one request; the array of `tool_result` blocks lives in one `user` message. Splitting them across turns produces an `invalid_request_error`.
- **Order in the `content` list does not matter.** The API matches tool_use_id to id. But leaving one out **does** matter — the model will refuse to proceed or, worse, hallucinate around the missing data.

## Handling a parallel OpenAI turn

Same shape, same rules. Iterate over `assistant_msg.tool_calls` and append one `role: "tool"` message per call *before* the next `chat.completions.create`.

```python
assistant_msg = response.choices[0].message
messages.append(assistant_msg.model_dump(exclude_none=True))

for tool_call in assistant_msg.tool_calls:
    args = json.loads(tool_call.function.arguments)
    output = TOOL_IMPLS[tool_call.function.name](**args)
    messages.append({
        "role": "tool",
        "tool_call_id": tool_call.id,
        "content": json.dumps(output),
    })
```

OpenAI-specific note: the tool result messages are separate array entries, one per call. You may still send them in any order, but every `tool_call_id` from the assistant turn must appear in a subsequent tool message before the next `create` call, or the API will 400.

## Executing tool calls concurrently

The API returning multiple tool_use blocks in one turn is the model saying "these are independent — go do them in parallel." Your code should oblige. Serial execution wastes the very win parallel tool calling was designed to deliver.

```python
from concurrent.futures import ThreadPoolExecutor

def execute_all(tool_uses, tool_impls):
    with ThreadPoolExecutor(max_workers=8) as pool:
        futures = {
            block.id: pool.submit(tool_impls[block.name], **block.input)
            for block in tool_uses
        }
        return {tid: fut.result() for tid, fut in futures.items()}
```

Guidelines for concurrent execution:

- **Only parallelise pure or side-effect-safe tools.** If two tool calls both mutate the same row in a database, running them concurrently is a race condition. If the model requested them in parallel, that is the model's mistake — but you own the fallout. When in doubt, serialise the ones that touch shared state.
- **Cap concurrency.** A ThreadPoolExecutor with an unbounded pool will happily open 200 sockets if the model asks. Use a sensible max_workers and enforce per-tool timeouts.
- **Async is often cleaner.** If your tool implementations are `async def` (an HTTP client, a database call), `asyncio.gather(*coros)` is more efficient than threads. Mod-003 goes deep on this; for now, know it is an option.
- **Preserve the id → result mapping.** The tool_use_id is your primary key. If the mapping drifts (e.g., you use list indices instead of ids), you will feed the model the wrong data and it will produce a confidently wrong answer.

## Turning parallel calls off

Sometimes you want the model to make one call at a time — so you can validate the result before the next question is asked, or so a tool with expensive side effects cannot fire more than once per turn.

- **OpenAI:** pass `parallel_tool_calls=False` on the request. The model will emit at most one tool_call per turn.
- **Anthropic:** set `disable_parallel_tool_use: True` inside the `tool_choice` object. Reference: the parallel-tool-use page linked above.

The trade-off is straightforward: turning parallel off makes each turn cheaper to reason about but adds a round trip per additional tool call. Default to parallel-on unless you have a concrete reason not to.

## Full example: three cities, one turn

Here is a worked Anthropic example that will trigger parallel tool use on a capable model. Notice that the loop body is the same as chapter 2 — only the number of blocks in `response.content` changes.

```python
import anthropic, json
from concurrent.futures import ThreadPoolExecutor

client = anthropic.Anthropic()

TOOLS = [{
    "name": "get_current_weather",
    "description": "Return the current weather for a city.",
    "input_schema": {
        "type": "object",
        "required": ["city"],
        "properties": {"city": {"type": "string"}},
    },
}]

def get_current_weather(city):
    # imagine this hits an HTTP API; keep it a bit slow to make parallelism visible
    return {"city": city, "temperature": 18, "conditions": "cloudy"}

messages = [{
    "role": "user",
    "content": "Give me the current weather in Paris, Tokyo, and Buenos Aires.",
}]

response = client.messages.create(
    model="claude-opus-4-7",
    max_tokens=1024,
    tools=TOOLS,
    messages=messages,
)

while response.stop_reason == "tool_use":
    messages.append({"role": "assistant", "content": response.content})

    tool_uses = [b for b in response.content if b.type == "tool_use"]

    with ThreadPoolExecutor(max_workers=4) as pool:
        futures = {b.id: pool.submit(get_current_weather, **b.input) for b in tool_uses}
        results_by_id = {tid: fut.result() for tid, fut in futures.items()}

    messages.append({
        "role": "user",
        "content": [
            {
                "type": "tool_result",
                "tool_use_id": tid,
                "content": json.dumps(result),
            }
            for tid, result in results_by_id.items()
        ],
    })

    response = client.messages.create(
        model="claude-opus-4-7",
        max_tokens=1024,
        tools=TOOLS,
        messages=messages,
    )

print(next(b.text for b in response.content if b.type == "text"))
```

If you drop `tool_uses[0]` from the results (simulating a missing tool_result), the next API call errors out with a message about a tool_use block without a matching tool_result. That is the guard you want — silent success would be worse than a hard error.

## Sequential vs parallel: when the model splits vs bundles

The model's choice of one call vs many is not random. It bundles when it thinks the calls are independent and splits when one depends on another. Two things follow:

- **If the model bundled calls that are secretly dependent, that is a schema / prompt problem.** Either your descriptions did not signal the dependency ("call `get_user_email` only after `resolve_user_id`") or you should collapse the two tools into one that does both.
- **If the model split calls that could have been parallel, that is a modelling nudge.** You can hint in the system prompt: "When you need data for multiple items, request them all in one turn." Do not force it — the model will still serialise when it needs to.

## Common bugs to prepare for

- **Missing a tool_result.** Every tool_use in the assistant turn needs a matching tool_result before the next call. Miss one and the API rejects the request. This is a *good* failure mode; the API is protecting you.
- **Mismatched IDs.** If you swap IDs across parallel calls (result A carrying tool_use_id of call B, and vice versa), the API accepts the request but the model reads the wrong data and answers wrong. The tests you write in the exercise should catch this class of bug specifically.
- **Race conditions.** Two parallel tool calls that share mutable state produce nondeterministic bugs. Prefer pure tools; serialise the impure ones.
- **Unbounded fan-out.** A permissive prompt can convince the model to request twenty parallel calls it did not actually need. Cap `max_workers`, set per-tool timeouts, and monitor the average parallel width per turn.
- **Assuming parallel calls arrive in a specific order.** They do not. Always look up results by id.

## Summary

- The tool-call loop from chapter 2 already handles multiple tool_use blocks in one turn — provided you iterate over all of them and return exactly one matching tool_result each.
- All tool_results for one assistant turn go together in one follow-up message. Do not spread them across multiple turns.
- Match by ID, not by list order. The two providers reject missing ids explicitly; both accept mismatched ids silently.
- Execute independent tool calls concurrently (threads or async) so parallel tool use actually pays off in latency. Serialise the ones that share mutable state.
- Turn parallel off with `parallel_tool_calls=False` (OpenAI) or `disable_parallel_tool_use` in `tool_choice` (Anthropic) when a tool has side effects you cannot safely batch.

The next chapter takes the loop out of the happy path: what to do when the model calls a tool that does not exist, sends malformed arguments, or when your tool itself throws.
