# Exercise 02 — Tool call response loop

Paired with [chapter 2 — the request → tool_call → tool_result → response loop](../02-tool-call-response-loop.md).

**Estimated effort:** 90–120 minutes.

## Objective

Build a working tool-call loop from scratch. Drive it end-to-end on at least one real tool. The point of the exercise is to internalise the *state machine* — REQUEST → TOOL_CALL → TOOL_RESULT → FINAL_RESPONSE — so that later chapters (parallel calls, error handling) are refinements of a mental model you already own.

## Problem statement

Implement a small assistant that can look up the current UTC time in any IANA timezone via a `get_current_time` tool. The tool takes one argument, `timezone` (a string like `"Europe/Paris"`), and returns the current ISO-8601 timestamp in that timezone.

Wire the loop yourself. Do not use an SDK helper that hides the state machine — you are building the helper, not consuming one.

## Requirements

1. Implement `get_current_time(timezone: str) -> str` in plain Python using `zoneinfo` and `datetime`. If the timezone is invalid, raise a `ValueError`.
2. Implement a `run_tool_conversation(messages, tools, tool_impls, max_turns=6)` helper that:
   - Sends the initial request with tool definitions attached.
   - While the stop reason is `tool_use` / `tool_calls`, executes every requested tool, appends both the assistant turn and the tool result(s) to the transcript in the correct shape, and re-issues the request.
   - Returns the final assistant text once the stop reason is `end_turn` / `stop`.
   - Raises a specific exception (e.g. `ToolLoopExhausted`) if the loop exceeds `max_turns`.
3. Log the sequence of state transitions: stop reason, tool name, arguments, tool result (redacted if long), and elapsed time per turn.
4. Test the loop on the three prompts below and confirm it produces sensible answers:
   - `"What time is it in Paris right now?"` — one tool call, one final text answer.
   - `"What time is it in Paris and in Tokyo?"` — the model may fan out (chapter 3 goes deeper on parallel; this exercise handles either serial or parallel gracefully).
   - `"What time zone is Buenos Aires in?"` — the model should answer from its own knowledge without calling the tool. If yours calls the tool, that is fine; note that it did.

## Requirements — provider dialects

Choose one provider for this exercise. If you have credit for both, do both — the two variants will make the shared abstractions visible.

### If you use Anthropic

- The tool_result content block lives inside a `user` message, keyed by `tool_use_id`.
- Append the raw `response.content` list as the assistant turn's content — the SDK accepts it as-is on the next call.
- Reference: <https://docs.anthropic.com/en/docs/build-with-claude/tool-use>.

### If you use OpenAI

- Tool call arguments arrive as a JSON *string*. Parse with `json.loads`.
- Tool results go in a message with `role: "tool"` and `tool_call_id` matching the assistant's `tool_calls[i].id`.
- Reference: <https://platform.openai.com/docs/guides/function-calling>.

## Starter guidance

The end-to-end example in chapter 2 is close to what you need. Do **not** copy it verbatim into a project without at least these three changes:

- Replace the stub `get_current_weather` with the real `get_current_time`.
- Add the `max_turns` cap and the `ToolLoopExhausted` exception.
- Add structured logging (a per-turn dict is enough — the exercise is not asking you to set up an observability stack).

## Acceptance criteria

- Running your script on `"What time is it in Paris right now?"` prints a coherent one- or two-sentence answer containing a plausible current time in Paris.
- The transcript log shows: (1) an assistant turn with a `tool_use` / `tool_calls` block, (2) a follow-up turn with a `tool_result` / `role: "tool"` message, (3) a final assistant text turn with stop reason `end_turn` / `stop`.
- Injecting `raise RuntimeError("boom")` into `get_current_time` produces a controlled failure — not a bare traceback bubbling out of the loop. (Chapter 4 gets more precise about the *shape* of the failure; for this exercise, "not a bare traceback" is enough.)
- Setting `max_turns=1` on a prompt that requires a tool call raises `ToolLoopExhausted` rather than hanging or returning an empty answer.
- Every line of your log is enough to reproduce the tool call locally: model, tool name, arguments, result, elapsed time.

## Stretch goals

- Rewrite the same loop against the *other* provider. Push the differences behind a small `Provider` adapter class and confirm the top-level loop is identical.
- Add a `--dry-run` flag that logs what the tool call *would* be but returns a fixed synthetic response — useful when you want to test the loop mechanics without spending API credit on the tool side.
- Replace `zoneinfo` with a fake clock so tests can pin the current time. Confirm the loop still behaves correctly with a mocked tool.
