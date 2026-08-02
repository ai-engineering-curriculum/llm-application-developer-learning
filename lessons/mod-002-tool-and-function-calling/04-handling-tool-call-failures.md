# Chapter 4 — Handling tool-call failures

The previous two chapters walked the happy path: the model calls the right tool with the right arguments, your code executes it, the model composes a good answer. Real systems spend a large share of their debugging time in the *other* paths. This chapter is the taxonomy of those other paths and, for each, the wrong reflex and the right handler.

## Motivation

The failure that will hurt your product most is not an exception — it is a *silently wrong answer*. The model called a tool, your code returned nonsense, the model wrote a confident paragraph around the nonsense, and the user believed it. Every failure class in this chapter has one theme: **do not let a bad tool call become a bad answer.** Return an error the model can see, cap the retries, and, above all, do not fabricate.

## The four failure classes

| # | Class | Root cause | Silent-failure risk |
|---|---|---|---|
| 1 | Malformed arguments | Model sent JSON that violates the schema or your semantic invariants. | If you paper over it (default values, blank fields), the model answers as if everything worked. |
| 2 | Unknown tool | Model requested a tool name your dispatcher does not have. | Returning an empty string looks like "tool succeeded with no data" — the model will invent a plausible answer. |
| 3 | Tool-side exception | Your code raised — bug, network flake, downstream 500. | If the exception bubbles out of the loop, your product returns a 500 too. If you swallow it, the model gets no signal and makes something up. |
| 4 | Semantically wrong tool call | The model called the wrong tool, or called the right tool for the wrong reason. | Impossible for the schema layer to detect. Guardrails and evals catch this — the loop cannot. |

We will handle the first three in this chapter. The fourth belongs to mod-006 (minimal eval).

## Class 1 — Malformed arguments

Even with strict-mode tools (Anthropic tool use, OpenAI strict function tools), you will occasionally receive arguments that violate a *semantic* rule your JSON Schema cannot express. Examples:

- The user ID is a valid string but refers to nobody.
- The date is well-formed but in the past when your tool requires a future date.
- The two arguments are individually valid but jointly nonsensical (start_date > end_date).

You also — rarely, but it happens — get syntactically invalid arguments, most often when generation hit `max_tokens` mid-JSON on the OpenAI side.

### Validate before you execute

Wrap each tool's arguments in a pydantic model (or an equivalent validator) and run it before you call the underlying function. Two rules:

1. Semantic validation lives in the validator, not in the tool body. If start_date > end_date is illegal, that is a `field_validator` on your input model. The tool implementation should be able to trust its inputs.
2. When validation fails, **return the error as a tool_result the model can read.** Do not raise.

```python
from pydantic import BaseModel, ValidationError, field_validator
from datetime import date

class BookingArgs(BaseModel):
    check_in: date
    check_out: date

    @field_validator("check_out")
    @classmethod
    def _after_check_in(cls, v, info):
        if v <= info.data.get("check_in"):
            raise ValueError("check_out must be strictly after check_in")
        return v

def run_book_room(raw_args: dict) -> str:
    try:
        args = BookingArgs.model_validate(raw_args)
    except ValidationError as exc:
        return json.dumps({"error": "invalid_arguments", "details": exc.errors()})
    return json.dumps(book_room(args))
```

The model receives the error payload as tool_result content, notices the failure, and typically re-issues the call with corrected arguments — or gives up and asks the user for clarification. Both behaviours are dramatically better than silently booking the wrong dates.

### Send errors back inside the tool_result envelope

Both providers have a specific shape for tool errors. Use it.

- **Anthropic:** on the `tool_result` content block, set `"is_error": true` and place the human-readable error in `content`. Reference: <https://docs.anthropic.com/en/docs/build-with-claude/tool-use/handling-tool-use-errors>.
- **OpenAI:** there is no dedicated error field on the `role: "tool"` message. Convention is to prefix the content with `"ERROR: "` and put a machine-readable JSON error inside — e.g. `{"error": "invalid_arguments", ...}`.

```python
# Anthropic
{
    "type": "tool_result",
    "tool_use_id": block.id,
    "content": "check_out must be strictly after check_in",
    "is_error": True,
}

# OpenAI
{
    "role": "tool",
    "tool_call_id": tool_call.id,
    "content": json.dumps({"error": "invalid_arguments", "message": "check_out must be strictly after check_in"}),
}
```

The point of using the explicit error channel is that both providers train the model to interpret those signals — a plain `{"result": null}` will not read as "error" and the model will happily paper over it.

## Class 2 — Unknown tool

A robust dispatcher is a `dict` lookup with an explicit else branch. It is not a chain of `if/elif` that falls through to `pass`.

```python
TOOL_IMPLS = {
    "get_current_weather": get_current_weather,
    "book_room": run_book_room,
}

def dispatch(name: str, raw_args: dict) -> tuple[str, bool]:
    """Returns (content, is_error)."""
    impl = TOOL_IMPLS.get(name)
    if impl is None:
        return (
            json.dumps({"error": "unknown_tool", "requested": name, "available": list(TOOL_IMPLS)}),
            True,
        )
    try:
        return (impl(raw_args), False)
    except Exception as exc:
        return (json.dumps({"error": "tool_exception", "type": type(exc).__name__, "message": str(exc)}), True)
```

