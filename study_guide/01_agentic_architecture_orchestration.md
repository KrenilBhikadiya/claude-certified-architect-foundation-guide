# Domain 1: Agentic Architecture & Orchestration (27%)

> This domain is about how you structure an agent: the loop that drives it, how several agents cooperate, how you force rules that the model must never break, how you break big jobs into pieces, and how you keep or discard conversation state between runs. It is the heaviest domain (27% of the exam) because every production agent decision, from "who calls whom" to "what happens when the model ignores the prompt", lives here, and the other domains (tool design, context management, Claude Code workflows) all plug into these ideas. It leans hardest on Scenario 1 (Customer Support Resolution Agent), Scenario 3 (Multi-Agent Research System) and Scenario 4 (Developer Productivity), and shows up in Scenario 5 (CI code review decomposition).

## Contents

- [Mental model](#mental-model)
- [1.1 Design and implement agentic loops for autonomous task execution](#11-design-and-implement-agentic-loops-for-autonomous-task-execution)
- [1.2 Orchestrate multi-agent systems with coordinator-subagent patterns](#12-orchestrate-multi-agent-systems-with-coordinator-subagent-patterns)
- [1.3 Configure subagent invocation, context passing, and spawning](#13-configure-subagent-invocation-context-passing-and-spawning)
- [1.4 Implement multi-step workflows with enforcement and handoff patterns](#14-implement-multi-step-workflows-with-enforcement-and-handoff-patterns)
- [1.5 Apply Agent SDK hooks for tool call interception and data normalization](#15-apply-agent-sdk-hooks-for-tool-call-interception-and-data-normalization)
- [1.6 Design task decomposition strategies for complex workflows](#16-design-task-decomposition-strategies-for-complex-workflows)
- [1.7 Manage session state, resumption, and forking](#17-manage-session-state-resumption-and-forking)
- [Decision cheat-table](#decision-cheat-table)
- [Sources fetched](#sources-fetched)

## Mental model

An **agent** is a loop: send the conversation to Claude, look at *why* it stopped, run whatever tools it asked for, append the results, and go round again until Claude stops asking for tools. Everything else in this domain is a variation on, or a control placed around, that loop.

```text
                       +-------------------------------------------+
   user request  --->  |  coordinator agent (the "hub")            |
                       |  - decomposes the task                    |
                       |  - picks which subagents to call          |
                       |  - passes EXPLICIT context in each prompt |
                       |  - aggregates, checks gaps, re-delegates  |
                       +----+------------------+------------------+
                            |                  |                  |   (parallel: several Agent/Task
                            v                  v                  v    calls in ONE response)
                      +-----------+      +-----------+      +-----------+
                      | subagent  |      | subagent  |      | subagent  |   each has its own fresh
                      | web search|      | doc analy.|      | synthesis |   context, own tools, own prompt
                      +-----------+      +-----------+      +-----------+
                            \                  |                  /
                             \-- results flow BACK to the coordinator only (hub-and-spoke) --/

   Around every tool call:   PreToolUse hook (block / rewrite / redirect)  ->  tool runs  ->  PostToolUse hook (normalize / annotate)
   Across runs:              resume (context still valid) | fork_session (branch) | fresh session + summary (context stale)
```

Rules of thumb that decide most exam questions in this domain:

1. **The loop is driven by `stop_reason`, nothing else.** `tool_use` means run tools and continue; `end_turn` means finished. Never parse prose ("I'm done") and never make an iteration cap the *primary* stop condition.
2. **Model-driven beats hard-coded sequences for open-ended work.** Let Claude choose the next tool from context. Use fixed pipelines only when the steps are genuinely predictable.
3. **Subagents start blank.** The only thing that reaches a subagent is the prompt string you give it (plus its own system prompt and project CLAUDE.md). If it needs findings, URLs, or decisions, put them in the prompt.
4. **Hub-and-spoke: all communication goes through the coordinator.** That gives you one place for observability, error handling and information control.
5. **Parallel means one response, many tool calls.** Emit several `Task`/`Agent` calls in a single coordinator turn, not one per turn.
6. **When a rule must hold 100% of the time, enforce it in code (hooks, prerequisite gates), not in the prompt.** Prompts and few-shot examples have a non-zero failure rate. This is the single most-tested idea in the domain.
7. **Fix the root cause at the lowest effort that works.** If the report only covers visual arts, the coordinator's decomposition is wrong, not the downstream agents.
8. **Resume only when the old context is still true.** If files changed or tool results are stale, start fresh with a structured summary (or resume and explicitly tell the agent what changed).

---

## 1.1 Design and implement agentic loops for autonomous task execution

### What the exam tests

- The **loop lifecycle**: send a request to Claude, inspect `stop_reason`, execute requested tools, return the results, repeat.
- How **tool results are appended to the conversation history** so Claude can reason about the next step (the assistant message with its `tool_use` blocks, then a user message with matching `tool_result` blocks).
- The difference between **model-driven decision-making** (Claude chooses the next tool based on context) and **pre-configured decision trees or fixed tool sequences** (your code decides).
- Skills: write control flow that **continues on `tool_use` and terminates on `end_turn`**; **append tool results between iterations**; and **avoid three anti-patterns**: parsing natural-language signals to decide when to stop, using an arbitrary iteration cap as the primary stopping mechanism, and treating "the assistant produced some text" as a completion signal.

### Concepts explained

#### The contract: the model asks, your code runs

Tool use is a contract. You describe the available tools; Claude decides when to call them; the model never executes anything itself. For tools that run in your application (user-defined tools and Anthropic-schema tools like bash or text editor) the API response contains `tool_use` blocks, your code runs them, and you send the output back in `tool_result` blocks on the next request. The docs describe the canonical shape as a `while` loop keyed on `stop_reason`:

1. Send a request with your `tools` array and the user message.
2. Claude replies with `stop_reason: "tool_use"` and one or more `tool_use` blocks (each has `id`, `name`, `input`).
3. Run each tool and format the outputs as `tool_result` blocks (each carries the matching `tool_use_id`).
4. Send a new request containing the original messages, the assistant's response, and a user message holding the `tool_result` blocks.
5. Repeat while `stop_reason` is `"tool_use"`.

The loop exits on any other stop reason.

#### The stop_reason values

| `stop_reason` | Meaning | What your loop does |
| :- | :- | :- |
| `tool_use` | Claude wants one or more tools run | Execute them, append results, **continue** |
| `end_turn` | Claude finished naturally | **Stop**, use the final text |
| `max_tokens` | Output hit your `max_tokens` limit | Raise the limit or continue; a truncated `tool_use` block must be retried with a higher limit |
| `stop_sequence` | One of your `stop_sequences` was emitted | Inspect `stop_sequence` to see which |
| `pause_turn` | A *server-side* tool loop (e.g. web search) hit its iteration limit | Send the assistant content back unchanged so Claude continues |
| `refusal` | Claude declined (still HTTP 200) | Inspect `stop_details`, handle or retry on a fallback |
| `model_context_window_exceeded` | The response filled the context window | Treat as truncated |

The exam only expects you to reason about `tool_use` versus `end_turn`, but a production loop must also handle the others. Note that `stop_reason` arrives in the body of a *successful* response; real failures are HTTP 4xx/5xx. Two caveats from the stop-reasons page: a response that leaves one of *your* client `tool_use` blocks waiting never has `pause_turn` (that case is always `tool_use`), and Claude can occasionally return an *empty* `end_turn` (2-3 tokens, no content), typically right after tool results; resending that empty response unchanged does not help, so a loop that returns "the final text" should expect an empty string. (In the Agent SDK, a hook that returns `defer` ends the run with `stop_reason: "tool_deferred"` instead; see 1.5.)

#### How results are appended

Three formatting rules from the docs matter because getting them wrong either breaks the API call or silently kills parallel tool use:

- The `tool_result` message must **immediately follow** the assistant message that contained the `tool_use` blocks (no messages in between).
- In that user message the **`tool_result` blocks come first**; any extra text goes *after* all of them. Text before a result causes a 400 error.
- If Claude made several tool calls in one turn, **return all results together in one user message**. This also keeps parallel tool use working on later turns.

Each iteration therefore grows the history: assistant (with `tool_use`) then user (with `tool_result`). That growing history *is* how Claude "remembers" what it has learned and decides what to do next. If a tool fails, return the error text with `"is_error": true`. The docs recommend instructive errors ("Rate limit exceeded. Retry after 60 seconds.") rather than a bare "failed", so Claude can recover.

#### Model-driven versus pre-configured

| | Model-driven loop | Pre-configured decision tree / fixed sequence |
| :- | :- | :- |
| Who picks the next tool | Claude, from the conversation so far | Your code (`if intent == "refund": call A then B then C`) |
| Strength | Handles ambiguous, multi-issue, unpredictable requests; adapts to what tools return | Predictable, cheap, testable |
| Weakness | Probabilistic: may skip or reorder steps | Brittle; cannot handle the case you did not anticipate |
| Use when | Open-ended tasks (support resolution, research, code exploration) | Truly fixed steps (a chain of known transforms) |

The exam rewards a **hybrid**: keep the loop model-driven, but add *deterministic guardrails* around specific steps that must never be skipped (that is 1.4 and 1.5). You do not replace the loop with a decision tree just because one rule is critical.

#### The Agent SDK runs this loop for you

With the Claude Agent SDK you do not write the `while` loop. The SDK runs the same loop that powers Claude Code: Claude evaluates, requests tools, the SDK executes them and feeds results back, repeating until Claude produces a response with **no tool calls**. Then you get a final `AssistantMessage` followed by a `ResultMessage`. Key SDK facts:

- A **turn** is one pass: Claude output, tool execution, results fed back. Turns continue until Claude produces output with no tool calls.
- `max_turns` / `maxTurns` and `max_budget_usd` / `maxBudgetUsd` exist as **guard rails**. When hit, the `ResultMessage` has subtype `error_max_turns` or `error_max_budget_usd` and **no final result**. Without limits, the loop runs until Claude finishes by itself.
- Check `ResultMessage.subtype` (`success`, `error_max_turns`, `error_max_budget_usd`, `error_during_execution`, `error_max_structured_output_retries`) before reading `result`.
- The `ResultMessage` also has a `stop_reason` (for example `end_turn`, `max_tokens`, `refusal`).
- Read-only tools can run concurrently; state-changing tools run sequentially. Custom tools default to sequential unless you set `readOnlyHint` in their annotations.

**Safety cap versus primary stop condition.** The anti-pattern the exam names is using an iteration cap *as the primary stopping mechanism*. A cap or budget as a **backstop** (circuit breaker against runaway cost) is good engineering and the SDK itself recommends a budget for production. The distinction: the loop must normally end because Claude said `end_turn`; the cap only trips in pathological cases, and when it trips you should surface it (log, escalate, resume), not silently treat the half-finished work as done. (*Building effective agents* makes the same point: agents usually end on completion, but it is common to add stopping conditions "such as a maximum number of iterations" to maintain control.)

### Worked examples

#### Example 1: a complete agentic loop on the Messages API (Python)

Scenario 1 (support agent). This is the manual loop you would write when you call the API directly instead of using the Agent SDK.

```python
import json
import anthropic

client = anthropic.Anthropic()
MODEL = "claude-sonnet-5-5"          # use whichever current model ID you deploy
MAX_SAFETY_TURNS = 40                # BACKSTOP only, never the normal exit

TOOLS = [
    {
        "name": "get_customer",
        "description": "Look up a customer by email or customer ID. Returns the verified customer record. "
                       "Call this BEFORE any order or refund operation.",
        "input_schema": {
            "type": "object",
            "properties": {"email": {"type": "string"}, "customer_id": {"type": "string"}},
        },
    },
    {
        "name": "lookup_order",
        "description": "Fetch an order by order number for a verified customer.",
        "input_schema": {
            "type": "object",
            "properties": {"order_id": {"type": "string"}, "customer_id": {"type": "string"}},
            "required": ["order_id", "customer_id"],
        },
    },
    # process_refund, escalate_to_human ... defined the same way
]

def run_tool(name: str, args: dict) -> dict:
    """Dispatch to your real backend. Raise ToolError for failures."""
    ...

class ToolError(Exception):
    pass

def run_agent(user_message: str, system: str) -> str:
    messages = [{"role": "user", "content": user_message}]
    turns = 0

    while True:
        turns += 1
        if turns > MAX_SAFETY_TURNS:                       # circuit breaker, surfaced loudly
            raise RuntimeError("Agent exceeded safety turn limit; escalate / inspect transcript")

        response = client.messages.create(
            model=MODEL,
            max_tokens=4096,
            system=system,
            tools=TOOLS,
            messages=messages,
        )

        # 1. Always record the assistant turn (text + tool_use blocks) in the history.
        messages.append({"role": "assistant", "content": response.content})

        # 2. Decide what to do from stop_reason, and ONLY from stop_reason.
        if response.stop_reason == "end_turn":
            return "".join(b.text for b in response.content if b.type == "text")

        if response.stop_reason == "tool_use":
            tool_results = []
            for block in response.content:
                if block.type != "tool_use":
                    continue
                try:
                    output = run_tool(block.name, block.input)
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,                 # must match the tool_use id
                        "content": json.dumps(output),
                    })
                except ToolError as exc:
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": f"{exc}. Check the input and retry, or escalate if it keeps failing.",
                        "is_error": True,                        # Claude can read the error and adapt
                    })
            # 3. ALL results go back in ONE user message, tool_result blocks first.
            messages.append({"role": "user", "content": tool_results})
            continue

        if response.stop_reason == "max_tokens":
            # truncated: raise max_tokens and retry, or ask Claude to continue
            raise RuntimeError("Output truncated; increase max_tokens")

        if response.stop_reason == "refusal":
            return "I can't help with that request."             # or route to a fallback / human

        # pause_turn, stop_sequence, model_context_window_exceeded: handle per your design
        raise RuntimeError(f"Unhandled stop_reason: {response.stop_reason}")
```

What to notice: the only branch that *ends normally* is `end_turn`; the cap is a loud failure, not a quiet exit; every `tool_result` echoes its `tool_use_id`; and all results of one turn are sent together.

#### Example 2: the same job on the Agent SDK (no loop to write)

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

async def main():
    async for message in query(
        prompt="Customer says order #12345 arrived damaged and wants a refund.",
        options=ClaudeAgentOptions(
            mcp_servers={"support": support_server},                       # your in-process MCP server
            allowed_tools=["mcp__support__get_customer",
                           "mcp__support__lookup_order",
                           "mcp__support__process_refund",
                           "mcp__support__escalate_to_human"],
            max_turns=30,            # guard rail
            max_budget_usd=2.0,      # guard rail
        ),
    ):
        if isinstance(message, ResultMessage):
            if message.subtype == "success":
                print(message.result)
            else:
                print(f"Stopped early: {message.subtype}")   # e.g. error_max_turns

asyncio.run(main())
```

#### Bad versus good termination logic

```python
# BAD 1: parse natural language to decide we are done
if "I have completed" in response_text or "anything else" in response_text.lower():
    break

# BAD 2: the iteration cap IS the stop condition
for _ in range(5):
    response = client.messages.create(...)
    ...                      # if the task needs 6 steps it is silently cut off

# BAD 3: "the model produced text" means done
if any(b.type == "text" for b in response.content):
    break                    # Claude often writes text AND a tool_use block in the same turn

# GOOD: stop_reason is the single source of truth
if response.stop_reason == "end_turn":
    break
```

Why BAD 3 is wrong: the docs' own example response contains a `text` block ("I'll check the current weather...") *and* a `tool_use` block in the same message with `stop_reason: "tool_use"`. Text presence says nothing about completion.

### Exam traps

- **"Terminate when the response contains the phrase 'task complete'."** Natural-language parsing is brittle (the model can phrase it any way, or say it mid-task). The structured field `stop_reason` exists for exactly this.
- **"Stop after N iterations."** Acceptable only as a safety net. As the *primary* mechanism it truncates legitimate long tasks and lets short tasks run on pointlessly.
- **"Stop when the assistant message has text content."** Assistant turns routinely mix text and tool calls.
- **"Replace the loop with a hard-coded decision tree for reliability."** Overkill and brittle; the right fix for a must-happen step is a targeted enforcement hook (1.4/1.5), not abandoning model-driven control.
- **Tool results in separate user messages or with text first.** Breaks the API contract and degrades parallel tool use.
- Distractors that mention `pause_turn`, `max_tokens` etc. as the "completion" signal: only `end_turn` means the model finished its answer.

### Check yourself

1. Your loop sometimes exits early when Claude writes "Let me check that for you" before calling a tool. What is the bug and the fix?
   <details><summary>Answer</summary>The loop uses text content as a completion signal. The turn actually has `stop_reason == "tool_use"`. Branch only on `stop_reason`: continue on `tool_use`, finish on `end_turn`.</details>
2. A reviewer proposes `for i in range(10):` as the loop because "agents must be bounded". What is the better design?
   <details><summary>Answer</summary>Loop until `end_turn`, and keep a separate safety limit (turn cap, budget such as `max_turns`/`max_budget_usd`) that fails loudly. The cap is a backstop, not the stopping rule.</details>
3. Claude requests three independent lookups in one turn. How do you return the results?
   <details><summary>Answer</summary>Execute them (concurrently if they are read-only), then send one user message whose content is three `tool_result` blocks, each with its own `tool_use_id`, placed before any text.</details>

### Official docs

- [How tool use works](https://platform.claude.com/docs/en/agents-and-tools/tool-use/how-tool-use-works.md) — the contract, where tools run, the canonical `while stop_reason == "tool_use"` loop.
- [Handle tool calls](https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls.md) — `tool_use`/`tool_result` structure, ordering rules, `is_error`.
- [Parallel tool use](https://platform.claude.com/docs/en/agents-and-tools/tool-use/parallel-tool-use.md) — several tool calls per turn; one user message for all results; `disable_parallel_tool_use` in `tool_choice`.
- [Handling stop reasons](https://platform.claude.com/docs/en/build-with-claude/handling-stop-reasons.md) — the full `stop_reason` table.
- [How the agent loop works (Agent SDK)](https://code.claude.com/docs/en/agent-sdk/agent-loop.md) — turns, message types, `max_turns`, result subtypes.
- [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) — workflows versus agents; agents as "LLMs using tools based on environmental feedback in a loop"; iteration limits as a common stopping condition.

---

## 1.2 Orchestrate multi-agent systems with coordinator-subagent patterns

### What the exam tests

- **Hub-and-spoke architecture**: one coordinator manages all communication between subagents, plus error handling and information routing.
- **Isolated context**: subagents do not inherit the coordinator's conversation history automatically.
- The coordinator's jobs: **task decomposition, delegation, result aggregation, and deciding which subagents to invoke** based on query complexity.
- The **risk of overly narrow decomposition**, which leaves broad topics partially covered.
- Skills: design coordinators that **dynamically select subagents** (not always the full pipeline); **partition scope** to minimise duplication; run **iterative refinement loops** (evaluate synthesis for gaps, re-delegate targeted queries, re-synthesise until coverage is sufficient); and **route everything through the coordinator** for observability, consistent error handling and controlled information flow.

### Concepts explained

#### Hub-and-spoke (orchestrator-workers)

In Anthropic's workflow vocabulary this is the **orchestrator-workers** pattern: a central LLM dynamically breaks down a task, delegates subtasks to worker LLMs, and synthesises the results. It differs from plain parallelisation because the subtasks are *not predefined*; the orchestrator decides them from the input. The Anthropic multi-agent research system uses exactly this: a lead agent plans, spawns subagents that search in parallel with their own context windows, then synthesises, optionally spawns more, and finally hands off to a citation step.

```text
research topic ──> COORDINATOR ──> [web-search subagent]    ──┐
                      ^    |   ──> [document-analysis agent] ──┤  results return
                      |    |   ──> [synthesis subagent]      ──┤  ONLY to coordinator
                      |    └─  ──> [report subagent]         ──┘
                      └────── gaps found? re-delegate with targeted queries
```

Why route everything through the hub instead of letting subagents talk to each other?

- **Observability**: one place to log what was asked and returned.
- **Consistent error handling**: one policy for timeouts, partial results, retries.
- **Controlled information flow**: the coordinator decides what each agent sees (important for least privilege and for avoiding context bloat).
- Uncontrolled agent-to-agent chatter is hard to reason about. Anthropic's research-system write-up reports that early agents distracted each other with excessive updates, and warns that multi-agent systems show emergent behaviour (small changes in the lead agent can unpredictably change subagent behaviour). The write-up does not itself argue for or against peer links; the "route everything through the coordinator" rationale above is the exam guide's.

#### Isolation is a feature

Each subagent runs in its own conversation. Its intermediate tool calls and results stay inside; only its **final message** returns to the parent. Benefits listed in the SDK docs: context isolation, parallelisation, specialised instructions, and tool restrictions. The cost: the subagent knows only what you tell it (see 1.3).

#### The coordinator's four jobs

1. **Decompose**: turn "impact of AI on creative industries" into subtopics that *cover the space* (visual arts, music, writing, film, games...).
2. **Delegate**: choose a subagent per subtopic or source type, with explicit goals and context.
3. **Aggregate**: merge results, keep source attribution, reconcile conflicts.
4. **Decide dynamically**: a simple factual lookup might need one search subagent only; a broad survey needs several in parallel; a request that only needs summarising existing material should skip search entirely. Always routing through the full pipeline wastes cost and latency. Anthropic's write-up describes embedding rules that scale effort to query complexity in the lead's prompt (roughly: simple fact-finding, 1 agent with 3-10 tool calls; direct comparisons, 2-4 subagents; complex research, more than 10), after early versions spawned dozens of subagents (it cites 50) for simple queries.

#### Overly narrow decomposition (Sample Question 7)

If every subagent succeeds yet the final report covers only part of the topic, look at what the coordinator *assigned*. In the official sample, the coordinator split "AI in creative industries" into three visual-arts subtasks; web search, document analysis and synthesis each did their jobs correctly and the report still missed music, writing and film. **Root cause: the coordinator's decomposition**, not the downstream agents. Mitigations: instruct the coordinator to enumerate the dimensions of a topic before splitting; have it check coverage against that enumeration; add a refinement loop.

#### Partitioning to minimise duplication

Anthropic reported that vague delegations ("research the semiconductor shortage") led subagents to duplicate work: one studied an old crisis while two others covered the same current supply chain. Fix: give each subagent a **distinct slice** with explicit boundaries, either by subtopic (music vs film), by source type (academic vs news vs company filings), by time window, or by geography. Also state what *not* to cover.

#### Iterative refinement loop

```text
plan -> delegate (parallel) -> collect -> synthesise
                                    |
                      coverage check: any gaps, conflicts, uncited claims?
                          yes: delegate targeted follow-up queries -> re-synthesise (loop)
                          no / budget reached: finish
```

The coordinator itself evaluates the synthesis for gaps, re-invokes search or analysis subagents with *targeted* queries (not the whole topic again), and re-invokes synthesis until coverage is sufficient. Give the loop an explicit exit rule (coverage criteria met or a bounded number of rounds) so it terminates.

#### Error propagation in a hub-and-spoke system (Sample Question 8)

When a web-search subagent times out, the coordinator needs enough information to decide *what to do next*. The best design returns **structured error context**: failure type, the attempted query, any partial results, and possible alternatives. The three tempting alternatives are wrong for stated reasons: a generic "search unavailable" hides context; returning an empty set marked successful suppresses the failure and risks silently incomplete reports; and killing the whole workflow throws away recoverable work. Local recovery (a retry for transient errors) inside the subagent is fine, but unrecoverable failures must be reported with detail.

#### Scoped tools to avoid round-trips (Sample Question 9)

Latency problem: the synthesis agent keeps bouncing control back to the coordinator for simple fact-checks (85% of cases). The answer that wins: give the synthesis agent a narrowly scoped `verify_fact` tool for the common simple lookups and keep complex verification going through the coordinator and the search subagent. This applies **least privilege**; giving synthesis full web-search tools over-provisions it and blurs responsibilities, batching creates blocking dependencies, and speculative caching cannot predict what will need checking.

### Worked examples

#### Example 1: a coordinator system prompt that avoids narrow decomposition

```text
You are the research coordinator. For every research request:

1. DIMENSIONS FIRST. Before delegating, list the dimensions of the topic that a complete
   report must cover (for "AI in creative industries": visual arts, music, writing,
   film/video, games, advertising, plus economics, legal/IP, labour impact). Do not
   delegate until this list exists.
2. PARTITION. Assign each subagent a distinct slice (subtopic OR source type) and say
   explicitly what is out of scope for it, so slices do not overlap.
3. SCALE EFFORT. Simple fact lookup: one search subagent. Comparison: 2-4 subagents.
   Broad survey: one subagent per dimension. Skip subagents the request does not need.
4. COVERAGE CHECK. After synthesis, compare the draft against your dimension list.
   For each uncovered dimension or unsupported claim, delegate a targeted follow-up.
   Stop when every dimension is covered or after 3 refinement rounds.
5. FAILURES. If a subagent reports an error, read its failure type and partial results.
   Retry with a modified query, use an alternative source, or proceed with partial
   results and flag the gap in the final report. Never present a gap as complete.
```

#### Example 2: bad versus good decomposition (Scenario 3)

```text
Topic: "impact of AI on creative industries"

BAD (what the sample question's coordinator did)
  1. AI in digital art creation
  2. AI in graphic design
  3. AI in photography
  -> every subagent "succeeds"; report silently omits music, writing, film.

GOOD
  1. Visual arts and design (art, graphic design, photography)
  2. Music (composition, production, licensing)
  3. Writing and publishing
  4. Film, TV and animation
  5. Games and interactive media
  6. Cross-cutting: copyright/IP, labour and economic impact
  -> distinct slices, full coverage; follow-ups added after the coverage check.
```

#### Example 3: structured error from a subagent to the coordinator

```json
{
  "status": "error",
  "failure_type": "timeout",
  "attempted_query": "AI music licensing disputes 2025 lawsuits",
  "elapsed_seconds": 60,
  "partial_results": [
    {"title": "Label sues AI music startup", "url": "https://example.com/a", "snippet": "..."}
  ],
  "alternatives": [
    "retry with narrower query: 'AI music copyright lawsuit'",
    "search legal-news sources only",
    "proceed with partial results and flag the gap"
  ]
}
```

### Exam traps

- **Blaming the downstream agent** when the coordinator's logs show the narrow decomposition (Q7). If subagents did their assigned work correctly, the fault is the assignment.
- **"Let subagents communicate directly to cut latency."** Breaks observability and controlled information flow. The exam's hub-and-spoke answer routes through the coordinator; reduce round-trips with *scoped tools* (Q9), not peer links.
- **Generic errors or swallowed errors** ("search unavailable", empty result marked success) versus structured error context.
- **"Always run the full pipeline for consistency."** The guide explicitly wants dynamic selection based on query complexity.
- **Over-provisioning**: giving one agent every tool "so it never needs the coordinator".
- **Terminating the whole workflow** on one subagent failure.

### Check yourself

1. A research report on "renewable energy policy" covers only solar. Search, analysis and synthesis agents all logged success. Where do you look first?
   <details><summary>Answer</summary>The coordinator's decomposition (what subtasks it created). The downstream agents did what they were told; the assignment omitted wind, hydro, storage, grid policy and so on.</details>
2. Why should subagent A not pass its findings directly to subagent B?
   <details><summary>Answer</summary>Hub-and-spoke routes everything through the coordinator so there is one point for logging, error handling and controlling what each agent sees. The coordinator forwards what B needs, explicitly.</details>
3. A user asks "What year was the company founded?" Should the coordinator launch the full search, analysis, synthesis and report pipeline?
   <details><summary>Answer</summary>No. It should select only what the query needs (likely one search agent or a direct answer). Dynamic subagent selection by complexity is part of the coordinator's job.</details>

### Official docs

- [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) — orchestrator-workers pattern and when to use it.
- [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) — lead agent plus parallel subagents, delegation lessons, scaling effort to query complexity, early failure modes.
- [Subagents in the SDK](https://code.claude.com/docs/en/agent-sdk/subagents.md) — benefits (isolation, parallelisation, specialisation, tool restrictions).
- [Create custom subagents (Claude Code)](https://code.claude.com/docs/en/sub-agents.md) — "choose between subagents and main conversation", chaining and parallel research patterns.

---

## 1.3 Configure subagent invocation, context passing, and spawning

### What the exam tests

- The **`Task` tool** is how subagents are spawned, and `allowedTools` must include `"Task"` for a coordinator to invoke them.
- **Context must be explicit**: subagents do not automatically inherit parent context or share memory between invocations.
- **`AgentDefinition`**: description, system prompt, and tool restrictions per subagent type.
- **Fork-based session management** for exploring divergent approaches from a shared baseline (covered in depth in 1.7).
- Skills: put **complete findings from prior agents directly in the subagent prompt**; use **structured data to separate content from metadata** (URLs, document names, page numbers) to preserve attribution; **spawn parallel subagents by emitting multiple `Task` calls in a single coordinator response**; and write coordinator prompts that specify **goals and quality criteria rather than step-by-step procedures**.

### Concepts explained

#### Terminology check: `Task` versus `Agent`

**The exam says `Task`; the current docs say `Agent`.** In Claude Code v2.1.63 the Task tool was renamed to Agent. The current SDK docs show `allowed_tools=["Read", "Grep", "Glob", "Agent"]`, tell you to detect subagent invocation by matching `tool_use` blocks whose `name` is `"Agent"`, and note that the tool still appears as `"Task"` in the `system:init` tools list; before v2.1.63, `tool_use` blocks also said `"Task"`. The Claude Code subagent docs add that existing `Task(...)` references in settings and agent definitions still work as aliases. So:

| | Exam wording | Current docs wording |
| :- | :- | :- |
| Tool that spawns subagents | `Task` | `Agent` (alias `Task` still accepted) |
| Coordinator permission | `allowedTools` includes `"Task"` | `allowedTools` / `allowed_tools` includes `"Agent"` |

**On the exam, answer with the guide's wording (`Task`, `allowedTools`).** In real code against a current SDK, include `"Agent"` (and when detecting invocations, match both names to be safe).

One more current-docs nuance: the agent-loop page describes `allowed_tools` / `allowedTools` as an **auto-approve** list ("Tools not listed are still available"; calls that need approval fall through to the permission mode and `canUseTool`), while `disallowed_tools` / `disallowedTools` is what blocks a tool outright. The exam guide still states the requirement as "`allowedTools` must include `Task`", so keep that as the exam answer; just do not read the current docs as saying an unlisted `Agent` tool is physically absent. To stop Claude spawning subagents (or one type, e.g. `Agent(Explore)` or `Agent(fork)`), the docs use deny rules.

#### Defining subagents: `AgentDefinition`

Programmatic definition (recommended for SDK apps) uses the `agents` option; Claude then invokes the matching subagent through the Agent/Task tool. Filesystem definitions are Markdown files with YAML frontmatter in `.claude/agents/` (project) or `~/.claude/agents/` (user); programmatically defined agents win over filesystem agents with the same name.

`AgentDefinition` fields (from the SDK subagents page):

| Field | Required | Purpose |
| :- | :- | :- |
| `description` | Yes | Natural-language statement of **when** to use this agent; Claude reads it to decide whether to delegate |
| `prompt` | Yes | The agent's **system prompt** (role, behaviour) |
| `tools` | No | Allow-list of tool names. **If omitted, the subagent inherits every tool available to subagents** |
| `disallowedTools` | No | Tools removed from its set (MCP patterns like `mcp__server__*` accepted) |
| `model` | No | Alias (`sonnet`, `opus`, `haiku`, ...), `inherit`, or a full model ID |
| `skills`, `memory`, `mcpServers` | No | Preloaded skills, memory scope, MCP servers for this agent |
| `maxTurns` | No | Turn cap; output is marked partial when reached and the agent can be resumed |
| `background` | No | Run as a non-blocking background task |
| `effort`, `permissionMode` | No | Reasoning effort and permission mode for this agent |

Per-language spelling: the field names above are the TypeScript names. In the Python SDK `AgentDefinition(...)` multi-word fields keep their camelCase spelling (the docs give `disallowedTools` and `mcpServers` as examples), unlike the snake_case `ClaudeAgentOptions` fields (`allowed_tools`, `max_turns`). `omitClaudeMd` and `autoCompactWindow` exist only in the TypeScript SDK's `AgentDefinition` (and in `--agents` JSON / frontmatter).

Tool restriction is a design tool: a `doc-reviewer` with only `Read` and `Grep` can analyse but cannot modify. A tool you leave out "isn't in the subagent's session at all". Some tools are always withheld from subagents (for instance `AskUserQuestion`), and the `Agent` tool itself is withheld at the nesting depth limit.

A concise filesystem definition:

```markdown
---
name: web-researcher
description: Searches the web for sources on an assigned subtopic. Use for any question needing current external information.
tools: WebSearch, WebFetch
model: sonnet
---

You are a web research specialist. Given a subtopic and quality criteria, find authoritative
sources, and return findings as structured JSON with claim, source_url, retrieved_at.
```

#### Subagent context is isolated: what actually reaches it

Unless it is a *fork* (see below), a subagent's context starts fresh. Per the docs, the only content passed from parent to subagent is the Agent tool's **prompt string**. A non-fork subagent receives: its own system prompt, the Agent tool's prompt, project CLAUDE.md (unless `omitClaudeMd`), and its tool definitions. It does **not** receive the parent's conversation history or tool results, the parent's system prompt, or preloaded skill content unless listed in `skills`. Consequences:

- Anything the subagent needs (file paths, error messages, earlier findings, user constraints, decisions) must be **written into the prompt**.
- There is **no shared memory** between invocations. Two parallel subagents cannot see each other's work; a later subagent does not remember an earlier one unless you resume it or pass its output. (Claude Code's optional `memory` field gives a subagent its own persistent notes directory across conversations; that is the subagent's private store, not a channel from the parent or from sibling subagents.)
- The parent gets back only the subagent's final message (which it may summarise). If you need verbatim preservation, say so in the prompt.

A **fork** is the documented exception: a subagent that inherits the full conversation so far (same system prompt, tools, model and history), shares the parent's prompt cache, and returns only its final result. Fork mode is on by default from Claude Code v2.1.302 (interactive terminal sessions: v2.1.232); a fork can be started with `/subtask` (whether or not fork mode is on), or by Claude calling the Agent tool with `subagent_type: "fork"`. A fork cannot spawn further forks. Do not confuse this with the exam's "fork-based session management" (`fork_session`), which branches a *saved session*; see 1.7.

#### Passing context with attribution intact

When search results and document analysis flow into synthesis, a flat prose blob loses where each claim came from. Pass **structured data that separates content from metadata**:

```json
{
  "findings": [
    {
      "claim": "Generative tools reduced stock-illustration commissions by roughly a third at three surveyed agencies.",
      "source_type": "web",
      "source_url": "https://example.com/survey-2025",
      "source_title": "Creative Labour Survey 2025",
      "page": null,
      "retrieved_at": "2026-10-12",
      "confidence_note": "single survey, self-reported"
    },
    {
      "claim": "Section 4 proposes opt-out registries for training data.",
      "source_type": "document",
      "document_name": "EU-AI-copyright-brief.pdf",
      "page": 17
    }
  ]
}
```

The synthesis subagent can then cite precisely, notice conflicts between sources, and the final report keeps a claim-to-source mapping. (Anthropic's write-up notes similar benefits from having subagents store outputs externally and pass lightweight references back, to reduce information loss and token overhead.)

#### Spawning in parallel

Parallelism is created by the **coordinator emitting several `Task`/`Agent` tool calls in a single response**. Because each subagent call is just a tool call, several `tool_use` blocks in one assistant turn means they can run concurrently, so independent subtasks finish in the time of the slowest rather than the sum. If the coordinator spawns one subagent, waits, then spawns the next in a later turn, the work is sequential.

Practical notes from the docs:

- Claude decides when and how many subagents to spawn; say in the coordinator prompt that independent subtasks should be launched together.
- By default in current Claude Code a spawned subagent runs in the **background** and its report arrives in a later turn; Claude waits when its next action needs the report. In `claude -p` and the SDK, Claude can wait on its own judgment or when asked.
- Limits exist: the concurrent subagent cap defaults to 20 (`CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS`), nesting depth defaults to 3 (`CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH`), and `max_budget_usd` counts subagent spend.
- **Can subagents spawn subagents?** In current Claude Code, yes by default: a subagent can spawn its own subagents up to three layers below the main conversation (so one prompt can grow into a tree). To keep the strict hub-and-spoke shape the exam describes, set the depth variable to `1`, or give that subagent an explicit `tools` list without `Agent` (or add `Agent` to its `disallowedTools`). Exam answer: all communication still goes through the coordinator.
- Parallel research costs tokens: Anthropic reports multi-agent systems using on the order of 15 times the tokens of a chat, so reserve them for high-value, parallelisable work.

#### Goals and quality criteria, not scripts

Anthropic's lessons: delegation needs an **objective, an output format, guidance on tools/sources, and clear task boundaries**; vague one-liners cause duplicated or misdirected work. The same write-up says its prompting strategy favours good heuristics over rigid rules, and the exam guide adds the design point that step-by-step scripts reduce subagent adaptability (the subagent cannot adjust when its first approach fails). So the coordinator prompt to a subagent should say *what good looks like* (goal, scope, quality bar, output schema) and let the subagent choose the *how*:

```text
BAD (procedural, brittle)
  "Step 1: search for 'AI music'. Step 2: open the first three results. Step 3: copy the
   headline of each. Step 4: return a list."

BAD (vague)
  "Research AI in music."

GOOD (goal + criteria + boundaries + format)
  "Goal: establish how generative AI is changing music production and licensing, 2023-2026.
   Scope: composition, production tooling, licensing/rights. Exclude: live performance.
   Quality bar: prefer primary sources (labels, courts, regulators, peer-reviewed work) over
   blogs; flag any claim supported by a single source; cover at least 5 distinct sources.
   Return JSON: findings[] with claim, source_url, source_title, retrieved_at, and gaps[]
   describing what you could not find."
```

### Worked examples

#### Example 1: coordinator with four subagents (Python Agent SDK)

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, AgentDefinition, ResultMessage

COORDINATOR_PROMPT = """You coordinate a research team. Decompose the request into distinct,
non-overlapping slices that together cover the topic. Launch independent slices IN PARALLEL
by issuing all the subagent calls in one response. Each subagent starts with NO knowledge of
this conversation, so every delegation prompt must contain: the goal, scope and exclusions,
quality criteria, the required output format, and any prior findings it needs (verbatim,
with source metadata). After results return, check coverage and delegate targeted follow-ups."""

agents = {
    "web-searcher": AgentDefinition(
        description="Finds and summarises web sources on an assigned subtopic.",
        prompt="You are a web research specialist. Return JSON findings with claim, source_url, retrieved_at.",
        tools=["WebSearch", "WebFetch"],
        model="sonnet",
    ),
    "doc-analyzer": AgentDefinition(
        description="Analyses supplied documents and extracts claims with page references.",
        prompt="You analyse documents. Return JSON findings with claim, document_name, page.",
        tools=["Read", "Grep", "Glob"],
    ),
    "synthesizer": AgentDefinition(
        description="Combines structured findings into a coherent, cited narrative. Use after research is collected.",
        prompt="You synthesise findings. Preserve every claim's source. Flag conflicts and gaps. "
               "Use verify_fact only for quick checks of dates, names and statistics.",
        tools=["mcp__research__verify_fact"],      # scoped tool: least privilege (Sample Q9)
    ),
    "report-writer": AgentDefinition(
        description="Formats the synthesised content into the final report.",
        prompt="You turn a synthesis into a formatted report with a references section.",
        tools=["Write"],
    ),
}

async def main():
    async for message in query(
        prompt="Produce a cited report on the impact of AI on creative industries.",
        options=ClaudeAgentOptions(
            system_prompt=COORDINATOR_PROMPT,
            agents=agents,
            mcp_servers={"research": research_server},
            # Exam wording: allowedTools must include "Task". Current docs: "Agent".
            allowed_tools=["Agent", "mcp__research__verify_fact"],
            max_budget_usd=5.0,
        ),
    ):
        if isinstance(message, ResultMessage) and message.subtype == "success":
            print(message.result)

asyncio.run(main())
```

#### Example 2: what a parallel delegation turn looks like

```text
coordinator assistant turn (ONE response, stop_reason = tool_use):
  tool_use #1  name=Task  input={subagent_type:"web-searcher", prompt:"Goal: music ... Exclude: ... Return JSON ..."}
  tool_use #2  name=Task  input={subagent_type:"web-searcher", prompt:"Goal: film/TV ... Exclude: ... Return JSON ..."}
  tool_use #3  name=Task  input={subagent_type:"doc-analyzer", prompt:"Analyse /data/eu-copyright-brief.pdf for ... "}

-> all three run concurrently, then three results return together.

NEXT coordinator turn:
  tool_use     name=Task  input={subagent_type:"synthesizer", prompt:"<ALL findings pasted here as JSON with sources> ..."}
```

The synthesiser's prompt contains the complete findings because it cannot see the earlier subagents' work.

#### Example 3: detecting subagent calls across SDK versions

```python
# Older SDKs/streams say "Task", current ones say "Agent": match both.
for block in message.content:
    if isinstance(block, ToolUseBlock) and block.name in ("Task", "Agent"):
        print("subagent:", block.input.get("subagent_type"))
```

### Exam traps

- **"The subagent can read the coordinator's conversation, so just say 'use the findings above'."** False; it sees only its prompt. Include the findings.
- **Forgetting the spawn permission.** The exam states that a coordinator without `Task` in `allowedTools` cannot invoke subagents (exam answer: `Task`; current docs: `Agent`, where `allowedTools` is described as the auto-approve list; see the terminology note above).
- **Sequential spawning across turns** when the subtasks are independent. The guide wants multiple calls in one response.
- **Passing only summaries** that drop URLs, document names, pages: attribution is lost and later citation is impossible.
- **Rigid step-by-step scripts** in the coordinator prompt, which block adaptation. Goals and quality criteria are preferred.
- **Granting `tools` omission lightly**: omitting `tools` means *all* subagent-available tools, the opposite of least privilege.
- **Assuming a resumed or later subagent remembers a previous one** without passing context.

### Check yourself

1. The synthesis subagent returns a report with no citations, though the search subagent found URLs. Most likely cause?
   <details><summary>Answer</summary>The coordinator passed prose summaries (or nothing) rather than the full structured findings with source metadata in the synthesis subagent's prompt. Subagents do not inherit context; include content plus metadata explicitly.</details>
2. You want three independent research slices to run at the same time. What must the coordinator do?
   <details><summary>Answer</summary>Emit all three `Task` (current docs: `Agent`) tool calls in a single response, not one per turn. Also make sure `Task`/`Agent` is in `allowedTools`.</details>
3. What happens if an `AgentDefinition` omits `tools`?
   <details><summary>Answer</summary>The subagent inherits every tool available to subagents, so it is not restricted. Specify an explicit list to apply least privilege.</details>

### Official docs

- [Subagents in the SDK](https://code.claude.com/docs/en/agent-sdk/subagents.md) — `agents` option, `AgentDefinition` table, what subagents inherit, detecting invocation (`Agent` vs `Task`), resuming subagents, depth/concurrency/spend caps.
- [Create custom subagents](https://code.claude.com/docs/en/sub-agents.md) — frontmatter reference, built-in subagents, tool restrictions, forks, foreground/background, `Task` renamed to `Agent` in v2.1.63.
- [Agent SDK overview](https://code.claude.com/docs/en/agent-sdk/overview.md) — capabilities (subagents, hooks, sessions, MCP).
- [Parallel tool use](https://platform.claude.com/docs/en/agents-and-tools/tool-use/parallel-tool-use.md) — several `tool_use` blocks in one assistant turn can be executed concurrently (execution order is your decision); results go back in one user message.
- [Multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) — delegation instructions, parallelisation, structured hand-offs.

---

## 1.4 Implement multi-step workflows with enforcement and handoff patterns

### What the exam tests

- **Programmatic enforcement** (hooks, prerequisite gates) versus **prompt-based guidance** for ordering steps in a workflow.
- Knowing that when **deterministic compliance** is required (for example identity verification before a financial operation), prompt instructions alone have a **non-zero failure rate**.
- **Structured handoff protocols** for mid-process escalation: customer details, root-cause analysis, recommended actions.
- Skills: implement **prerequisites that block downstream tool calls** until upstream steps have completed (block `process_refund` until `get_customer` returned a verified customer ID); **decompose multi-concern requests** into distinct items and investigate each **in parallel with shared context**, then synthesise a unified resolution; and **compile structured handoff summaries** (customer ID, root cause, refund amount, recommended action) for human agents **who cannot see the transcript**.

### Concepts explained

#### Why prompts are not enough

Sample Question 1: in 12% of cases the agent skipped `get_customer` and used only the customer's stated name for `lookup_order`, sometimes refunding the wrong account. The correct fix is a **programmatic prerequisite** that blocks `lookup_order` and `process_refund` until `get_customer` has returned a verified customer ID. The wrong answers are instructive:

| Option | Why it fails |
| :- | :- |
| Stronger system-prompt wording ("verification is mandatory") | Still probabilistic; a model can ignore or forget instructions, especially deep into a long conversation |
| Few-shot examples showing `get_customer` first | Same: raises the odds, does not guarantee them |
| Routing classifier that enables only some tools per request type | Solves **tool availability**, not **tool ordering**; both tools are legitimately available in the same request |
| **Programmatic prerequisite / gate** | Deterministic: the call literally cannot proceed |

Rule: the higher the consequence of a violation (money, identity, compliance, data deletion), the more the answer should be "enforce in code". Prompts remain the right tool for *style, judgment and soft preferences* (and for explaining to Claude *why* a call got blocked so it can recover).

#### Three places to put enforcement

1. **A `PreToolUse` hook (the gate)**: intercept the call, check state, allow or deny with a reason Claude can act on (see 1.5 for hook mechanics). Works for built-in, MCP and custom tools, and fires on every tool call Claude makes, including calls made inside subagents (but not for content inserted without a tool call, such as `@file` references; see 1.5).
2. **Inside the tool itself**: `process_refund` re-validates that the customer was verified (for example by requiring a verification token that only `get_customer` issues). Strong because it cannot be bypassed by any agent, but it only protects that one tool.
3. **In your orchestration code**: run the workflow as explicit stages and expose only the tools relevant to the current stage. Strong, but moves you toward a rigid pipeline; use it where steps really are fixed.

A good design often combines (1) or (2) for the critical constraint with a model-driven loop for everything else.

#### Prerequisite gate state

A gate needs memory: "has this session verified a customer?" Because SDK hook callbacks run **in your application process** (not in the model's context), you can keep that state in ordinary Python variables keyed by the `session_id` that every hook input carries. A `PostToolUse` hook on `get_customer` records the verified ID; a `PreToolUse` hook on the dependent tools checks it. Two details:

- Hooks matching the same event run in parallel and in non-deterministic completion order, so write each hook to act **independently**, not relying on another hook having run first. A gate that checks recorded state (set by the earlier, completed tool call) is fine because the tool result exists before the next dependent call is issued.
- Tie the check to the data, not just the sequence: require that the `customer_id` argument passed to `process_refund` equals the verified ID, so the agent cannot verify customer A and then refund customer B.

#### Multi-concern requests

Real support messages bundle problems: "I was charged twice for order #A, my other order #B never arrived, and I need to change my address." The pattern from the guide:

1. **Decompose** into distinct items (duplicate charge, missing delivery, address change).
2. **Investigate each in parallel using shared context**: the verified customer record is fetched once, then each item is investigated independently (parallel tool calls in one turn, or one subagent per item with the verified context pasted into each prompt, because subagents do not inherit it).
3. **Synthesise** one unified resolution that addresses every item, including items that need escalation, instead of replying to the first issue and forgetting the others.

#### Handoff to a human who has no transcript

When the agent escalates mid-process, the human agent typically sees only what the escalation tool passes. Dumping the raw transcript is unavailable and unreadable; sending "customer is unhappy, please help" forces them to start over. A good handoff is a **structured summary** with everything needed to act:

| Field | Why the human needs it |
| :- | :- |
| `customer_id` (verified), name, tier | Identify the right account immediately |
| `issue_summary` | One or two sentences, in plain language |
| `root_cause` | What the agent actually found (for example, "order shipped to old address after profile change") |
| `actions_taken` | So nothing is repeated or contradicted |
| `refund_amount` (and currency, order ID) | The money involved; also why a policy limit was hit |
| `escalation_reason` | Policy limit, customer asked for a human, policy gap, repeated failure |
| `recommended_action` | A concrete next step the human can approve |
| `customer_preferences` / sentiment notes | For example, "customer explicitly asked for a human" |
| `evidence` / references | Order IDs, ticket links, timestamps |

### Worked examples

#### Example 1: prerequisite gate with SDK hooks (blocks `process_refund` and `lookup_order` before verification)

```python
from claude_agent_sdk import ClaudeAgentOptions, HookMatcher

# Server key in mcp_servers is "support", so tools are mcp__support__<tool>.
GET_CUSTOMER   = "mcp__support__get_customer"
GATED_TOOLS    = "mcp__support__lookup_order|mcp__support__process_refund"   # exact-name list

verified_customers: dict[str, str] = {}      # session_id -> verified customer_id (your process memory)

def _extract_text(tool_response) -> str:
    # Shape of tool_response for in-process MCP tools: log it once and adapt (verify in docs).
    if isinstance(tool_response, str):
        return tool_response
    if isinstance(tool_response, dict) and isinstance(tool_response.get("content"), list):
        return "".join(p.get("text", "") for p in tool_response["content"] if isinstance(p, dict))
    return str(tool_response)

async def record_verification(input_data, tool_use_id, context):
    """PostToolUse on get_customer: remember a verified customer ID for this session."""
    import json
    try:
        record = json.loads(_extract_text(input_data["tool_response"]))
    except ValueError:
        return {}
    if record.get("verified") is True and record.get("customer_id"):
        verified_customers[input_data["session_id"]] = record["customer_id"]
    return {}

async def require_verified_customer(input_data, tool_use_id, context):
    """PreToolUse on lookup_order / process_refund: deny until get_customer verified someone."""
    session_id = input_data["session_id"]
    verified_id = verified_customers.get(session_id)
    requested_id = input_data["tool_input"].get("customer_id")

    if verified_id is None:
        reason = ("Blocked: no verified customer yet. Call get_customer with the customer's "
                  "email or customer ID first, then retry this call with the returned customer_id.")
    elif requested_id != verified_id:
        reason = (f"Blocked: customer_id {requested_id!r} does not match the verified customer "
                  f"{verified_id!r}. Use the verified customer_id.")
    else:
        return {}                                  # allowed: let normal permission flow continue

    return {
        "hookSpecificOutput": {
            "hookEventName": input_data["hook_event_name"],
            "permissionDecision": "deny",
            "permissionDecisionReason": reason,   # Claude reads this and self-corrects
        }
    }

options = ClaudeAgentOptions(
    mcp_servers={"support": support_server},
    allowed_tools=[GET_CUSTOMER, "mcp__support__lookup_order",
                   "mcp__support__process_refund", "mcp__support__escalate_to_human"],
    hooks={
        "PostToolUse": [HookMatcher(matcher=GET_CUSTOMER, hooks=[record_verification])],
        "PreToolUse":  [HookMatcher(matcher=GATED_TOOLS,  hooks=[require_verified_customer])],
    },
)
```

Resulting trace when the model tries to skip verification:

```text
assistant  : tool_use lookup_order {order_id:"12345", customer_id:"jane smith"}
PreToolUse : deny -> "Blocked: no verified customer yet. Call get_customer ..."
assistant  : tool_use get_customer {email:"jane@example.com"}
PostToolUse: records verified customer_id C-8841
assistant  : tool_use lookup_order {order_id:"12345", customer_id:"C-8841"}
PreToolUse : allow
```

The prompt can *also* tell Claude to verify first (it reduces how often the gate fires), but the gate is what guarantees the outcome.

#### Example 2: a handoff tool whose schema forces a complete summary

```python
ESCALATE_TOOL = {
    "name": "escalate_to_human",
    "description": ("Hand the case to a human agent. The human CANNOT see this conversation, so the "
                    "summary must be self-contained. Use when: policy exception or limit exceeded, "
                    "customer explicitly requests a human, or you cannot make progress."),
    "input_schema": {
        "type": "object",
        "properties": {
            "customer_id":        {"type": "string", "description": "Verified customer ID from get_customer"},
            "issue_summary":      {"type": "string"},
            "root_cause":         {"type": "string", "description": "What investigation found, not what the customer said"},
            "actions_taken":      {"type": "array", "items": {"type": "string"}},
            "refund_amount":      {"type": ["number", "null"], "description": "USD amount at issue, null if none"},
            "order_ids":          {"type": "array", "items": {"type": "string"}},
            "escalation_reason":  {"type": "string", "enum": ["policy_limit", "policy_exception",
                                   "customer_requested_human", "unable_to_resolve", "other"]},
            "recommended_action": {"type": "string"},
            "customer_notes":     {"type": "string", "description": "Preferences, sentiment, anything promised"},
        },
        "required": ["customer_id", "issue_summary", "root_cause", "actions_taken",
                     "escalation_reason", "recommended_action"],
    },
}
```

A good payload:

```json
{
  "customer_id": "C-8841",
  "issue_summary": "Customer reports order 12345 arrived damaged and wants a full refund.",
  "root_cause": "Photo evidence confirms damage in transit; carrier claim already open.",
  "actions_taken": ["Verified identity via get_customer", "Confirmed order 12345 delivered 2026-10-12",
                    "Attempted process_refund for 640.00: blocked by 500.00 policy limit"],
  "refund_amount": 640.00,
  "order_ids": ["12345"],
  "escalation_reason": "policy_limit",
  "recommended_action": "Approve full refund of 640.00; carrier claim ref CLM-7731 already filed.",
  "customer_notes": "Polite, wants resolution today. Prefers email follow-up."
}
```

A bad payload (what the exam's "no transcript access" warning is about): `{"summary": "customer upset about order, please help"}`.

#### Example 3: multi-concern request, parallel investigation with shared context

```text
Customer: "Charged twice for order A-100, order B-200 never arrived, and I need my address changed."

1. get_customer(email) -> verified C-8841        (once; shared by everything below)
2. Decompose into 3 items: [duplicate charge A-100] [missing delivery B-200] [address change]
3. ONE assistant turn with parallel calls:
     lookup_order(A-100, C-8841)   lookup_order(B-200, C-8841)   get_account_settings(C-8841)
   (or, with subagents: three Task calls, each prompt containing "Verified customer C-8841 ...")
4. Synthesise: refund duplicate charge (within limit) + replacement shipment for B-200
   + address update confirmation, in a single reply; escalate only the part that needs a human.
```

### Exam traps

- **"Strengthen the prompt / add few-shot examples"** for a *must-never-happen* ordering rule. Probabilistic; reject when the stem stresses financial or identity consequences.
- **"Add a routing classifier / enable subsets of tools"** when the real problem is ordering. It changes availability, not sequence.
- **Handoff by transcript dump** or by a one-line summary. The human has no transcript; send a structured summary with customer ID, root cause, amounts, recommended action.
- **Treating a multi-issue request as one issue**, or handling the items strictly one after another when they are independent.
- **Gate that checks "get_customer was called"** but not that it returned a *verified* result for the *same* customer.
- Over-engineering: "train a classifier" or "build a state machine for the whole conversation" when one targeted gate fixes the observed failure.

### Check yourself

1. An agent occasionally calls `delete_account` without confirming with the user first, despite clear instructions. What is the most reliable fix?
   <details><summary>Answer</summary>Enforce it programmatically: a `PreToolUse` hook (or tool-side check) that denies `delete_account` unless a recorded confirmation exists, with a reason telling Claude to obtain confirmation. Prompt wording cannot guarantee compliance.</details>
2. Why must an escalation summary include the root cause and the actions already taken?
   <details><summary>Answer</summary>The human agent has no transcript access; without these they would repeat work, contradict earlier steps, or miss why the case escalated. The structured summary is their only context.</details>
3. A customer raises three unrelated problems in one message. What workflow does the guide favour?
   <details><summary>Answer</summary>Decompose into distinct items, investigate them in parallel using shared (already verified) context, then synthesise one unified resolution.</details>

### Official docs

- [Intercept and control agent behavior with hooks (Agent SDK)](https://code.claude.com/docs/en/agent-sdk/hooks.md) — `PreToolUse` deny, matcher syntax, multiple-hook precedence (hooks run in parallel), callbacks that run in your own application process.
- [Hooks reference](https://code.claude.com/docs/en/hooks.md) — decision control, exit code 2 behaviour per event, `PostToolUse` input with `tool_response`.
- [Subagents in the SDK](https://code.claude.com/docs/en/agent-sdk/subagents.md) — what a subagent receives from its parent (only the Agent tool's prompt string), which is why shared context must be pasted into each parallel subagent's prompt.
- [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) — guardrails, checkpoints, human-in-the-loop pauses.

---

## 1.5 Apply Agent SDK hooks for tool call interception and data normalization

### What the exam tests

- **Hook patterns that intercept tool results** (such as `PostToolUse`) to transform them before the model processes them.
- **Hook patterns that intercept outgoing tool calls** to enforce compliance (for example blocking refunds above a threshold).
- The distinction between **deterministic guarantees via hooks** and **probabilistic compliance via prompts**.
- Skills: implement `PostToolUse` hooks that **normalise heterogeneous data** (Unix timestamps, ISO 8601, numeric status codes) coming from different MCP tools; implement interception hooks that **block policy-violating actions** (refunds above $500) and **redirect to alternative workflows** (human escalation); and **choose hooks over prompts** when business rules demand guaranteed compliance.

### Concepts explained

#### What a hook is

A hook is a callback (in the Agent SDK) or a handler configured in Claude Code settings (a shell command or HTTP endpoint in the common cases; the reference also lists MCP-tool, prompt and agent handlers) that runs at a defined point in the agent lifecycle. The docs list the cadence: some events fire per session (`SessionStart`, `SessionEnd`), some per turn (`UserPromptSubmit`, `Stop`), and `PreToolUse` / `PostToolUse` fire **on every tool call inside the agentic loop**. Hooks run in your application process, not inside the model's context window, so they consume no context, and they can short-circuit the loop: a `PreToolUse` hook that rejects a call prevents it from executing, and Claude receives the rejection instead.

How a hook is wired (SDK):

1. An event fires (for example Claude is about to call a tool: `PreToolUse`).
2. The SDK collects hooks registered for that event (callbacks in `options.hooks`, plus settings-file command hooks if the relevant `setting_sources` are enabled).
3. A `matcher` filters by the event's target (for tool events, the **tool name**); omit the matcher to run for every occurrence.
4. Your callback receives `(input_data, tool_use_id, context)`.
5. It returns an output object: allow, block, modify input, or add context.

Matcher rules (from the hooks reference): `"*"`, empty or omitted matches everything; a string made only of letters, digits, `_`, `-`, spaces, `,` and `|` is an **exact name or exact-name list** (`Edit|Write`); any other characters make it a **JavaScript regex, unanchored** (`^mcp__` matches all MCP tools; use `^Edit$` for a whole-string match). **Matchers match tool names only, not arguments or file paths**; to filter by argument, inspect `tool_input` inside the hook (settings-file handlers can also use a handler-level `if` field in permission-rule syntax, such as `Bash(git *)` or `Edit(*.ts)`; the SDK callback `HookMatcher` has no such field). MCP tools are named `mcp__<server>__<action>`, where `<server>` is the key you used in `mcpServers`.

#### PreToolUse: the interception point

Input includes `tool_name`, `tool_input`, `tool_use_id` plus common fields (`session_id`, `cwd`, `hook_event_name`, ...). Output goes inside `hookSpecificOutput`:

| Field | Effect |
| :- | :- |
| `permissionDecision` | `"allow"` (skip the prompt; deny and ask rules in your settings are still evaluated), `"deny"` (block the call), `"ask"` (prompt the user), `"defer"` (end the turn so the call can be resumed later; non-interactive `-p` mode only, ends the SDK run with `stop_reason: "tool_deferred"`, and is ignored when Claude makes several tool calls in one turn) |
| `permissionDecisionReason` | For `deny`, the text **shown to Claude**, so write it as an instruction that steers the next action (for `ask` it is shown to the user; for `allow`/`defer` it only goes to the debug log) |
| `updatedInput` | **Replaces the whole input object** before execution (include unchanged fields); pair with `allow` (auto-approve) or `ask` (show the user the modified input), or omit the decision and it still flows through normal permission evaluation; ignored with `defer`. Avoid having more than one hook rewrite the same tool's input: Claude Code does not merge rewrites |
| `additionalContext` | Extra string added to Claude's context alongside the result |

Top-level fields such as `systemMessage` (shown to the **user**, not the model) and `continue` also exist. When several `PreToolUse` hooks disagree, precedence is **deny, then defer, then ask, then allow**; one hook's `deny` does not stop sibling hooks from running.

**A hook can deny and explain; it does not make Claude call another tool.** (The hooks guide notes that command hooks communicate only through stdout, stderr and exit codes and cannot trigger commands or tool calls.) "Redirect to human escalation" is implemented by denying with a reason that tells Claude to call `escalate_to_human` (and optionally rewriting arguments with `updatedInput` where that makes sense). Claude then issues the escalation call in the next turn, itself subject to any hooks you registered.

Other facts worth knowing:

- Shell-command hooks (Claude Code settings) block with **exit code 2** (stderr becomes Claude's feedback) or by exiting 0 and printing JSON. **Exit code 1 is a non-blocking error**: the action still proceeds, so enforcement scripts must use `exit 2`. Choose one approach per hook: exit codes alone, or exit 0 plus JSON.
- An SDK callback `PreToolUse` hook that **times out blocks the tool call** (fail-closed). Timeouts are configurable in seconds on the `HookMatcher`. The opposite holds for settings-file `command`/`http` hooks: a timeout, a crash or a script that cannot start (for example a wrong path) is a non-blocking error and the call proceeds, unless you set `"onFailure": "block"` on the handler (Claude Code v2.1.295 or later). A policy gate must be tested so it cannot silently fail open.
- A hook's `deny` is not bypassed by permission modes: it blocks the tool even in `bypassPermissions` mode, because `PreToolUse` hooks run before the permission-mode check (hooks guide). Conversely, a hook `allow` cannot lift a deny rule from settings.
- **Hooks also fire inside subagents.** Hooks from settings files, managed policy and plugins run for a subagent's tool calls just as in the main conversation; the input then carries `agent_id` and `agent_type`. Subagent lifecycle has its own events: `SubagentStart` (cannot block, but can inject `additionalContext` into the subagent) and `SubagentStop` (same decision format as `Stop`: `decision: "block"` with a `reason` keeps the subagent running; to add context to the *parent* after a subagent returns, use a `PostToolUse` hook on the `Agent` tool). Hooks defined in a subagent's own frontmatter run only while that subagent is active.
- `PreToolUse` fires only when Claude *calls* a tool. Content inserted without a tool call (for example `@file` references) does not trigger it, and a matcher on `Write|Edit` does not cover a `Bash` command that writes the same file. Match every route to a protected resource.

#### PostToolUse: transform what the model sees

Runs **after a tool completes successfully**. Input has both `tool_input` and `tool_response`. Output fields (in `hookSpecificOutput`, with `hookEventName: "PostToolUse"`):

| Field | Effect |
| :- | :- |
| `updatedToolOutput` | **Replaces the tool's output before Claude sees it**; works for any tool in both SDKs. Must match the tool's output shape (built-in tools with the wrong shape are ignored; MCP tool output is passed through without schema validation) |
| `updatedMCPToolOutput` | Older field, replaces **MCP tool** output only; prefer `updatedToolOutput` |
| `additionalContext` | String appended to what Claude sees |
| top-level `decision: "block"` plus `reason` | Adds the reason next to the result; Claude still sees the original output |

Critical limits: **PostToolUse cannot undo anything.** The tool already ran, so files were written and requests were sent. `updatedToolOutput` only changes what *Claude sees*. To prevent or modify a call, use `PreToolUse`. Also, do not strip error details that Claude needs; it may proceed on a false assumption. For failed calls there is a separate `PostToolUseFailure` event (it does not fire for calls rejected before execution, such as unknown tools, schema-invalid input, or permission denials).

(Version note: the SDK hooks page also mentions a TypeScript-only `PostToolBatch` event that fires once after a whole parallel batch resolves. The Python SDK lacks some events, for instance `SessionStart`/`SessionEnd` callbacks; those exist as settings-file shell hooks.)

#### Why hooks beat prompts here

A refund limit stated in a prompt is a request; a `PreToolUse` hook is a rule. The model might misjudge the amount, be talked into an exception by the customer, or lose the instruction in a long conversation. The hook sees the actual `tool_input` and decides identically every time. Similarly, a prompt that says "convert timestamps to ISO 8601" depends on the model doing arithmetic correctly on each result; a `PostToolUse` normaliser does it deterministically and keeps formats consistent regardless of which MCP server returned the data.

When *not* to use a hook: for judgment calls ("is this customer frustrated enough to escalate?"), tone, and anything that needs language understanding. Those belong in the prompt, with explicit criteria and examples (Sample Question 3). Hooks are for crisp, checkable rules.

### Worked examples

#### Example 1: PreToolUse hook blocking refunds above $500 and redirecting to escalation

```python
from claude_agent_sdk import ClaudeAgentOptions, HookMatcher

REFUND_LIMIT_USD = 500.00

async def enforce_refund_limit(input_data, tool_use_id, context):
    args = input_data["tool_input"]
    amount = float(args.get("amount", 0))

    if amount > REFUND_LIMIT_USD:
        return {
            # Shown to the human operator / UI, not to the model
            "systemMessage": f"Refund of {amount:.2f} blocked by policy (limit {REFUND_LIMIT_USD:.2f}).",
            "hookSpecificOutput": {
                "hookEventName": input_data["hook_event_name"],
                "permissionDecision": "deny",
                # Shown to Claude: tells it exactly what to do instead
                "permissionDecisionReason": (
                    f"Refunds above {REFUND_LIMIT_USD:.2f} USD require human approval and cannot be "
                    "processed automatically. Do NOT retry with a smaller amount or split the refund. "
                    "Call escalate_to_human now with customer_id, order_ids, refund_amount, root_cause, "
                    "actions_taken and recommended_action, then tell the customer a specialist will follow up."
                ),
            },
        }
    return {}      # within limit: no opinion, normal permission flow continues

options = ClaudeAgentOptions(
    mcp_servers={"support": support_server},
    allowed_tools=["mcp__support__get_customer", "mcp__support__lookup_order",
                   "mcp__support__process_refund", "mcp__support__escalate_to_human"],
    hooks={"PreToolUse": [HookMatcher(matcher="mcp__support__process_refund",
                                      hooks=[enforce_refund_limit])]},
)
```

Notice the deliberate "do not split the refund" text: a pure threshold check can be gamed by several smaller refunds, so a production gate should also track cumulative refunds per order or customer in your own state (as in 1.4).

The same rule as a Claude Code settings-file command hook (`.claude/settings.json`):

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "mcp__support__process_refund",
        "hooks": [
          { "type": "command", "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/refund-limit.sh" }
        ]
      }
    ]
  }
}
```

```bash
#!/bin/bash
# refund-limit.sh : reads the hook JSON on stdin, blocks with exit 2 above the limit
INPUT=$(cat)
AMOUNT=$(echo "$INPUT" | jq -r '.tool_input.amount // 0')
if (( $(echo "$AMOUNT > 500" | bc -l) )); then
  echo "Refunds above 500 USD need human approval. Call escalate_to_human instead." >&2
  exit 2          # exit 1 would NOT block
fi
exit 0
```

#### Example 2: PostToolUse hook normalising timestamps and status codes

Three MCP tools return different dialects: the order system sends Unix seconds and numeric status codes, billing sends ISO 8601 with offsets, the shipping API sends milliseconds. The model should see one format.

```python
import json
from datetime import datetime, timezone

ORDER_STATUS = {0: "pending", 1: "processing", 2: "shipped", 3: "delivered", 4: "cancelled", 5: "refunded"}
DATE_KEYS = {"created", "created_at", "updated_at", "shipped_at", "delivered_at", "paid_at", "timestamp"}

def to_iso_utc(value):
    """Unix seconds/milliseconds or ISO 8601 (any offset) -> ISO 8601 UTC string."""
    if isinstance(value, (int, float)):
        seconds = value / 1000 if value > 1e11 else value        # ms vs s heuristic
        return datetime.fromtimestamp(seconds, tz=timezone.utc).isoformat().replace("+00:00", "Z")
    if isinstance(value, str):
        try:
            dt = datetime.fromisoformat(value.replace("Z", "+00:00"))
        except ValueError:
            return value                                         # unknown format: leave untouched
        if dt.tzinfo is None:
            dt = dt.replace(tzinfo=timezone.utc)                 # document this assumption
        return dt.astimezone(timezone.utc).isoformat().replace("+00:00", "Z")
    return value

def normalise(node):
    if isinstance(node, dict):
        out = {}
        for key, val in node.items():
            if key in DATE_KEYS:
                out[key] = to_iso_utc(val)
            elif key == "status" and isinstance(val, int):
                out[key] = ORDER_STATUS.get(val, f"unknown({val})")   # never invent: flag unknowns
            else:
                out[key] = normalise(val)
        return out
    if isinstance(node, list):
        return [normalise(item) for item in node]
    return node

async def normalise_tool_results(input_data, tool_use_id, context):
    raw = input_data["tool_response"]
    # MCP tool results arrive as content blocks; exact shape: log once and confirm (verify in docs).
    if isinstance(raw, dict) and isinstance(raw.get("content"), list):
        blocks = []
        for part in raw["content"]:
            if isinstance(part, dict) and part.get("type") == "text":
                try:
                    part = {**part, "text": json.dumps(normalise(json.loads(part["text"])))}
                except ValueError:
                    pass                                          # not JSON: leave as-is
            blocks.append(part)
        updated = {**raw, "content": blocks}
    else:
        return {}                                                 # unknown shape: do nothing rather than corrupt it

    return {
        "hookSpecificOutput": {
            "hookEventName": input_data["hook_event_name"],
            "updatedToolOutput": updated,
        }
    }

hooks = {"PostToolUse": [HookMatcher(matcher="^mcp__(orders|billing|shipping)__", hooks=[normalise_tool_results])]}
```

Before and after, as the model sees them:

```text
orders   : {"order_id":"A-100","created":1760227200,"status":2}
billing  : {"charge_id":"ch_9","paid_at":"2025-10-12T02:00:00+02:00"}

after hook:
orders   : {"order_id":"A-100","created":"2025-10-12T00:00:00Z","status":"shipped"}
billing  : {"charge_id":"ch_9","paid_at":"2025-10-12T00:00:00Z"}
```

Why a hook and not a prompt: the conversion is mechanical, so make it deterministic; it also keeps the dialect differences out of the model's context and prompt entirely. If you own the MCP servers, normalising at the source is equally valid; the hook is the answer when you cannot change them (third-party servers) or want one policy across all tools.

#### Example 3: bad versus good enforcement

```text
BAD : System prompt: "NEVER issue refunds above $500. Escalate instead."
      -> model usually complies; occasionally rounds, splits, or is persuaded. Non-zero failure rate.

GOOD: PreToolUse hook on process_refund that denies amount > 500 and instructs escalation,
      plus (optionally) the same sentence in the prompt so Claude plans for it and the gate rarely fires.
```

### Exam traps

- **Using `PostToolUse` to block a refund.** Too late: the tool already ran. Blocking and rewriting arguments is `PreToolUse`.
- **"Add the threshold to the system prompt"** where the stem demands guaranteed compliance.
- **Exit code 1** to block a shell hook; only exit 2 (or JSON `deny`) blocks.
- **Expecting the hook to call `escalate_to_human` itself.** It denies with an instruction; Claude makes the escalation call.
- **Matcher on file paths or amounts.** Matchers match tool names; inspect `tool_input` in code.
- **Hooks that only cover one route** (for example `Write|Edit` but not `Bash`).
- **Confusing `systemMessage` (user-facing) with `permissionDecisionReason` (model-facing).**
- **Using a hook for a judgment call** (sentiment, "is this complex?"). That is prompt/criteria territory.

### Check yourself

1. Which hook event and field normalise a Unix timestamp returned by an MCP tool before Claude sees it?
   <details><summary>Answer</summary>`PostToolUse`, returning `hookSpecificOutput.updatedToolOutput` (older, MCP-only: `updatedMCPToolOutput`) containing the converted value, matched on the MCP tool names.</details>
2. A `PreToolUse` hook returns `permissionDecision: "deny"` with a reason. Who sees the reason?
   <details><summary>Answer</summary>Claude (the model) sees `permissionDecisionReason` as the denial feedback and can adjust. A top-level `systemMessage` would be what the user sees.</details>
3. Why is a $500 refund cap better as a hook than as a prompt rule?
   <details><summary>Answer</summary>The hook inspects the real `amount` and decides deterministically every time; the prompt is probabilistic and can be misapplied or overridden. Financial rules need guaranteed compliance.</details>

### Official docs

- [Intercept and control agent behavior with hooks (Agent SDK)](https://code.claude.com/docs/en/agent-sdk/hooks.md) — callbacks, matchers, `permissionDecision`, `updatedInput`, `updatedToolOutput`, timeouts, troubleshooting.
- [Hooks reference](https://code.claude.com/docs/en/hooks.md) — event list, matcher patterns, exit codes, JSON output, PreToolUse/PostToolUse schemas.
- [Automate actions with hooks (guide)](https://code.claude.com/docs/en/hooks-guide.md) — shell-command examples such as blocking edits to protected files, combining results from several hooks.
- [Give Claude custom tools](https://code.claude.com/docs/en/agent-sdk/custom-tools.md) — in-process MCP tools, `mcp__server__tool` naming, `isError` results.

---

## 1.6 Design task decomposition strategies for complex workflows

### What the exam tests

- When to use **fixed sequential pipelines (prompt chaining)** versus **dynamic, adaptive decomposition** driven by intermediate findings.
- **Prompt chaining** for reviews: analyse each file individually, then run a **cross-file integration pass**.
- The value of **adaptive investigation plans** that generate subtasks from what each step discovers.
- Skills: pick the pattern that fits (chaining for predictable multi-aspect reviews; dynamic decomposition for open-ended investigation); **split large code reviews into per-file local passes plus an integration pass to avoid attention dilution**; and decompose open-ended tasks like "add comprehensive tests to a legacy codebase" by **mapping structure first, finding high-impact areas, then building a prioritised plan that adapts as dependencies appear**.

### Concepts explained

#### Anthropic's workflow patterns (the vocabulary behind the question)

From *Building effective agents*: **workflows** are systems where LLMs and tools follow predefined code paths; **agents** are systems where the LLM dynamically directs its own process. The five workflow patterns, with when each fits:

| Pattern | Idea | Use when |
| :- | :- | :- |
| **Prompt chaining** | Each call processes the previous call's output; optional programmatic checks between steps | The task cleanly breaks into fixed subtasks; you trade latency for accuracy |
| **Routing** | Classify the input, send it to a specialised handler | Distinct categories handled better separately (for example simple questions to a small model) |
| **Parallelisation** | Sectioning (independent subtasks at once) or voting (same task several times) | Speed, or multiple perspectives for confidence |
| **Orchestrator-workers** | A central LLM decides subtasks dynamically, delegates, synthesises | Subtasks cannot be predicted in advance (coding across many files, multi-source research) |
| **Evaluator-optimizer** | One LLM generates, another critiques, loop | Clear evaluation criteria and iteration measurably helps |

The article's guidance: start with the simplest solution and add complexity only when it clearly helps.

For the exam, two of these matter most:

- **Prompt chaining (fixed pipeline)**: the steps are known up front. Example: for each changed file run a local analysis, then run one integration pass over all the per-file results.
- **Dynamic decomposition (adaptive plan / orchestrator-workers)**: what to do next depends on what you find. Example: adding tests to a legacy codebase, where discovering a shared module changes the order and scope of everything else.

#### Why split a large review (attention dilution)

Sample Question 12: a PR touches 14 files; a single pass gives deep feedback on some files, shallow on others, misses obvious bugs, and contradicts itself (flagging a pattern in one file while approving the identical code elsewhere). Root cause: **attention dilution** when too many files are analysed at once. Fix: **focused passes**:

1. **Per-file local pass** (same prompt and depth for every file): finds issues inside the file; consistent depth.
2. **Integration pass**: examines cross-file concerns using the per-file findings plus the interfaces involved: data flow, contract mismatches, duplicated logic, ordering and transaction boundaries.

Why the alternatives lose: forcing developers to submit smaller PRs shifts burden without improving the system; a bigger model or context window does not fix attention quality ("larger context windows don't solve attention quality issues"); and running three passes and keeping only issues found in two of three would suppress real bugs that are caught intermittently.

The per-file passes are independent, so they can run in parallel (parallelisation: sectioning) before the sequential integration step. The overall shape is a **chain whose first link fans out**.

#### Fixed pipeline versus adaptive plan: how to choose

| Question | If yes, lean to |
| :- | :- |
| Are the steps the same every time and known in advance? | Prompt chaining |
| Are the inputs of the same shape (every PR, every invoice)? | Prompt chaining |
| Does the next step depend on what the previous step finds? | Dynamic decomposition |
| Is the scope open-ended ("improve", "understand", "add tests to everything")? | Dynamic decomposition |
| Are you tempted to hard-code a long if/else tree for edge cases? | Probably dynamic, with guardrails |

Fixed pipelines are cheaper, more predictable, and easier to test; adaptive plans handle surprises but cost more and are less predictable. Many real systems use both: a fixed outer skeleton (map, then work, then verify) with adaptive inner steps.

#### Adaptive investigation: the legacy-codebase pattern

"Add comprehensive tests to a legacy codebase" cannot be a fixed pipeline because you do not know the structure. The guide's approach:

1. **Map the structure** (directory layout, entry points, module boundaries, existing tests, coverage gaps). Use read-only exploration (in Claude Code this is where plan mode or an Explore subagent fits).
2. **Identify high-impact areas** (business-critical paths, high-churn files, code with no tests and many dependents).
3. **Create a prioritised plan** (for example: pricing engine, auth, order state machine, then utilities).
4. **Execute and adapt**: as each area is worked, record newly discovered dependencies (for instance a hidden global config needs a test seam first) and **revise the plan**. Sub-tasks are generated from findings, not from a script written in advance.

### Worked examples

#### Example 1: per-file passes plus integration pass (CI code review, Scenario 5)

```python
import asyncio
import json
import anthropic

client = anthropic.AsyncAnthropic()
MODEL = "claude-sonnet-5-5"

LOCAL_PROMPT = """Review ONE file from a pull request. Report only issues local to this file:
bugs, error handling, security, unclear logic. For each issue give file, line, severity,
explanation and a suggested fix. Also list this file's public interfaces and the external
calls it makes (for the later cross-file pass). Respond as JSON: {{"issues":[...],"interfaces":[...],"external_calls":[...]}}.

FILE: {path}
{content}"""

INTEGRATION_PROMPT = """You are reviewing the CROSS-FILE behaviour of a pull request. You are given each file's
local findings and its interfaces. Do not repeat local issues. Look only for problems that span files:
contract mismatches, inconsistent validation, data-flow errors, duplicated logic, transaction or
ordering problems. Where the same pattern is treated differently across files, flag the inconsistency.

PER-FILE RESULTS (JSON):
{per_file_results}

CHANGED INTERFACES AND CALL SITES:
{cross_refs}"""

async def review_file(path: str, content: str) -> dict:
    resp = await client.messages.create(
        model=MODEL, max_tokens=2000,
        messages=[{"role": "user", "content": LOCAL_PROMPT.format(path=path, content=content)}],
    )
    return {"path": path, "result": json.loads(resp.content[0].text)}

async def review_pr(files: dict[str, str]) -> dict:
    # Pass 1: identical focused prompt per file, in parallel (sectioning).
    per_file = await asyncio.gather(*(review_file(p, c) for p, c in files.items()))

    # Pass 2: a separate integration-focused call over the per-file output.
    integration = await client.messages.create(
        model=MODEL, max_tokens=3000,
        messages=[{"role": "user", "content": INTEGRATION_PROMPT.format(
            per_file_results=json.dumps(per_file), cross_refs=build_cross_refs(files))}],
    )
    return {"local": per_file, "integration": integration.content[0].text}
```

(`build_cross_refs` is your code: the changed signatures and their call sites. In production, validate the JSON and retry on parse failures; that belongs to Domain 4.)

#### Example 2: adaptive plan for "add tests to a legacy codebase"

```text
STEP 1  MAP        (read-only exploration)
        -> modules: billing/, orders/, auth/, utils/ ; 12% coverage ; no test seams in billing
STEP 2  PRIORITISE
        -> plan v1: [1] billing.calculate_tax (revenue-critical, 0 tests, 14 dependents)
                    [2] orders.state_machine  [3] auth.session  [4] utils.*
STEP 3  EXECUTE [1]  -> discovers calculate_tax reads a global CONFIG and hits the network
        -> NEW SUBTASKS generated from the finding:
           [1a] introduce an injectable config/clock seam   [1b] stub the rates service
        -> plan v2: 1a, 1b, 1, then 2 ... (order changed because of a discovered dependency)
STEP 4  After each area: run tests, update coverage map, re-rank remaining areas.
```

Bad versus good:

```text
BAD : fixed list written up front: "write tests for every file alphabetically."
      Misses priorities, ignores dependencies, burns effort on trivial files.
GOOD: map -> prioritise -> execute -> re-plan loop; subtasks emerge from findings.
```

### Exam traps

- **A single big pass** with "more context" or "a stronger model" for the 14-file review: attention quality, not window size, is the issue.
- **Consensus voting** (flag only if 2 of 3 runs agree) for bug-finding: suppresses true positives.
- **Pushing the problem to developers** (smaller PRs) instead of fixing the pipeline.
- **A rigid pipeline for open-ended work** (and, conversely, a fully open-ended agent for a perfectly predictable review).
- **Treating the integration pass as optional**: per-file passes alone cannot see cross-file data-flow bugs.
- **Writing the full plan before looking at the code**: the guide wants the plan to adapt as dependencies are discovered.

### Check yourself

1. A weekly job summarises 40 customer contracts with the same five questions each. Chain or adaptive?
   <details><summary>Answer</summary>A fixed pipeline (prompt chaining or parallel per-document passes plus an aggregation step): same steps, same shape, known in advance.</details>
2. Why does the integration pass receive per-file findings and interfaces instead of all raw files again?
   <details><summary>Answer</summary>Re-feeding everything recreates the attention-dilution problem. Compact local findings plus the relevant interfaces let the pass concentrate on cross-file relationships.</details>
3. You are asked to "improve performance of this legacy service". What is the first step in a good decomposition?
   <details><summary>Answer</summary>Map and measure (identify structure and hotspots), then prioritise high-impact areas, then plan adaptively as findings arrive; not a pre-written list of edits.</details>

### Official docs

- [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) — prompt chaining, routing, parallelisation, orchestrator-workers, evaluator-optimizer, "start simple".
- [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) — dynamic decomposition, scaling effort to query complexity, start wide then narrow.
- [Create custom subagents](https://code.claude.com/docs/en/sub-agents.md) — parallel research and chained-subagent patterns, when to use the main conversation instead.
- [Subagents in the SDK](https://code.claude.com/docs/en/agent-sdk/subagents.md) — running per-file or per-area analysis in parallel.

---

## 1.7 Manage session state, resumption, and forking

### What the exam tests

- **Named session resumption** with `--resume <session-name>` to continue a specific prior conversation.
- **`fork_session`** to create independent branches from a shared analysis baseline and explore divergent approaches.
- The need to **tell the agent about changes to previously analysed files** when resuming after code modifications.
- Why **starting a new session with a structured summary** can be more reliable than resuming with **stale tool results**.
- Skills: use `--resume` with names across work sessions; use `fork_session` for parallel exploration branches (comparing two testing strategies or two refactoring approaches); **choose between resuming (prior context mostly valid) and starting fresh with an injected summary (prior tool results stale)**; and **inform a resumed session about specific file changes** for targeted re-analysis rather than full re-exploration.

### Concepts explained

#### What a session is

A **session** is the conversation history the agent accumulates: your prompts, every tool call, every tool result, every response. It is written to disk (JSONL transcript files under `~/.claude/projects/<encoded-cwd>/`) so you can return to it. Crucially:

- Sessions persist the **conversation, not the filesystem**. If files changed since, the transcript still contains the *old* file contents the agent read earlier.
- Session files are **local to the machine** that created them. Resuming on another host needs a session store adapter, copying the transcript file, or (often more robust) passing the needed results as application state into a fresh session.

#### Continue, resume, fork

| Mechanism | What it does | CLI | Agent SDK |
| :- | :- | :- | :- |
| **Continue** | Reopen the most recent session in the current directory (no ID needed) | `claude --continue` | `continue_conversation=True` (Python) / `continue: true` (TypeScript) |
| **Resume** | Reopen a *specific* session | `claude --resume <name or id>`; `/resume`; `claude --resume` opens the picker | `resume=<session_id>` (SDK takes the session ID from `ResultMessage.session_id`) |
| **Fork** | New session that starts with a **copy** of the original's history; original untouched | `claude --continue --fork-session`, `claude --resume <name> --fork-session`, or `/branch [name]` inside a session | `resume=<id>` plus `fork_session=True` (Python) / `forkSession: true` (TypeScript) |
| **Fresh start with summary** | New session; you inject a structured summary | `claude` (new), paste the summary | new `query()` call with the summary in the prompt |

**Named sessions (CLI).** Give a session a name at startup with `claude -n auth-refactor` (or `--name`), during a session with `/rename auth-refactor`, or in the picker with `Ctrl+R`. Later: `claude --resume auth-refactor` or `/resume auth-refactor` resumes it directly (an exact match resumes directly; an ambiguous name opens the picker with the name pre-filled). Sessions you never name get only a generated title (which does work as a resume handle) and a default display name such as `my-app-3f` (which does **not**). The exam's `--resume <session-name>` is this CLI behaviour. In the Agent SDK, the handle is the **session ID**, though the SDK also exposes `rename_session`/`tag_session`-style helpers to attach human-readable titles.

**What resuming restores.** The full conversation (messages, tool calls, results; if it was compacted, the last compaction summary plus later messages), the model, the agent (if started with `--agent`), the permission mode in most terminal cases, and so on. It does **not** restore everything about the original launch: flags such as `--mcp-config`, `--settings`, `--plugin-dir`, `--add-dir` must be passed again.

**Fork semantics.** A fork gets its own session ID; the original's history is unchanged and can be resumed separately. Forking branches the **conversation, not the filesystem**: if a forked agent edits files, those edits are real and visible to every session working in that directory. For isolated file changes combine forks with separate git worktrees or file checkpointing. Resuming the same session in two terminals *without* forking interleaves both conversations into one transcript, which is another reason to fork when you want parallel branches.

Fork is the right tool for: comparing two testing strategies, trying two refactoring approaches, or exploring "what if we used OAuth2 instead of JWT" from a shared, expensive codebase analysis. (Do not confuse this with the *forked subagent* of 1.3, which copies the conversation into a helper that returns a result.)

#### Resume or start fresh? The key judgment

Resuming gives the agent everything it knew, which is a big advantage **only while that knowledge is still true**. The danger: the agent trusts old `Read`/`Grep`/`Bash` results in its history. If files were edited, branches switched or tests changed in between, the resumed agent may reason from stale evidence and confidently report wrong conclusions.

| Situation | Best choice | Why |
| :- | :- | :- |
| Same task, nothing changed, pick up after a break | **Resume** | Context fully valid; saves re-exploration |
| Code changed a little in a few known files | **Resume + tell it exactly what changed** | Targeted re-analysis is cheap; keeps the rest of the context |
| Many files changed, or a big refactor/merge landed | **Fresh session + structured summary** | Most tool results are stale; a clean context avoids contradictions |
| Long session that has been compacted several times or is cluttered | **Fresh session + summary** | Compaction replaced early detail with a summary; start from curated facts |
| Want to try approach B without losing approach A | **Fork** | Independent branches from the same baseline |
| Need the work on a different machine / stateless worker | **Fresh session with results passed as state** (or a session store) | Session files are local; passing results is often more robust |
| Hit `error_max_turns` or `error_max_budget_usd` | **Resume with a higher limit** | Context is still valid; the run just ran out of budget |

#### Informing a resumed session about changes

Do not assume the agent will notice. Give it a precise change notice and ask for targeted re-analysis:

```text
Since our last session these files changed:
  - src/billing/tax.py        (rewrote calculate_tax to use the new RateTable; signature unchanged)
  - src/billing/rates.py      (new file)
  - tests/test_tax.py         (deleted two obsolete tests)
Treat any earlier reads of those files as OUT OF DATE. Re-read them, update your analysis of the
billing module and the test plan accordingly, and leave conclusions about orders/ and auth/ as they are.
```

This is the "targeted re-analysis rather than full re-exploration" skill.

#### Fresh start with an injected summary

When resuming would drag in stale results, extract the durable knowledge into a compact, structured summary and begin a new session with it. A good summary contains facts and decisions, not raw tool output:

```text
CONTEXT FROM PRIOR INVESTIGATION (verified 2026-10-28, commit 3f9a1c2)
Goal: add regression tests to the billing module.
Architecture: billing/ has tax.py (pure functions), rates.py (RateTable loader), invoice.py (uses both).
Findings: calculate_tax is called from invoice.py:88 and reports/monthly.py:41; rounding is half-even.
Decisions: use pytest; stub RateTable via fixture; do not touch legacy_adapter.py.
Open items: coverage of refunds path unknown; multi-currency untested.
Known-stale: anything about invoice.py before commit 3f9a1c2.
Next step: write tests for calculate_tax edge cases, then re-read invoice.py before testing it.
```

The SDK sessions page endorses this in the cross-host context: capture the results you need (analysis output, decisions, file diffs) as application state and pass them into a fresh session's prompt; it "is often more robust than shipping transcript files around". The same logic applies when the transcript's contents are out of date.

### Worked examples

#### Example 1: named session across work sessions (CLI, Scenario 4)

```bash
# Day 1: start a named investigation
claude -n legacy-billing-audit
#   > map the billing module and list high-risk areas ... (agent reads many files)
#   (end of day: exit)

# Day 2, nothing in billing/ changed: resume by name
claude --resume legacy-billing-audit

# Day 2, a teammate merged changes to two files: resume and tell it what changed
claude --resume legacy-billing-audit
#   > FYI src/billing/tax.py and src/billing/rates.py changed (details...). Re-read them and update your risk list.
```

#### Example 2: fork to compare two strategies (Agent SDK, Python)

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

async def run(prompt, **opts):
    session_id, text = None, None
    async for m in query(prompt=prompt, options=ClaudeAgentOptions(
            allowed_tools=["Read", "Grep", "Glob"], **opts)):
        if isinstance(m, ResultMessage):
            session_id = m.session_id
            text = getattr(m, "result", None)
    return session_id, text

async def main():
    # 1. Shared baseline: one expensive analysis
    base_id, _ = await run("Analyse the payments module: structure, seams, current test gaps.")

    # 2. Two independent branches from that same baseline
    a_id, plan_a = await run("Propose a test strategy based on unit tests with heavy mocking.",
                             resume=base_id, fork_session=True)
    b_id, plan_b = await run("Propose a test strategy based on contract/integration tests.",
                             resume=base_id, fork_session=True)

    # 3. base_id, a_id, b_id are three separate sessions; resume whichever you pick
    print(plan_a, plan_b, sep="\n---\n")

asyncio.run(main())
```

Key points: both forks begin with the identical analysis context (no repeated exploration), neither branch pollutes the other, and the original `base_id` remains available. Remember forks share the **filesystem**; because these branches are read-only planning, that is safe.

#### Example 3: decision trace

```text
Q: Two days ago I analysed the auth module in session "auth-audit". Since then the team
   refactored auth/ (14 files changed). Resume or fresh?
A: Fresh start + summary. Most of the earlier Read/Grep results are stale; resuming would mix
   old and new facts. Carry over only durable knowledge (goals, decisions, known risks) in a
   structured summary, then re-explore the changed area.

Q: Same session, but only auth/token.py changed.
A: Resume by name and say: "auth/token.py changed (summary of change); re-read it and update
   conclusions that depended on it."

Q: I want to evaluate JWT versus OAuth2 starting from the same analysis.
A: Fork the analysis session twice (fork_session), one branch per approach.
```

### Exam traps

- **"Always resume; it preserves everything."** Preserves stale evidence too. The guide explicitly prefers a fresh session with a structured summary when prior tool results are stale.
- **"Always start fresh to be safe."** Wasteful when the context is mostly valid; resume (plus a change notice) is better.
- **Resuming and silently assuming the agent knows files changed.** You must tell it which files changed so it can re-analyse them.
- **Using two resumes of one session for parallel exploration.** They interleave into one transcript; use `fork_session` for independent branches.
- **Assuming a fork isolates file edits.** It branches the conversation only.
- **Confusing the exam's `--resume <name>` (CLI) with the SDK, where you pass a session ID.**
- Using `--resume` to move work to a different machine without the transcript: session files are local.

### Check yourself

1. You analysed a service yesterday in a named session. Overnight, 30 files were refactored. How do you proceed?
   <details><summary>Answer</summary>Start a new session seeded with a structured summary of the still-valid findings and decisions; do not resume, because most earlier tool results are stale. (If only a handful of files had changed, resume and name them explicitly.)</details>
2. When do you use `fork_session` instead of `--resume`?
   <details><summary>Answer</summary>When you want independent branches from the same baseline (for example comparing two refactoring approaches) without altering the original session or letting branches interleave.</details>
3. After resuming, the agent still quotes an old version of a function. What did you skip?
   <details><summary>Answer</summary>Informing it of the file change. Tell it which files changed and ask it to re-read and update only the affected analysis.</details>

### Official docs

- [Work with sessions (Agent SDK)](https://code.claude.com/docs/en/agent-sdk/sessions.md) — continue vs resume vs fork, `fork_session`/`forkSession`, session ID capture, resuming across hosts, "don't rely on session resume" advice.
- [Manage sessions (Claude Code)](https://code.claude.com/docs/en/sessions.md) — `--continue`, `--resume <name>`, `-n`, `/rename`, `/branch`, `--fork-session`, what a resumed session restores, where transcripts live.
- [How the agent loop works](https://code.claude.com/docs/en/agent-sdk/agent-loop.md) — session IDs on `ResultMessage`, compaction, recovering from `error_max_turns`.
- [Subagents in the SDK](https://code.claude.com/docs/en/agent-sdk/subagents.md) — resuming a subagent by `agentId`.

---

## Decision cheat-table

| If the question says ... | Prefer ... | Because ... |
| :- | :- | :- |
| Loop should know when the agent is done | Branch on `stop_reason`: `tool_use` continue, `end_turn` stop | Structured, reliable; text and iteration counts are not completion signals |
| A cap or budget on iterations | Keep it only as a backstop (`max_turns`, `max_budget_usd`) that fails loudly | Primary stop must be `end_turn` |
| Agent skips a required step (verify identity, confirm deletion) | Programmatic prerequisite gate (`PreToolUse` hook or tool-side check) | Prompts and few-shot examples have a non-zero failure rate |
| Rule must block an action above a threshold (refund over 500) | `PreToolUse` hook: deny with an instructive reason (and redirect to escalation) | Deterministic and happens before the action runs |
| Need to unify timestamp or status-code formats across tools | `PostToolUse` hook with `updatedToolOutput` | Mechanical transform, applied before the model sees it |
| Want to undo or prevent a tool's side effect | `PreToolUse`, not `PostToolUse` | `PostToolUse` runs after the tool already executed |
| Final report misses whole areas though every subagent succeeded | Fix the coordinator's decomposition | Subagents did what they were assigned |
| Subagent needs earlier agents' findings | Put complete findings (content plus source metadata) in its prompt | Subagents do not inherit context or share memory |
| Run independent subagents concurrently | Several `Task`/`Agent` calls in one coordinator response | Separate turns make it sequential |
| Subagent needs a quick fact-check without a coordinator round-trip | Give it a narrowly scoped tool (`verify_fact`) | Least privilege; keeps complex cases on the coordinator path |
| Subagent fails or times out | Return structured error context (type, query, partial results, alternatives) | Enables intelligent recovery; no swallowed or generic errors |
| Human agent takes over, no transcript | Structured handoff (customer ID, root cause, amounts, recommended action) | The summary is all they have |
| Same predictable steps every time (multi-aspect review) | Prompt chaining / fixed pipeline | Predictable, cheap, testable |
| Open-ended investigation ("add tests to legacy code") | Dynamic decomposition: map, prioritise, adapt | Next steps depend on discoveries |
| 14-file PR review is inconsistent and contradictory | Per-file local passes plus a cross-file integration pass | Attention dilution; larger windows or voting do not fix it |
| Return to a prior investigation, context still valid | `--resume <name>` (CLI) or `resume=<session_id>` (SDK) | Keeps the analysis; add a change notice if anything moved |
| Prior tool results are stale (big code changes) | New session with a structured summary | Stale results cause confident wrong conclusions |
| Compare two approaches from one analysis | `fork_session` | Independent branches from a shared baseline |
| Exam says `Task` tool / `allowedTools` | Answer with the guide's wording; in current docs the tool is `Agent` | `Task` was renamed `Agent` in Claude Code v2.1.63 (alias still works) |

---

## Sources fetched

All fetched on 2026-10-30.

- https://code.claude.com/docs/en/agent-sdk/overview.md (2026-10-30)
- https://code.claude.com/docs/en/agent-sdk/agent-loop.md (2026-10-30)
- https://code.claude.com/docs/en/agent-sdk/subagents.md (2026-10-30)
- https://code.claude.com/docs/en/agent-sdk/hooks.md (2026-10-30)
- https://code.claude.com/docs/en/agent-sdk/sessions.md (2026-10-30)
- https://code.claude.com/docs/en/agent-sdk/custom-tools.md (2026-10-30)
- https://code.claude.com/docs/en/sub-agents.md (2026-10-30; read in full via two fetches)
- https://code.claude.com/docs/en/hooks-guide.md (2026-10-30; sections used: blocking edits to protected files, combining results from multiple hooks, hook output and exit codes, limitations, hooks and permission modes)
- https://code.claude.com/docs/en/hooks.md (2026-10-30; sections used: lifecycle, hook locations, matcher patterns, hook input/output, exit codes and timeouts, JSON output and decision control, PreToolUse, PostToolUse, PostToolUseFailure, PostToolBatch, SubagentStart, SubagentStop, Stop, async hooks; the remaining event sections (SessionStart, Elicitation, worktree and model-switch events, prompt/agent hooks, etc.) were not used for this domain)
- https://code.claude.com/docs/en/sessions.md (2026-10-30)
- https://platform.claude.com/docs/en/agents-and-tools/tool-use/how-tool-use-works.md (2026-10-30)
- https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls.md (2026-10-30)
- https://platform.claude.com/docs/en/agents-and-tools/tool-use/parallel-tool-use.md (2026-10-30; execution semantics, formatting rules and `disable_parallel_tool_use` sections used)
- https://platform.claude.com/docs/en/build-with-claude/handling-stop-reasons.md (2026-10-30; quick-reference table, `end_turn` empty-response caveat, `max_tokens` incomplete tool use, `tool_use`, `pause_turn`, `refusal`, `model_context_window_exceeded`, loop pattern)
- https://www.anthropic.com/engineering/building-effective-agents (2026-10-30)
- https://www.anthropic.com/engineering/multi-agent-research-system (2026-10-30)
- Official exam guide v1.0 (July 2026) text file: Domain 1 task statements 1.1 to 1.7, scenarios 1, 3, 4, 5, sample questions 1, 7, 8, 9, 12, in-scope and out-of-scope lists.
