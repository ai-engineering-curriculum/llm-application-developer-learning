# Exercise 04 — Malformed arguments and missing tools

Paired with [chapter 4 — handling tool-call failures](../04-handling-tool-call-failures.md).

**Estimated effort:** 60–90 minutes.

## Objective

Turn the happy-path loop from exercises 02–03 into one that survives — and recovers from — the three tool-call failure classes: malformed arguments, unknown tool names, and tool-side exceptions. The exercise is about giving the model enough signal to correct itself, and about **never** letting a bad tool call silently become a bad answer.

## Problem statement

Take the two-tool assistant from exercise 03 (`get_current_time`, `get_country_capital`). Introduce the following failure surfaces and confirm your loop handles each one visibly and safely:

1. **Malformed arguments.** Add a `book_room(check_in: date, check_out: date, guests: int)` tool. Its semantic invariant: `check_out > check_in` and `1 <= guests <= 8`. Send a prompt that will trip the invariant, e.g. `"Book a room for 3 nights: check in July 10, check out July 8, for 2 guests."`
2. **Unknown tool.** Add a system prompt line that hints at a tool your dispatcher does not implement (e.g. `"You may also call cancel_booking(booking_id) if the user asks."`) and send a prompt that would trigger it: `"Actually, cancel my booking ABC-123."`
3. **Tool-side exception.** Modify `get_current_time` to raise `RuntimeError("upstream 500")` when the timezone is `"Etc/Broken"`. Send `"What time is it in Etc/Broken right now?"`

For each failure, the assistant must (a) surface a controlled, model-visible error rather than a silent success, (b) let the model attempt a recovery on the next turn, and (c) never fabricate a result to cover for a failed tool.

## Requirements

1. Add a `BookingArgs` pydantic model (or equivalent validator) that enforces `check_out > check_in` and `1 <= guests <= 8`. Validate arguments before calling the tool body.
2. On validation failure, return a tool_result with the provider's explicit error channel:
   - Anthropic: `{"type": "tool_result", "tool_use_id": ..., "content": "...", "is_error": True}`
   - OpenAI: `{"role": "tool", "tool_call_id": ..., "content": "ERROR: ..."}` — with a machine-parseable JSON error inside the content string.
3. Add an explicit "unknown tool" branch to your dispatcher. Return an error listing the available tools by name.
4. Wrap every tool body in `try / except` at the dispatch layer. On exception, return an error via the same channel. **Never `except Exception: pass`.** **Never return an empty dict / null result as a "safe default".**
5. Give every tool call a wall-clock timeout of 10 seconds. On timeout, return an error via the same channel.
6. Cap the loop with `MAX_TURNS=6`. On exhaustion raise `ToolLoopExhausted` — do not return a "best-effort" answer.
7. Log: for each turn, the tool name, arguments, whether the call succeeded, and the error class if it did not.

## Requirements — the assertions

For each of the three failure prompts above, write assertions that:

- **Prompt 1 (bad date range):** the transcript contains at least one tool_result flagged as an error (with `is_error: true` on Anthropic, or `"ERROR:"` prefix on OpenAI). The final assistant text either asks the user to correct the dates or explains it cannot proceed. It does **not** confirm a booking.
- **Prompt 2 (unknown tool):** the transcript contains an error tool_result naming the requested tool and the available ones. The final assistant text tells the user cancellation is not supported. It does **not** claim to have cancelled anything.
- **Prompt 3 (tool exception):** the transcript contains an error tool_result mentioning the exception. The final assistant text says the lookup failed. It does **not** invent a timestamp.

## Starter guidance

- Anthropic error-handling reference: <https://docs.anthropic.com/en/docs/build-with-claude/tool-use/handling-tool-use-errors>
- OpenAI function-calling errors: <https://platform.openai.com/docs/guides/function-calling>
- pydantic validation: <https://docs.pydantic.dev/latest/concepts/validators/>

The correction loop is important. After you send back an error tool_result, the model usually re-issues the call with corrected arguments (or gives up gracefully). Do not short-circuit — let the loop iterate at least once so you can observe the recovery.

## Acceptance criteria

- All three assertions above pass on your assistant.
- Grepping your codebase for `except Exception: pass` returns no matches. Grepping for a default-empty-dict return (`return {}`, `return None`) in tool bodies returns no matches either.
- Injecting a tool that hangs forever produces a `TimeoutError` (or an `is_error` tool_result mentioning the timeout) within roughly 10 seconds — not a hung process.
- Setting `MAX_TURNS=2` on a prompt that requires a recovery (e.g. deliberate bad arguments on turn 1) raises `ToolLoopExhausted` cleanly. Setting `MAX_TURNS=6` allows the same prompt to recover and produce a final text answer.

## Stretch goals

- Distinguish between *retriable* and *non-retriable* tool errors. Have your tool implementations retry retriable failures (e.g. HTTP 429, 5xx) inside the tool body with a small jittered backoff, and only surface non-retriable failures to the model.
- Add a metric that counts the parse-failure rate, tool-exception rate, and loop-exhaustion rate over a batch of test prompts. Regressions on any of the three should be visible to you at a glance.
- Fuzz the loop: generate 20 prompts that plausibly stress the tools (nonsense dates, weird timezones, unicode names) and confirm none of them produce a fabricated-looking success answer.