Why the model would ever call a tool that does not exist:

- You removed a tool but the model saw its name in an earlier example or a stale system prompt.
- The user's phrasing pattern-matched to a hypothetical tool the model expected to see.
- You have two related tools and the model conflated them (`get_user_email` vs `get_user_details`).

In all three cases, telling the model the tool does not exist and listing the ones that do lets it recover. Silently returning nothing does not.

## Class 3 — Tool-side exception

Your tool implementation *will* throw. Network flakes, upstream 500s, permission errors, timeouts, an assertion you wrote and forgot about. The loop must not crash.

The dispatcher above already handles it — it catches, formats a structured error, and returns it as tool_result content with `is_error=True`. Two additional refinements:

### Timeouts are non-negotiable

Every tool call must have a wall-clock timeout. A tool that hangs holds the entire conversation hostage — the user sees a spinner, your worker is stuck, and the model never gets a signal. `concurrent.futures.wait(..., timeout=...)`, `asyncio.wait_for`, or a per-HTTP-request `timeout=` — pick one and use it uniformly.

```python
from concurrent.futures import TimeoutError

try:
    result = future.result(timeout=15.0)
except TimeoutError:
    result = ("tool timed out after 15s", True)
```

### Retry inside the tool, not in the loop

If a tool call fails with a retriable error (429, transient 5xx), retry inside the tool body — with a jittered backoff — before you return an error to the model. The model is not a smart retry policy: it will either give up too fast or hammer the tool inappropriately. Your tool code knows the semantics; use them.

Non-retriable failures (400s that mean the arguments were wrong, 401/403, 404) should be returned to the model immediately so it can adjust its behaviour on the next turn.

## The `max_iterations` cap

The loop from chapter 2 must be bounded. A model that mishandles an error can, in principle, request the same failing tool call repeatedly. Cap the total assistant turns per user request.

```python
MAX_TURNS = 6

for turn in range(MAX_TURNS):
    if response.stop_reason != "tool_use":
        break
    # ... run tools, send results, get next response ...
else:
    # loop exhausted without a final answer
    raise ToolLoopExhausted(
        f"Model kept calling tools after {MAX_TURNS} turns; giving up."
    )
```

Two things about this cap:

- **Fail hard when you hit it, do not "return what we have".** A truncated tool loop should be visible on your dashboards, not stitched together into a "best-effort" answer.
- **Pick the number empirically.** Too low and legitimate workflows fail; too high and buggy loops burn tokens. Log the observed depth for a week and set the cap at the 99th percentile plus a small buffer.

## Never let the loop lie

There is one anti-pattern that shows up in every codebase and is worth naming explicitly: **fabricating a tool_result when the tool failed.**

```python
# NEVER do this
try:
    result = tool(**args)
except Exception:
    result = {}  # "safe default" — actually a lie
```

The empty dict tells the model the tool succeeded and returned nothing. The model will now compose a fluent answer that treats the absence as significant ("It appears there is no data for that user"). The user believes it. You have not silenced a bug; you have upgraded it to a data-integrity issue.

If the tool failed, say so. Return an error object with `is_error: true`. Let the model see the failure and decide what to do — retry, ask for clarification, tell the user it does not know. All three are strictly better than a confident hallucination.

## Failure-handling checklist

Before you ship a tool-calling feature, work through this list:

- [ ] Every tool validates arguments with a pydantic model. Semantic invariants live in validators, not in the tool body.
- [ ] Validation failures return an error via the provider's error channel (`is_error: true` on Anthropic; `"ERROR: ..."` in the content on OpenAI).
- [ ] The dispatcher has an explicit "unknown tool" branch that lists the available tools in the error.
- [ ] Every tool body has a wall-clock timeout.
- [ ] Retriable failures retry inside the tool with jittered backoff; non-retriable failures are reported to the model immediately.
- [ ] The loop has a `MAX_TURNS` cap that raises a specific exception on exhaustion.
- [ ] There is a metric on parse-failure rate, tool-exception rate, and loop-exhaustion rate. Regressions on any of the three page a human.
- [ ] There is no `except Exception: pass` and no "empty dict as safe default" in the code path.

## Summary

- The failure that hurts most is a silently wrong answer. Make failures visible to the model instead of papering over them.
- Validate arguments *before* the tool body runs. Semantic rules go in the validator, not in the function.
- Use each provider's explicit error signal on tool_result (`is_error: true` for Anthropic; `ERROR: ...` convention for OpenAI). A structured error payload lets the model recover; a blank result invites hallucination.
- Every tool must have a timeout. Retry retriable errors inside the tool; return non-retriable errors up to the model.
- Cap the loop with `MAX_TURNS`. Fail hard on exhaustion; never "return what we have".
- The single anti-pattern to avoid: fabricating an empty or default tool_result when the tool actually failed.

The next chapter zooms back out to the design question: should you have been using a tool call in the first place, or is the task really just prompting or structured output?
