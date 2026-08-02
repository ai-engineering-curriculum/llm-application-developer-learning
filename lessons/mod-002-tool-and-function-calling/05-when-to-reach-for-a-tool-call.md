# Chapter 5 — When to reach for a tool call (and when not to)

You now know how to declare tools, drive the request/tool_use/tool_result loop, fan out parallel calls, and handle the failure modes without lying to the user. The last question this module answers is when you should use any of this at all. Tool calling is a powerful hammer, and it makes every LLM feature look nailable. It is not.

## Motivation

Every tool call is at least one extra round trip and one extra model invocation. Compared to a single-turn text generation:

- **Latency doubles or triples.** You are paying for at least two model responses instead of one, plus the wall-clock of the tool itself.
- **Cost roughly doubles.** The full prompt (system + tool schemas + history) is re-processed on every turn. Mod-004 will teach you prompt caching, which softens this — but it does not eliminate it.
- **Failure surface expands.** Every failure mode from chapter 4 now applies: malformed arguments, tool exceptions, loop exhaustion. None of these exist in a plain-generation call.

Those costs are worth paying when tools give you something plain generation cannot. When they do not, adding a tool is technical-debt-with-a-latency-tax.

## What tools genuinely add

A tool call is the right choice when you need at least one of these:

### 1. Access to information the model does not have

- Data behind an auth boundary — your database, your customer's CRM, a paid API.
- Data newer than the model's knowledge cutoff.
- Data specific to *this* user, *this* tenant, *this* session.

If the answer requires facts the model cannot know, a tool is the mechanism for delivering those facts. (Retrieval, covered in mod-005, is a specialised case of this.)

### 2. Actions with side effects

- Sending an email, booking a room, charging a card, writing to a database.
- Anything the user's world needs to remember happened.

Plain text generation cannot do these. A tool call is the only in-loop mechanism. Note carefully: the model *requests* the action; your code *decides* whether to execute it. For high-stakes side effects, insert a confirmation step. The model asked to charge $10,000 does not mean you should.

### 3. Computation the model is bad at

- Precise arithmetic, unit conversion, timezone math.
- Deterministic string operations (regex validation, exact string matching).
- Querying a spreadsheet, filtering a list, sorting a dataset.

Any deterministic task a small function does better than a probabilistic sampler is a tool candidate. The classic example is a calculator: a two-line Python function outperforms a state-of-the-art model on arithmetic by every metric.

### 4. Format enforcement (in disguise)

You already saw this in mod-001 chapter 4: on Anthropic, forcing a tool call is the primary mechanism for schema-constrained output. Even if the "tool" does nothing but record data, defining it as a tool lets you leverage the strong argument-schema guarantees.

## What tools do not add

Reach for a tool for any of these and you are usually making the system slower, more expensive, and less reliable for no benefit:

### 1. Anything the model can already do reliably

Rephrasing, summarising, translating, tone-shifting, categorising into a fixed set of buckets — these are the model's core competencies. A `summarize(text)` tool that just calls the same model again is pure overhead.

### 2. Classification you can express as structured output

If the task is "pick one of these three labels and give a confidence score", that is `response_format: json_schema` (OpenAI) or a forced tool call (Anthropic) — a *single-turn* interaction. Do not build a multi-turn tool-calling loop around a task that has no external dependencies.

### 3. Static knowledge inside the model's training window

If a well-tuned prompt gets the right answer with high reliability, adding a tool call to look up something the model already knows is a regression. Test the plain-generation baseline first. Add tools only when the baseline is not good enough.

### 4. Wrapping every step of your business logic

The temptation to define one tool per function in your codebase — `get_user`, `get_user_prefs`, `get_user_billing`, `get_user_notifications` — is real and almost always wrong. It turns the model into a slow, chatty orchestrator when a two-line SQL join would do the job in your service before you ever call the model. Do as much as you can in code; leave the model to decide only what actually requires judgment.

## A decision framework

When you are unsure whether a task warrants a tool, walk through these three questions in order:

1. **Does the answer depend on data the model does not have, or does it have a side effect?** If no, do not use a tool. Use plain generation or structured output.
2. **Is the operation better done deterministically than probabilistically?** If yes, a tool is likely the right shape. If no, ask whether you are really just avoiding tuning the prompt.
3. **Is the marginal cost — one extra round trip, one extra failure surface — acceptable for this feature's latency and reliability budget?** If not, redesign so the extra state or data is delivered *in the prompt* on the first turn, not requested by the model on a second.

