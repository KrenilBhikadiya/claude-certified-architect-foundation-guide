# Domain 2: Tool Design & MCP Integration (18%)

> This domain is about the **interface between the model and the outside world**: how you describe tools so Claude picks the right one, how tools report failure so the agent can recover intelligently, how many tools (and which ones) each agent gets, how MCP servers are wired into Claude Code and the Agent SDK, and how to use the built-in file/search tools well. It carries 18% of the exam because almost every agentic scenario depends on it: the **Customer Support Resolution Agent** (`get_customer`, `lookup_order`, `process_refund`, `escalate_to_human`), the **Multi-Agent Research System** (scoped subagent tool lists, error propagation, `verify_fact`), and **Developer Productivity with Claude** (built-in tools plus MCP servers) all lean on it.

## Contents

- [Mental model](#mental-model)
- [Terminology and currency notes](#terminology-and-currency-notes)
- [2.1 Design effective tool interfaces with clear descriptions and boundaries](#21-design-effective-tool-interfaces-with-clear-descriptions-and-boundaries)
- [2.2 Implement structured error responses for MCP tools](#22-implement-structured-error-responses-for-mcp-tools)
- [2.3 Distribute tools appropriately across agents and configure tool choice](#23-distribute-tools-appropriately-across-agents-and-configure-tool-choice)
- [2.4 Integrate MCP servers into Claude Code and agent workflows](#24-integrate-mcp-servers-into-claude-code-and-agent-workflows)
- [2.5 Select and apply built-in tools (Read, Write, Edit, Bash, Grep, Glob) effectively](#25-select-and-apply-built-in-tools-read-write-edit-bash-grep-glob-effectively)
- [Decision cheat-table](#decision-cheat-table)
- [Sources fetched](#sources-fetched)

## Mental model

A tool is a **contract between a deterministic system (your code) and a non-deterministic reader (the model)**. The model never sees your implementation. It sees exactly three things: the tool's **name**, its **description**, and its **input schema**. Everything in this domain follows from that.

```
                 ┌──────────────── what Claude sees ────────────────┐
 user request -> │ name + description + input_schema  (per tool)     │ -> chooses a tool
                 │ + your system prompt (same prompt, same context)  │
                 └───────────────────────────────────────────────────┘
                                       │ tool_use {name, input}
                                       ▼
                         your code / MCP server executes
                                       │ tool_result {content, is_error}
                                       ▼
                 Claude reads the result (including the error text!) and decides what to do next
```

Rules of thumb for this domain:

1. **Descriptions are the primary selection mechanism.** If two tools get confused, the first and cheapest fix is a better description (input formats, example queries, edge cases, boundaries versus the neighbouring tool), not few-shot examples, a routing layer, or a merge.
2. **Fewer, sharper tools beat many vague ones.** Each tool should have one clear purpose with a defined input/output contract. Overlapping names and descriptions cause misrouting.
3. **A tool failure is information.** Return it as a structured error (`isError: true` plus category, retryability, human-readable message) so the agent can choose between retry, fix-the-input, explain-to-customer, or escalate. A generic "Operation failed" removes that choice.
4. **Never confuse "failed" with "found nothing".** An empty result from a successful query is a valid answer; an access failure is an error. Mixing them makes the agent either give up on good data or trust missing data.
5. **Scope tools to roles (least privilege).** Around 4-5 relevant tools per agent selects better than 18. A synthesis agent with a web-search tool will use it. Give cross-role access only as a narrow, constrained tool (`verify_fact`) and route the hard 15% through the coordinator.
6. **`tool_choice` controls whether and which tool is called**: `auto` (model decides), `any` (must call some tool), forced `{"type":"tool","name":...}` (must call that tool), `none` (no tools). Use it for guarantees about *the first step*, not for long-term workflow ordering.
7. **MCP: tools act, resources inform, prompts are user-invoked.** Put shared team servers in project-scoped `.mcp.json` (secrets via `${VAR}`), personal/experimental ones in user scope (`~/.claude.json`). Use resources as a catalog so the agent does not burn tool calls on exploration. Prefer existing community servers for standard integrations.
8. **Pick the built-in tool that matches the job**: Grep searches *contents*, Glob matches *paths*, Read/Write handle *whole files*, Edit does *targeted unique-match replacement*, Bash is the general escape hatch. Build understanding incrementally (Grep for entry points, then Read to follow imports) instead of reading everything up front.

The reasoning pattern the exam rewards across all five task statements: **fix the root cause with the lowest-effort, highest-leverage change first** (usually the description or the tool boundary), and **use programmatic enforcement rather than prompt wording when compliance must be guaranteed**.

## Terminology and currency notes

Where today's documentation (fetched 2026-10-30) differs from the exam guide's wording, **exam answers follow the guide's wording**, but you should know both:

| Exam guide says | Current docs say | What to do |
| --- | --- | --- |
| The `Task` tool spawns subagents | The tool is now called **`Agent`** in `tool_use` blocks and in `allowedTools`; it still appears as `"Task"` in the `system:init` tools list, and older versions emitted `"Task"`. SDK docs say to match both names. | Treat `Task` (exam) and `Agent` (current) as the same subagent-spawning tool. Do not confuse it with the separate task-tracking tools (`TaskCreate`, `TaskList`, ...) listed in the Claude Code tools reference. |
| `tool_choice`: `"auto"`, `"any"`, forced `{"type":"tool","name":"..."}` | A fourth option, `none`, exists. **Forced tool use (`any` and `tool`) is not supported on some current models** (the Define tools page lists Claude Opus 5.5, Sonnet 5.5, Fable 5.1 and Mythos 5.1 as returning a 400 error) **and not with manual extended thinking**; the page's alternative is `auto` plus strict tool use, or structured outputs. | Learn the three modes exactly as the guide states them; know that `none` exists and that model support for forcing varies (check the page for your model). |
| User-level MCP config is `~/.claude.json` | Same: user scope is stored in `~/.claude.json`. Also, the default **local** scope is stored in `~/.claude.json` too (under the project's path). Precedence is local, then project, then user. | Matches the guide. Know the three scopes, not just two. |
| Tools from all configured MCP servers are discovered at connection time and available simultaneously | Still true in effect, but **Claude Code now defers MCP tool *definitions* by default** (tool search): only tool names and server instructions load at session start, and full schemas load on demand (on Opus 4, Sonnet 4, Haiku 4.5 and later models; earlier models load everything upfront). | Answer with the guide's wording (all servers' tools are available together); know tool search is why many servers no longer bloat context. |
| Built-ins: Read, Write, Edit, Bash, Grep, Glob | Same tools. On macOS, Linux and WSL, Claude Code leaves **Glob and Grep out of the default tool set** and searches with `find`/`grep` through Bash (they come back if you name them in `--tools`/`--allowedTools`, if Bash is removed from the session, or if a subagent lists them in `tools` without Bash). On Windows they are default. | Exam scenarios treat Grep and Glob as available built-ins. |
| `isError`, `errorCategory`, `isRetryable` | MCP defines the **`isError`** flag for tool failures. `errorCategory`, `isRetryable`, `retriable` are *design conventions* you add to the result body; the MCP spec has no such fields. | Know which part is protocol (`isError`) and which is your own metadata. |

---

## 2.1 Design effective tool interfaces with clear descriptions and boundaries

### What the exam tests

- **Knowledge**
  - Tool descriptions are the *primary* mechanism an LLM uses to pick a tool. Minimal descriptions ("Retrieves customer information") lead to unreliable selection among similar tools.
  - A good description includes input formats, example queries, edge cases, and boundary explanations (when to use this tool *versus* its neighbours).
  - Ambiguous or overlapping descriptions cause misrouting (the guide's example: `analyze_content` vs `analyze_document` with near-identical descriptions).
  - System prompt wording matters: keyword-sensitive instructions can create unintended tool associations that override well-written descriptions.
- **Skills**
  - Write descriptions that differentiate each tool's purpose, expected inputs, outputs, and when to use it versus similar alternatives.
  - Rename tools and update descriptions to remove functional overlap (`analyze_content` becomes `extract_web_results` with a web-specific description).
  - Split a generic tool into purpose-specific tools with defined input/output contracts (`analyze_document` becomes `extract_data_points`, `summarize_content`, `verify_claim_against_source`).
  - Review system prompts for keyword-sensitive instructions that might override good tool descriptions.

### Concepts explained

#### The description is the interface

When you call the API with tools, the platform builds a special system prompt from your tool definitions, tool configuration and your own system prompt (the Define tools page shows this template). Two consequences:

1. The model has **no other information** about a tool than its name, description and schema. It cannot infer behaviour from your code.
2. **Tool definitions and your system prompt live in the same prompt** (the page says the API assembles them into one prompt). The exam's reasoning is that they therefore compete: a strongly worded sentence in the system prompt can outweigh a thin tool description (see "keyword-sensitive instructions" below; this competition is exam-guide reasoning, not a statement on the docs page).

The official guidance on definitions says detailed descriptions are "by far the most important factor in tool performance". It asks you to explain: what the tool does; when it should be used *and when it should not*; what each parameter means and how it changes behaviour; and important caveats or limitations, including what information the tool does **not** return. It suggests at least 3-4 sentences per tool, more for complex tools. The exam guide turns this into a checklist you should memorize:

| Include in the description | Example for `lookup_order` |
| --- | --- |
| Purpose and output | "Returns order status, line items, totals, shipment tracking and refund eligibility." |
| **Input formats** | "Order number as digits, 5-8 characters, with or without a leading `#` (e.g. `12345`, `#12345`)." |
| **Example queries** it handles | "Use for: 'where is my order #12345', 'was order 88231 delivered', 'can I return the blender I bought last week'." |
| **Edge cases** | "Orders older than 24 months return `archived: true` with limited fields. An unknown order number returns `found: false`, not an error." |
| **Boundaries versus similar tools** | "Does NOT look up customers by name or email; use `get_customer` for that. Requires a verified `customer_id`." |

For tools with complex or format-sensitive inputs, the API also supports an optional **`input_examples`** array (each example must validate against `input_schema`, otherwise the API returns a 400; adds roughly 20-50 tokens for simple examples and 100-200 for complex nested ones). Descriptions come first; examples supplement them. A user-defined tool is `name` + `description` + `input_schema` (+ optional `input_examples`, `strict`, ...); the API requires `name` to match `^[a-zA-Z0-9_-]{1,128}$` (the MCP spec's tool-name rules are similar and also allow `.`).

Other design guidance from the docs and Anthropic's "Writing tools for agents" article (all consistent with the exam):

- **Name parameters unambiguously** (`customer_id`, not `id` or `user`).
- **Namespace** tool names by service or resource when you have many (`github_list_prs`, `slack_send_message`, `asana_projects_search`); it also helps tool search.
- **Return only high-signal information**: stable, readable identifiers instead of opaque internals; a `response_format` option (`concise`/`detailed`) where useful.
- **Consolidate by workflow, not by API endpoint.** Prefer `schedule_event` over `list_users` + `list_events` + `create_event`, or `get_customer_context` over three separate lookups. More tools are not automatically better, and overlapping tools "distract" agents.
- **Make error messages actionable** (covered in 2.2).
- **Iterate with evaluations**: run realistic tasks, read transcripts, and look at which tool calls were redundant or had invalid parameters. Small description edits can have large effects.

#### How overlap causes misrouting

Two tools whose descriptions are near-identical give the model nothing to discriminate on, so selection becomes a coin flip that varies with phrasing. The exam's version:

- `get_customer`: "Retrieves customer information"
- `lookup_order`: "Retrieves order details"
- Both accept similar identifier formats, so a user message like "check my order #12345" is plausibly either one. Logs show `get_customer` being called for order questions.

The troubleshooting page states the cause and cure in one line: *description ambiguity; differentiate tools by WHEN to use them, not only WHAT they do.*

#### The fix ladder (lowest effort first)

When selection is unreliable, climb this ladder and stop at the first rung that solves the problem:

| Rung | Change | When it is the right answer |
| --- | --- | --- |
| 1 | **Expand descriptions** (formats, examples, edge cases, boundaries) | The tools are semantically distinct but described thinly. This is the "most effective *first step*" in the official sample question. |
| 2 | **Rename + rewrite** to remove overlap (`analyze_content` to `extract_web_results`) | The *name itself* implies the wrong scope, or two tools sound the same. |
| 3 | **Split** a generic tool into purpose-specific tools with defined contracts | One tool is doing several different jobs with different inputs/outputs (extract vs summarize vs verify). |
| 4 | **Consolidate** overlapping tools into one | Two tools genuinely do the same job (e.g. `lookup_entity`); valid but more effort than a "first step" warrants. |
| (no) | Few-shot routing examples, keyword-routing layer | Few-shot adds token overhead without fixing the cause; a pre-routing layer is over-engineered and bypasses the model's language understanding. |

**"Consolidate" vs "split" is not a contradiction.** The docs recommend fewer, more capable tools (group `create_pr`/`review_pr`/`merge_pr` into one tool with an `action` parameter; build workflow-level tools). The exam guide recommends *splitting* `analyze_document`. The shared principle is: **each tool has one clear, distinct purpose with a well-defined contract.** Merge when you have several thin, near-identical tools or a chain of low-level calls that always occur together; split when one vague tool hides several different jobs with different inputs and outputs.

#### Keyword-sensitive system prompt wording

The system prompt and the tool descriptions are read together. If the system prompt contains blanket instructions built from words that echo one tool's description, the model can develop an unintended association.

Example (customer support; illustrative wording, not from the exam or docs): a system prompt line "Always check the customer's account details before answering any question" contains "customer" and "account", which echo `get_customer`'s description. The model now calls `get_customer` first for *every* message, including pure order-status questions where `lookup_order` is the right tool, regardless of how good the `lookup_order` description is. The fix is not to bolt on more rules; it is to **review the prompt for keyword collisions** and restate the policy in terms of the workflow ("Identify the customer with `get_customer` before any refund or account change; for order status questions with an order number, use `lookup_order`").

This mechanism is described in the exam guide rather than on a specific docs page; the docs page that supports it is the tool-use system-prompt construction on Define tools (tool definitions and your system prompt are assembled into one prompt).

#### Ordering is a different problem

If the issue is that the agent skips a *required* step (e.g. calls `lookup_order` without verifying the customer, 12% of the time), better descriptions will not give a guarantee. The official sample (Q1) answers this with a **programmatic prerequisite** that blocks `lookup_order` and `process_refund` until `get_customer` returned a verified ID. Rule: **selection problems are fixed in descriptions; ordering/compliance problems are fixed in code.**

### Worked examples

#### Example 1: bad vs good definitions for `get_customer` and `lookup_order`

Bad (the situation in the exam's Q2):

```json
[
  {
    "name": "get_customer",
    "description": "Retrieves customer information",
    "input_schema": {
      "type": "object",
      "properties": { "id": { "type": "string" } },
      "required": ["id"]
    }
  },
  {
    "name": "lookup_order",
    "description": "Retrieves order details",
    "input_schema": {
      "type": "object",
      "properties": { "id": { "type": "string" } },
      "required": ["id"]
    }
  }
]
```

Problems: identical parameter names, no formats, no examples, no edge cases, no boundary between the tools.

Good:

```json
[
  {
    "name": "get_customer",
    "description": "Look up ONE customer account and return a verified customer_id, name, email, tier (standard/gold/platinum) and account status. Accepts either a customer ID (format C- followed by 6 digits, e.g. C-104233) or an exact email address. Use this FIRST in any conversation that involves a refund, an account change, or order access, because it produces the verified customer_id that lookup_order and process_refund require. Does NOT accept order numbers and does NOT return order data; if the user only gives an order number such as #12345, call lookup_order instead. Name-only lookups are not supported: ask the user for an email. If several accounts match, returns status 'ambiguous' with up to 5 masked candidates rather than guessing. A customer that does not exist returns found: false (this is a valid result, not an error).",
    "input_schema": {
      "type": "object",
      "properties": {
        "customer_id": { "type": "string", "pattern": "^C-[0-9]{6}$", "description": "Customer ID, e.g. C-104233. Provide this OR email." },
        "email": { "type": "string", "description": "Exact account email, e.g. jane.doe@example.com. Provide this OR customer_id." }
      }
    },
    "input_examples": [
      { "customer_id": "C-104233" },
      { "email": "jane.doe@example.com" }
    ]
  },
  {
    "name": "lookup_order",
    "description": "Return the details of ONE order: status (processing/shipped/delivered/returned), line items, totals, shipment tracking and refund eligibility. Input is an order number of 5 to 8 digits, with or without a leading # (e.g. 12345 or #12345), plus the verified customer_id from get_customer. Use this whenever the user mentions an order number, delivery status, tracking, or wants to return or refund a purchase. Does NOT return customer profile data; use get_customer for that. Orders older than 24 months return archived: true with limited fields. An unknown order number returns found: false (valid result, not an error). Orders that belong to a different customer return a permission error.",
    "input_schema": {
      "type": "object",
      "properties": {
        "order_number": { "type": "string", "pattern": "^#?[0-9]{5,8}$", "description": "Order number, e.g. 12345 or #12345." },
        "customer_id": { "type": "string", "pattern": "^C-[0-9]{6}$", "description": "Verified customer ID returned by get_customer." }
      },
      "required": ["order_number", "customer_id"]
    },
    "input_examples": [
      { "order_number": "#12345", "customer_id": "C-104233" }
    ]
  }
]
```

Note what changed: formats, example queries, edge cases, and an explicit **boundary sentence in each tool pointing at the other**. The schema also differentiates the parameters (`customer_id` vs `order_number`) and uses `pattern` to constrain formats. This is the "low-effort, high-leverage" fix the exam wants as the first step.

#### Example 2: rename to remove overlap (`analyze_content` to `extract_web_results`)

In the Multi-Agent Research scenario (illustrative setup; the exam guide gives only the two tool names), suppose the web-search subagent has `analyze_content` and the document-analysis subagent (or a shared coordinator toolset) has `analyze_document`, both described as "Analyzes content and returns key information". The model then alternates.

```json
{
  "name": "extract_web_results",
  "description": "Extract structured findings from the HTML or text of a WEB PAGE or search result that has already been fetched. Input: the page text plus the research question. Output: a list of {claim, quote, url, published_date} items, at most 10, most relevant first. Use for web articles, news, blog posts and search snippets. Do NOT use for PDFs, reports or uploaded files; use extract_data_points for those. Returns an empty list (not an error) if the page contains nothing relevant.",
  "input_schema": {
    "type": "object",
    "properties": {
      "page_text": { "type": "string" },
      "source_url": { "type": "string", "description": "URL the text came from, kept for citation." },
      "question": { "type": "string", "description": "The research question guiding extraction." }
    },
    "required": ["page_text", "source_url", "question"]
  }
}
```

The name now says *what it does* (extract) and *on what* (web results); the description states the boundary against the document tool.

#### Example 3: split a generic tool (`analyze_document` into three)

```json
[
  {
    "name": "extract_data_points",
    "description": "Pull specific named values (dates, amounts, names, statistics) out of ONE loaded document. Input: document_id and a list of field names. Output: one {field, value, page, quote} per field; fields not found are returned with value null. Use when the question is 'what does the document say the X is'. Does not summarize and does not judge truth.",
    "input_schema": {
      "type": "object",
      "properties": {
        "document_id": { "type": "string" },
        "fields": { "type": "array", "items": { "type": "string" }, "description": "e.g. [\"publication_date\", \"sample_size\"]" }
      },
      "required": ["document_id", "fields"]
    }
  },
  {
    "name": "summarize_content",
    "description": "Produce a faithful summary of ONE loaded document or a page range. Input: document_id, optional page range, max_words (default 200). Output: summary text plus the page ranges it covers. Use for 'what is this document about'. Does not extract specific values and does not verify claims.",
    "input_schema": {
      "type": "object",
      "properties": {
        "document_id": { "type": "string" },
        "pages": { "type": "string", "description": "e.g. 1-5. Omit for whole document." },
        "max_words": { "type": "integer", "minimum": 50, "maximum": 1000 }
      },
      "required": ["document_id"]
    }
  },
  {
    "name": "verify_claim_against_source",
    "description": "Check whether ONE specific claim is supported by ONE loaded document. Input: document_id and claim text. Output: verdict (supported / contradicted / not_addressed), the supporting or contradicting quote, and page. Use when another agent's finding must be checked against its cited source. Does not search the web.",
    "input_schema": {
      "type": "object",
      "properties": {
        "document_id": { "type": "string" },
        "claim": { "type": "string" }
      },
      "required": ["document_id", "claim"]
    }
  }
]
```

Each tool now has a distinct input/output contract, so a downstream consumer (and the model) knows exactly which one to call and what comes back.

#### Example 4: keyword-sensitive system prompt, bad vs good

```text
BAD system prompt fragment
  "You are a support agent. ALWAYS look up the customer's account information
   before responding to any request about their account, orders or billing."

  -> "account information" and "customer" echo get_customer's description.
     The agent calls get_customer for "check my order #12345" and never
     reaches lookup_order, even though lookup_order's description is good.

GOOD system prompt fragment
  "You are a support agent.
   - Refunds and account changes: first identify the customer with get_customer
     (verified customer_id), then act.
   - Questions that contain an order number (e.g. #12345) about status,
     tracking or returns: call lookup_order directly with that number.
   - When unsure which tool applies, follow each tool's own description."
```

#### Example 5: reading a misrouting log (illustrative)

| User message | Tool called | Why | Fix |
| --- | --- | --- | --- |
| "check my order #12345" | `get_customer` | Both descriptions are one line; 5-digit string looks like an ID | Expand both descriptions (formats, boundary) |
| "refund order 88231" | `process_refund` without `get_customer` | Ordering requirement not enforced | Programmatic prerequisite (not a description fix) |
| "what's in this PDF?" | `analyze_content` | Name/description overlap with `analyze_document` | Rename or split (rung 2/3) |
| Everything routes to `get_customer` after a prompt edit | `get_customer` | New system-prompt wording echoes the tool | Review prompt for keyword collisions |

### Exam traps

- **"Add 5-8 few-shot examples to the system prompt."** Plausible but wrong as a *first step*: it adds tokens and does not repair the missing information in the descriptions.
- **"Build a keyword/regex router in front of the model."** Over-engineered; it replaces the model's language understanding with brittle rules.
- **"Merge both into `lookup_entity`."** A legitimate architecture, but more work than a first step when the immediate defect is thin descriptions. (If the question asks for the *most effective first step*, expand the descriptions.)
- **Confusing selection with ordering.** If the symptom is "skips a required step", the answer is programmatic enforcement, not better descriptions.
- **Blaming the tool when the system prompt is the cause.** If a prompt change coincided with misrouting, look for keyword-sensitive wording.
- **Over-trusting parameter names.** Two tools both taking `id` make the problem worse; distinct names plus formats help.

### Check yourself

1. Your `search_docs` and `search_tickets` tools both say "Searches the knowledge base for relevant items". The agent uses them interchangeably. What is the most effective first step, and what would you put in the descriptions?
   <details><summary>Answer</summary>Rewrite both descriptions to differentiate them: what each searches (documentation articles vs support tickets), accepted query formats and filters, example queries, edge cases (empty results), and an explicit "do not use for X; use the other tool" boundary. Renaming/splitting/merging or few-shot/routing layers come later if that does not fix it.</details>
2. After editing the system prompt to say "always confirm the user's order details first", the agent calls `lookup_order` even for password-reset requests. What is the likely root cause and fix?
   <details><summary>Answer</summary>A keyword-sensitive instruction ("order details") created an unintended association with `lookup_order`, overriding the tool descriptions. Fix by rewording the prompt in workflow terms (scope the instruction to order-related requests) and reviewing the prompt for other wording that echoes tool descriptions.</details>
3. A single `analyze_document` tool is used for extracting dates, summarizing, and fact-checking, and its output format varies. Which design change fits, and why not just improve the description?
   <details><summary>Answer</summary>Split into purpose-specific tools (`extract_data_points`, `summarize_content`, `verify_claim_against_source`) with defined input/output contracts. A better description cannot give one tool three different output contracts; the splitting resolves both selection ambiguity and output predictability.</details>

### Official docs

- [Define tools](https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools.md) — best practices for tool definitions, example good/poor descriptions, `input_examples`, how tools become a system prompt.
- [Troubleshooting tool use](https://platform.claude.com/docs/en/agents-and-tools/tool-use/troubleshooting-tool-use.md) — "Claude calls the wrong tool" symptom/cause/fix table.
- [Writing tools for agents](https://www.anthropic.com/engineering/writing-tools-for-agents) — consolidation, namespacing, high-signal responses, evaluating tools.
- [How tool use works](https://platform.claude.com/docs/en/agents-and-tools/tool-use/how-tool-use-works.md) — the tool-use contract and the agentic loop.

---

## 2.2 Implement structured error responses for MCP tools

### What the exam tests

- **Knowledge**
  - The MCP **`isError`** flag is how a tool reports failure back to the agent.
  - Four error families: **transient** (timeouts, service unavailable), **validation** (invalid input), **business** (policy violations), **permission**.
  - Why uniform errors ("Operation failed") prevent good recovery decisions.
  - Retryable vs non-retryable errors, and how structured metadata prevents wasted retries.
- **Skills**
  - Return structured error metadata: `errorCategory` (transient/validation/permission), an `isRetryable` boolean, and a human-readable description.
  - For business-rule violations include a `retriable: false` flag and a **customer-friendly explanation** so the agent can communicate appropriately.
  - **Recover locally inside subagents** for transient failures; propagate to the coordinator only what cannot be resolved locally, together with **partial results and what was attempted**.
  - Distinguish **access failures** (need a retry decision) from **valid empty results** (a successful query with no matches).

### Concepts explained

#### Two layers of errors in MCP

The MCP tools specification separates failures into two mechanisms:

| | Protocol error | Tool execution error |
| --- | --- | --- |
| What it means | The *request* is broken: unknown tool, malformed request, server error | The tool *ran* (or tried to) and the operation failed |
| How it is reported | Standard JSON-RPC `error` object (e.g. code `-32602`) | A normal `result` with **`isError: true`** and explanatory `content` |
| Typical causes | Unknown tool name, request fails the schema, server fault | API failures, **input validation errors**, **business logic errors** |
| Does the model see it? | Clients *may* pass it on; the spec says these are less likely to lead to successful recovery | Clients *should* pass it to the model so it can **self-correct** |

So for the failure kinds in this task statement, the **reporting channel is `isError: true` inside a normal result**, not a thrown JSON-RPC error. On the Claude API the equivalent is a `tool_result` block with `"is_error": true`. In the Agent SDK's in-process MCP servers, an uncaught exception in a handler is converted into an error result carrying the **raw exception message**; if you catch it yourself and return `isError: true` (Python key: `is_error`), Claude sees the message **you** composed.

#### `isError` is protocol; the rest is your convention

MCP defines `isError` and the content blocks; it does **not** define `errorCategory` or `isRetryable`. These are fields you design so the agent (and your orchestration code) can branch on them. Two portable ways to carry them:

- as JSON text inside a `text` content block (works everywhere; the Python in-process SDK forwards only `content` and `is_error`), and/or
- in `structuredContent` (supported by the MCP spec and the TypeScript SDK; the Python in-process `@tool` decorator does not forward it). Two caveats: the spec says a tool returning structured content SHOULD also return the serialized JSON in a text block, and in the Agent SDK's in-process server, when `structuredContent` is set, Claude receives that JSON plus any image/resource blocks but **not** the text blocks. A tool may also declare an optional `outputSchema`; if it does, the server MUST return conforming `structuredContent` and clients SHOULD validate it. (Neither is required for the exam's `isError` pattern.)

The exam guide uses `isRetryable` in one place and `retriable: false` in another; treat them as the same idea, one retry flag.

#### The four categories and what the agent should do

| Category | Meaning | Example | Retry the same call? | Agent's correct move |
| --- | --- | --- | --- | --- |
| **transient** | Temporary infrastructure fault | Timeout, 503, connection reset | **Yes** (bounded, with backoff) | Retry locally; if still failing, report with what was tried |
| **validation** | The input was wrong | `order_number` not 5-8 digits, date in the past | **No, not unchanged** (same input fails again) | Fix the input (or ask the user) and call again |
| **business** | A policy/rule refused the action | Refund requested after the 30-day window; order not eligible | **No** (`isRetryable: false`) | Do not retry; explain to the customer using the provided friendly text, or escalate if an exception is warranted |
| **permission** | Caller not allowed | Order belongs to a different customer; missing scope | **No** | Do not retry; escalate or ask for the right authorization |

A common design practice (not from the docs or the exam guide) is to define `isRetryable` as "**repeating the identical call could succeed**". Under that definition a validation error is `false` (you must change the input), which is why metadata matters: the agent should not hammer the same bad call three times. Note that the exam guide's `errorCategory` skill line lists `transient`/`validation`/`permission` and signals business-rule violations with `retriable: false` plus a customer-friendly message; using `business` as a fourth category value (as in the examples below) is a reasonable convention for the four error families named in the knowledge bullet, not an exam requirement.

#### Why generic errors hurt

"Operation failed" collapses four situations that need four different responses. The agent cannot tell whether to retry (transient), correct the input (validation), apologize and explain (business), or escalate (permission). Typical results: wasted retries on a policy refusal, or giving up on a one-second blip, or telling the customer something wrong. Structured metadata turns recovery from guessing into branching. The Claude API docs make the same point: write instructive error messages that say what went wrong and what to try next (e.g. "Rate limit exceeded. Retry after 60 seconds").

#### Valid empty result vs access failure

| | Valid empty result | Access failure |
| --- | --- | --- |
| Meaning | The query ran correctly and nothing matched | The query could not run or could not see the data |
| `isError` | `false` | `true` |
| Example | "0 orders for C-104233 in the last 90 days" | "Order database timed out" / "403 for this customer" |
| What the agent concludes | There really are none; proceed on that basis | Unknown! Retry, use another source, or flag the gap |

Swallowing an exception and returning an empty list marked successful is the classic anti-pattern: it **hides a failure as a fact** ("the customer has no orders") and prevents any recovery. MCP's resource spec makes the same point: a server must not return an empty `contents` array for a *non-existent* resource, because the array is ambiguous between "exists but empty" and "does not exist"; a missing resource is a JSON-RPC error.

#### Local recovery in subagents, and what to propagate

In a multi-agent system, handle failures at the lowest level that can fix them:

1. **Subagent handles transient faults itself**: bounded retries with backoff, maybe a reformulated query or alternate source. Do not bother the coordinator with a 2-second blip.
2. **If it cannot resolve the failure**, it returns to the coordinator a *structured* failure: what failed (category), what was attempted (query, retries, alternatives tried), **partial results** gathered so far, and suggested next steps.
3. **The coordinator decides** (retry with a different query, route to another subagent, proceed with partial results and annotate the gap, or escalate).

Official sample Q8 (web search subagent times out) is exactly this: the best answer returns structured error context (failure type, attempted query, partial results, alternatives). The wrong answers each fail a principle:

- Generic "search unavailable" after retries **hides context**.
- An empty result marked successful **suppresses the error**.
- Throwing to a top-level handler that kills the workflow **over-reacts** when recovery was possible.

### Worked examples

#### Example 1: the raw MCP result for each error type

All four use `isError: true` and put the machine-readable detail in the text block (JSON) so any client can read it. The `errorCategory`, `isRetryable` etc. are your design, not MCP fields.

Transient:

```json
{
  "jsonrpc": "2.0",
  "id": 11,
  "result": {
    "resultType": "complete",
    "isError": true,
    "content": [
      {
        "type": "text",
        "text": "{\"errorCategory\":\"transient\",\"isRetryable\":true,\"message\":\"Order service timed out after 8s (HTTP 504).\",\"attempted\":{\"tool\":\"lookup_order\",\"order_number\":\"#12345\"},\"suggestion\":\"Retry up to 2 more times with a 1-2s backoff; if it still fails, tell the user the order system is temporarily unavailable.\"}"
      }
    ]
  }
}
```

Validation:

```json
{
  "isError": true,
  "content": [
    {
      "type": "text",
      "text": "{\"errorCategory\":\"validation\",\"isRetryable\":false,\"message\":\"order_number '12-45' is invalid: expected 5 to 8 digits, optionally prefixed with #.\",\"field\":\"order_number\",\"suggestion\":\"Ask the customer to re-check the order number, then call lookup_order again with the corrected value.\"}"
    }
  ]
}
```

Business (note the customer-friendly text and `retriable: false`):

```json
{
  "isError": true,
  "content": [
    {
      "type": "text",
      "text": "{\"errorCategory\":\"business\",\"isRetryable\":false,\"retriable\":false,\"message\":\"Refund denied by policy RET-30: order delivered 47 days ago, return window is 30 days.\",\"customerMessage\":\"This order was delivered more than 30 days ago, which is outside our standard return window, so I can't process the refund myself. I can connect you with a specialist who can review whether an exception is possible.\",\"suggestion\":\"Do not retry. Offer escalate_to_human if the customer wants an exception.\"}"
    }
  ]
}
```

Permission:

```json
{
  "isError": true,
  "content": [
    {
      "type": "text",
      "text": "{\"errorCategory\":\"permission\",\"isRetryable\":false,\"message\":\"Order 88231 does not belong to customer C-104233.\",\"suggestion\":\"Do not retry and do not reveal the other account's details. Verify the customer's identity again or escalate.\"}"
    }
  ]
}
```

Protocol error (not one of the four; for contrast):

```json
{
  "jsonrpc": "2.0",
  "id": 12,
  "error": { "code": -32602, "message": "Unknown tool: lookup_orders" }
}
```

#### Example 2: a handler that produces these (Python, in-process SDK server)

```python
import json
from typing import Any
from claude_agent_sdk import tool

def err(category: str, message: str, retryable: bool, **extra) -> dict[str, Any]:
    body = {"errorCategory": category, "isRetryable": retryable, "message": message, **extra}
    return {"content": [{"type": "text", "text": json.dumps(body)}], "is_error": True}

@tool("process_refund", "Refund an eligible order. Requires verified customer_id and order_number. "
      "Refunds are only allowed within 30 days of delivery; otherwise returns a business error with a "
      "customer-friendly explanation. Does not retry on business or permission errors.",
      {"customer_id": str, "order_number": str, "amount": float})
async def process_refund(args: dict[str, Any]) -> dict[str, Any]:
    try:
        order = await orders.get(args["order_number"])           # may raise TimeoutError
    except TimeoutError:
        return err("transient", "Order service timed out.", True)
    if order is None:
        return {"content": [{"type": "text", "text": "No order found with that number."}]}  # valid empty result
    if order.customer_id != args["customer_id"]:
        return err("permission", "Order belongs to another customer.", False)
    if order.days_since_delivery > 30:
        return err("business", "Refund window (30 days) exceeded.", False,
                   retriable=False,
                   customerMessage="This order is outside our 30-day return window, so I can't refund it directly. "
                                   "I can pass your request to a specialist.")
    ...
```

(`orders` is a stand-in for your backend client. The shape of the return value, `content` plus `is_error`, is from the Agent SDK custom tools page; the metadata keys are your own convention.)

#### Example 3: valid empty result vs access failure

```text
tools/call search_orders { "customer_id": "C-104233", "since": "2026-08-01" }

(a) Query ran, nothing matched
    result: { "isError": false,
              "content": [{ "type": "text", "text": "0 orders found for C-104233 since 2026-08-01." }],
              "structuredContent": { "orders": [], "count": 0 } }
    -> agent: "You haven't placed orders since August 1."

(b) Query could not run
    result: { "isError": true,
              "content": [{ "type": "text",
                 "text": "{\"errorCategory\":\"transient\",\"isRetryable\":true,\"message\":\"Order DB connection reset\"}" }] }
    -> agent: retry; if it still fails, say "I couldn't check your orders right now", NOT "you have no orders".

ANTI-PATTERN
    catch (e) { return { content: [{ type: "text", text: "[]" }] } }   // failure disguised as (a)
```

#### Example 4: local recovery in a subagent, propagation to the coordinator

```python
import asyncio

async def web_search_with_recovery(query: str, max_attempts: int = 3) -> dict:
    attempts, partial = [], []
    for i in range(1, max_attempts + 1):
        try:
            return {"status": "ok", "results": await search(query)}        # success: nothing to propagate
        except TimeoutError as e:
            attempts.append({"attempt": i, "query": query, "error": "timeout"})
            await asyncio.sleep(2 ** (i - 1))                               # local recovery for transient faults
        except SearchPermissionError as e:
            # not locally recoverable: stop immediately, no pointless retries
            return failure("permission", query, attempts, partial)
    # transient faults persisted: escalate with context, not a bare "unavailable"
    partial = await cached_snippets(query)                                  # whatever we could salvage
    return failure("transient", query, attempts, partial)

def failure(category, query, attempts, partial):
    return {
        "status": "failed",
        "errorCategory": category,
        "isRetryable": category == "transient",
        "attempted": {"query": query, "attempts": attempts},
        "partialResults": partial,
        "alternatives": ["narrow the query to one sub-topic", "try a news-only search", "proceed with partial results and flag the gap"],
    }
```

The coordinator receives this one message and can retry with a modified query, hand the topic to another subagent, or continue with `partialResults` and an explicit coverage-gap note in the final report.

#### Example 5: a conversation trace, business error handled well

```text
User:      I want a refund for order #77120. It arrived two months ago.
Agent:     [get_customer {email}]            -> verified C-104233
Agent:     [lookup_order {#77120, C-104233}] -> delivered 61 days ago, eligible: false
Agent:     [process_refund {..}]             -> isError: true, errorCategory: business, retriable: false,
                                               customerMessage: "...outside our 30-day return window..."
Agent -> user: "This order was delivered more than 30 days ago, which is outside our standard
                return window, so I can't refund it directly. I can connect you with a specialist
                who can review whether an exception is possible. Would you like that?"
```

The agent did not retry (retriable false), did not expose internal policy codes, and offered a path forward.

### Exam traps

- **"Retry with exponential backoff, then return a generic 'unavailable' status."** Backoff is fine locally for transient errors; the trap is the generic status that discards context the coordinator needs.
- **"Return an empty result marked successful."** Suppresses the failure; always wrong where recovery or accurate reporting matters.
- **"Propagate every exception to a top-level handler that stops the workflow."** Over-reaction; only unrecoverable errors go up, and they go up with context and partial results.
- **Retrying business or permission errors.** A policy refusal will not succeed on the fifth try. Use the category and `retriable: false`.
- **Treating `isError` as optional prose.** If you only write "Error: ..." text with `isError` false, the client/model may treat it as ordinary data.
- **Mixing protocol and execution errors.** Unknown tool or malformed request is a JSON-RPC error; "the refund was refused" is a tool execution error (`isError: true`).
- **Conflating "no results" with "could not search".** The two need opposite behaviour.

### Check yourself

1. A tool returns `{"isError": true, "content": [{"type":"text","text":"Operation failed"}]}` for every problem. Name two concrete wrong behaviours this causes.
   <details><summary>Answer</summary>Examples: the agent retries a policy refusal or permission error pointlessly (it cannot tell they are non-retryable); it gives up immediately on a transient timeout; it cannot tell the customer *why* something failed; it cannot decide between escalating and fixing input. The fix is `errorCategory`, `isRetryable` and a human-readable message.</details>
2. `search_tickets` for customer C-9 finds nothing. Should it return `isError: true`? What if the ticket database is unreachable?
   <details><summary>Answer</summary>No: finding nothing is a valid empty result (success, count 0). An unreachable database is an access failure: `isError: true`, category `transient`, `isRetryable: true`. The agent must not report "no tickets" in the second case.</details>
3. A document-analysis subagent hits a 503 from the document store. Describe what should be handled locally and what, if anything, goes to the coordinator.
   <details><summary>Answer</summary>Handle locally: bounded retries with backoff (transient). If it still fails, return to the coordinator a structured failure with category, what was attempted, any partial results, and alternatives, so the coordinator can decide. Do not return a bare "failed", an empty success, or crash the whole workflow.</details>

### Official docs

- [MCP specification: Tools (error handling)](https://modelcontextprotocol.io/specification/2026-07-28/server/tools.md) — protocol errors vs tool execution errors, `isError`, result content types, `structuredContent`.
- [Handle tool calls](https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls.md) — `tool_result`, `is_error`, instructive error messages.
- [Give Claude custom tools (Agent SDK)](https://code.claude.com/docs/en/agent-sdk/custom-tools.md) — "Handle errors": uncaught exception vs returning `isError: true`.
- [MCP specification: Resources (error handling)](https://modelcontextprotocol.io/specification/2026-07-28/server/resources.md) — why an empty array must not stand for "does not exist".

---

## 2.3 Distribute tools appropriately across agents and configure tool choice

### What the exam tests

- **Knowledge**
  - Giving one agent too many tools (the guide's contrast: 18 instead of 4-5) degrades selection reliability by increasing decision complexity.
  - Agents given tools outside their specialization tend to misuse them (a synthesis agent attempting web searches).
  - **Scoped tool access**: only the tools needed for the role, plus a *limited* cross-role tool for specific high-frequency needs.
  - `tool_choice` options: `"auto"`, `"any"`, and forced selection `{"type": "tool", "name": "..."}`.
- **Skills**
  - Restrict each subagent's tool set to its role to prevent cross-specialization misuse.
  - Replace a generic tool with a constrained alternative (replace `fetch_url` with `load_document`, which validates document URLs).
  - Provide scoped cross-role tools for high-frequency needs (a `verify_fact` tool for the synthesis agent) while routing complex cases through the coordinator.
  - Use forced `tool_choice` to make a specific tool run first (e.g. `extract_metadata` before enrichment tools), then handle subsequent steps in follow-up turns.
  - Use `tool_choice: "any"` to guarantee the model calls *a* tool rather than replying with conversational text.

### Concepts explained

#### Why fewer tools per agent

Every tool in the list is something the model must weigh on every turn. The docs quantify the scaling problem at the platform level: selection accuracy "degrades once you exceed 30-50 available tools" and, on the context side, a multi-server setup can burn tens of thousands of tokens on definitions. The exam's framing is the *small-scale* version: an agent with 18 tools makes worse choices than one with 4-5 well-differentiated tools, because there are more near-neighbours to confuse and more irrelevant options.

Two distinct remedies exist; know which the question is asking for:

| Situation | Remedy | Source |
| --- | --- | --- |
| Several *roles* share one flat toolset; agents misuse tools outside their job | **Scope tools per agent/subagent** (least privilege) | Exam guide, Agent SDK subagents |
| One agent legitimately needs a very large catalog (dozens to thousands of tools, e.g. many MCP servers) | **Tool search** (load definitions on demand; keep 3-5 most-used tools non-deferred) | Tool search docs |

The second is covered in 2.4 for Claude Code. For the exam's multi-agent scenarios, the answer is nearly always the first.

#### Scoped access and cross-role tools

- A **synthesis** agent combines findings. Given `web_search`, it will search (it is a tool, it looks relevant, and the model is helpful), duplicating work, bypassing the coordinator's decomposition and citation tracking.
- A **web-search** agent should not have `write_report`. A **report** agent should not run searches.
- Real work sometimes crosses roles. In the sample, 85% of the synthesis agent's verification needs are simple fact-checks (dates, names, statistics) and 15% need real investigation. Round-tripping all of them through the coordinator costs 2-3 round trips and 40% extra latency. The best answer (Q9): give synthesis a **scoped `verify_fact` tool** for the simple 85%, and keep routing the complex 15% through the coordinator to the web-search agent. This is least privilege applied pragmatically: it adds exactly what is needed for the common case and preserves the coordination pattern for the rest.
- Distractors in that question, and why they lose: *batching* verifications to the end creates blocking dependencies (later synthesis steps may depend on earlier verified facts); *giving synthesis all web-search tools* over-provisions it and violates separation of concerns; *speculative caching* by the search agent cannot reliably predict what synthesis will need.

How scoping is enforced in the Agent SDK: each subagent's `AgentDefinition` has a **`tools`** array (allowed tool names) and a `disallowedTools` array; if `tools` is omitted the subagent inherits every tool available to subagents. A tool you leave out "isn't in the subagent's session at all". MCP servers can be attached per agent via the `mcpServers` field. Do not confuse this with **`allowedTools`** (SDK/CLI option): `allowedTools` only **pre-approves** listed tools (no permission prompt); it does *not* remove other tools. To restrict availability use `tools` (or a bare name in `disallowedTools`).

Common combinations from the SDK docs: read-only analysis = `Read`, `Grep`, `Glob`; test execution = `Bash`, `Read`, `Grep`; code modification = `Read`, `Edit`, `Write`, `Grep`, `Glob`.

#### Constrained tools instead of generic ones

A generic `fetch_url(url)` can fetch anything: internal addresses, huge binaries, arbitrary content that then lands in the model's context. A **constrained** tool encodes the policy in code, so the model cannot misuse it even if it tries:

- `load_document(url)`: accepts only `https` URLs on an allow-listed set of hosts or file types (PDF/HTML/DOCX), enforces a size cap, extracts text, and returns a `document_id` plus metadata. Failures come back as structured errors (2.2).

Same principle as 2.1: the narrower the tool, the less the model has to guess and the less damage a wrong guess can do.

#### `tool_choice`: the four modes

`tool_choice` is a request parameter next to `tools`. Behaviours below are from the Define tools page.

| Mode | JSON | Behaviour | Use when |
| --- | --- | --- | --- |
| **auto** (default when tools are given) | `{"type": "auto"}` | Claude decides whether to call a tool and which one; may answer in plain text | A normal agent turn: some requests need tools, some do not; the model may explain before acting |
| **any** | `{"type": "any"}` | Claude **must** call one of the provided tools, but chooses which | Output must be structured/tool-shaped and you have several candidate tools (e.g. different extraction schemas for unknown document types); you want to avoid conversational text instead of a tool call |
| **forced tool** | `{"type": "tool", "name": "extract_metadata"}` | Claude **must** call exactly that tool | A specific step must run first (extract metadata before enrichment); a deterministic first move |
| **none** | `{"type": "none"}` | Claude cannot call tools (default when no tools are provided) | A final text-only turn while the tool definitions stay in the request |

Important behaviours:

- With `any` or forced `tool`, the API **prefills** the assistant message, so the model does **not** emit a natural-language explanation before the `tool_use` block, even if asked. If you want text plus a tool call, use `auto` and instruct in the user message ("Use the get_weather tool in your response").
- Forced tool use also **skips thinking** on every model that accepts it.
- `disable_parallel_tool_use: true` goes **inside** `tool_choice` (it is not a top-level parameter). With `auto` it means *at most one* tool call per response; with `any`/`tool` it means *exactly one*. Set it on the request that returns the `tool_use`.
- Combine `any` with **strict tool use** (`"strict": true` on tools) to guarantee both that a tool is called and that its input matches the schema (grammar-constrained sampling).
- **Currency note:** the Define tools page states that `any` and `tool` are not supported on some current models (listed there as returning a 400 error) and not with manual extended thinking (`thinking: {type: "enabled"}`; adaptive thinking by itself does not block forced tool use); `auto` and `none` always work. Exam answers still follow the guide's three-mode wording.
- Changing `tool_choice` between requests invalidates cached message blocks (tool definitions and system prompt stay cached).

Forced selection controls **one step**. It is the tool for "make sure X is called first", then you process results and send a follow-up turn (usually `auto`) for the rest. Forcing the same tool on every turn would loop on that tool forever.

### Worked examples

#### Example 1: scoped subagent tool lists (Scenario 3), with a cross-role `verify_fact`

```python
from claude_agent_sdk import query, ClaudeAgentOptions, AgentDefinition

research_agents = {
    "web-searcher": AgentDefinition(
        description="Finds and ranks web sources for a research subtopic.",
        prompt="Search the web for the assigned subtopic. Return source URLs, titles, dates and key excerpts. Do not write reports.",
        tools=["mcp__research__web_search"],                    # search only
    ),
    "doc-analyst": AgentDefinition(
        description="Analyzes documents the coordinator has already identified.",
        prompt="Load the given documents and extract data points, summaries or claim checks as asked.",
        tools=[
            "mcp__research__load_document",                     # constrained, validates URLs
            "mcp__research__extract_data_points",
            "mcp__research__summarize_content",
            "mcp__research__verify_claim_against_source",
        ],
    ),
    "synthesizer": AgentDefinition(
        description="Combines findings from other agents into a cited narrative.",
        prompt=("Synthesize ONLY the findings you are given. For a simple fact check (a date, name, number) "
                "call verify_fact. If a claim needs deeper investigation, list it under 'needs_investigation' "
                "and stop; the coordinator will handle it. Never search the web yourself."),
        tools=["mcp__research__verify_fact"],                   # narrow cross-role tool
    ),
    "report-writer": AgentDefinition(
        description="Formats the final cited report.",
        prompt="Turn the synthesis into the final report with citations.",
        tools=["Write"],
    ),
}

options = ClaudeAgentOptions(
    mcp_servers={"research": research_server},                  # your SDK or external MCP server
    agents=research_agents,
    allowed_tools=["Agent", "mcp__research__*", "Write"],       # allowedTools = pre-approve, not restrict
)
```

Each subagent sees only 1-4 tools. The coordinator spawns them via the `Agent` tool (the exam calls it the `Task` tool). Note the intentional absence of `web_search` from `synthesizer`.

The `verify_fact` definition (narrow by design):

```json
{
  "name": "verify_fact",
  "description": "Verify ONE simple, checkable fact such as a date, a person or organization name, or a published statistic, against a trusted reference source. Input: the fact as a short statement and an optional expected value. Output: verdict (confirmed / contradicted / unverifiable), the source URL, and the value found. Use for quick single-lookup checks while synthesizing. Do NOT use for interpretation, causal claims, comparisons across several sources, or anything that needs reading a full document; report those as needs_investigation instead.",
  "input_schema": {
    "type": "object",
    "properties": {
      "statement": { "type": "string", "description": "e.g. 'The EU AI Act entered into force on 1 August 2024'" },
      "expected_value": { "type": "string", "description": "Optional value to compare, e.g. '2024-08-01'" }
    },
    "required": ["statement"]
  }
}
```

#### Example 2: replace `fetch_url` with a constrained `load_document`

```text
BEFORE  fetch_url(url: string)
        - any scheme, any host, any size; returns raw body into context
        - the analysis agent also uses it to wander the web

AFTER   load_document(url: string)
        - https only; host must be in the approved sources list
        - content-type must be PDF, HTML or DOCX; max size enforced
        - returns { document_id, title, page_count } (NOT the full text)
        - invalid URL -> isError: true, errorCategory: "validation", message names the rule broken
```

#### Example 3: `tool_choice` modes with the extraction-first example (Scenario 6)

Goal: for every incoming document, **always** run `extract_metadata` first, then let the model use enrichment tools as needed.

Turn 1: force the first step.

```python
import json
import anthropic
client = anthropic.Anthropic()

tools = [extract_metadata_tool, lookup_vendor_tool, classify_document_tool]   # each a full definition

resp1 = client.messages.create(
    model=MODEL, max_tokens=1024, tools=tools,
    tool_choice={"type": "tool", "name": "extract_metadata"},       # FORCED: this tool first
    messages=[{"role": "user", "content": document_text}],
)
# resp1.content[0] is a tool_use block for extract_metadata (no text before it, no thinking block).
tool_use = next(b for b in resp1.content if b.type == "tool_use")
metadata = run_tool(tool_use.name, tool_use.input)
```

Turn 2: return the result and release the model (`auto`, or `any` to demand another tool call).

```python
resp2 = client.messages.create(
    model=MODEL, max_tokens=1024, tools=tools,
    tool_choice={"type": "auto"},                                    # model picks enrichment tools or finishes
    messages=[
        {"role": "user", "content": document_text},
        {"role": "assistant", "content": resp1.content},
        {"role": "user", "content": [
            {"type": "tool_result", "tool_use_id": tool_use.id, "content": json.dumps(metadata)},
        ]},
    ],
)
```

When to use each mode in this system:

| Step | Mode | Why |
| --- | --- | --- |
| Always extract metadata first | forced `{"type":"tool","name":"extract_metadata"}` | Deterministic first move; no chance of chatty text or a skipped step |
| Document type unknown; you have `extract_invoice`, `extract_receipt`, `extract_contract` and want a structured output regardless | `any` | Guarantees a tool call (structured output), lets the model pick the matching schema |
| Open-ended follow-up (enrich, validate, or just answer) | `auto` | Model decides whether another tool is needed |
| Final turn: write a plain-language summary from collected data | `none` | Tools remain defined but cannot be called |

(Model-support caveat: see the currency note above. On models that reject forced selection, use `auto` with strict tool use, or structured outputs, and enforce ordering in your own code.)

#### Example 4: the same idea as an orchestration rule

```text
Programmatic ordering (guarantee):     step 1 tool_choice = forced(extract_metadata)
                                        step 2 tool_choice = auto
Prompt-only ordering (no guarantee):    "Always call extract_metadata first."  (probabilistic)
```

If the business needs the guarantee, choose the first. This mirrors the guide's rule that deterministic enforcement beats prompt instructions for critical sequencing.

### Exam traps

- **"Give the synthesis agent all the search tools so it never waits."** Over-provisioning; breaks separation of concerns.
- **"Batch all verifications and send them at the end."** Looks efficient, but later synthesis steps depend on earlier verified facts.
- **"Use a routing classifier to enable only some tools per request"** as the fix for tool *ordering* failures. It addresses availability, not sequence (Q1 option D).
- **Confusing `any` with forced.** `any` = some tool, model's choice; forced = this exact tool.
- **Using `auto` and expecting a guaranteed tool call.** `auto` may return plain text.
- **Forcing the same tool for the whole conversation.** Force the step you need, then switch to `auto`.
- **Expecting an explanation before a forced/any tool call.** The prefill means no preamble text.
- **`allowedTools` as access control.** It pre-approves; scoping is done with the subagent's `tools` list.
- **Keeping 18 tools "for flexibility".** The guide's point is the opposite: scope to 4-5 role-relevant tools.

### Check yourself

1. The report-writing subagent keeps issuing web searches. Name the root cause and the fix.
   <details><summary>Answer</summary>It has access to search tools outside its role. Fix: restrict its `tools` list to what the role needs (e.g. `Write`); it should receive findings from the coordinator, not search itself.</details>
2. You want the model to always call *some* extraction tool (never reply with prose), but you do not know the document type in advance. Which `tool_choice`? And if one specific metadata tool must always run first?
   <details><summary>Answer</summary>`{"type": "any"}` for "some tool, model picks the schema"; forced `{"type": "tool", "name": "extract_metadata"}` for the specific first step, followed by a follow-up turn (usually `auto`).</details>
3. 70% of your support agent's `escalate_to_human` calls are preceded by an unnecessary web search the agent should not be able to run. What do you change?
   <details><summary>Answer</summary>Remove the web search tool from that agent's toolset (scope tools to the role). Prompt wording would be a probabilistic fix; removing the tool is deterministic.</details>

### Official docs

- [Define tools: forcing tool use](https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools.md) — `tool_choice` options, prefill behaviour, model-support table.
- [Parallel tool use](https://platform.claude.com/docs/en/agents-and-tools/tool-use/parallel-tool-use.md) — `disable_parallel_tool_use` inside `tool_choice`.
- [Strict tool use](https://platform.claude.com/docs/en/agents-and-tools/tool-use/strict-tool-use.md) — guaranteed schema-valid inputs.
- [Tool search tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool.md) — selection accuracy past 30-50 tools; when to use it.
- [Manage tool context](https://platform.claude.com/docs/en/agents-and-tools/tool-use/manage-tool-context.md) — tool search vs caching vs context editing.
- [Subagents in the SDK](https://code.claude.com/docs/en/agent-sdk/subagents.md) — `AgentDefinition.tools`, tool restrictions, common combinations.

---

## 2.4 Integrate MCP servers into Claude Code and agent workflows

### What the exam tests

- **Knowledge**
  - Scoping: project-level `.mcp.json` for shared team tooling vs user-level `~/.claude.json` for personal/experimental servers.
  - Environment variable expansion in `.mcp.json` (e.g. `${GITHUB_TOKEN}`) so credentials are not committed.
  - Tools from *all* configured MCP servers are discovered at connection time and available simultaneously.
  - MCP **resources** expose content catalogs (issue summaries, documentation hierarchies, database schemas) and reduce exploratory tool calls.
- **Skills**
  - Configure shared servers in project-scoped `.mcp.json` with env-var expansion for tokens.
  - Configure personal/experimental servers in user-scoped `~/.claude.json`.
  - Enhance MCP tool descriptions (capabilities and outputs in detail) so the agent does not prefer built-ins like Grep over more capable MCP tools.
  - Prefer existing community MCP servers for standard integrations (e.g. Jira); reserve custom servers for team-specific workflows.
  - Expose content catalogs as MCP resources so agents see what data exists without exploratory tool calls.

### Concepts explained

#### MCP in one picture

MCP is a client-server protocol. The **host** (Claude Code, Claude Desktop, your Agent SDK app) creates one **client** per **server**. A server exposes three primitives, differing in *who decides to use them*:

| Primitive | Controlled by | What it is | Claude Code surface |
| --- | --- | --- | --- |
| **Tools** | The **model** | Functions that do something or fetch something (API call, query, file write) | Called automatically; named `mcp__server__tool` |
| **Resources** | The **application** | Read-only data identified by URI: files, DB schemas, docs, issue lists | `@server:protocol://path` mentions; auto-provided list/read tools (`ListMcpResourcesTool`, `ReadMcpResourceTool`) |
| **Prompts** | The **user** | Parameterized instruction templates | Slash commands such as `/mcp__github__list_prs` |

Protocol methods: `tools/list`, `tools/call`; `resources/list`, `resources/templates/list`, `resources/read`; `prompts/list`, `prompts/get`. Transports: **stdio** (local process) and **Streamable HTTP** (remote). In Claude Code, `--transport http` is recommended for remote servers; SSE is deprecated.

In the current protocol revision (2026-07-28) the protocol is stateless: every request carries version and capabilities in `_meta`, and a `server/discover` request lets the client learn a server's capabilities. You do not need that level of detail for the exam.

#### Scopes: where the config lives

| Scope | Loads in | Shared with team | Stored in | Typical use |
| --- | --- | --- | --- | --- |
| **Local** (default) | Current project only | No | `~/.claude.json` (under that project's path) | Your private settings for one project |
| **Project** | Current project | **Yes**, via version control | **`.mcp.json`** in the project root | **Shared team tooling** (GitHub, Jira, team DB) |
| **User** | All your projects | No | **`~/.claude.json`** | **Personal / experimental** servers |

Add commands: `claude mcp add --transport http <name> <url> [--scope local|project|user]`; stdio: `claude mcp add --transport stdio <name> --env KEY=value -- <command> [args...]` (the `--` separates Claude's options from the server command). Manage with `claude mcp list|get|remove` and `/mcp` inside a session. If the same server name exists in several places, precedence is **local, then project, then user** (then plugin servers, then claude.ai connectors); the whole entry is used, without field merging.

Security: Claude Code asks for approval before using project-scoped servers in interactive sessions; in `claude -p` runs and Agent SDK sessions it loads them without asking (use `disabledMcpjsonServers`, `--setting-sources`/`settingSources`, or `--strict-mcp-config` to keep one out).

#### Environment variable expansion (no secrets in git)

A committed `.mcp.json` must not contain tokens. Claude Code expands, in a server's `command`, `args`, `env`, `url` and `headers`:

- `${VAR}`: value of `VAR`
- `${VAR:-default}`: `VAR` if set, else `default`

If a variable is unset and has no default, the file still loads with the literal `${VAR}` text and `claude mcp list` warns. An empty-string variable counts as set. For remote servers' `url`/`headers`, Claude Code deliberately reads certain credential variables as **empty** instead of expanding them (the docs name `ANTHROPIC_API_KEY`, `ANTHROPIC_AUTH_TOKEN`, `AWS_BEARER_TOKEN_BEDROCK`, `HTTPS_PROXY` and `NPM_TOKEN` as examples of the covered set; a `:-default` on such a name is ignored) so a project's `.mcp.json` cannot send your credentials to a server it names. To pass one of them, copy it into a variable with your own name first. Each developer sets `GITHUB_TOKEN` in their own shell/CI secret store; the file in git contains only the placeholder.

#### Discovery and availability

At connection, Claude Code lists each server's tools (and prompts/resources), so tools from all configured servers sit in the same toolbox **at the same time**; the agent can combine GitHub, Jira and database tools in one task. The Agent SDK exposes the connection status in the `system:init` message (`connected`, `pending`, `failed`, `needs-auth`, `disabled`); a failed server does **not** raise an exception, the run just continues without its tools (Claude may silently fall back to built-ins, so check the status).

Two practical consequences:

1. **Name collisions and clarity.** Tool names are namespaced `mcp__<server>__<tool>`; keep descriptions distinct across servers (the 2.1 rules apply to MCP tools too).
2. **Many servers = lots of definitions.** Claude Code's **tool search** is on by default: only tool names and the server's `instructions` load at session start, and Claude searches for full definitions on demand (`ENABLE_TOOL_SEARCH`: unset/`true` = defer, `auto`/`auto:N` = threshold, `false` = load upfront). Set `alwaysLoad: true` on a server to keep its few essential tools always visible. Default description limits to remember (adjustable with `CLAUDE_CODE_MAX_MCP_DESCRIPTION_LENGTH`): server instructions and upfront tool descriptions are cut at **4,096 characters**; tool-search-loaded descriptions at **16,384**, so put the key capability and "when to use" words first (tool search also matches on the start of the description).

In the Agent SDK, MCP tools need explicit permission: list them in `allowedTools`, using `mcp__github__*` for a whole server or `mcp__db__query` for one tool. Prefer this over broad permission modes.

#### MCP tool descriptions must compete with built-ins

The agent has built-ins (Grep, Read, Bash) and your MCP tools in the same list. If an MCP tool's description is thin ("Searches code"), the model reaches for the familiar built-in (Grep) even when the MCP tool is far more capable. The skill: **describe the MCP tool's capabilities, outputs and when to prefer it** so it wins when it should. Same discipline as 2.1; "enhancing MCP tool descriptions" is the exam's phrase. For MCP authors there are also tool **annotations** (`readOnlyHint`, `destructiveHint`, `idempotentHint`, `openWorldHint`); Claude Code runs `readOnlyHint` tools in parallel with other read-only calls, and annotations are hints, not enforcement.

#### Community server or custom server?

| Need | Choose | Why |
| --- | --- | --- |
| Standard integration (Jira, GitHub, Slack, Postgres) | **Existing community/vendor server** | Maintained, tested, already designed for agents; building your own duplicates effort and maintenance |
| Team-specific workflow (your deployment checklist, internal service with custom rules) | **Custom server** | No off-the-shelf equivalent |
| Simple custom function inside your own SDK app | **SDK in-process MCP server** (`createSdkMcpServer` / `create_sdk_mcp_server`) | No separate process |

(Deploying and hosting MCP servers is out of scope for the exam.)

#### Resources as a content catalog

Without a catalog, an agent exploring "what issues/docs/tables exist?" burns tool calls (`list_issues`, `search`, `get`, then more `get`s). A **resource** gives it a ready-made map up front:

- an **issue summary** resource (open issues with title/status/labels),
- a **documentation hierarchy** resource (the table of contents),
- a **database schema** resource (tables and columns).

Resources are read-only, identified by URI, with `name`, `description`, `mimeType`; **resource templates** parameterize them (`travel://activities/{city}/{category}`). In Claude Code you type `@` to list resources from all connected servers and reference one as `@server:protocol://path`; referenced resources are fetched and attached automatically. Use resources for *context the app can hand over*, use tools for *actions or parameterized queries the model must decide to run*, use prompts for *user-invoked workflows*.

### Worked examples

#### Example 1: shared project `.mcp.json` with `${GITHUB_TOKEN}`

Committed at the repo root:

```json
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/",
      "headers": {
        "Authorization": "Bearer ${GITHUB_TOKEN}"
      }
    },
    "jira": {
      "command": "npx",
      "args": ["-y", "example-jira-mcp-server"],
      "env": {
        "JIRA_URL": "${JIRA_URL:-https://yourcompany.atlassian.net}",
        "JIRA_API_TOKEN": "${JIRA_API_TOKEN}"
      }
    },
    "core-tools": {
      "type": "http",
      "url": "https://mcp.example.com/mcp",
      "alwaysLoad": true
    }
  }
}
```

(`example-jira-mcp-server` and the Jira URL are placeholders; use the community/vendor server your team picks. The GitHub URL and header pattern are the ones used in the Agent SDK MCP docs.) Each developer runs `export GITHUB_TOKEN=...` locally; the secret never enters git, and every teammate who clones the repo gets the same servers.

You can also create project-scope entries with `claude mcp add --transport http github --scope project <url>` (and `--header`), which writes `.mcp.json`. When you do, check the resulting file: a shell may expand `${GITHUB_TOKEN}` inside double quotes before Claude Code sees it (ordinary shell behaviour, not something the docs page states), so single-quote the header or edit the file so that it contains the literal `${GITHUB_TOKEN}` placeholder.

#### Example 2: a personal/experimental server in user scope

```bash
# Only for me, available in every project, never committed
claude mcp add --transport http hubspot --scope user https://mcp.hubspot.com/anthropic

# Personal stdio experiment
claude mcp add --transport stdio scratch-notes --scope user --env NOTES_DIR=$HOME/notes -- npx -y example-notes-mcp
```

The entries land in `~/.claude.json` (user scope; the docs place them at the top level of that file, whereas local-scope entries sit under `projects.<path>.mcpServers`). A typical result looks like the following (illustrative; the exact key name for user scope is not spelled out on the page, so verify in docs):

```json
{
  "mcpServers": {
    "hubspot": { "type": "http", "url": "https://mcp.hubspot.com/anthropic" }
  }
}
```

Rule of thumb: **if the team should have it, `.mcp.json` (project); if only you want it, user scope**. Default `local` scope is also personal but limited to one project.

#### Example 3: the same wiring in the Agent SDK

```python
import os
from claude_agent_sdk import query, ClaudeAgentOptions

options = ClaudeAgentOptions(
    # Option A: let the project's .mcp.json load (default 'project' setting source), or
    # Option B: pass servers in code:
    mcp_servers={
        "github": {
            "type": "http",
            "url": "https://api.githubcopilot.com/mcp/",
            "headers": {"Authorization": f"Bearer {os.environ['GITHUB_TOKEN']}"},
        }
    },
    allowed_tools=["mcp__github__list_issues", "mcp__github__get_issue"],   # pre-approve only what is needed
)
```

Then inspect the `system:init` message and treat `failed`/`needs-auth` as "tools unavailable", not as success.

#### Example 4: enhancing an MCP tool description so it beats Grep

Bad (the agent keeps using Grep):

```json
{ "name": "find_references", "description": "Searches code." }
```

Good:

```json
{
  "name": "find_references",
  "description": "Symbol-aware search over the indexed repositories. Given a function, class or type name, returns every definition and every call site with file path, line number, enclosing function and the argument expressions, resolving re-exports and aliases (so renamed imports are found). Use this INSTEAD of Grep when you need all callers or all usages of a symbol, or when the symbol may be imported under another name. Grep remains better for literal text, log messages, comments and config values. Output: up to 200 results, grouped by file; if truncated, narrow with the path_prefix parameter. Returns an empty list (not an error) when the symbol has no usages.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "symbol": { "type": "string", "description": "Exact symbol name, e.g. processRefund" },
      "path_prefix": { "type": "string", "description": "Optional directory filter, e.g. src/billing/" }
    },
    "required": ["symbol"]
  }
}
```

The "use this INSTEAD of Grep when ..." boundary, plus output shape and limits, is what moves selection.

#### Example 5: an MCP resource catalog instead of exploratory calls

The server advertises a catalog:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "resultType": "complete",
    "resources": [
      { "uri": "issues://summary/open", "name": "open-issues", "title": "Open issues summary",
        "description": "All open issues with id, title, status, labels, assignee.", "mimeType": "application/json" },
      { "uri": "docs://hierarchy", "name": "docs-tree", "title": "Documentation hierarchy",
        "description": "Table of contents of the developer docs with page ids.", "mimeType": "text/markdown" },
      { "uri": "schema://orders-db", "name": "orders-schema", "title": "Orders database schema",
        "description": "Tables, columns, types and foreign keys.", "mimeType": "application/json" }
    ]
  }
}
```

Using it in Claude Code: `Compare @orders:schema://orders-db with the refund logic in src/refunds/ and list queries that miss an index.`

```text
WITHOUT resources                         WITH resources
  tool: list_tables                         context: schema://orders-db attached
  tool: describe_table(orders)              -> model already knows tables/columns
  tool: describe_table(order_items)         -> goes straight to the targeted query
  tool: describe_table(refunds) ...
  (N exploratory calls, tokens, latency)
```

#### Example 6: choosing a community server vs building one

```text
"We need Claude to read and update Jira tickets."     -> use an existing Jira MCP server (project .mcp.json)
"We need a one-click 'release candidate' workflow
 that runs our checklist across 4 internal systems."  -> custom MCP server (team-specific)
```

### Exam traps

- **Putting a token directly in `.mcp.json`** or committing a personal config: use `${VAR}` and keep secrets in the environment.
- **Putting team servers in user scope** (`~/.claude.json`) so "every developer has them": user scope is personal; shared tooling belongs in `.mcp.json`.
- **"Build a custom Jira server."** Reserve custom servers for team-specific workflows; use the existing server for standard integrations.
- **Thin MCP descriptions** that lead the agent to prefer Grep/built-ins: fix the description (capabilities, outputs, when to prefer), not the system prompt alone.
- **Only the first server's tools are available**: false; all configured servers' tools are available simultaneously.
- **Using a tool where a resource fits** (or vice versa): catalogs of existing content = resources; actions and parameterized queries = tools; user-invoked workflows = prompts.
- **Confusing `allowedTools` (approval) with availability** (see 2.3).
- **Treating a `failed` or `needs-auth` server as working.** Check the init status. (`pending` is not automatically a failure: the server may still be connecting or may connect on first use, so re-check status later.)

### Check yourself

1. Your team wants every developer to get the GitHub and Jira servers when they clone the repo, without anyone's token in git. What do you create?
   <details><summary>Answer</summary>A project-scoped `.mcp.json` at the repo root with the servers defined, using `${GITHUB_TOKEN}` / `${JIRA_API_TOKEN}` expansion in `headers`/`env`; each developer sets the variables locally.</details>
2. You are trying a new experimental server and do not want teammates affected. Where does it go?
   <details><summary>Answer</summary>User scope (`claude mcp add --scope user ...`), stored in `~/.claude.json`. (Local scope, the default, would also be private but only for the current project.)</details>
3. Agents keep running 6-8 `list`/`get` calls to find out which documentation pages exist before each task. What MCP feature reduces this?
   <details><summary>Answer</summary>Expose the documentation hierarchy (and similar catalogs) as an MCP **resource**, so the agent gets the map up front instead of exploring through tool calls.</details>

### Official docs

- [Connect Claude Code to tools via MCP](https://code.claude.com/docs/en/mcp.md) — scopes, `.mcp.json`, `${VAR}` / `${VAR:-default}`, transports, resources via `@`, prompts as commands, tool search, output limits.
- [Agent SDK: Connect to external tools with MCP](https://code.claude.com/docs/en/agent-sdk/mcp.md) — `mcpServers`, `allowedTools` wildcards, tool naming, connection status.
- [Agent SDK: custom tools](https://code.claude.com/docs/en/agent-sdk/custom-tools.md) — in-process MCP servers, annotations.
- [Agent SDK: tool search](https://code.claude.com/docs/en/agent-sdk/tool-search.md) — deferring tools at scale, `ENABLE_TOOL_SEARCH`.
- [MCP spec: server overview](https://modelcontextprotocol.io/specification/2026-07-28/server/index.md), [Tools](https://modelcontextprotocol.io/specification/2026-07-28/server/tools.md), [Resources](https://modelcontextprotocol.io/specification/2026-07-28/server/resources.md), [Prompts](https://modelcontextprotocol.io/specification/2026-07-28/server/prompts.md) — who controls each primitive and their message shapes.
- [MCP architecture](https://modelcontextprotocol.io/docs/2026-07-28/learn/architecture.md) and [server concepts](https://modelcontextprotocol.io/docs/2026-07-28/learn/server-concepts.md) — hosts, clients, servers, discovery, primitives.

---

## 2.5 Select and apply built-in tools (Read, Write, Edit, Bash, Grep, Glob) effectively

### What the exam tests

- **Knowledge**
  - **Grep** searches file *contents* (function names, error messages, import statements).
  - **Glob** matches file *paths* by pattern (by name or extension).
  - **Read/Write** operate on whole files; **Edit** makes targeted modifications using unique text matching.
  - When Edit fails because the text is not unique, use **Read + Write** as the reliable fallback.
- **Skills**
  - Pick Grep to search code across a codebase (all callers of a function, locating an error message).
  - Pick Glob to find files by naming pattern (e.g. `**/*.test.tsx`).
  - Use Read to load the full file, then Write, when Edit cannot find a unique anchor.
  - Build codebase understanding **incrementally**: Grep for entry points, then Read to follow imports and trace flows, rather than reading everything up front.
  - Trace function usage across **wrapper modules**: first identify all exported names, then search for each name across the codebase.

### Concepts explained

#### Choosing the tool

| Tool | Use it for | Not for | Behaviour worth knowing (current docs) |
| --- | --- | --- | --- |
| **Grep** | Searching **inside files**: where a function is defined/called, where an error message appears, who imports a module | Finding files by name | Built on ripgrep: **ripgrep regex syntax**, so escape metacharacters (`interface\{\}`). Output modes: `files_with_matches` (default, paths only), `content` (lines with file and line number), `count`. Scope with `glob` (e.g. `**/*.tsx`) or `type` (e.g. `py`). Single-line matching by default; `multiline: true` spans lines. **Skips gitignored files.** |
| **Glob** | Finding **files by path/name pattern**: `**/*.test.tsx`, `src/**/*.ts`, `*.{json,yaml}` | Searching file contents | Results sorted by modification time, **capped at 100 files** (truncation flag tells Claude to narrow the pattern). Does **not** respect `.gitignore` by default. |
| **Read** | Loading a file's contents (with line numbers) to understand or to prepare a full rewrite; also images, PDFs (page ranges), notebooks | Directories (use a shell `ls`) | One call returns at most ~25,000 tokens; a whole file over 256 KB is refused by default, so read with `offset` and `limit` (or search with Grep); from Claude Code v2.1.296 Claude can set `allow_large: true` to read more in one call. Absolute paths. |
| **Write** | **Creating** a file or **replacing a whole file** | Small changes to existing files | Overwrites with the full content provided; never appends or merges. For existing files, models may require a prior Read in the conversation. |
| **Edit** | **Targeted** change: replace one exact piece of text with another | Many identical occurrences you can't disambiguate; creating files | Exact string replacement, **no regex or fuzzy matching**. `old_string` must match the file exactly (whitespace counts) and must occur **exactly once**; otherwise supply more surrounding context, or set `replace_all`. Older models always require a prior Read; newer ones may edit an unread file in some cases. |
| **Bash** | Running commands: tests, builds, `git`, package managers, listing directories, anything no dedicated tool covers | Routine file search/read/edit when a dedicated tool exists | Each command is a separate process; working directory can carry over in the main session, but **environment variables do not persist**. |

Why prefer dedicated tools over Bash for search/read/edit: they have purpose-built behaviour (structured results, output modes, permission rules scoped by tool, hook matchers by tool name) and give the model clearer, safer semantics. Bash is the general escape hatch.

Platform note: on macOS, Linux and WSL, Claude Code by default leaves Grep and Glob out of the tool set and searches with `find`/`grep` through Bash; naming them in `--tools`/`--allowedTools` (or in a subagent's `tools` when the subagent leaves out Bash) restores them, as does removing Bash from the session. The exam treats Grep and Glob as standard built-ins.

#### Edit's unique-match requirement and the Read + Write fallback

Edit replaces text only when the target is unambiguous:

- **0 matches** (typo, wrong whitespace, stale content): Edit fails; re-Read and copy the text exactly.
- **More than one match** (repeated boilerplate like `return null;`, identical `try/except` blocks, duplicated imports): Edit fails because it cannot tell which one you mean.

Resolution options, in the order to try them:

1. **Lengthen the anchor** with neighbouring lines until it is unique (the docs describe this as what Claude does).
2. If you truly want every occurrence changed, use `replace_all`.
3. If no unique anchor exists (or the change touches many scattered spots), **Read the whole file, produce the full modified content, and Write it back**. This is the exam's fallback: slower and heavier, but deterministic. Afterward, re-check that nothing else in the file changed unintentionally.

Do not use Write for a one-line change in a big file as a habit: you resend the whole file and risk collateral damage. Edit is the default for targeted changes; Read + Write is the fallback, and Write alone is for new files or deliberate full rewrites.

#### Build understanding incrementally

Reading every file in a repository up front wastes context, hits Read size limits, and buries the relevant code. The efficient pattern:

1. **Orient** with Glob/Grep: find entry points (`main`, routes, handlers) or the symbol you care about. Start with `files_with_matches` to see *where*, then `content` for the relevant lines.
2. **Read** only the files that matter; follow imports to the next layer.
3. **Repeat** with a new Grep for each newly discovered name, narrowing as you learn.
4. Stop when you can answer the question; keep a running summary rather than re-reading.

#### Tracing usage through wrapper modules

Callers often don't use the original name: a wrapper or barrel module re-exports under a different name. Grepping for the original function misses them.

Method (the guide's): **first identify every exported name of the wrapper, then search the codebase for each of those names** (and for the underlying one).

### Worked examples

#### Example 1: which tool? (Scenario 4: Developer productivity)

| Task | Tool | Call sketch |
| --- | --- | --- |
| Find every caller of `processRefund` | Grep | pattern `processRefund\(`, mode `content` |
| Find which files contain the error text "Refund window exceeded" | Grep | pattern `Refund window exceeded`, mode `files_with_matches` |
| List all test files for React components | Glob | `**/*.test.tsx` |
| Find only tests that mention `processRefund` | Grep (+ Glob filter) | pattern `processRefund`, `glob: "**/*.test.tsx"` |
| Understand `src/billing/refunds.ts` | Read | whole file (or `offset`/`limit` if large) |
| Create `docs/refund-flow.md` | Write | new file, full content |
| Change the timeout constant in one unique line | Edit | `old_string`: `const REFUND_TIMEOUT_MS = 5000;` |
| Run the unit tests | Bash | `npm test -- refunds` |
| See which files changed | Bash | `git status` |

#### Example 2: Edit fails on non-unique text; Read + Write fallback

File `src/billing/guards.ts`:

```ts
export function canRefund(order: Order): boolean {
  if (!order) return false;
  if (order.status === "cancelled") return false;
  return true;
}

export function canExchange(order: Order): boolean {
  if (!order) return false;
  if (order.status === "cancelled") return false;
  return true;
}
```

Goal: make only `canExchange` also reject `order.status === "final_sale"`.

```text
Edit  old_string: "  if (order.status === \"cancelled\") return false;\n  return true;"
      -> FAILS: text appears twice (non-unique)

Option 1 (lengthen anchor): include the function line
Edit  old_string: "export function canExchange(order: Order): boolean {\n  if (!order) return false;\n  if (order.status === \"cancelled\") return false;"
      new_string: same text plus "\n  if (order.status === \"final_sale\") return false;"   -> unique, succeeds

Option 2 (fallback when no unique anchor is practical): 
  Read  src/billing/guards.ts          -> full contents in context
  Write src/billing/guards.ts          -> complete updated file content
```

The exam answer for "Edit fails because the text is not unique" is the Read + Write fallback; lengthening the anchor is the lighter alternative you should try when a unique context exists.

#### Example 3: incremental exploration in an unfamiliar codebase

Question: "How does a refund get processed?"

```text
1. Grep   pattern: refund   mode: files_with_matches      -> 14 files (too many to read)
2. Grep   pattern: "export (async )?function .*[Rr]efund"  mode: content
          -> src/billing/refunds.ts:42 export async function processRefund(...)
          -> src/api/routes/refund.ts:11 router.post("/refund", handleRefund)
3. Read   src/api/routes/refund.ts                         -> imports handleRefund from ../handlers/refundHandler
4. Read   src/handlers/refundHandler.ts                    -> calls processRefund, auditLog
5. Grep   pattern: auditLog\(  mode: content               -> see where audits are written
6. Read   src/billing/refunds.ts (offset/limit around 42)  -> the actual logic
Stop. Answer with the trace entry point -> handler -> processRefund -> audit.
```

Anti-pattern: `Read` on all 14 files (plus their imports) before asking a question.

#### Example 4: tracing through a wrapper module

`src/payments/index.ts` (the wrapper):

```ts
export { charge as processPayment } from "./stripeClient";
export { refund as issueRefund } from "./stripeClient";
export { getBalance } from "./ledger";
```

Task: find all usages of the underlying `refund` function.

```text
WRONG: Grep "refund(" -> finds the definition and some internal calls; misses every
       caller that imports { issueRefund } from "../payments".

RIGHT:
1. Read (or Grep "^export" in) src/payments/index.ts
   -> exported names: processPayment, issueRefund, getBalance
2. Grep each exported name across the codebase:
   Grep "\bissueRefund\b"      -> callers via the wrapper
   Grep "\bprocessPayment\b"   -> (if relevant)
   Grep "\brefund\b" in files importing ./stripeClient directly -> direct callers
3. Merge the results: direct callers + callers through each re-exported name.
```

#### Example 5: Glob vs Grep mix-ups

```text
"Find all *.test.tsx files"            -> Glob  **/*.test.tsx            (names/paths)
"Find where 'ECONNRESET' is handled"   -> Grep  ECONNRESET               (contents)
"Find test files that mock fetch"      -> Grep  pattern "jest.mock\(.fetch", glob "**/*.test.tsx"
```

### Exam traps

- **Using Read to search** ("read every file to find callers"): wasteful and bounded by Read limits; use Grep.
- **Using Grep to find files by name** or **Glob to find text inside files**: wrong tool. Glob = paths, Grep = contents.
- **Using Write for a small edit** on a large file: unnecessary risk; Edit is for targeted changes.
- **Retrying the same Edit** with the same non-unique `old_string`: it will keep failing; lengthen the anchor or fall back to Read + Write.
- **Grepping only the original function name** when callers go through a wrapper: enumerate exports first.
- **Reading the entire codebase up front** "to be safe": the guide wants incremental exploration from entry points.
- **Using Bash for everything** (`cat`, `grep`, `sed`) when dedicated tools exist.
- **Assuming Edit does regex or fuzzy matching.** It does exact string replacement.

### Check yourself

1. You need every file that imports `legacyAuth`. Grep or Glob? What output mode first?
   <details><summary>Answer</summary>Grep (content search for the import statement), starting with `files_with_matches` to list the files, then `content` if you need the lines.</details>
2. An Edit to change `timeout: 30` to `timeout: 60` fails; the string appears in four config blocks and you only want to change the one under `payments:`. What do you do?
   <details><summary>Answer</summary>Include more surrounding text (e.g. the `payments:` line) so the `old_string` is unique. If no unique anchor is practical, Read the file and Write the full updated content (the fallback). `replace_all` would wrongly change all four.</details>
3. `src/utils/index.ts` re-exports `parseDate as toDate`. How do you find all uses of `parseDate`?
   <details><summary>Answer</summary>Read the wrapper to list its exported names (including `toDate`), then Grep for each name (`toDate`, `parseDate`) across the codebase and combine the results, since callers using the alias would be missed by searching only the original name.</details>

### Official docs

- [Tools reference (Claude Code)](https://code.claude.com/docs/en/tools-reference.md) — built-in tools table, plus Read/Write/Edit/Grep/Glob/Bash behaviour sections (unique-match rule, output modes, caps).
- [Subagents in the SDK: tool restrictions](https://code.claude.com/docs/en/agent-sdk/subagents.md) — common tool combinations per role.

---

## Decision cheat-table

| If the question says ... | Prefer ... | Because ... |
| --- | --- | --- |
| Agent confuses two tools with one-line descriptions; "most effective first step" | **Expand the descriptions** (formats, examples, edge cases, boundaries) | Descriptions are the primary selection mechanism; cheapest root-cause fix |
| Two tools named/described almost identically (`analyze_content` / `analyze_document`) | **Rename and sharpen**, or **split** into purpose-specific tools with contracts | Removes functional overlap |
| Misrouting began right after a system-prompt edit | **Review the prompt for keyword-sensitive wording** | Prompt and descriptions compete in one prompt |
| Agent skips a required prerequisite tool (verify customer before refund) | **Programmatic prerequisite / enforcement** | Descriptions and prompts are probabilistic; compliance must be guaranteed |
| Tool returns a generic "Operation failed" | **Structured error**: `isError: true`, `errorCategory`, `isRetryable`, message | Lets the agent choose retry vs fix vs explain vs escalate |
| Refund denied by policy | **Business error**, `retriable: false`, **customer-friendly message** | Retrying cannot help; agent must communicate it well |
| Search returned zero matches vs database unreachable | **Valid empty result** (success) vs **access failure** (`isError: true`) | Opposite follow-up behaviour; never disguise failure as empty success |
| Subagent hits a transient timeout | **Retry locally**; propagate only unresolved failures with attempted work and **partial results** | Local recovery first; coordinator needs context to decide |
| Synthesis agent keeps doing web searches / agent has 18 tools | **Scope each agent to 4-5 role-relevant tools** | Less decision complexity; blocks cross-role misuse |
| Synthesis needs frequent simple fact-checks (85/15 split) | **Scoped `verify_fact` tool**; complex cases via coordinator | Least privilege for the common case, keeps coordination for the rest |
| Generic `fetch_url` lets the agent wander | **Constrained `load_document`** that validates URLs | Policy encoded in the tool |
| Must run `extract_metadata` before anything else | **Forced `tool_choice`** `{"type":"tool","name":...}`, then follow-up turns | Deterministic first step |
| Must always get a tool call, not prose, but any of several tools is fine | **`tool_choice: "any"`** | Guarantees a tool call without naming one |
| Shared team servers with a token | **Project `.mcp.json` with `${GITHUB_TOKEN}`** | Version-controlled, no secrets committed |
| Personal or experimental MCP server | **User scope (`~/.claude.json`)** | Not shared, available across projects |
| Agent prefers Grep over a richer MCP tool | **Rewrite the MCP tool description** (capabilities, outputs, when to prefer) | Selection depends on description |
| Standard integration (Jira) | **Existing community server** | Custom servers only for team-specific workflows |
| Agent wastes calls discovering what data exists | **MCP resources as a content catalog** | Visibility without exploratory calls |
| Need all callers of a function | **Grep**; find test files by pattern: **Glob** | Content vs path search |
| Edit fails: text not unique | **Lengthen the anchor**, else **Read + Write** | Edit requires exactly one match |
| Understand an unfamiliar codebase | **Grep entry points, Read to follow imports**; list wrapper exports before searching | Incremental, not read-everything |

## Sources fetched

All fetched on 2026-10-30 (all returned successfully). A later review pass re-checked this file against full local copies of these pages; the four largest (`code.claude.com/docs/en/mcp.md`, tools-reference, parallel-tool-use, strict-tool-use) plus the tool-search-tool page were searched by passage and the relevant sections read:

- https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools.md
- https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls.md
- https://platform.claude.com/docs/en/agents-and-tools/tool-use/how-tool-use-works.md
- https://platform.claude.com/docs/en/agents-and-tools/tool-use/parallel-tool-use.md
- https://platform.claude.com/docs/en/agents-and-tools/tool-use/strict-tool-use.md
- https://platform.claude.com/docs/en/agents-and-tools/tool-use/manage-tool-context.md
- https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool.md
- https://platform.claude.com/docs/en/agents-and-tools/tool-use/troubleshooting-tool-use.md
- https://modelcontextprotocol.io/specification/2026-07-28/server/index.md
- https://modelcontextprotocol.io/specification/2026-07-28/server/tools.md
- https://modelcontextprotocol.io/specification/2026-07-28/server/resources.md
- https://modelcontextprotocol.io/specification/2026-07-28/server/prompts.md
- https://modelcontextprotocol.io/docs/2026-07-28/learn/architecture.md
- https://modelcontextprotocol.io/docs/2026-07-28/learn/server-concepts.md
- https://code.claude.com/docs/en/mcp.md
- https://code.claude.com/docs/en/tools-reference.md
- https://code.claude.com/docs/en/agent-sdk/mcp.md
- https://code.claude.com/docs/en/agent-sdk/custom-tools.md
- https://code.claude.com/docs/en/agent-sdk/tool-search.md
- https://code.claude.com/docs/en/agent-sdk/subagents.md
- https://www.anthropic.com/engineering/writing-tools-for-agents

Also used: the official exam guide v1.0 (July 2026), Domain 2 task statements 2.1-2.5, scenarios 1, 3, 4 and 6, and sample questions Q1, Q2, Q8, Q9.

