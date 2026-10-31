# Domain 5: Context Management & Reliability (15%)

> This domain is about keeping an agent *correct* once conversations, tool outputs and multi-agent hand-offs get long, noisy or broken. It covers what you keep in context (and what you must never let a summary eat), when a case should leave the agent and go to a human, how failures travel between agents, how to survive huge codebases and crashed sessions, how to decide how much human review an automated extraction needs, and how to keep every claim traceable to its source. It is weighted 15% because every production scenario depends on it: it is a primary domain for the Customer Support agent (Scenario 1), the Multi-Agent Research system (Scenario 3), Code Generation with Claude Code (Scenario 2) and Structured Data Extraction (Scenario 6). Expect scenario questions that ask "what is the root cause?" or "what is the most effective first change?" rather than recall questions.

## Contents

- [Mental model](#mental-model)
- [5.1 Manage conversation context to preserve critical information across long interactions](#51-manage-conversation-context-to-preserve-critical-information-across-long-interactions)
- [5.2 Design effective escalation and ambiguity resolution patterns](#52-design-effective-escalation-and-ambiguity-resolution-patterns)
- [5.3 Implement error propagation strategies across multi-agent systems](#53-implement-error-propagation-strategies-across-multi-agent-systems)
- [5.4 Manage context effectively in large codebase exploration](#54-manage-context-effectively-in-large-codebase-exploration)
- [5.5 Design human review workflows and confidence calibration](#55-design-human-review-workflows-and-confidence-calibration)
- [5.6 Preserve information provenance and handle uncertainty in multi-source synthesis](#56-preserve-information-provenance-and-handle-uncertainty-in-multi-source-synthesis)
- [Decision cheat-table](#decision-cheat-table)
- [Sources fetched](#sources-fetched)

## Mental model

Think of the context window as **scarce working memory, not storage**. Everything in a request counts against it: the system prompt, every message (including every tool result), and the tool definitions. The docs add that more context is not automatically better: as the token count grows, accuracy and recall degrade, a phenomenon they call *context rot*. The Anthropic engineering article on context engineering frames the goal as "the smallest possible set of high-signal tokens" that makes the desired behavior likely.

From that, eight ideas organise the whole domain:

1. **Facts that must survive go in a structured block outside the summarised history.** Summaries are lossy by design. Amounts, dates, order numbers, statuses and customer-stated expectations are exactly what lossy summaries blur ("the customer had a billing problem"). Extract them verbatim into a persistent "case facts" block that you re-send on every turn.
2. **Trim before you accumulate.** A tool result stays in context for the rest of the session. Cut a 40-field order record down to the 5 fields the task needs *when it comes back*, not after the window is full.
3. **Position matters.** The exam guide's wording: models reliably use information at the start and end of long inputs but may omit findings in the middle. Put a key-findings summary first, use explicit section headers, and make upstream agents return compact structured data, not their whole reasoning chain.
4. **Escalate on rules you can state, not on vibes.** Customer asks for a human, policy is silent or ambiguous, or no progress is possible. Sentiment scores and the model's own confidence number do not track case difficulty.
5. **Errors are information.** A failing subagent returns a structured error (what failed, what was tried, what partial results exist, what could be tried next). It never hides the failure as an empty success and never kills the whole workflow. Transient failures are retried locally first.
6. **Persist state outside the context window.** Scratchpad files, manifests, memory files and exported state survive compaction, `/clear`, crashes and new sessions. Delegate verbose exploration to subagents so the main context stays clean.
7. **Accuracy numbers need segmentation.** A 97% aggregate can hide a document type or a field that is wrong half the time. Sample high-confidence output at random (stratified), calibrate confidence against labelled data, and route scarce reviewer time to the cases that need it.
8. **Provenance is data, not decoration.** Every claim keeps a pointer to its source, date and method through every summarisation step. Conflicts are annotated with both values and both sources, not silently resolved.

The recurring exam reasoning pattern: pick the **lowest-effort, highest-leverage fix that addresses the root cause** (explicit criteria + few-shot examples before classifiers; a persistent facts block before fancier memory systems), and prefer **programmatic/structural** guarantees over hoping the model remembers.

> Out of scope for this exam (do not study for it): token-counting algorithms, prompt-caching internals, pricing and rate limits, embeddings/vector databases. The docs pages used here mention them; skip those parts.

---

## 5.1 Manage conversation context to preserve critical information across long interactions

### What the exam tests

**Knowledge of**

- **Progressive summarisation risk.** Repeatedly condensing a conversation compresses numbers, percentages, dates and the customer's stated expectations into vague prose.
- **"Lost in the middle."** Models reliably use information at the beginning and end of a long input but may skip findings buried in the middle.
- **Tool-result accumulation.** Tool outputs pile up and eat tokens far out of proportion to their usefulness (the guide's example: 40+ fields per order lookup when 5 matter).
- **Complete history on every request.** Each API request must carry the complete conversation so far, or the model loses coherence.

**Skills in**

- Extract transactional facts (amounts, dates, order numbers, statuses) into a persistent **case facts** block included in each prompt, *outside* the summarised history.
- Extract and persist structured issue data into a separate context layer for **multi-issue sessions**.
- **Trim verbose tool outputs** to the relevant fields before they accumulate.
- Put **key-findings summaries first** in aggregated inputs and organise details under **explicit section headers**, to counter position effects.
- Require subagents to attach **metadata** (dates, source locations, methodological context) to structured outputs.
- Change upstream agents to return **structured data** (key facts, citations, relevance scores) rather than verbose content and reasoning chains when downstream agents have limited context budgets.

### Concepts explained

#### How context actually fills up

The context-windows doc describes the loop plainly. Every turn consists of an input phase (all prior history plus the new user message) and an output phase (Claude's reply, which becomes input for the next turn). "Everything in the request counts": system prompt, all messages including tool results, and tool definitions; Claude's own output counts too. Previous turns are "preserved completely" unless *you* (or a server-side feature) remove them.

Two consequences matter for the exam:

- **The model has no memory between requests other than what you send.** If your code sends only the latest user message, the agent forgets the order number discussed two turns ago. Passing the complete history (including the assistant's `tool_use` blocks and your `tool_result` blocks) is what keeps the conversation coherent. The Agent SDK's session handling does this for you inside a session; if you call the Messages API yourself, your loop must append to a single `messages` list and resend it.
- **Context is cumulative.** A tool result you receive on turn 3 is re-read by the model on turns 4, 5, 6... A 3,000-token order record that contributes 200 tokens of useful information costs you its full weight for the rest of the session.

#### Why long contexts degrade

The docs use the term *context rot* (accuracy and recall fall as tokens grow) and the engineering article says models have an "attention budget" that every new token draws down. The exam guide names one visible symptom, *lost in the middle*: information at the start and end of a long input is used more reliably than information in the middle. (The phrase "lost in the middle" and the start/end/middle claim are the exam guide's wording; the fetched docs do not use that phrase or describe a middle-of-the-input effect. They describe general degradation as context grows (context rot) and give their own prompt-layout advice for long inputs, covered below. Answers on the exam follow the guide.)

Practical implication: a bigger window does not fix this. Curating *what* is in the window, and *where* it sits, does.

#### Progressive summarisation and the case-facts block

When a conversation gets long, a common design is to summarise old turns to save space. The danger is that each summarisation pass is lossy, and loss is not random: precise values (`$89.99`, `2026-03-14`, `ORD-88213`, "refund promised within 5 business days") are the first things a prose summary generalises away.

The fix is to separate **facts** from **narrative**:

- **Narrative** (the back-and-forth, tone, explanations) may be summarised.
- **Facts** (identifiers, amounts, dates, statuses, commitments, customer-stated expectations) are extracted into a structured block, kept verbatim, updated as tool results arrive, and **injected into every prompt outside the summarised history**, typically at the top of the system prompt or the first user message of each request.

Why this works: the facts block is rebuilt from authoritative tool outputs (and the customer's own words for expectations), so it is never a summary of a summary. It is also small and cheap to resend.

When *not* to use it: very short sessions where the full history fits comfortably and nothing is summarised, or content that is genuinely narrative (a brainstorm). Do not store whole tool dumps in the block; only the fields later turns will need.

#### Multi-issue sessions: a separate context layer

A customer says "my order arrived damaged and I was also double-charged on invoice 4471." Two issues, two sets of facts. Keep a structured **issues layer**: one record per issue with its own order IDs, amounts, status and promised next step. This stops facts from issue A leaking into issue B ("which refund amount was that?") and lets the agent report status per issue. It is the same idea as the case-facts block, indexed by issue.

#### Trimming verbose tool outputs

Tool results are the biggest source of "disproportionate" token use. The skill is to reduce them to the relevant fields **before they accumulate**. Where can you do that?

| Where | How | Notes |
|---|---|---|
| In your tool/MCP server | Return only the fields the agent needs, or expose a `fields` parameter | Best when you own the tool. The data never enters context. |
| In a hook after the tool runs | A `PostToolUse` hook rewrites the result before Claude sees it | The exam guide's PostToolUse hook pattern (Task Statement 1.5: intercept tool results for transformation before the model processes them), applied to trimming. Works when you cannot change the tool. |
| Automatic clearing of old results | API context editing: `clear_tool_uses_20250919` | Generic, age-based. It clears *old* results above a token threshold; it does not know which fields matter. |

Current Claude Code hooks docs: a `PostToolUse` hook (it runs after a tool completes successfully; failed calls go to a separate `PostToolUseFailure` event) receives `tool_name`, `tool_input` and `tool_response` as JSON on stdin, and can return `hookSpecificOutput.updatedToolOutput`, which "replaces the tool's output with the provided value before it is sent to Claude". The field works for all tools, not only MCP tools. The value must match the tool's output shape; for built-in tools a non-matching value is ignored, whereas MCP tool output "is passed through without schema validation". A second field, `updatedMCPToolOutput`, replaces output for MCP tools only, and the docs say to prefer `updatedToolOutput`. Do not confuse this with `decision: "block"`, which only adds a `reason` next to the result: Claude still sees the original output. Caveats from the docs worth knowing: the tool has already run (the hook cannot undo side effects), telemetry still records the original output, and stripping error details Claude needs can make it proceed on a false assumption. So **trim success payloads; never strip error information.**

The API-level context-editing page describes server-side `clear_tool_uses_20250919` (beta header `context-management-2025-06-27`): once the prompt passes a `trigger` (default 100,000 input tokens; it can also be set in `tool_uses`), it clears the oldest tool results beyond the `keep` count (default 3 tool uses), replacing them with placeholder text; `exclude_tools` protects chosen tools, `clear_at_least` sets a minimum amount to clear (if the API cannot clear that much, the strategy is not applied; the point is to make breaking the prompt cache worthwhile), and `clear_tool_inputs` (default `false`) also removes the tool call parameters. A separate strategy on the same page, `clear_thinking_20251015` (parameter `keep`), manages earlier `thinking` blocks, not tool results. The tool-context page positions this as the answer when "early results are no longer relevant". It complements, and does not replace, field-level trimming and a facts block, because it cannot tell an old order ID you still need from an old search result you do not. If you rely on it, save must-keep facts first (the docs pair it with the memory tool so Claude can write important information to memory before results are cleared).

Server-side **compaction** is the API's own summariser: it replaces older turns with a summary Claude writes on the server, and the context-windows page calls it the primary strategy for long-running conversations. The compaction overview describes two kinds, both beta. *On-demand compaction* (beta header `compact-2026-09-04`) is triggered by your application: you send a request with the top-level `compaction` parameter and get back a signed `compaction` block that goes first in `messages` in place of the messages it summarises; it cannot be combined with `context_management` on the same request. *Compaction at a token threshold* is triggered by the API when input tokens reach the trigger you set (the context-editing page names its edit type `compact_20260112`); it has its own page that was not among the pages checked for this file. For on-demand compaction you can replace the default summary prompt with `instructions` (up to 16,384 characters; the docs' example says "Preserve every entity and field name agreed so far..."), which is the platform-level cousin of the case-facts idea, but the exam pattern is still: *do not rely on any summary to carry exact transactional facts; keep them in a structure you control.* (Compaction's API details are not exam content.)

Do not conflate the mechanisms: **server-side compaction** summarises older turns; **context editing** (`clear_tool_uses_*`, `clear_thinking_*`) clears specific tool results or thinking blocks by rule without summarising; the **memory tool** is a client-side file store under `/memories` that your application implements, so information survives clearing or summarising; and **Claude Code's `/compact` and `/clear`** are Claude Code session commands (`/compact` summarises the conversation, `/clear` starts an empty context), not API parameters.

#### Position effects: layout of aggregated inputs

When a coordinator concatenates many subagent results into one prompt for synthesis, the middle gets under-used (the exam guide's "lost in the middle" framing). Two layout techniques from the exam guide:

1. **Key-findings summary at the very beginning** of the aggregated input (for example the handful of facts the final answer must not miss; the number is illustrative).
2. **Explicit section headers** around each detailed block, so the model can locate and cite sections instead of skimming a wall of text.

The prompting best-practices doc gives related but different layout advice for long inputs (20k+ tokens): put long documents near the top and the query/instructions after them (the docs report that queries at the end can improve response quality "by up to 30 percent in tests"), wrap each document in tags such as `document`, `source` and `document_content`, and ask the model to quote the relevant passages first and work from those quotes. That page does not mention a key-findings summary or "lost in the middle"; the summary-first rule and the position-effect rationale come from the exam guide. A reasonable way to combine them (this combination is the guide author's suggestion, not a docs statement): orient the model at the start, keep sections labelled, and put the question where the model will see it last.

#### Subagent outputs for limited downstream budgets

If the synthesis agent has a limited context budget, the research agents upstream should return **compact structured data** (key facts, citations, relevance scores, dates, source locations, method notes) instead of long prose and chain-of-thought. The Anthropic engineering write-up on sub-agent architectures says specialised sub-agents explore in clean context windows and return condensed summaries (often 1,000-2,000 tokens); the multi-agent research article adds that subagents can store large outputs externally and pass lightweight references to the coordinator rather than copying everything through the conversation. Metadata (publication dates, where in the document a fact came from, how it was measured) must ride along, or the synthesis step cannot weigh or reconcile claims (see 5.6).

### Worked examples

#### Example A. Progressive summarisation failure vs a persistent case-facts block

**Bad: only a rolling prose summary survives.**

Raw turns (abridged):

```text
Customer: Order ORD-88213 arrived on 2026-03-14 with a cracked screen. I paid $89.99 plus $6.50 shipping.
Agent: (lookup_order -> status DELIVERED, item SKU-4410) I can refund the $89.99 item price. Shipping is non-refundable under policy RET-2.
Customer: I was told on the phone by Maria that shipping would be refunded too. I need it within 5 business days or I'm disputing with my bank.
Agent: I've noted that. Let me check...
```

After two summarisation passes the only context left is:

```text
Summary: The customer had a billing problem with a recent order and wants it resolved quickly.
```

The amounts, the date, the order number, the claimed phone promise, the 5-day deadline and the chargeback threat are gone. The next turn cannot answer "how much did we say we'd refund?" and may even re-ask for the order number.

**Good: a case-facts block, kept outside the summarised history and re-sent on every request.**

```json
{
  "case_facts": {
    "customer_id": "CUST-20931",
    "verified": true,
    "issues": [
      {
        "issue_id": "I-1",
        "type": "damaged_item_return",
        "order_id": "ORD-88213",
        "sku": "SKU-4410",
        "delivered_on": "2026-03-14",
        "order_status": "DELIVERED",
        "item_amount_usd": 89.99,
        "shipping_amount_usd": 6.50,
        "policy_refs": ["RET-2: shipping non-refundable"],
        "customer_expectation": "Shipping also refunded within 5 business days (says agent Maria promised by phone)",
        "stated_consequence": "Will dispute with bank if not met",
        "agent_commitments": ["Item refund 89.99 approved in principle"],
        "open_questions": ["Is there a record of the phone promise?"],
        "status": "awaiting_policy_decision"
      }
    ]
  }
}
```

How it is used: your orchestration code (not the model's memory) updates the block after each tool result, then builds each request like this.

```python
def build_request(case_facts: dict, summarized_history: str, recent_turns: list[dict]):
    system = (
        SUPPORT_AGENT_PROMPT
        + "\n\n## Case facts (authoritative; never contradict; update via tools only)\n"
        + json.dumps(case_facts, indent=2)
    )
    messages = []
    if summarized_history:
        messages.append({"role": "user", "content": f"Earlier conversation (summary): {summarized_history}"})
        messages.append({"role": "assistant", "content": "Understood."})
    messages.extend(recent_turns)   # last N turns verbatim, including tool_use/tool_result blocks
    return {"system": system, "messages": messages}
```

Why it works: the summary may be as vague as it likes; the numbers, dates and commitments live in structured data derived from tool outputs and the customer's own words.

#### Example B. Multi-issue session state layer

```json
{
  "case_facts": {
    "customer_id": "CUST-20931",
    "issues": [
      {"issue_id": "I-1", "type": "damaged_item_return", "order_id": "ORD-88213", "refund_usd": 89.99, "status": "refund_processed", "refund_id": "RF-5521"},
      {"issue_id": "I-2", "type": "duplicate_charge",     "invoice_id": "INV-4471", "charged_usd": 54.00, "expected_usd": 27.00, "status": "investigating"}
    ],
    "active_issue": "I-2"
  }
}
```

The agent can answer "what happened with my refund?" (I-1) while working on I-2 without mixing the two amounts.

#### Example C. Trim a 40-field order lookup to the 5 return-relevant fields

`lookup_order` returns about 40 fields: warehouse codes, carrier internals, tax breakdown, marketing attributes, audit fields. For a **return** the agent needs five: `order_id`, `status`, `delivered_at`, `items` (SKU, price) and `return_eligible_until`.

A `PostToolUse` command hook (matcher on the MCP tool; MCP tool names follow `mcp__<server>__<tool>`; the server and script names here are illustrative) that rewrites the result:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "mcp__backend__lookup_order",
        "hooks": [
          { "type": "command", "command": "python3 .claude/hooks/trim_order.py" }
        ]
      }
    ]
  }
}
```

```python
# .claude/hooks/trim_order.py  (illustrative; adapt to the real tool_response shape)
import json, sys

event = json.load(sys.stdin)                # includes tool_name, tool_input, tool_response
raw = event["tool_response"]
order = json.loads(raw) if isinstance(raw, str) else raw   # shape depends on the MCP server (verify in docs)

trimmed = {
    "order_id": order["order_id"],
    "status": order["status"],
    "delivered_at": order.get("delivered_at"),
    "items": [{"sku": i["sku"], "price": i["price"]} for i in order.get("items", [])],
    "return_eligible_until": order.get("return_eligible_until"),
}

print(json.dumps({
    "hookSpecificOutput": {
        "hookEventName": "PostToolUse",
        "updatedToolOutput": trimmed      # MCP output is not schema-validated; keep it a faithful subset
    }
}))
```

Rules for a good trimmer: keep a *whitelist* of fields (not a blacklist), preserve the identifiers later calls need, and pass errors through untouched. If you control the MCP server, returning only those five fields from the tool itself is simpler and removes the hook. (If the question is phrased in the Agent SDK, the exam guide's Task Statement 1.5 describes the same idea: a PostToolUse hook that transforms tool results before the model processes them. The Agent SDK's hook API was not among the pages checked for this file, so treat the exact field names there as `(verify in docs)`.)

#### Example D. Lost in the middle: summary first, headers around the rest

**Bad: raw concatenation of eight subagent reports.**

```text
[report 1 full text ... 3,000 words]
[report 2 full text ... 3,000 words]
...
[report 8 full text ... 3,000 words]
Now write the final report.
```

Findings in reports 3-6 are the ones most likely to be dropped.

**Good: key findings first, labelled sections after.**

```text
## KEY FINDINGS (read first; the final report must address every item)
1. Generative-AI adoption in music production grew faster than in film (source: S3, 2026-01).
2. Two sources conflict on freelance illustrator income change: -14% (S2, 2025) vs -5% (S6, 2026): see section 2.
3. Film VFX: no usable sources found (search timed out; see COVERAGE GAPS).
...

## SECTION 1 - Visual arts        (agent: doc-analysis-1, 4 sources)
...
## SECTION 2 - Writing            (agent: web-search-2, 6 sources)
...
## COVERAGE GAPS
- Film production: web search timed out twice; no results retrieved.

## TASK
Write the report. Cite sources by id. Mark any key finding you could not support.
```

#### Example E. Require structured subagent output (not reasoning chains)

**Bad subagent return:** 2,500 words describing every search it ran and how it thought about each result.

**Good subagent return:**

```json
{
  "subagent": "web-search-music",
  "query": "AI music generation adoption 2025",
  "findings": [
    {
      "claim": "42% of surveyed producers used AI-assisted tools in 2025",
      "relevance": 0.92,
      "source": {"url": "https://example.org/report-2025", "title": "Producer Tools Survey", "publication_date": "2025-11-02"},
      "location": "section 3, table 2",
      "excerpt": "42% of respondents reported ...",
      "method": "online survey, n=1,204, self-reported"
    }
  ],
  "coverage": {"searched": ["web"], "gaps": []}
}
```

Fixed schema = predictable size, easy merge, provenance preserved. The reasoning chain stays in the subagent's own context, which is discarded.

### Exam traps

- **"Increase the context window / switch to the 1M model."** A larger window does not fix context rot or lost-in-the-middle, and it does nothing about summarised-away facts. Rule it out when the problem is *what* is preserved.
- **"Summarise more aggressively to save tokens."** That makes the specific failure worse. The fix is structured facts outside the summary.
- **"Ask the model to remember the amounts"** or "add an instruction to preserve numbers" in the summarisation prompt. Probabilistic. A structural block is deterministic.
- **Trimming by clearing old tool results only.** Automatic clearing is age-based; it does not choose fields. It is a complement, not the answer to "40 fields, 5 relevant".
- **Dropping history to save tokens** ("send only the last user message"). Breaks conversational coherence; the guide requires the complete history (with the facts block and/or a summary of older turns).
- **Putting the key summary in the middle or end of a long aggregate.** The guide says *beginning*.
- **Having the model pass its full reasoning chain to the next agent.** Bloats the downstream context; return structured facts.

### Check yourself

**Q1.** A support agent summarises the conversation every 10 turns. By turn 40, it keeps asking the customer to repeat the order number and quotes the wrong refund amount. What is the most effective fix?

<details><summary>Answer</summary>

Extract transactional facts (customer ID, order IDs, amounts, dates, statuses, commitments, stated expectations) into a structured case-facts block that is injected into every request outside the summarised history, updated from tool results. Making the summariser "try harder" or enlarging the window leaves the root cause (lossy prose summaries of exact values) in place.
</details>

**Q2.** `get_account` returns 60 fields; the billing-dispute flow needs four. Context fills up after about 15 lookups. What do you do first?

<details><summary>Answer</summary>

Trim the result to the needed fields at the source (tool/MCP server) or with a `PostToolUse` hook that replaces the tool output before it accumulates. Keep errors intact. Age-based clearing of old results is a useful complement but does not address the 56 irrelevant fields in every result.
</details>

**Q3.** A synthesis agent receives eight 3,000-word subagent reports and consistently ignores findings from the fourth and fifth. Name two changes that address the likely cause.

<details><summary>Answer</summary>

Lost-in-the-middle position effect. (1) Put a key-findings summary at the very beginning of the aggregated input. (2) Organise the detailed results under explicit section headers. Optionally, have upstream agents return compact structured data instead of long prose so the aggregate is shorter.
</details>

### Official docs

- [Context windows](https://platform.claude.com/docs/en/build-with-claude/context-windows.md) — how every turn accumulates, what counts toward the window, context rot, compaction as the main long-run strategy.
- [Context editing](https://platform.claude.com/docs/en/build-with-claude/context-editing.md) — `clear_tool_uses_20250919` options (`trigger`, `keep`, `exclude_tools`, `clear_at_least`, `clear_tool_inputs`), the separate `clear_thinking_20251015` strategy, and how it pairs with the memory tool.
- [Compaction overview](https://platform.claude.com/docs/en/build-with-claude/compaction.md) and [Compaction on demand](https://platform.claude.com/docs/en/build-with-claude/compaction-on-demand.md) — server-side summarisation (on-demand vs token-threshold) and, for on-demand compaction, the custom `instructions` field (skim; API details are not exam content).
- [Manage tool context](https://platform.claude.com/docs/en/agents-and-tools/tool-use/manage-tool-context.md) — the four ways to cut tool-driven context bloat and when each fits.
- [Prompting best practices: long context prompting](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices.md) — documents on top, query last, tag structure, quote-first.
- [Hooks reference (PostToolUse)](https://code.claude.com/docs/en/hooks.md) — `tool_response` input, the `updatedToolOutput` output field (all tools) and the MCP-only `updatedMCPToolOutput`.
- [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) — context rot, attention budget, high-signal tokens, sub-agent summaries.

---

## 5.2 Design effective escalation and ambiguity resolution patterns

### What the exam tests

**Knowledge of**

- The legitimate **escalation triggers**: the customer asks for a human; a **policy exception or gap** (not merely "a complex case"); the agent cannot make meaningful progress.
- The difference between **escalating immediately** when the customer explicitly demands a human and **offering to resolve** when the issue is straightforward and within the agent's capability.
- Why **sentiment-based escalation** and **self-reported confidence scores** are unreliable proxies for real case complexity.
- Why **multiple customer matches** call for clarification (ask for more identifiers), not heuristic selection.

**Skills in**

- Write **explicit escalation criteria with few-shot examples** into the system prompt, showing when to escalate and when to resolve autonomously.
- **Honour an explicit request for a human immediately**, without first attempting investigation.
- **Acknowledge frustration and offer resolution** when the issue is within the agent's capability; escalate only if the customer **reiterates** their preference.
- **Escalate when policy is ambiguous or silent** on the specific request (the guide's example: competitor price matching when policy only covers own-site adjustments).
- When a tool returns **multiple matches**, instruct the agent to **ask for additional identifiers** instead of picking one by heuristic.

### Concepts explained

#### The three real triggers

Escalation is a routing decision: "this case should leave the agent." The guide gives three defensible triggers, each of which you can state in a prompt and check in a log:

1. **The customer asks for a human.** An explicit request. It is the customer's right to make that request, and the agent should not argue or investigate first.
2. **Policy exception or gap.** The request is outside what the agent is authorised to decide: policy is silent, ambiguous, or requires an exception. Note the wording "not just complex cases": complexity alone is not a trigger. A multi-step damaged-goods replacement with photo evidence is *complex* but fully covered by policy, so the agent should resolve it. A one-line request to price-match a competitor is *simple* but policy only addresses own-site adjustments, so the agent should escalate.
3. **Inability to make meaningful progress.** The agent has tried reasonable avenues (tools failing, repeated clarification not converging, needed system access missing) and further turns will not help.

In the sample question for this scenario (Q3 in the official guide), an agent at 55% first-contact resolution escalates easy standard replacements while trying to handle policy-exception cases itself. The correct answer is to add **explicit escalation criteria with few-shot examples** because the root cause is unclear decision boundaries and this is "the proportionate first response before adding infrastructure".

#### Immediate escalation vs offering resolution

The same unhappy customer can call for different behaviour depending on what they actually said:

| Customer signal | Agent behaviour | Why |
|---|---|---|
| "I want to speak to a human." / "Get me a real agent." (explicit request) | Escalate **now**. Do not investigate or try to resolve first. | Honouring the stated preference is the requirement. Investigating first is exactly the behaviour the customer is rejecting. |
| Frustrated tone but no request for a human; issue is within the agent's tools and policy | Acknowledge the frustration, **offer a concrete resolution**, and carry it out. | Frustration is not a request. Most such cases are quickly resolvable. |
| Same customer, after the offer: "No, I still want a person." | Escalate. | They reiterated. The preference now overrides the attempt to resolve. |
| Request outside policy (competitor price match, exception not described) | Escalate with a summary, even if the customer is calm. | Policy gap. The agent must not invent policy. |
| Several customers match the identifier given | Ask for another identifier (email, phone, order number, postcode). | Selecting by heuristic risks acting on the wrong account. |
| Agent has tried and is stuck (tool outage, loops) | Escalate with what was tried. | No meaningful progress is possible. |

#### Why sentiment and self-reported confidence are poor proxies

- **Sentiment measures mood, not difficulty.** An angry customer with a straightforward damaged-item return needs a quick refund, not a human. A polite customer asking for something no policy covers needs a human. Official sample Q3's option D is wrong because "sentiment doesn't correlate with case complexity, which is the actual issue".
- **Self-reported confidence (e.g. "rate your confidence 1-10") is poorly calibrated.** The model that is wrongly handling a hard case is typically also confidently wrong about it. Official sample Q3's option B is wrong for this reason: "the agent is already incorrectly confident on hard cases".
- A **separately trained classifier** on historical tickets (sample Q3 option C) is over-engineered as a first step: it needs labelled data and ML infrastructure before the cheaper prompt-level fix has been tried.

There is an important distinction with Domain 5.5. In 5.5 the model's field-level confidence *is* used, but only after **calibrating it against a labelled validation set** and monitoring it with sampling. Uncalibrated, self-reported confidence as an escalation switch is the trap. Calibrated confidence for routing review attention is the pattern.

#### Ambiguity: multiple matches

`get_customer` with a name returns three "Alex Morgan" records. The wrong move is to guess ("the most recently active one", "the one with an open order"). A wrong guess can leak another customer's data or process a refund on the wrong account. The right move is a clarification turn asking for an identifier that discriminates (email on file, phone number, order number, billing postcode), then re-querying. Put this rule in the system prompt and, where practical, have the tool return a `match_count` and a "needs disambiguation" signal rather than one arbitrary record.

#### Where escalation criteria live

- **System prompt**: explicit criteria + few-shot examples of both outcomes (escalate and resolve), including borderline cases. This is the exam's first-line answer to miscalibrated escalation.
- **Tool design**: the `escalate_to_human` tool should take a structured handoff (customer ID, issue summary, what was tried, amounts, recommended action) because the human may not see the transcript. (Hand-off structure is also a Domain 1 topic.)
- **Programmatic enforcement**: if a business rule is a hard limit (for example, refunds above a threshold must always go to a human), enforce it in code with a hook or tool precondition rather than hoping the prompt is followed. Soft judgment (is this a policy gap?) belongs in prompt criteria.

#### Do not confuse escalation with approval prompts

In the Agent SDK, `canUseTool` and the `AskUserQuestion` tool let *your application* pause the run to get a person's approval or a clarifying answer; the run then continues. This is **human-in-the-loop approval**, not hand-off of the case. Facts from the Agent SDK user-input doc worth knowing: the callback fires when a tool needs approval or when Claude asks a clarifying question; it does not fire for tools already auto-approved by a rule or mode (use a `PreToolUse` hook for logic that must apply to every call); the callback can stay pending indefinitely, and if the user may take longer than your process can stay up, a `PreToolUse` hook can return a `defer` decision so the process exits and later resumes from the persisted session; and `AskUserQuestion` is not currently available in subagents spawned via the Agent tool (the subagents page also lists it among the tools removed from every subagent), so a design that needs a clarifying question from a user has to ask it from the main agent (that consequence is this guide's inference, not a docs statement). Escalation to a human *agent* (Scenario 1's `escalate_to_human`) transfers the whole case.

### Worked examples

#### Example A. System-prompt escalation section with few-shot examples

```text
## When to escalate vs resolve

ESCALATE (call escalate_to_human) when ANY of these is true:
1. The customer explicitly asks for a human, agent, manager or supervisor.
   -> Escalate immediately. Do NOT look up orders or try to resolve first.
2. The request needs a policy exception, or our policy does not address it
   (e.g. price-matching a competitor, goodwill credits, legal threats).
   -> Do not invent policy. Escalate with a summary.
3. You cannot make progress (a required tool keeps failing, or two clarification
   attempts have not produced what you need).

RESOLVE YOURSELF when the request is covered by policy and your tools can do it,
even if the case has several steps or the customer is upset:
- Standard damage/defect replacement or refund with photo evidence.
- Order status, address changes before dispatch, return label requests.
When the customer is frustrated but has NOT asked for a human: acknowledge it in one
sentence, state the concrete fix, and do it. If they then repeat that they want a
person, escalate.

Customer lookups: if get_customer returns more than one match, ask for one more
identifier (email, phone, order number). Never choose a match yourself.

### Examples
<example>
Customer: "This is ridiculous, just give me a human."
Action: escalate_to_human immediately. Reply: "Of course. I'm connecting you with a
person now and passing along your details so you won't need to repeat them."
(No investigation first.)
</example>
<example>
Customer: "My blender arrived cracked, here are photos. I'm furious, this is the second time!"
Action: verify customer and order, confirm photo evidence, offer replacement or refund
per policy, process it. Reply: "I'm sorry, that's frustrating. Here's what I can do right
now: ..." (Covered by policy; no human needed.)
</example>
<example>
Customer: "Another store sells this for $20 less. Match it."
Action: policy only covers adjustments on our own site -> escalate with summary.
</example>
<example>
Customer (after your offer): "I appreciate that, but I'd still rather talk to a person."
Action: escalate_to_human. They have repeated their preference.
</example>
```

#### Example B. Transcript: frustration without a request

```text
Customer: I've been charged twice for the same order and nobody is helping. This is useless.
Agent: I'm sorry about the double charge, that's frustrating. Let me fix it. Could you give me the email on your account?
Customer: sam.lee@example.com
Agent: (get_customer -> 1 match; lookup_order -> duplicate charge confirmed)
        Found it: two charges of $42.10 on ORD-5512. I've refunded the duplicate; it will show in 3-5 business days.
Customer: Fine.
```

Frustration was acknowledged, the issue was within the tools and policy, so the agent resolved it. No escalation. Had the customer instead said "Stop, I want a person", the agent would have escalated at that point.

#### Example C. Multiple matches

```text
Tool result: get_customer(name="Priya Shah") -> {"match_count": 3, "matches": [ {"id": "C-102", "city": "Leeds"}, {"id": "C-877", "city": "Leeds"}, {"id": "C-2301", "city": "Bristol"} ]}

Bad  : Agent picks C-102 "because it has the most recent order" and issues a refund.
Good : Agent: "I found more than one account under that name. Could you confirm the email address or phone number on the account, or the order number?"
```

### Exam traps

- **Sentiment threshold, 1-10 confidence score, or a trained classifier as the first fix.** All three are the guide's named distractors. The correct first fix for miscalibrated escalation is explicit criteria + few-shot examples.
- **"Always try to resolve before escalating."** Wrong when the customer explicitly requests a human: escalate immediately, no investigation. Equally wrong in the other direction: escalating every frustrated customer.
- **"Escalate complex cases."** Complexity is not the criterion; policy exceptions/gaps and lack of progress are.
- **Picking the "most likely" customer on multiple matches.** Ask for identifiers.
- **Inventing a policy answer** for competitor price-matching because it seems reasonable. Policy silence means escalate.
- **Confusing a `canUseTool` approval pause with escalation.** Different mechanism and purpose.

### Check yourself

**Q1.** A customer writes: "Your refund policy says nothing about gift-card purchases. I want a refund for one." The agent's tools can issue refunds. What should it do?

<details><summary>Answer</summary>

Escalate (with a summary of the request and what was checked). Policy is silent on the specific request, so the agent must not invent an answer even though the tool could technically process a refund. It is a policy gap, not a complexity issue.
</details>

**Q2.** A customer says "this is the third time, I'm sick of this" but never asks for a person, and the issue is a standard shipping-address correction before dispatch. Escalate?

<details><summary>Answer</summary>

No. Acknowledge the frustration, resolve the address change, and escalate only if the customer then reiterates a wish for a human. Frustration is not an explicit request and does not indicate a case beyond the agent's capability.
</details>

**Q3.** Your team proposes: "Have the agent output a 1-10 confidence and auto-escalate below 6." Give the strongest reason this is a poor first step.

<details><summary>Answer</summary>

Self-reported confidence is poorly calibrated: on exactly the hard cases the agent is mishandling it is already confidently wrong, so the threshold will not separate easy from hard cases. The proportionate first fix is explicit escalation criteria with few-shot examples. (Calibrated confidence against a labelled validation set is a different, valid technique for review routing; see 5.5.)
</details>

### Official docs

- [Handle approvals and user input (Agent SDK)](https://code.claude.com/docs/en/agent-sdk/user-input.md) — `canUseTool`, `AskUserQuestion`, `defer`; the human-in-the-loop mechanics (not the same as escalation).
- [Claude prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices.md) — examples and structuring prompts with tags (supports the few-shot escalation section).
- Exam guide sample question 3 and Scenario 1 (in the official guide, section 9 and section 5).

---

## 5.3 Implement error propagation strategies across multi-agent systems

### What the exam tests

**Knowledge of**

- **Structured error context** (failure type, attempted query, partial results, alternative approaches) lets the coordinator make intelligent recovery decisions.
- The difference between an **access failure** (timeout; a retry decision is needed) and a **valid empty result** (query succeeded, nothing matched).
- Why **generic statuses** ("search unavailable") hide context the coordinator needs.
- Why **suppressing errors** (empty result marked as success) and **terminating the whole workflow on one failure** are both anti-patterns.

**Skills in**

- Return structured error context: failure type, what was attempted, partial results, possible alternatives.
- Distinguish access failures from valid empty results in error reporting.
- Have subagents perform **local recovery for transient failures** and propagate only what they cannot resolve, including what was attempted and any partial results.
- Structure synthesis output with **coverage annotations** showing which findings are well supported and which topic areas have gaps because sources were unavailable.

### Concepts explained

#### Who decides what

In a coordinator-subagent design, **subagents own local recovery**; the **coordinator owns strategy**. A subagent can cheaply retry a flaky call. It cannot know whether, say, the report should be re-scoped, a different subagent should try, or the user should be told a section is thin. For the coordinator to decide, it needs facts, which is why the error must be *structured* and *informative*.

#### Retry or not: classify the failure

The platform error docs list the HTTP error classes. Server-side conditions are 500 `api_error` (the docs say to retry with exponential backoff), 504 `timeout_error` (the docs suggest streaming for long-running requests) and 529 `overloaded_error` ("temporarily overloaded"); these are the natural candidates for bounded retries. The official SDK already retries transient failures such as connection errors and 5xx server errors with exponential backoff, twice by default, honouring `retry-after` when present (the docs list rate limits there too; rate-limit mechanics are not exam content). Request-side errors are 400 `invalid_request_error`, 401 `authentication_error`, 403 `permission_error`, 404 `not_found_error` and 413 `request_too_large`: they describe a problem with the request, key, permissions, resource or size, so resending the same request unchanged cannot be expected to succeed. (The docs do not literally label these "non-retryable"; that conclusion is reasoning from what each error means.)

The exam's in-scope list uses a business-level version of the same idea: **transient vs business-rule vs permission errors**. Map them like this:

| Class | Example | Retry the same call? | What the subagent should do |
|---|---|---|---|
| Transient | Timeout, 5xx server error, connection reset | Yes, a bounded number of times with backoff | Recover locally. Propagate only if still failing, with attempts recorded. |
| Business / validation | "Order not eligible for refund", malformed query | No | Do not retry. Report the reason; the coordinator or user decides (alternative query, escalate). |
| Permission / access | 403, source requires login | No | Do not retry. Report so the coordinator can choose another source or tell the user. |
| Valid empty result | Search ran fine, 0 matches | Not an error | Return success with an empty list and say what was searched. |

#### Access failure vs valid empty result

Both can look like "no data", but they mean opposite things:

- **Valid empty**: the system answered; the answer is "nothing matches". The coordinator may conclude "no evidence found for this topic" (a finding in itself) or broaden the query.
- **Access failure**: the system did not answer. The coordinator knows *nothing* about the topic from this source and must not report absence of evidence.

Collapsing them into `[]` makes the final report assert "there is no research on X" when the truth is "we could not reach the source for X".

#### The four ways to get this wrong

The guide names three explicitly (Q8 options B, C, D). The fourth below is a common companion mistake that follows from the "local recovery" skill.

| Anti-pattern | What it looks like | Why it fails |
|---|---|---|
| 1. Generic status | `{"status": "search unavailable"}` after retries | The coordinator cannot tell timeout from permission denial, cannot see the query, partial results or alternatives. (Option B in sample Q8.) |
| 2. Silent suppression | Catch the timeout, return `[]` as success | Failure is disguised as "no results". No recovery occurs and the report is quietly incomplete. (Option C.) |
| 3. Workflow termination | Let the exception bubble to a top-level handler that aborts the research run | One failed subagent destroys all completed work when recovery (retry, alternative source, proceed with partial) was available. (Option D.) |
| 4. Raw propagation with no local recovery | Every blip, including a 1-second network hiccup, goes straight to the coordinator | Coordinator is flooded with noise and spends tokens on problems a retry would fix. Handle transient faults locally; propagate only what cannot be resolved. |

The multi-agent research article is consistent with this: it combines letting the agent know a tool failed and adapt with deterministic safeguards (retry logic, checkpoints), and builds systems that resume from where the agent was rather than restarting from scratch.

#### Structured error payload

```json
{
  "status": "error",
  "failure_type": "timeout",
  "retryable": true,
  "attempted": {
    "tool": "web_search",
    "query": "AI film production VFX adoption 2025",
    "attempts": 3,
    "backoff_ms": [500, 1500, 4000],
    "elapsed_s": 94
  },
  "partial_results": [
    {"claim": "Studio X piloted AI previsualisation", "source_url": "https://example.org/a", "publication_date": "2025-06-10"}
  ],
  "coverage_impact": "film_production subtopic: 1 of ~6 expected sources retrieved",
  "alternatives": [
    "retry with a narrower query: 'AI previsualisation studios 2025'",
    "try the document-analysis agent on the trade-press PDF set",
    "proceed with partial results and flag the gap"
  ]
}
```

What each field buys the coordinator: `failure_type` (retry vs reroute), `attempted` (do not repeat the same query), `partial_results` (use what exists), `coverage_impact` (how bad), `alternatives` (a menu of next moves).

For a valid empty result the shape is a *success*:

```json
{
  "status": "success",
  "query": "AI music generation regulation Iceland 2025",
  "results": [],
  "sources_searched": ["web_search"],
  "note": "Query completed; no matching documents."
}
```

#### Coverage annotations in the final report

The synthesis agent reports not just findings but *how well each area is supported*. Example:

```text
## Coverage
| Topic | Status | Basis |
|---|---|---|
| Visual arts | Well supported | 6 sources, 2 independent surveys, 2024-2026 |
| Music | Well supported | 5 sources |
| Writing | Partial | 2 sources; one is a vendor blog |
| Film production | GAP | Web search timed out after 3 attempts; only 1 source retrieved (see details) |
```

This lets a reader trust the strong sections and treat the gap as a gap.

### Worked examples

#### Example A. Subagent with local recovery and structured propagation

```python
import time

TRANSIENT = {"timeout", "overloaded", "connection_error", "server_error"}

def run_search(query: str, max_attempts: int = 3) -> dict:
    partial, attempts = [], []
    for n in range(1, max_attempts + 1):
        try:
            hits = web_search(query)                      # may raise
            return {"status": "success", "query": query,
                    "results": hits, "sources_searched": ["web_search"]}   # [] is a VALID empty result
        except SearchError as e:
            attempts.append({"attempt": n, "error": e.kind, "detail": str(e)})
            partial.extend(getattr(e, "partial_results", []))
            if e.kind not in TRANSIENT:                   # permission / validation: retrying is pointless
                break
            time.sleep(0.5 * 3 ** (n - 1))                # bounded exponential backoff
    last = attempts[-1]
    return {
        "status": "error",
        "failure_type": last["error"],
        "retryable": last["error"] in TRANSIENT,
        "attempted": {"tool": "web_search", "query": query, "attempts": attempts},
        "partial_results": partial,
        "alternatives": suggest_alternatives(query, last["error"]),
    }
```

#### Example B. Coordinator recovery decision

```python
def handle(result, task):
    if result["status"] == "success":
        return record(task, result)                       # even when results == [] (valid empty)
    if result["failure_type"] == "timeout" and result["retryable"] and not task.retried:
        return spawn(task.with_query(result["alternatives"][0]), retried=True)
    if result["partial_results"]:
        record(task, result["partial_results"], coverage="partial")
    mark_gap(task, reason=result["failure_type"])         # surfaces in the final coverage table
    # keep going: other subagents' work is untouched
```

The coordinator retries once with a narrower query, accepts partial results otherwise, flags the gap, and *continues*. It never aborts the run for a single failed branch.

#### Example C. Generic vs structured, side by side

```text
Bad : {"status": "error", "message": "search unavailable"}
Bad : {"status": "ok", "results": []}                    # but the search actually timed out
Good: {"status": "error", "failure_type": "timeout", "attempted": {...}, "partial_results": [...], "alternatives": [...]}
```

### Exam traps

- **Option "automatic retry with backoff, then return a generic status"** (sample Q8-B). Retrying locally is fine; the trap is the generic status that hides context.
- **"Return an empty result so the pipeline keeps flowing."** Silent suppression; marks failure as success.
- **"Propagate the exception to a top-level handler."** Over-reaction; kills completed work.
- **Treating zero hits as an error, or an error as zero hits.** Keep valid-empty and access-failure distinct.
- **Retrying non-transient failures** (permission denied, invalid request). Pointless; propagate with context.
- **Assuming the coordinator should handle retries for every blip.** Transient faults are recovered locally by the subagent.
- **Blaming downstream agents for a coverage hole that starts upstream.** Official sample Q7 (a research report covers only visual arts) shows every subagent succeeding while the coordinator's decomposition was too narrow ("digital art", "graphic design", "photography"). Coverage annotations and good error payloads tell you *where* a gap came from; check the coordinator's task decomposition first when all subagents report success but whole topic areas are missing. Do not "fix" it by adding gap-detection to the synthesis agent or widening a search agent's queries.

### Check yourself

**Q1.** The document-analysis subagent receives HTTP 403 when opening a paywalled PDF. What should it return?

<details><summary>Answer</summary>

A structured error with failure type "permission/access denied", the document identifier it tried, any partial results (for example, the abstract if it was available), and alternatives (use another source, ask for credentials, mark the gap). It should not retry the same call and must not return an empty success.
</details>

**Q2.** A search subagent returns `{"status": "success", "results": []}`. In the final report, the coordinator writes "no studies exist on this topic". When is this valid and when is it a bug?

<details><summary>Answer</summary>

Valid only if the query genuinely completed and matched nothing (a valid empty result, ideally with the sources searched recorded). It is a bug if the empty list masks an access failure such as a timeout; that is why failures must be reported distinctly and why reports carry coverage annotations.
</details>

**Q3.** One of four parallel research subagents fails after three retries. Describe the best coordinator behaviour.

<details><summary>Answer</summary>

Use the structured error to choose a recovery (retry with a narrower or different query, route to another agent/source, or proceed with partial results), keep the other three subagents' results, and mark the affected topic in the final coverage annotations as a gap. Do not terminate the workflow and do not paper over the failure.
</details>

### Official docs

- [Claude API errors](https://platform.claude.com/docs/en/api/errors.md) — the HTTP error classes (server-side 500, 504, 529 versus request-side 400, 401, 403, 404, 413) and the SDK's default retries; only 500 is explicitly tagged "retry with exponential backoff". (Skip the rate-limit material.)
- [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) — "Production reliability" section: compounding errors, resuming from where the agent was, adapting after tool failure plus deterministic safeguards.
- [Subagents](https://code.claude.com/docs/en/sub-agents.md) — `maxTurns` partial output that can be resumed; subagent results return as a summary; and the "API errors in subagents" section (a foreground subagent cut off by an API error returns its partial output with a note; a background one is marked failed and its last output is included).

---

## 5.4 Manage context effectively in large codebase exploration

### What the exam tests

**Knowledge of**

- **Context degradation** in extended sessions: inconsistent answers, and references to "typical patterns" instead of the specific classes discovered earlier.
- The role of **scratchpad files** for persisting key findings across context boundaries.
- **Subagent delegation** to isolate verbose exploration output while the main agent coordinates high-level understanding.
- **Structured state persistence for crash recovery**: each agent exports state to a known location; the coordinator loads a **manifest** on resume.

**Skills in**

- Spawn subagents for specific questions ("find all test files", "trace refund flow dependencies") while the main agent keeps the high-level picture.
- Have agents keep **scratchpad files** of key findings and consult them for later questions.
- **Summarise findings from one exploration phase before spawning subagents for the next**, injecting the summary into their initial context.
- Design **crash recovery** with state exports (manifests) that the coordinator loads on resume and injects into agent prompts.
- Use **`/compact`** to shrink context during long exploration when it fills with verbose discovery output.

### Concepts explained

#### Recognising context degradation

In Scenario 4 (developer productivity agent exploring an unfamiliar legacy codebase) the agent reads dozens of files over a long session. Symptoms that the context is saturated or polluted:

- It answers the same question differently than it did an hour ago (**inconsistent answers**).
- It talks about "typical patterns" ("repositories in this kind of app usually...") instead of naming the `RefundService.applyPolicy()` it found earlier. The specific discovery has been crowded out, summarised away, or sits in a low-attention part of the window, and the model fills the gap with generic priors.
- It re-reads files it already read or re-asks questions already answered.

Cause: accumulating raw file contents and grep output ("verbose discovery output"), plus automatic summarisation that loses specifics. The Claude Code docs are explicit that file reads and instructions unrelated to the task "fill the context window ... costing tokens and degrading Claude's performance" in large codebases.

#### Four tools, each for a different job

| Technique | What it does | Use when |
|---|---|---|
| **Scratchpad file** | Agent writes key findings (class names, file paths, flow notes) to a file on disk and re-reads it later | Findings must outlive compaction, `/clear` and context pressure; you want facts exact, not summarised. |
| **Subagent delegation** | A subagent explores in **its own context window** and returns only a summary | The exploration produces lots of output you will not need verbatim ("find all test files", "trace refund dependencies"). |
| **Phase summaries** | Summarise phase N's findings and inject the summary into the prompts of phase N+1 agents | Multi-phase investigations where later agents should not re-explore. |
| **`/compact`** | Replace history with a summary (optionally focused) | Session is long and full of verbose discovery output, and you want to continue in the same session. |

Plus **manifests / state export** for crash recovery.

##### Scratchpad files

A scratchpad is just a file the agent maintains: durable, exact, cheap to re-read, and independent of the conversation. It counters degradation because the *specifics* sit on disk and can be loaded into a fresh context on demand. The same idea appears across the docs:

- Anthropic's context-engineering article calls it **structured note-taking (agentic memory)**: persistent notes outside the window, pulled back in later.
- The prompting best-practices doc recommends **structured formats for state** (for example a `tests.json`), **unstructured text for progress notes** (`progress.txt`), and git for checkpoints and history; and notes that for multi-window work, starting a fresh window and having the model rediscover state from files can beat compaction ("Review progress.txt, tests.json, and the git logs").
- The memory tool doc describes a **multisession pattern**: an initializer session creates a progress log, a feature checklist and a startup-script reference; each later session opens by reading them and ends by updating the log. Its key principle: mark a feature complete only after end-to-end verification.

##### Subagent delegation

Per the Claude Code docs, each subagent runs in its **own context window** with its own system prompt and tool access, does the noisy work there, and "returns only the summary". Use one "when a side task would flood your main conversation with search results, logs, or file contents you won't reference again." A non-fork subagent does not see your conversation history (a fork does), so the delegation message must contain everything it needs. The built-in Explore agent is a fast, read-only agent that Claude delegates to for broad codebase searches. What comes back to the main conversation is the subagent's result, so many subagents that each return detailed results can still consume significant context (the docs warn about this); that is why the guide says to ask for short summaries.

*Current docs differ in emphasis from the exam guide:* the Claude Code best-practices page lists "use subagents to protect your context window" among advice that is largely outdated, because most current models have a 1M-token window and compaction runs on its own, so running out of context is rare; it says subagents remain good for parallel work, keeping noisy output out of the way and independent review, but each costs extra tokens. The exam guide still teaches subagent delegation to isolate verbose exploration output, so answer with the exam's rationale.

Costs, from the best-practices doc: subagents spend tokens (each sends its own requests) and a fresh one must gather context your conversation already had, so it can be slower than doing a small task inline. To contain this: ask for **short summaries back**, give simple subagents a cheaper model, and restrict their tools (for example `tools: Read, Grep, Glob`).

Terminology note: the exam guide and many exam answers say the **`Task`** tool spawns subagents. In current Claude Code (v2.1.63 and later) the tool was **renamed `Agent`**; existing `Task(...)` references still work as aliases. Same mechanism. Follow the guide's wording on the exam.

##### Summarise-then-spawn phases

Instead of letting phase 2's agents start cold (re-reading everything) or inheriting a bloated history, you:

1. Run phase 1 (e.g. "map the modules") with subagents.
2. Consolidate their returns into a compact, structured **phase summary**.
3. Inject that summary into the initial context of the phase 2 subagents (e.g. "trace refund flow in the modules below").

This keeps the main agent coordinating at a high level, and gives each worker only what it needs.

##### `/compact`, `/clear`, `/context`

From the Claude Code docs:

- **`/compact [instructions]`** replaces the conversation with a structured summary; add instructions to steer what is kept (the docs' example: `/compact focus on the auth bug fix`). Claude Code also compacts automatically as you approach the limit.
- **`/clear`** starts fresh (use between unrelated tasks); the previous session can be reopened with `/resume`.
- **`/context`** shows what is consuming context.
- What survives compaction (context-window doc): the system prompt and output style still apply; project-root CLAUDE.md and unscoped rules, and auto memory, are re-injected from disk; plan-mode plans are re-injected; Claude Code re-reads up to five recently modified files (large files come back as path references); invoked skill bodies are re-injected within size caps. Things that were *loaded into the conversation*, such as path-scoped rules and nested CLAUDE.md files, are summarised away with everything else and reload on demand. Lesson: **durable facts belong in files on disk, not in the conversation.**
- Compact *before* quality degrades and with a focus, rather than trusting the automatic pass to guess what matters.

The large-codebase docs add a complementary habit: scope what Claude reads in the first place (per-directory CLAUDE.md files, `claudeMdExcludes`, `Read` deny rules for generated/vendored code, code-intelligence plugins instead of scanning files), and write long cross-package plans to a plan file, which Claude Code re-injects after compaction.

##### Crash recovery with manifests

Agents crash, time out, or lose their session. If their findings only live in a transcript or in the coordinator's head, you restart from zero. The guide's pattern:

1. **Each agent exports state to a known location** as it works (for example `state/<agent_id>.json`: what it covered, findings, open questions, status).
2. The coordinator maintains (or can rebuild) a **manifest**: an index of agents, their state files, their status and what remains.
3. **On resume**, the coordinator loads the manifest, decides which agents are complete, which to re-run or continue, and **injects the relevant saved state into the new agents' prompts**.

This is an *application-level* pattern from the exam guide. No docs page describes agents exporting state to a known location or a coordinator loading a manifest, and it is not a named Claude Code or SDK feature. The documented platform pieces it can sit on top of are: SDK sessions that persist the conversation (resume by session ID; `fork_session` / `forkSession` to branch; session files are local to the machine that created them, so the docs describe a session-store adapter or moving the file for other hosts), and the Agent SDK docs' own advice for cross-host cases: if you cannot rely on session resume, "capture the results you need (analysis output, decisions, file diffs) as application state and pass them into a fresh session's prompt". That advice is in the same spirit as state export plus injection, but it is not the manifest pattern. Notes from the docs: sessions persist the *conversation*, not the filesystem (use file checkpointing for file changes); resumed Claude Code sessions do not re-run tool calls interrupted by a crash; resumed subagents keep their full history and transcripts live under the project's `subagents` folder; and a subagent that hit `maxTurns` is marked partial and can be resumed. The multi-agent research article describes the same goal: resume from where the agent was rather than restart.

### Worked examples

#### Example A. Degradation symptoms (what to look for in a transcript)

```text
Turn 12  User: Where do refunds get validated?
         Agent: In RefundValidator.validate() (src/billing/refund_validator.py:44), which checks window and SKU class.
...  (60 more turns, many file reads) ...
Turn 74  User: Remind me where refunds get validated?
         Agent: Typically refund validation lives in a service or controller layer, e.g. a RefundService...
```

Turn 74 is the tell: generic "typically" language instead of the specific class named at turn 12. Remedy: the finding should have been written to a scratchpad, and the session should be compacted (with focus) or restarted from the scratchpad.

#### Example B. Scratchpad file

Instruction in the agent's prompt:

```text
Maintain notes in .scratch/findings.md. After every significant discovery append: the
exact symbol and file:line, what it does, and what it calls. Before answering any
question about code you explored earlier, re-read .scratch/findings.md first.
Never answer from memory of "typical patterns".
```

File content:

```markdown
# Findings: refund flow (updated turn 38)
- Entry: POST /api/refunds -> RefundController.create (src/api/refund_controller.py:21)
- Validation: RefundValidator.validate (src/billing/refund_validator.py:44): window 30d, SKU class check
- Policy lookup: PolicyEngine.get("RET-2") (src/policy/engine.py:88), reads policies/returns.yaml
- Payment: PaymentGateway.refund (src/payments/gateway.py:130): idempotency key = order_id + attempt
- Tests: tests/billing/test_refund_validator.py, tests/api/test_refund_controller.py
## Open questions
- Where is the shipping-fee exclusion applied? (not in validator; check PolicyEngine callers)
```

Why it counters degradation: the findings are exact, stay out of the window until needed, and survive `/compact` and `/clear` because they are on disk.

#### Example C. Delegate verbose exploration to subagents

Custom subagent definition (`.claude/agents/code-explorer.md`):

```markdown
---
name: code-explorer
description: Answers one specific question about the codebase (find files, trace a flow). Use for exploration that would produce verbose output.
tools: Read, Grep, Glob
model: haiku
---
You answer exactly one question about this codebase. Search as needed, then return:
1. A direct answer in at most 10 lines.
2. A list of the key symbols with file:line.
3. Anything you could not determine.
Do not paste file contents. Do not include your search process.
```

Main-agent delegations (each carries its own context because the subagent will not see the conversation):

```text
Task 1: "Find all test files that exercise refund logic. Return paths only."
Task 2: "Trace the dependencies of RefundController.create down to the payment call. Return a call chain with file:line."
Task 3: "List every place the string 'shipping_fee' is excluded from refunds."
```

(The subagent-spawning tool is `Task` in the exam guide; `Agent` in current Claude Code.)

#### Example D. Summarise-then-spawn

```text
Phase 1 (3 parallel explorers: api/, billing/, policy/) -> each returns <=15 lines.
Coordinator writes PHASE1_SUMMARY:
  modules: api (controllers), billing (validator, ledger), policy (engine + yaml)
  refund entrypoint: RefundController.create -> RefundValidator -> PaymentGateway
  known gaps: shipping fee rule location unknown
Phase 2 prompt to each investigator:
  "Context from phase 1: {PHASE1_SUMMARY}. Your task: find where the shipping fee exclusion is applied. Do not re-map the modules."
```

#### Example E. Manifest-based crash recovery

(Exam-guide pattern; the file names and JSON shape below are illustrative, not a documented format.)

```json
{
  "run_id": "refund-audit-2026-10-30",
  "updated_at": "2026-10-30T14:12:09Z",
  "agents": [
    {"id": "explore-api",     "status": "complete",    "state_file": "state/explore-api.json",     "summary": "Controllers mapped; entry RefundController.create"},
    {"id": "explore-billing", "status": "complete",    "state_file": "state/explore-billing.json", "summary": "Validator + ledger mapped"},
    {"id": "explore-policy",  "status": "in_progress", "state_file": "state/explore-policy.json",  "summary": "engine mapped; yaml rules 3/7 read"},
    {"id": "trace-shipping",  "status": "pending",     "state_file": null,                         "summary": null}
  ]
}
```

Coordinator on resume:

```python
manifest = json.load(open("state/manifest.json"))
done = {a["id"]: json.load(open(a["state_file"])) for a in manifest["agents"] if a["status"] == "complete"}
for agent in manifest["agents"]:
    if agent["status"] == "complete":
        continue
    prior = json.load(open(agent["state_file"])) if agent["state_file"] else {}
    spawn_agent(
        agent["id"],
        prompt=build_prompt(agent["id"], completed_findings=done, own_prior_state=prior),  # inject, don't make them rediscover
    )
```

Each agent also writes its state file *as it goes* (not only at the end), so a crash mid-task loses little.

### Exam traps

- **"Just use a bigger context window"** for degraded exploration. Degradation is about relevance and attention, not capacity.
- **Keeping all exploration in the main conversation** "so nothing is lost". The point of subagents is isolating verbose output.
- **Relying on automatic summarisation to preserve exact class names.** Persist specifics to a scratchpad/state file.
- **Restarting the whole run after a crash** instead of loading a manifest and injecting saved state.
- **Letting phase-2 agents inherit the full phase-1 history** instead of a compact phase summary.
- **Using `/compact` as a substitute for durable notes.** Compaction is lossy; use it to relieve pressure, while facts that must stay exact live in files.
- **Assuming a subagent sees the parent's conversation.** It does not (except a fork). Put the needed context in the delegation prompt.

### Check yourself

**Q1.** After 90 minutes exploring a legacy monolith, the agent answers a question about the payment flow with "typically such systems use a PaymentProcessor class", though it found `LedgerPoster` earlier. Diagnose and fix.

<details><summary>Answer</summary>

Context degradation: earlier specifics have been crowded out or lost to summarisation, so the model falls back to generic "typical patterns". Fix: keep a scratchpad of exact findings and re-read it before answering, delegate verbose exploration to subagents so the main context stays clean, and use `/compact` (with a focus) when context is full of discovery output.
</details>

**Q2.** A four-agent investigation crashes after two agents finish. What design lets you resume without redoing finished work?

<details><summary>Answer</summary>

Each agent exports its state to a known location as it works; the coordinator keeps/loads a manifest of agent statuses and state files; on resume it skips completed agents and injects their saved findings (and any partial state of the incomplete agents) into the prompts of the agents it re-runs.
</details>

**Q3.** You need to know "which tests cover the refund module" while designing a refactor in the main session. Why use a subagent rather than grepping inline?

<details><summary>Answer</summary>

The search output is verbose and mostly not needed afterwards; a subagent does it in its own context window and returns just the short answer, so the main session keeps its context for coordination. Trade-off: it costs extra tokens and cold-start time, so for a tiny one-off lookup inline may be quicker.
</details>

### Official docs

- [Explore the context window](https://code.claude.com/docs/en/context-window.md) — what loads, what survives compaction, `/compact` with focus, `/clear`, `/context`, delegating large reads.
- [Set up Claude Code in a monorepo or large codebase](https://code.claude.com/docs/en/large-codebases.md) — layered CLAUDE.md, `claudeMdExcludes`, `Read` deny rules, code intelligence, plan files surviving compaction.
- [Create custom subagents](https://code.claude.com/docs/en/sub-agents.md) — own context window, summary return, `Agent` (formerly `Task`) tool, frontmatter fields, resuming.
- [Best practices for Claude Code](https://code.claude.com/docs/en/best-practices.md) — "Know what subagents cost", "Keep cost and context down", steering with `/clear` and `/rewind`.
- [Manage sessions](https://code.claude.com/docs/en/sessions.md) — `--continue`, `--resume`, `--fork-session`, what a resumed session restores.
- [Work with sessions (Agent SDK)](https://code.claude.com/docs/en/agent-sdk/sessions.md) — continue, resume by ID, fork, resume across hosts, "capture results as application state". (Documents session persistence only; it does not describe a manifest pattern.)
- [Memory tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool.md) — client-side, file-based memory under `/memories` (Claude requests file operations; your application runs them) and the multisession development pattern.
- [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) — compaction, structured note-taking, sub-agent architectures.
- [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) — plan saved to memory, subagents writing artifacts to external storage and passing references, resuming from where the agent was.

---

## 5.5 Design human review workflows and confidence calibration

### What the exam tests

**Knowledge of**

- Why **aggregate accuracy** (e.g. 97% overall) can **mask poor performance** on particular document types or fields.
- **Stratified random sampling** of high-confidence extractions to measure error rates and detect **novel error patterns**.
- **Field-level confidence scores calibrated on labelled validation sets** to route review attention.
- Validating accuracy **by document type and field segment before automating** high-confidence extractions.

**Skills in**

- Implement stratified random sampling of high-confidence extractions for ongoing error-rate measurement and novel-pattern detection.
- Analyse accuracy by document type and field to confirm consistent performance across segments before reducing human review.
- Have the model output **field-level confidence scores**, then calibrate review thresholds on labelled validation sets.
- Route extractions with **low model confidence** or **ambiguous/contradictory source documents** to human review, prioritising limited reviewer capacity.

### Concepts explained

This is Scenario 6 (structured data extraction): documents in, schema-validated JSON out, downstream systems depend on it, and some human review is affordable but not unlimited. The question is *which* extractions need a person.

#### Why a single accuracy number is dangerous

Aggregate accuracy is dominated by whatever is most common. If 60% of volume is clean invoices at 99.5%, the average can look excellent while a small, hard segment is badly wrong. Automation decisions are made per *case*, so you need accuracy per **segment** (document type, field, and combinations such as "handwritten forms x date fields").

Worked illustration (validation set of 10,000 labelled documents):

| Document type | Share of volume | Documents | Accuracy | Errors |
|---|---|---|---|---|
| Printed invoices | 60% | 6,000 | 99.5% | 30 |
| Receipts | 30% | 3,000 | 98.0% | 60 |
| Contracts | 7% | 700 | 88.0% | 84 |
| Handwritten forms | 3% | 300 | 60.0% | 120 |
| **Total** | 100% | 10,000 | **97.06%** | **294** |

Headline: "97% accurate". Reality: handwritten forms are 3% of documents but 41% of all errors, and contracts are 7% of documents but 29% of errors. Auto-approving "everything the model is confident about" would push those errors straight into downstream systems. The same logic applies **by field**: `total_amount` may be 99.8% while `payment_terms` is 91%.

Rule: **before reducing human review for any segment, show that segment's accuracy (per document type and per field) is consistently high.** Do not generalise from the overall number.

#### Field-level confidence, calibrated

Ask the model to output a **confidence per extracted field** (not one score per document) so review can focus on the shaky fields.

But raw model-reported confidence is not trustworthy as-is (the same reason 5.2 rejects self-reported confidence as an escalation switch). The fix is **calibration**:

1. Run the extractor on a **labelled validation set** (human-verified ground truth).
2. Group predictions into confidence buckets, per field (and ideally per document type).
3. Measure *actual* accuracy in each bucket.
4. Choose the confidence **threshold** at which actual accuracy meets your target; below it, send to a human.

Worked example, field `payment_terms`, 2,000 labelled extractions, target: at least 98% accuracy for auto-accept.

| Reported confidence | n | Correct | Actual accuracy |
|---|---|---|---|
| 0.95 - 1.00 | 900 | 891 | 99.0% |
| 0.90 - 0.95 | 500 | 470 | 94.0% |
| 0.80 - 0.90 | 350 | 301 | 86.0% |
| below 0.80 | 250 | 140 | 56.0% |

- Auto-accepting from 0.90 up gives (891 + 470) / 1,400 = **97.2%**, below the 98% target. 
- Auto-accepting from 0.95 up gives **99.0%** and automates 900 of 2,000 = **45%** of this field.
- So the threshold for `payment_terms` is **0.95**. A different field (say `total_amount`) may reach the target at 0.85; thresholds are set **per field** (and per document type where they differ).
- If reviewers can only handle 30% of volume, do not simply lower the bar: rank the review queue so the *lowest calibrated confidence and highest business impact* come first, and revisit the capacity/target trade-off explicitly.

Note what is being claimed: confidence is *useful as a ranking signal after calibration*, and its meaning is *measured*, not assumed.

#### Stratified random sampling of high-confidence output

Even calibrated thresholds drift: new vendors, new layouts, a model update. If you only review low-confidence items, you never look at the large "confident" pool where *confident-but-wrong* errors and novel failure modes hide. So continuously **sample from the auto-approved pool**:

- **Stratify** by document type (and field group, and possibly confidence band) so small or risky segments are not drowned by big ones. Give each stratum a minimum sample, and over-sample rare or high-risk strata.
- **Random within each stratum** (not "the first 50" or "the ones that look odd") so the error estimate is unbiased.
- **Measure** error rate per stratum over time, with enough sample size for the estimate to mean something. (General statistics, not from the docs: with zero errors in n = 100 random samples, the 95% upper bound on the true error rate is roughly 3/n = 3%. A clean small sample proves less than it feels like.)
- **Read the errors you find.** The second purpose of sampling is **novel pattern detection**: a cluster of errors with a shared cause (a new invoice template, a currency format) is a signal to fix prompts/schema, recalibrate, or pull that segment back to full review.
- If a stratum's error rate crosses its limit, **revert that stratum to human review** and re-validate.

#### Routing: what goes to people

Send to review when any of these holds:

- A required field's calibrated confidence is below its threshold.
- **Ambiguous or contradictory source documents**: for example the invoice total differs from the sum of line items, the PO and the invoice disagree on the vendor, two pages give different dates, or the document is illegible. Contradiction is a routing trigger even when the model is "confident".
- Schema/semantic validation fails (and a retry cannot fix it because the information is not in the document).
- The stratum is on probation (recent drift).

Prioritise by calibrated confidence and impact when capacity is limited, and show the reviewer *which field* and *why* it was flagged (value, confidence, conflicting alternatives, source location) so review is fast.

### Worked examples

#### Example A. Extraction output with field-level confidence and conflict flags

```json
{
  "document_id": "DOC-20431",
  "document_type": "invoice",
  "fields": {
    "invoice_number": {"value": "INV-77120", "confidence": 0.99, "source": "p1, header"},
    "invoice_date":   {"value": "2026-03-04", "confidence": 0.96, "source": "p1, top right"},
    "total_amount":   {"value": 1240.50, "confidence": 0.71, "source": "p2, totals box",
                       "alternatives": [{"value": 1204.50, "source": "p1, line-item sum"}]},
    "payment_terms":  {"value": null, "confidence": 0.0, "source": null, "note": "not stated"}
  },
  "validation": {"line_items_sum_equals_total": false},
  "conflicts": ["total_amount: stated total differs from line-item sum"]
}
```

Nullable `payment_terms` avoids a fabricated value (a Domain 4 principle). `conflicts` and the low `total_amount` confidence both route this document to review.

#### Example B. Routing function with per-field calibrated thresholds

```python
THRESHOLDS = {                         # illustrative values; in practice calibrated on a labelled validation set, per field
    "invoice_number": 0.85, "invoice_date": 0.90,
    "total_amount": 0.92, "payment_terms": 0.95,
}
PROBATION = {("handwritten_form", "*")}   # segments pulled back to full review

def route(doc: dict) -> dict:
    reasons = []
    if (doc["document_type"], "*") in PROBATION:
        reasons.append("segment on probation")
    for name, f in doc["fields"].items():
        if f["value"] is not None and f["confidence"] < THRESHOLDS.get(name, 0.95):
            reasons.append(f"{name}: confidence {f['confidence']:.2f} below {THRESHOLDS.get(name, 0.95)}")
    reasons += doc.get("conflicts", [])
    return {"route": "human_review" if reasons else "auto_accept", "reasons": reasons}
```

#### Example C. Stratified random sample of auto-accepted records

```python
import random
from collections import defaultdict

def weekly_sample(auto_accepted: list[dict], rate=0.02, min_per_stratum=40, seed=None):
    rng = random.Random(seed)
    strata = defaultdict(list)
    for d in auto_accepted:
        strata[d["document_type"]].append(d)          # stratum = document type (add field group if useful)
    sample = []
    for stratum, docs in strata.items():
        k = min(len(docs), max(min_per_stratum, round(rate * len(docs))))
        sample.extend(rng.sample(docs, k))             # random WITHIN each stratum
    return sample
```

Then: humans label the sample, you compute error rate per stratum and per field, you tabulate error *types*, and you act on thresholds (tighten, recalibrate, or pull the stratum back to review).

### Exam traps

- **"97% overall accuracy, so we can automate."** The guide's headline trap. Segment first.
- **Reviewing only low-confidence items.** You will never discover confident-but-wrong or novel patterns. Sample the high-confidence pool.
- **Simple random sampling across everything.** Small high-risk segments get too few samples. Stratify.
- **One document-level confidence number** or **trusting raw self-reported confidence** without a labelled calibration set.
- **A single global threshold** for all fields and document types.
- **Treating contradictory sources as "pick the likelier one".** Contradiction goes to review (or is annotated; see 5.6).
- **Calibrating once and forgetting it.** Ongoing sampling is what detects drift.

### Check yourself

**Q1.** An extractor reports 96.5% accuracy overall. Management wants to remove human review. What do you do first?

<details><summary>Answer</summary>

Break accuracy down by document type and by field (using a labelled validation set) to find weak segments, and only reduce review where every relevant segment is consistently strong. Keep ongoing stratified random sampling of auto-approved output to measure error rates and catch new patterns.
</details>

**Q2.** From a labelled validation set you find that for field `due_date`, extractions with confidence at or above 0.90 are 93% correct and those at or above 0.97 are 99% correct. Your target is 98%. What threshold and why?

<details><summary>Answer</summary>

0.97: it is the only level in the data that meets the 98% target (0.90 gives 93%). Lowering it toward 0.90 is not justified by what was measured; you would first measure intermediate bands (and check bucket sizes) before choosing a threshold below 0.97. The model's reported confidence is meaningful only through this measured relationship; thresholds are set per field from validation data, not assumed.
</details>

**Q3.** Why sample from high-confidence extractions at all, given that they are not sent to review?

<details><summary>Answer</summary>

Because confident errors and new failure modes are invisible if you only inspect flagged items. A stratified random sample gives an unbiased error-rate estimate per segment and lets reviewers spot novel error patterns; the result feeds recalibration or putting a segment back under review.
</details>

### Official docs

- Exam guide v1.0, Domain 5 task statement 5.5 and Scenario 6 (no dedicated product docs page covers this procedure).
- [Reduce hallucinations](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations.md) — allow "I don't know", quote-grounding, citations/verification and best-of-N consistency checks as complementary signals (they reduce, not eliminate, errors).
- [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) — "Effective evaluation": start with small samples, LLM-as-judge with a rubric, and human evaluation that catches what automation misses (for example sources of poor quality).

(The docs fetched for this domain describe the building blocks; the stratified-sampling and calibration procedures are the exam guide's patterns rather than a documented product feature.)

---

## 5.6 Preserve information provenance and handle uncertainty in multi-source synthesis

### What the exam tests

**Knowledge of**

- How **source attribution is lost** when findings are summarised without preserving claim-to-source mappings.
- The need for **structured claim-source mappings** that the synthesis agent preserves and merges.
- How to handle **conflicting statistics from credible sources**: annotate the conflict with source attribution instead of arbitrarily picking one value.
- **Temporal data**: require publication or collection dates in structured outputs so differences over time are not mistaken for contradictions.

**Skills in**

- Require subagents to output **claim-source mappings** (source URLs, document names, relevant excerpts) that downstream agents preserve through synthesis.
- Structure reports with explicit sections that separate **well-established** from **contested** findings, preserving original source characterisations and methodological context.
- Complete document analysis with **conflicting values included and explicitly annotated**, leaving the coordinator to decide how to reconcile before synthesis.
- Require **publication/data-collection dates** in subagent output for correct temporal interpretation.
- **Render content types appropriately**: financial data as tables, news as prose, technical findings as structured lists, not one uniform format.

### Concepts explained

#### How provenance gets lost

Each hop in a multi-agent pipeline is a chance to drop the "where did this come from?" Typical failure: a search subagent returns bullets with URLs; a summariser compresses them to "Studies show AI adoption in music rose sharply"; the synthesis agent writes a fluent paragraph with no citations. The claim may be right, but now nobody (including the final citation step) can verify it, weigh its source, or notice that two numbers came from different years.

The fix is structural: **every claim travels as a record** `claim + source + excerpt + date + method`, and every downstream agent is *required* to carry the record forward and merge records when claims coincide. The multi-agent research article's design reflects this: a dedicated **CitationAgent** attributes claims to sources at the end, and subagents pass lightweight references to stored artifacts instead of paraphrasing everything through the coordinator. The hallucination guidance adds the verification habit: have the model cite a supporting quote for each claim and **retract any claim for which it cannot find one**, and extract word-for-word quotes first for long documents.

#### Claim-source mapping schema

```json
{
  "claim_id": "C-017",
  "claim": "Freelance illustrator income fell 14% year over year",
  "value": {"number": -14, "unit": "percent", "metric": "median freelance income change"},
  "source": {
    "type": "survey",
    "title": "Creative Freelancers Income Survey 2025",
    "url": "https://example.org/cfis-2025",
    "document_name": "cfis-2025.pdf",
    "location": "p. 18, Table 4",
    "publisher_characterisation": "self-reported, members of two unions"
  },
  "excerpt": "Median income for illustrators declined 14% compared with 2024 (Table 4).",
  "publication_date": "2025-09-15",
  "data_collection_period": "2025-03 to 2025-06",
  "method": {"sample_size": 1204, "design": "online survey, non-random"},
  "found_by": "doc-analysis-2"
}
```

Merge rule for synthesis: two records with the same claim and compatible values become one claim with **multiple sources**; incompatible values become a **conflict record** (below), never a silent choice.

#### Conflicts: annotate, do not pick

Two credible sources give different numbers. The tempting shortcut is to choose "the more recent", "the bigger sample", or "the average". Those are judgments that depend on context the subagent lacks. The guide's pattern:

1. The analysis subagent **includes both values**, each with its source, date and method, and flags the conflict.
2. The **coordinator** decides how to reconcile (ask a verification agent, prefer one with stated rationale, or present both).
3. The final report shows the disagreement explicitly rather than hiding it.

```json
{
  "conflict_id": "X-003",
  "topic": "Change in freelance illustrator income",
  "values": [
    {"claim_id": "C-017", "value": "-14%", "source": "Creative Freelancers Income Survey 2025", "date": "2025-09-15", "method": "online survey, n=1,204"},
    {"claim_id": "C-052", "value": "-5%",  "source": "Platform Earnings Report 2026",          "date": "2026-02-10", "method": "platform payout data, all active freelancers"}
  ],
  "possible_explanations": ["different populations (members vs all platform users)", "different periods", "self-reported vs transactional"],
  "temporal_difference": false,
  "status": "needs_reconciliation"
}
```

#### Temporal data: dates prevent fake contradictions

"Market share is 12%" in one source and "18%" in another might be a contradiction, or it might be 2023 vs 2025. If subagents omit dates, synthesis cannot tell and will either "reconcile" a non-conflict or report change as error. Require `publication_date` (when the source was written) and `data_collection_period` (when the numbers were measured; they can differ by a year or more) in every structured finding. Then:

- Different dates, same metric and population: report as a **trend**, not a contradiction.
- Same date and population, different values: a genuine **conflict**.
- Unknown dates: say so ("date not stated") rather than assuming.

#### Report structure: established vs contested

Structure the report so the reader can tell what is solid:

```text
## Well-established findings
(multiple independent sources agree; each claim lists its sources)

## Contested findings
(sources disagree; both values, sources, dates and methods shown; coordinator note on likely reasons)

## Single-source / low-confidence findings
(one source, or weak method; flagged)

## Coverage gaps
(topics where sources were unavailable; see 5.3)
```

Preserve each source's own characterisation ("self-reported", "industry estimate") and method; do not smooth them into confident, uniform prose.

#### Render each content type appropriately

Do not flatten everything into the same bullet-point format:

| Content | Render as | Why |
|---|---|---|
| Financial / numeric comparisons (revenue, margins, growth by year) | **Table** | Readers compare across rows and columns. |
| News, events, narrative context | **Prose** | Causality and sequence read naturally. |
| Technical findings (APIs, vulnerabilities, versions, steps) | **Structured list** | Discrete items, easy to scan and reference. |

Mixed reports use all three, each where it fits, with citations in all of them.

### Worked examples

#### Example A. Attribution lost vs preserved

**Bad (summarisation step):**

```text
Subagent returns: [ {"claim": "AI tools used by 42% of producers", "url": "https://example.org/survey"}, ... ]
Summariser output: "A large share of music producers now use AI tools."
Final report:     "Many producers use AI tools." (no source, no number, no date)
```

**Good (claim-source records survive every hop):**

```text
Summariser output (structured): the same records, grouped by theme, with claim_id, source, date preserved.
Final report: "AI-assisted tools were used by 42% of surveyed producers in 2025 [C-001: Producer Tools Survey, 2025-11-02, n=1,204, self-reported]."
```

#### Example B. Dates dissolving a "contradiction"

```text
Source A (published 2023-04, data from 2022):  "Remote work among developers: 61%"
Source B (published 2025-02, data from 2024):  "Remote work among developers: 48%"
```

With dates the finding is "remote work declined from 61% (2022) to 48% (2024)", a trend. Without dates the pipeline flags a contradiction or picks one. Only same-period disagreement (e.g. two 2024 figures of 48% and 57%) is a genuine conflict to annotate.

#### Example C. Document-analysis subagent prompt

```text
You are analysing documents for a research report. For each factual claim you extract, return a
JSON record with: claim, value, source (url, document_name, location), an exact excerpt,
publication_date, data_collection_period (or "not stated"), and method (sample, design).
If two documents give different values for the same metric, include BOTH values in a
"conflicts" entry with their sources and dates. Do not choose between them and do not average.
Finish analysing all documents even when you find conflicts; the coordinator will reconcile.
If you cannot find a supporting excerpt for a claim, omit the claim.
```

#### Example D. Final-report fragment with mixed rendering

```text
### Market snapshot (financial data: table)
| Segment | 2023 revenue (USD bn) | 2025 revenue (USD bn) | Source |
|---|---|---|---|
| Music tools | 1.2 | 2.9 | [C-021] Industry Report 2025 (analyst estimate) |
| Film tools  | 0.8 | 1.5 | [C-024] Vendor filings 2025 (reported) |

### What happened (news: prose)
In January 2026 two major labels announced licensing deals with AI music firms, following months of litigation [C-031, C-032].

### Technical findings (structured list)
- Model watermarking: detection rates fall below 70% after re-encoding [C-040, lab benchmark, 2025-08]
- Licensing metadata: no common standard in use [C-041, working group paper, 2026-01]

### Contested
Freelancer income change: -14% (CFIS 2025 survey, self-reported) vs -5% (Platform Earnings Report 2026, payout data). Populations differ; see conflict X-003.
```

### Exam traps

- **"Pick the more authoritative/recent/larger-sample figure and drop the other."** The guide wants both values annotated with sources; reconciliation is the coordinator's decision.
- **"Average the conflicting numbers."** Fabricates a value no source reported.
- **"Have the document-analysis agent stop and ask when it finds a conflict."** The guide: complete the analysis with conflicts included and annotated.
- **Omitting dates to save tokens**, then misreading a trend as a contradiction.
- **A summariser that "cleans up" source wording** and removes attribution or methodology.
- **Citing only at the end** with no claim-level records upstream: the citation step has nothing reliable to attach.
- **One uniform output format for everything** (all bullets, or all prose).

### Check yourself

**Q1.** Two subagents report different adoption rates (31% vs 38%) for the same technology. One source is dated 2023, the other 2025. What should synthesis do?

<details><summary>Answer</summary>

Check the dates and metric definitions first: if the figures differ because they were measured at different times, present them as a change over time (a trend), not a contradiction. If the date, population and definition match, annotate a genuine conflict with both values and sources and let the coordinator reconcile. Never silently choose or average.
</details>

**Q2.** What must a subagent's structured output contain so the synthesis agent can preserve provenance?

<details><summary>Answer</summary>

For each claim: the claim and value, the source (URL/document name and location in it), a relevant excerpt, publication and/or data-collection date, and method/context (sample, design, how the source characterises itself). Plus explicit conflict entries when values disagree.
</details>

**Q3.** A report has quarterly revenue figures for six companies, a recap of last month's merger news and a list of five API changes. How should each be presented?

<details><summary>Answer</summary>

Revenue figures as a table, the merger recap as prose, and the API changes as a structured list, each with citations, rather than forcing a single format.
</details>

### Official docs

- [Reduce hallucinations](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations.md) — allow uncertainty, quote first, cite every claim and retract unsupported ones.
- [Prompting best practices: long context prompting](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices.md) — wrap documents with `source` metadata, ground answers in quotes; the research-and-information-gathering tips (verify across sources).
- [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) — CitationAgent, artifacts and references, evaluation of citation accuracy and source quality.
- [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) — sub-agents returning condensed summaries (so provenance must be built into what they return).

---

## Decision cheat-table

| If the question says ... | Prefer ... | Because ... |
|---|---|---|
| Agent forgets amounts/dates/order IDs after many turns or summaries | Persistent **case-facts block** re-sent each request, outside summarised history | Summaries are lossy for exact values; structured facts are deterministic. |
| Customer has two or more issues in one session | Separate **per-issue structured layer** | Prevents facts crossing between issues. |
| Tool returns 40 fields, 5 matter, context fills up | **Trim at source or with a PostToolUse hook** (whitelist fields, keep errors) | Cuts tokens before they accumulate; age-based clearing does not choose fields. |
| Synthesis ignores findings from the middle of a big aggregated prompt | **Key-findings summary first + explicit section headers** | Position effects: start and end are used more reliably. |
| Downstream agent has a small context budget | Upstream agents return **structured facts, citations, relevance scores, dates** | Not reasoning chains; compact and mergeable. |
| Customer says "get me a human" | **Escalate immediately**, no investigation | Honour the explicit request. |
| Frustrated customer, issue is within agent policy and tools | **Acknowledge, offer resolution**; escalate only if they repeat the request | Frustration is not a request for a human. |
| Policy silent or ambiguous (e.g. competitor price match) | **Escalate** | Policy gap; do not invent policy. |
| Agent escalates easy cases and mishandles hard ones | **Explicit criteria + few-shot examples** in the system prompt | Root cause is unclear boundaries; cheapest effective fix. Not sentiment, self-confidence or a new classifier. |
| `get_customer` returns several matches | **Ask for additional identifiers** | Heuristic selection risks wrong-account actions. |
| Subagent times out | **Structured error** (type, query, partial results, alternatives) after local retries | Lets the coordinator choose recovery; not generic status, not empty success, not abort. |
| Search returned zero hits vs search failed | Report as **valid empty result** vs **access failure** | They mean different things for coverage claims. |
| Final report built from partial sources | **Coverage annotations** (supported vs gap) | Honest about what is missing. |
| Long exploration; agent says "typically..." not specific classes | **Scratchpad file**, subagent delegation, `/compact` with focus | Counters context degradation; exact findings live on disk. |
| Multi-phase investigation | **Summarise phase output, inject into next phase's subagents** | Avoids re-exploration and bloated inheritance. |
| Agent crashed mid-investigation | **State export per agent + manifest** loaded on resume and injected into prompts | Resume without redoing work. |
| "97% accurate overall, remove review?" | **Segment accuracy by document type and field first**; keep stratified sampling | Aggregates mask weak segments. |
| Want to route extractions to humans | **Field-level confidence calibrated on a labelled validation set**; route low-confidence and contradictory-source items | Calibrated thresholds, scarce reviewer capacity prioritised. |
| Sources disagree on a statistic | **Include both values with sources, dates, methods**; coordinator reconciles | No arbitrary pick or average. |
| Numbers differ and dates differ | Check **publication/collection dates** before calling it a conflict | Temporal difference is not contradiction. |
| Report mixes financial data, news and technical findings | **Table / prose / structured list** respectively | Match format to content type. |

---

## Sources fetched

All fetched on 2026-10-30.

- https://platform.claude.com/docs/en/build-with-claude/context-windows.md
- https://platform.claude.com/docs/en/build-with-claude/context-editing.md (complete page checked in the fact-check pass, including client-side SDK compaction and "Availability"; code samples skimmed)
- https://platform.claude.com/docs/en/build-with-claude/compaction.md
- https://platform.claude.com/docs/en/build-with-claude/compaction-on-demand.md (complete page checked in the fact-check pass: request, continue-from-summary, `instructions`, errors, limits, availability; code samples skimmed. The separate "Compaction at a token threshold" page was not fetched)
- https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool.md (overview, prompting guidance, security, error handling, compaction/context-editing pairing, multisession pattern; the handler implementation samples and per-command reference were not read line by line)
- https://platform.claude.com/docs/en/agents-and-tools/tool-use/manage-tool-context.md
- https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices.md (long-context prompting; context awareness, multiwindow workflows, state management; research and subagent orchestration)
- https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations.md
- https://platform.claude.com/docs/en/api/errors.md
- https://code.claude.com/docs/en/context-window.md
- https://code.claude.com/docs/en/large-codebases.md
- https://code.claude.com/docs/en/sessions.md
- https://code.claude.com/docs/en/best-practices.md (context and subagent-cost sections, "Steer the session" and "What you can stop doing")
- https://code.claude.com/docs/en/sub-agents.md (complete page checked in the fact-check pass, including forks, "Manage subagent context", resuming and "API errors in subagents")
- https://code.claude.com/docs/en/agent-sdk/sessions.md
- https://code.claude.com/docs/en/agent-sdk/user-input.md
- https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
- https://www.anthropic.com/engineering/multi-agent-research-system
- https://code.claude.com/docs/en/hooks.md (extra page, used only for the PostToolUse input and decision-control passages, `updatedToolOutput` / `updatedMCPToolOutput`, and MCP tool matcher naming)

Official exam guide v1.0 (July 2026), Domain 5 (task statements 5.1-5.6), Scenarios 1-6 and sample questions 3, 7 and 8, was used as the primary authority.