If the answer to any of the first two is yes, go tool. If both are no, do not.

## Cost, latency, and reliability trade-offs

A rough model, useful for sanity checks:

- **Round trips.** A tool-calling feature averages 2 to 4 model invocations per user request (one to plan, one or more to react to tool_results, one to compose the final answer). Plain generation is 1.
- **Token bill.** Each extra invocation re-sends the full transcript and re-processes it. On long conversations this dominates. Prompt caching (mod-004) is the main mitigation.
- **P99 latency.** The slowest link in the chain sets your tail latency. If any tool has a p99 of 3 seconds, so does your feature. Watch the tail, not the mean.
- **Reliability.** Every added component is another thing that fails. A feature with one tool call is roughly one order of magnitude flakier than a plain-generation feature. A feature with three is roughly two orders of magnitude flakier. Mod-006 evals catch this in staging; monitoring catches it in production.

None of these is disqualifying. But they should be present in your head when you decide to introduce a tool.

## Boundary to `agentic-ai-developer-learning`

This module deliberately stops short of *agent* design. The distinction is worth making explicit because the industry uses "agent" for at least three different things.

- **What this module teaches.** The tool-call API surface: schemas, the request/tool_use/tool_result loop, parallel calls, failure handling. The loop is short (a handful of turns), the tools are yours, and the model is the decision-maker only for "which tool with which arguments."
- **What the peer track `agentic-ai-developer-learning` (level 20) teaches.** Multi-step planning over longer horizons — the model choosing not just the next tool but the *strategy*. ReAct-style think-act-observe loops. Multi-agent coordination and hand-offs. Agent frameworks (LangGraph, CrewAI, and their kin) that abstract the loop you built by hand in this module.
- **What comes later still.** The `ai-systems-architect` track (higher level) covers when to *not* use an agent — the same lesson this chapter delivered for tool calls, one level up.

If your workflow needs the model to plan several steps ahead, revise its plan on new information, or coordinate with another agent, you are in the peer track's territory. The tool-call primitive you learned here is a building block those systems use, but the design questions are different: how to keep an agent's plan coherent across ten turns is a very different problem from how to make one tool call not lie.

The right way to sequence the two is to finish this module first. Building the loop by hand once — with your own dispatcher, your own error handling, your own cap — is what makes agent frameworks readable later instead of magical.

## A concrete example: three variants of the same feature

Consider a support-triage feature: given a user's message, produce a label (`bug`, `feature`, `question`), a priority (`low`, `med`, `high`), and a short reasoning note.

- **Variant A — plain generation with structured output.** One turn. `response_format: json_schema` with `strict: true` (OpenAI) or a forced tool call (Anthropic). Latency: 1 round trip. Cost: 1 invocation. Failure surface: format drift (mod-001 chapter 5). This is the right default.
- **Variant B — tool call for "look up similar past tickets".** Two to three turns. The model requests a `find_similar_tickets(text)` tool, reads the result, then produces the label. Right when past-ticket context materially changes classification quality. Wrong if the model already gets the label right without it.
- **Variant C — agentic loop.** N turns. The model plans a sequence of investigations, escalates to a human on low confidence, updates its plan mid-flight. This is out of scope for the LLM Application Developer track — it lives in the peer agentic track.

If you can ship variant A and the metrics are good enough, do. If they are not, and you can identify a specific piece of context that would materially improve the answer, add exactly the tool that fetches that context — variant B. Do not skip to C unless variants A and B have both been measured and found lacking.

## Summary

- Tools are the right tool when the model needs data it does not have, has to cause a side effect, or must run a deterministic computation.
- They are the wrong tool when the task is rephrasing, classification you can express in one turn, or knowledge already inside the model.
- Every tool call adds latency, cost, and failure surface. Reach for the minimum tool set that unlocks the feature.
- Multi-step planning, ReAct loops, and multi-agent coordination live in the peer `agentic-ai-developer-learning` track. Finish this module before going there.
- Prefer the simplest variant that clears your quality bar. Plain generation → structured output → single tool call → multi-tool loop → agent. Only step down the list when the level above measurably fails.

You have finished mod-002. The next module (`mod-003-streaming-async-and-orchestration`) picks up the response wire itself — how to stream tokens as they are generated, and how to run many calls concurrently without your program becoming its own bottleneck.
