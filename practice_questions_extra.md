# Claude Certified Architect — Foundations: Extra Practice Questions (aligned to the official guide)

**547 more questions**, all checked against the official exam guide v1.0 (see [`practice_questions_audit.md`](practice_questions_audit.md)). Click **Show answer** to reveal.

## Read this first

- **Part A — multiple-response (33).** The real exam mixes single- and multiple-answer items and states how many to select. These were **written by me from the official guide's task statements** (each cites its TS). They are original, not exam content, and not reviewed by Anthropic — treat them as drills of the guide's own wording.
- **Part B — mock exam (57).** MIT-licensed community mock (Szymon Paluch); single-answer.
- **Part C — community bank (457).** MIT-licensed community banks (haytamAroui, Connectry LABS), single-answer, options shuffled to remove an upstream bias toward "B". Upstream they are all marked `draft` (not human-reviewed); I removed everything the audit found to be out of scope, beyond the blueprint, flawed, or in conflict with the guide.
- None of this is real exam content. The exam is under NDA and I did not use leaked material.
- **If an answer ever contradicts the official guide, the guide wins.**

## Contents

- [Part A — Multiple-response (33)](#part-a--multiple-response-33)
- [Part B — Mock exam (57)](#part-b--mock-exam-57)
- [Part C — Community bank (457)](#part-c--community-bank-457)
  - [Domain 1 · Agentic Architecture & Orchestration (27%)](#part-c-domain-1) — 92
  - [Domain 2 · Tool Design & MCP Integration (18%)](#part-c-domain-2) — 87
  - [Domain 3 · Claude Code Configuration & Workflows (20%)](#part-c-domain-3) — 75
  - [Domain 4 · Prompt Engineering & Structured Output (20%)](#part-c-domain-4) — 89
  - [Domain 5 · Context Management & Reliability (15%)](#part-c-domain-5) — 114
- [Licenses and attribution](#licenses-and-attribution)

---

## Part A — Multiple-response (33)

Select **exactly** the number of answers the item asks for. Scenarios use the official names.

#### X1 · Customer Support Resolution Agent

*TS 1.1 · Agentic loops*

Your support agent runs in a custom loop built on the Claude API. Some sessions end before a refund tool call has executed. Which TWO changes implement correct loop control? (Select TWO.)

- **A)** Continue the loop while stop_reason is "tool_use", appending each tool result to the conversation before the next request.
- **B)** Use a fixed iteration cap of 10 as the primary stopping mechanism.
- **C)** Stop as soon as the response contains any text block.
- **D)** Terminate the loop when stop_reason is "end_turn".
- **E)** Terminate when the assistant text contains a closing phrase such as "Is there anything else I can help with?"

<details>
<summary><b>Show answer</b></summary>

**Correct answers: A, D**

- **A ✓** The loop must keep iterating on "tool_use" and feed results back so the model can reason about its next action.
- **D ✓** "end_turn" is the structured completion signal; it is the correct termination condition.

**Why the others are wrong:**

- **B** Arbitrary iteration caps as the primary stop condition are a listed anti-pattern.
- **C** Responses can mix text and tool_use blocks; checking for text as a completion indicator is an anti-pattern.
- **E** Parsing natural-language signals for termination is a listed anti-pattern.

</details>

---

#### X2 · Multi-Agent Research System

*TS 1.3 · Subagent invocation & context passing*

Your coordinator must spawn a document-analysis subagent, but the subagent keeps producing generic output that ignores earlier web-search findings. Which TWO statements describe what is required? (Select TWO.)

- **A)** The prior agents' complete findings must be included directly in the subagent's prompt.
- **B)** The subagent automatically inherits the coordinator's conversation history.
- **C)** Subagents share memory between invocations, so earlier findings persist.
- **D)** An AgentDefinition is unnecessary because the subagent reuses the coordinator's system prompt and tools.
- **E)** The coordinator's allowedTools must include "Task" so it can spawn subagents.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: A, E**

- **A ✓** Subagent context must be provided explicitly in the prompt.
- **E ✓** The Task tool is the spawning mechanism, and allowedTools must include it.

**Why the others are wrong:**

- **B** Subagents operate with isolated context and do not inherit history automatically.
- **C** They do not share memory between invocations.
- **D** AgentDefinition carries each subagent type's description, system prompt and tool restrictions.

</details>

---

#### X3 · Multi-Agent Research System

*TS 1.3 · Subagent invocation & context passing*

You want the research coordinator to reduce latency and stay adaptable. Which TWO design choices support this? (Select TWO.)

- **A)** Let subagents message each other directly to avoid routing through the coordinator.
- **B)** Emit multiple Task tool calls in a single coordinator response so independent subagents run in parallel.
- **C)** Spawn each subagent in a separate coordinator turn so results can be inspected one at a time.
- **D)** Write the coordinator prompt around research goals and quality criteria rather than step-by-step procedures.
- **E)** Specify the exact procedural steps each subagent must follow to guarantee consistency.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: B, D**

- **B ✓** Parallel spawning is done by emitting multiple Task calls in one response, not across separate turns.
- **D ✓** Goal-and-criteria prompts let subagents adapt; rigid procedures do not.

**Why the others are wrong:**

- **A** Hub-and-spoke routing through the coordinator provides observability, consistent error handling and controlled information flow.
- **C** Separate turns serialize independent work and add latency.
- **E** Procedural step-by-step instructions reduce subagent adaptability.

</details>

---

#### X4 · Customer Support Resolution Agent

*TS 1.4 · Workflow enforcement & handoff* (also TS 1.5)

Policy: refunds above $500 must never be issued automatically. Separately, two MCP tools return dates as Unix timestamps and as ISO 8601 strings, confusing the agent. Which TWO implementations address these requirements deterministically? (Select TWO.)

- **A)** A system-prompt rule stating that refunds above $500 must never be processed, plus few-shot examples.
- **B)** Ask the agent to report a 1-10 confidence score and escalate when it is low.
- **C)** A PostToolUse hook that normalizes timestamp formats and status codes before the model processes the results.
- **D)** Add few-shot examples showing how to convert each timestamp format by hand.
- **E)** A tool-call interception hook that blocks process_refund above $500 and redirects to a human-escalation workflow.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: C, E**

- **C ✓** PostToolUse hooks transform heterogeneous tool output before the model sees it.
- **E ✓** Hooks give deterministic compliance where prompts have a non-zero failure rate.

**Why the others are wrong:**

- **A** Prompt guidance is probabilistic; business rules needing guaranteed compliance require hooks.
- **B** Self-reported confidence is a poorly calibrated proxy and not a deterministic control.
- **D** This leaves normalization to the model instead of normalizing data deterministically.

</details>

---

#### X5 · Customer Support Resolution Agent

*TS 1.4 · Workflow enforcement & handoff*

A human agent who cannot see the conversation transcript will take over an escalated case. Which THREE items belong in the structured handoff summary? (Select THREE.)

- **A)** The complete verbatim transcript, with no summary, so nothing is lost.
- **B)** The model's self-reported 1-10 confidence as the sole justification for escalating.
- **C)** The refund amount and the recommended next action.
- **D)** The root cause of the issue.
- **E)** The customer ID.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: C, D, E**

- **C ✓** These are the decision-relevant facts for the human.
- **D ✓** Lets the human act without re-investigating.
- **E ✓** Identifies the account without the transcript.

**Why the others are wrong:**

- **A** The handoff should be a compiled structured summary, not an unstructured dump.
- **B** Self-reported confidence is an unreliable proxy and not a handoff content item.

</details>

---

#### X6 · Developer Productivity with Claude

*TS 1.6 · Task decomposition*

Which TWO statements about task decomposition are correct? (Select TWO.)

- **A)** Prompt chaining suits predictable multi-aspect work such as reviewing each file and then running a cross-file integration pass.
- **B)** Dynamic decomposition should be avoided whenever intermediate findings could change the plan.
- **C)** A fixed sequential pipeline is the best choice for open-ended investigation tasks.
- **D)** Reviewing all files in one pass is preferable because a larger context removes attention dilution.
- **E)** Open-ended tasks such as "add comprehensive tests to a legacy codebase" call for mapping structure, finding high-impact areas, then a prioritized plan that adapts as dependencies are discovered.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: A, E**

- **A ✓** Fixed sequential pipelines fit predictable workflows.
- **E ✓** Dynamic, adaptive decomposition fits open-ended investigation.

**Why the others are wrong:**

- **B** Adapting the plan to intermediate findings is exactly its purpose.
- **C** Open-ended tasks need plans that adapt to intermediate findings.
- **D** Larger context does not fix attention quality; per-file plus integration passes do.

</details>

---

#### X7 · Developer Productivity with Claude

*TS 1.7 · Session state, resumption & forking*

A named codebase-investigation session is resumed after teammates changed several files the agent had analyzed. Which TWO statements are correct? (Select TWO.)

- **A)** Use fork_session to merge the histories of two sessions into one.
- **B)** Resume and assume earlier tool results remain valid; the agent will notice changed files by itself.
- **C)** If prior context is mostly valid, resume with --resume <session-name> and tell the agent which files changed so it re-analyzes only those.
- **D)** If prior tool results are stale, starting a new session with an injected structured summary is more reliable than resuming.
- **E)** Always discard the session and re-read every file in the repository.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: C, D**

- **C ✓** Resumption plus targeted change notices avoids full re-exploration.
- **D ✓** Stale tool results make resumption less reliable than a fresh start with a summary.

**Why the others are wrong:**

- **A** fork_session creates independent branches from a shared baseline; it does not merge.
- **B** The agent must be informed of file changes after resumption.
- **E** Wasteful; resumption with targeted updates is preferred when context is mostly valid.

</details>

---

#### X8 · Multi-Agent Research System

*TS 1.2 · Coordinator-subagent orchestration*

Why should all subagent communication be routed through the coordinator in a hub-and-spoke design? (Select THREE.)

- **A)** Observability of every inter-agent exchange.
- **B)** It removes the need for the coordinator to decompose tasks.
- **C)** Controlled flow of information between agents.
- **D)** Consistent error handling.
- **E)** It lets subagents inherit the coordinator's full conversation history automatically.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: A, C, D**

- **A ✓** Central routing gives one place to observe.
- **C ✓** The coordinator decides what each subagent receives.
- **D ✓** The coordinator handles failures uniformly.

**Why the others are wrong:**

- **B** Decomposition, delegation and aggregation remain core coordinator responsibilities.
- **E** Subagents do not inherit history automatically.

</details>

---

#### X9 · Multi-Agent Research System

*TS 2.1 · Tool descriptions & boundaries*

analyze_content and analyze_document have near-identical descriptions and the agent routes between them unreliably. Which TWO changes address the root cause? (Select TWO.)

- **A)** Rename analyze_content to extract_web_results and give it a web-specific description.
- **B)** Give both tools the same long description so the model sees full context.
- **C)** Split analyze_document into purpose-specific tools such as extract_data_points, summarize_content and verify_claim_against_source with defined input/output contracts.
- **D)** Add a system-prompt rule that keyword-matches "analyze" to a preferred tool.
- **E)** Switch to a larger model and leave the descriptions unchanged.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: A, C**

- **A ✓** Renaming and re-describing removes functional overlap.
- **C ✓** Purpose-specific tools with clear contracts differentiate each tool's role.

**Why the others are wrong:**

- **B** Identical descriptions preserve the ambiguity.
- **D** Keyword-sensitive instructions can create unintended tool associations.
- **E** Descriptions are the primary selection mechanism; a bigger model does not fix ambiguity.

</details>

---

#### X10 · Customer Support Resolution Agent

*TS 2.2 · Structured error responses*

Every failure from your MCP tools returns the text "Operation failed", so the agent retries permanent errors. Which THREE pieces of structured error metadata enable correct recovery? (Select THREE.)

- **A)** An errorCategory such as transient, validation or permission.
- **B)** An isRetryable boolean.
- **C)** A numeric severity score between 0 and 1 generated by the model.
- **D)** A human-readable description (customer-friendly for business-rule violations).
- **E)** The raw stack trace of the MCP server process.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: A, B, D**

- **A ✓** Lets the agent pick the right recovery.
- **B ✓** Prevents wasted retries of permanent errors.
- **D ✓** Lets the agent communicate appropriately.

**Why the others are wrong:**

- **C** Not a defined error-metadata pattern and not deterministic.
- **E** Does not support a recovery decision and exposes internals.

</details>

---

#### X11 · Multi-Agent Research System

*TS 2.2 · Structured error responses* (also TS 5.3)

A search subagent can fail by timing out, or can succeed with zero matches. Which TWO behaviors are correct? (Select TWO.)

- **A)** Treat zero matches as an error so the coordinator always retries.
- **B)** For a timeout, return structured error context: failure type, attempted query, partial results and possible alternatives.
- **C)** Terminate the whole research workflow on the first timeout.
- **D)** Return a query that ran but matched nothing as a valid empty result, not as an error.
- **E)** Catch the timeout and return an empty result set marked as successful.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: B, D**

- **B ✓** This lets the coordinator choose to retry, rework the query or proceed with partial results.
- **D ✓** Valid empty results are successful queries and must be distinguishable from access failures.

**Why the others are wrong:**

- **A** A valid empty result is not a failure.
- **C** Single failures should not terminate the workflow when recovery is possible.
- **E** Silently suppressing errors prevents any recovery.

</details>

---

#### X12 · Multi-Agent Research System

*TS 2.3 · Tool distribution & tool_choice*

A synthesis agent sometimes attempts web searches, and an agent with 18 tools selects tools unreliably. Which TWO changes help? (Select TWO.)

- **A)** Give every subagent all available tools so any agent can handle any task.
- **B)** Give the synthesis agent the full web-search toolset to avoid coordinator round trips.
- **C)** Raise the loop's iteration cap so the agent can try more tools.
- **D)** Restrict each subagent to the tools relevant to its role.
- **E)** Give the synthesis agent a scoped verify_fact tool for common simple lookups, while complex verification still routes through the coordinator.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: D, E**

- **D ✓** Fewer, role-specific tools improve selection reliability and prevent cross-specialization misuse.
- **E ✓** Scoped cross-role tools cover high-frequency needs without over-provisioning.

**Why the others are wrong:**

- **A** More tools increase decision complexity and degrade selection.
- **B** Over-provisions the synthesis agent and violates separation of concerns.
- **C** Does not address tool-selection reliability.

</details>

---

#### X13 · Developer Productivity with Claude

*TS 2.4 · MCP server integration*

Which THREE statements about configuring MCP servers for Claude Code are correct? (Select THREE.)

- **A)** .mcp.json can reference credentials via environment-variable expansion such as ${GITHUB_TOKEN}.
- **B)** Hardcode tokens in .mcp.json so every teammate gets a reproducible setup.
- **C)** Shared team servers belong in the project-scoped .mcp.json.
- **D)** Tools from configured servers are only discovered one server at a time, on demand.
- **E)** Personal or experimental servers belong in the user-scoped ~/.claude.json.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: A, C, E**

- **A ✓** Avoids committing secrets.
- **C ✓** Project scope is shared via version control.
- **E ✓** User scope is for personal servers.

**Why the others are wrong:**

- **B** Secrets must not be committed; use environment-variable expansion.
- **D** Tools from all configured servers are discovered at connection time and available simultaneously.

</details>

---

#### X14 · Developer Productivity with Claude

*TS 2.5 · Built-in tools (Read/Write/Edit/Bash/Grep/Glob)*

A developer needs (1) every caller of a function named parseInvoice and (2) every file matching **/*.test.tsx. Which TWO tool choices are correct? (Select TWO.)

- **A)** Use Grep to enumerate files by their path pattern.
- **B)** Use Glob to find the callers of parseInvoice.
- **C)** Use Grep to find the callers of parseInvoice by content.
- **D)** Read every file in the repository first and search them in context.
- **E)** Use Glob to find files matching **/*.test.tsx.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: C, E**

- **C ✓** Grep searches file contents (function names, error messages, imports).
- **E ✓** Glob matches file paths by name or extension patterns.

**Why the others are wrong:**

- **A** Path-pattern matching is Glob's job.
- **B** Glob matches paths, not file contents.
- **D** Build understanding incrementally (Grep, then Read to trace) rather than reading everything upfront.

</details>

---

#### X15 · Code Generation with Claude Code

*TS 3.1 · CLAUDE.md hierarchy & modularity*

A new teammate is not getting the team's coding standards in Claude Code, while everyone else is. Which THREE statements are correct? (Select THREE.)

- **A)** Instructions in ~/.claude/CLAUDE.md are automatically shared with teammates through version control.
- **B)** A subdirectory CLAUDE.md applies to files across the entire repository regardless of location.
- **C)** The /memory command shows which memory files are loaded, which helps diagnose inconsistent behavior across sessions.
- **D)** Project-level CLAUDE.md (root or .claude/CLAUDE.md) is version-controlled and reaches every teammate.
- **E)** Instructions in ~/.claude/CLAUDE.md apply only to that user and are not shared through version control.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: C, D, E**

- **C ✓** The guide names /memory for this diagnosis.
- **D ✓** Team standards belong at project level.
- **E ✓** This is the likely cause when standards were placed at user level.

**Why the others are wrong:**

- **A** User-level files are not shared.
- **B** Directory-level files are directory-bound.

</details>

---

#### X16 · Code Generation with Claude Code

*TS 3.2 · Slash commands & skills*

Which THREE statements about skills in .claude/skills/ and slash commands are correct? (Select THREE.)

- **A)** argument-hint prompts developers for required parameters when they invoke the skill without arguments.
- **B)** allowed-tools in skill frontmatter restricts which tools the skill may use while running.
- **C)** A personal variant should reuse the project skill's exact name in ~/.claude/skills/ so it takes effect for teammates.
- **D)** Commands stored in ~/.claude/commands/ are shared with the team through version control.
- **E)** context: fork runs the skill in an isolated sub-agent context so verbose output does not pollute the main conversation.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: A, B, E**

- **A ✓** Documented frontmatter option.
- **B ✓** Limits destructive actions.
- **E ✓** Isolation of verbose or exploratory output.

**Why the others are wrong:**

- **C** Personal variants should use different names so they do not affect teammates.
- **D** That path is personal; project commands live in .claude/commands/.

</details>

---

#### X17 · Code Generation with Claude Code

*TS 3.3 · Path-specific rules*

Test files (Button.test.tsx next to Button.tsx) are spread across the repo and must follow one set of conventions. Which TWO statements are correct? (Select TWO.)

- **A)** Place a CLAUDE.md in every directory that contains tests.
- **B)** Create per-type skills and rely on Claude to load them when relevant.
- **C)** Create a .claude/rules/ file whose YAML frontmatter paths field contains a glob such as **/*.test.tsx.
- **D)** Path-scoped rules load only when editing matching files, reducing irrelevant context and token usage.
- **E)** Put all conventions under headers in the root CLAUDE.md and rely on Claude to infer which applies.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: C, D**

- **C ✓** Glob-scoped rules apply by file type regardless of directory.
- **D ✓** Conditional loading is the advantage over always-loaded files.

**Why the others are wrong:**

- **A** Directory-bound files cannot easily cover files spread across the codebase.
- **B** Skills need invocation or model choice, which contradicts deterministic automatic application.
- **E** Relies on inference, not explicit matching.

</details>

---

#### X18 · Code Generation with Claude Code

*TS 3.4 · Plan mode vs direct execution*

Which TWO tasks are best suited to plan mode rather than direct execution? (Select TWO.)

- **A)** Fixing a single-file bug with a clear stack trace.
- **B)** Adding a date-validation conditional to one function.
- **C)** Migrating a library whose usage spans 45+ files.
- **D)** Restructuring a monolith into microservices with decisions about service boundaries.
- **E)** Renaming one local variable inside a function.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: C, D**

- **C ✓** Large-scale, multi-file change with architectural implications.
- **D ✓** Multiple valid approaches and architectural decisions.

**Why the others are wrong:**

- **A** Well-scoped; direct execution fits.
- **B** Simple and well-scoped; direct execution fits.
- **E** Trivial; direct execution fits.

</details>

---

#### X19 · Code Generation with Claude Code

*TS 3.5 · Iterative refinement*

Claude interprets your prose description of a data transformation inconsistently across runs. Which TWO approaches best fix this? (Select TWO.)

- **A)** Ask for a full rewrite without sharing any failures.
- **B)** Provide 2-3 concrete input/output examples of the transformation.
- **C)** Rewrite the prose with more adjectives and emphasis.
- **D)** Write tests covering expected behavior and edge cases first, then iterate by sharing the test failures.
- **E)** Report interacting issues one at a time across separate messages.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: B, D**

- **B ✓** Examples communicate transformations when prose is interpreted inconsistently.
- **D ✓** Test-driven iteration guides progressive improvement.

**Why the others are wrong:**

- **A** Withholding failures removes the feedback signal.
- **C** Still prose; the interpretation problem remains.
- **E** Interacting issues should be raised together in one detailed message.

</details>

---

#### X20 · Claude Code for Continuous Integration

*TS 3.6 · Claude Code in CI/CD*

Which THREE statements about running Claude Code in a CI pipeline are correct? (Select THREE.)

- **A)** Provide testing standards, fixtures and review criteria through CLAUDE.md.
- **B)** The session that generated the code is the best reviewer of that code.
- **C)** Use the -p (or --print) flag to run non-interactively and avoid hanging on input.
- **D)** Combine --output-format json with --json-schema to get machine-parseable findings for inline PR comments.
- **E)** Set CLAUDE_HEADLESS=true to prevent interactive prompts.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: A, C, D**

- **A ✓** CLAUDE.md supplies project context to CI-invoked Claude Code.
- **C ✓** Documented non-interactive mode.
- **D ✓** Enforces structured CI output.

**Why the others are wrong:**

- **B** An independent review instance is more effective than self-review.
- **E** This environment variable is not a documented mechanism.

</details>

---

#### X21 · Claude Code for Continuous Integration

*TS 4.1 · Explicit criteria & false positives*

Developers have stopped trusting your automated reviewer because of false positives. Which TWO changes are most effective? (Select TWO.)

- **A)** Temporarily disable the high-false-positive categories while their prompts are improved.
- **B)** Add "be conservative" to the prompt.
- **C)** Add "only report high-confidence findings" to the prompt.
- **D)** Cap output at three findings per pull request.
- **E)** Define explicit categorical criteria for what to report (bugs, security) versus skip (minor style, local patterns).

<details>
<summary><b>Show answer</b></summary>

**Correct answers: A, E**

- **A ✓** Restores trust while the categories are fixed.
- **E ✓** Specific criteria improve precision.

**Why the others are wrong:**

- **B** Vague instructions do not define what to flag.
- **C** General confidence instructions fail to improve precision.
- **D** An arbitrary cap does not target the false-positive categories.

</details>

---

#### X22 · Claude Code for Continuous Integration

*TS 4.2 · Few-shot prompting*

Review comments are inconsistently formatted and ambiguous cases are handled unpredictably. Which TWO few-shot practices help most? (Select TWO.)

- **A)** Provide twenty examples of the easiest, most common case.
- **B)** Use examples only to list the exact patterns to match, so the model never generalizes.
- **C)** Include examples showing the desired output format: location, issue, severity and suggested fix.
- **D)** Replace all explicit criteria with examples.
- **E)** Use 2-4 targeted examples of ambiguous cases that show why one action was chosen over plausible alternatives.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: C, E**

- **C ✓** Format demonstrations produce consistent, actionable output.
- **E ✓** Targeted examples teach the judgment process.

**Why the others are wrong:**

- **A** Does not address the ambiguous cases.
- **B** Few-shot examples are valuable because they enable generalization to novel patterns.
- **D** Examples complement criteria; they do not replace them.

</details>

---

#### X23 · Structured Data Extraction

*TS 4.3 · Structured output via tool use & schemas*

Your pipeline receives documents of unknown type and has several extraction tools. A separate extract_metadata step must always run first. Which TWO configurations are correct? (Select TWO.)

- **A)** Set tool_choice to "any" so the model must call a tool, choosing which extraction schema fits.
- **B)** Force extract_metadata on every call in every turn.
- **C)** Ask for JSON in plain text and parse it with a regex.
- **D)** Force {"type": "tool", "name": "extract_metadata"} on the first call, then handle later steps in follow-up turns.
- **E)** Set tool_choice to "auto" and rely on the model to call a tool.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: A, D**

- **A ✓** "any" guarantees structured output when the document type is unknown.
- **D ✓** Forced selection ensures a specific tool runs first.

**Why the others are wrong:**

- **B** Forced selection for all turns would prevent later steps from running.
- **C** tool_use with JSON schemas is the most reliable structured-output approach.
- **E** "auto" may return conversational text instead of a tool call.

</details>

---

#### X24 · Structured Data Extraction

*TS 4.3 · Structured output via tool use & schemas*

Your extraction model sometimes fabricates values for fields missing from source documents. Which THREE schema design practices help? (Select THREE.)

- **A)** Rely on strict JSON schemas to guarantee that line items sum to the stated total.
- **B)** Make fields optional (nullable) when the source document may not contain the information.
- **C)** Mark every field required so no field is ever omitted.
- **D)** Add an "unclear" enum value for ambiguous cases.
- **E)** Use "other" plus a detail string for extensible categories.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: B, D, E**

- **B ✓** Prevents the model from fabricating values to satisfy required fields.
- **D ✓** Gives the model a truthful option.
- **E ✓** Handles categories not enumerated in advance.

**Why the others are wrong:**

- **A** Strict schemas remove syntax errors, not semantic errors.
- **C** Required fields push the model to fabricate.

</details>

---

#### X25 · Structured Data Extraction

*TS 4.4 · Validation, retry & feedback loops*

Extractions sometimes fail validation. Which TWO statements about validation-retry loops are correct? (Select TWO.)

- **A)** On retry, send the original document, the failed extraction and the specific validation errors.
- **B)** Extract calculated_total alongside stated_total to flag discrepancies.
- **C)** Tool-use strict schemas also guarantee that values land in the correct fields.
- **D)** Retry repeatedly when the needed information exists only in an external document that was not provided.
- **E)** Resend only the validation error without the document.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: A, B**

- **A ✓** Retry-with-error-feedback guides self-correction.
- **B ✓** A self-correction validation pattern for semantic errors.

**Why the others are wrong:**

- **C** They eliminate syntax errors but not semantic errors such as wrong-field placement.
- **D** Retries cannot succeed when the information is absent from the source.
- **E** The model needs the document and the failed extraction to correct itself.

</details>

---

#### X26 · Claude Code for Continuous Integration

*TS 4.5 · Batch processing*

Which TWO statements about the Message Batches API are correct? (Select TWO.)

- **A)** It offers 50% cost savings with up to a 24-hour processing window and no guaranteed latency SLA, suiting overnight or weekly workloads.
- **B)** Batches are guaranteed to complete within one hour.
- **C)** It is appropriate for blocking pre-merge checks because results usually arrive quickly.
- **D)** It supports multi-turn tool calling inside a single request.
- **E)** Use custom_id to correlate requests and responses, and resubmit only the failed documents.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: A, E**

- **A ✓** Latency-tolerant, non-blocking workloads fit.
- **E ✓** custom_id supports targeted resubmission.

**Why the others are wrong:**

- **B** Up to 24 hours, with no guarantee.
- **C** No latency SLA makes it unsuitable for blocking workflows.
- **D** It cannot execute tools mid-request and return results.

</details>

---

#### X27 · Claude Code for Continuous Integration

*TS 4.6 · Multi-instance & multi-pass review*

Which TWO statements about review architectures are correct? (Select TWO.)

- **A)** Split large reviews into per-file local passes plus a separate cross-file integration pass.
- **B)** Asking the same session to critically re-review its own output is as effective as an independent instance.
- **C)** Run three full-PR passes and report only issues found in at least two.
- **D)** An independent review instance without the generator's reasoning context is more effective at catching subtle issues than self-review.
- **E)** A larger context window removes the need for multi-pass review.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: A, D**

- **A ✓** Avoids attention dilution and contradictory findings.
- **D ✓** The generating session retains reasoning context and is less likely to question its decisions.

**Why the others are wrong:**

- **B** Self-review is limited by retained reasoning context.
- **C** Requiring consensus suppresses real, intermittently detected bugs.
- **E** Larger context does not fix attention quality.

</details>

---

#### X28 · Customer Support Resolution Agent

*TS 5.1 · Context preservation*

In long support sessions the agent loses order numbers and amounts, and tool results with 40+ fields per lookup crowd the context. Which THREE changes help? (Select THREE.)

- **A)** Keep transactional facts (amounts, dates, order numbers, statuses) in a persistent case-facts block outside the summarized history.
- **B)** Trim verbose tool outputs to only the relevant fields before they accumulate in context.
- **C)** Place the most important findings in the middle of long inputs.
- **D)** Summarize all numeric values into qualitative phrases to save space.
- **E)** Place key-findings summaries at the start of aggregated inputs and use explicit section headers.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: A, B, E**

- **A ✓** Prevents progressive summarization from degrading critical values.
- **B ✓** Reduces tokens spent on irrelevant fields.
- **E ✓** Mitigates lost-in-the-middle position effects.

**Why the others are wrong:**

- **C** Models process the beginning and end more reliably than the middle.
- **D** Condensing numbers and dates into vague summaries is the risk to avoid.

</details>

---

#### X29 · Customer Support Resolution Agent

*TS 5.2 · Escalation & ambiguity resolution*

Which TWO escalation behaviors are correct? (Select TWO.)

- **A)** Escalate when sentiment analysis detects frustration.
- **B)** When a lookup returns several matching customers, pick the most recent by heuristic.
- **C)** Escalate when the agent's self-reported confidence score falls below a threshold.
- **D)** Escalate when policy is silent or ambiguous on the request, such as competitor price matching when policy covers only own-site adjustments.
- **E)** Escalate immediately when the customer explicitly asks for a human, without first attempting investigation.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: D, E**

- **D ✓** Policy gaps are an appropriate trigger.
- **E ✓** Explicit requests are honored right away.

**Why the others are wrong:**

- **A** Sentiment does not correlate with case complexity.
- **B** Ask for additional identifiers instead of guessing.
- **C** Self-reported confidence is an unreliable proxy for complexity.

</details>

---

#### X30 · Multi-Agent Research System

*TS 5.3 · Error propagation in multi-agent systems*

A web-search subagent hits transient failures. Which TWO practices are correct? (Select TWO.)

- **A)** Abort the entire workflow on any single failure.
- **B)** Annotate the synthesis output with coverage gaps for topics whose sources were unavailable.
- **C)** Recover locally from transient failures and propagate only errors that cannot be resolved, including what was attempted and partial results.
- **D)** Return an empty result marked successful so the workflow keeps moving.
- **E)** Return a generic "search unavailable" status.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: B, C**

- **B ✓** Coverage annotations separate well-supported findings from gaps.
- **C ✓** Local recovery first, informative propagation second.

**Why the others are wrong:**

- **A** Unnecessarily discards recoverable work.
- **D** Silent suppression is an anti-pattern.
- **E** Generic statuses hide context the coordinator needs.

</details>

---

#### X31 · Developer Productivity with Claude

*TS 5.4 · Large-codebase context management*

During an extended exploration of a large unfamiliar codebase the agent begins citing "typical patterns" instead of specific classes found earlier. Which THREE actions help? (Select THREE.)

- **A)** Continue in the same context and trust the agent's general knowledge of typical patterns.
- **B)** Have the agent keep scratchpad files recording key findings and consult them for later questions.
- **C)** Delegate specific questions (such as finding all test files) to subagents so verbose output stays out of the main context.
- **D)** Use /compact to reduce context usage when it fills with verbose discovery output.
- **E)** Read all files upfront so everything is in context from the start.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: B, C, D**

- **B ✓** Persists findings across context boundaries.
- **C ✓** Subagents isolate verbose exploration.
- **D ✓** Documented context-reduction tool.

**Why the others are wrong:**

- **A** That is the degradation symptom, not a fix.
- **E** Build understanding incrementally instead.

</details>

---

#### X32 · Structured Data Extraction

*TS 5.5 · Human review & confidence calibration*

Your extraction pipeline reports 97% overall accuracy and the team wants to cut human review. Which TWO steps should come first? (Select TWO.)

- **A)** Sample high-confidence extractions with stratified random sampling to measure error rates and find novel error patterns.
- **B)** Review only the low-confidence extractions and never sample the rest.
- **C)** Use the model's uncalibrated self-reported confidence directly as the routing threshold.
- **D)** Treat 97% overall as sufficient to automate all high-confidence extractions.
- **E)** Analyze accuracy by document type and by field to confirm consistent performance across segments.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: A, E**

- **A ✓** Ongoing measurement before reducing review.
- **E ✓** Aggregate metrics can mask poor performance on specific segments.

**Why the others are wrong:**

- **B** High-confidence errors would go undetected.
- **C** Confidence should be calibrated with labeled validation sets.
- **D** Aggregate accuracy may hide weak segments.

</details>

---

#### X33 · Multi-Agent Research System

*TS 5.6 · Provenance & multi-source synthesis*

Your synthesis agent merges findings from many sources and sometimes loses attribution or misreads differing statistics as contradictions. Which THREE practices help? (Select THREE.)

- **A)** Require publication or data-collection dates in structured outputs.
- **B)** Keep the value from the most recent source and drop the other.
- **C)** Have subagents output structured claim-source mappings (URLs, document names, excerpts) that synthesis preserves and merges.
- **D)** Annotate conflicting statistics with their source attribution instead of arbitrarily choosing one value.
- **E)** Convert every content type into one uniform bulleted list.

<details>
<summary><b>Show answer</b></summary>

**Correct answers: A, C, D**

- **A ✓** Prevents temporal differences from being misread as contradictions.
- **C ✓** Prevents attribution loss during summarization.
- **D ✓** Lets the coordinator reconcile with context.

**Why the others are wrong:**

- **B** Silently picking one value loses information and attribution.
- **E** Render financial data as tables, news as prose and technical findings as structured lists.

</details>

---

## Part B — Mock exam (57)

#### B1 · Customer Support Resolution Agent

*TS 1.1 · Agentic loops*

A customer support resolution agent built with the Claude Agent SDK reaches backend systems through custom MCP tools (get_customer, lookup_order, process_refund, escalate_to_human). The engineering team wrote a wrapper loop that ends the conversation whenever the assistant's text matches a closing phrase like "let me know if there's anything else." Production logs show that in 11% of sessions the loop terminates while a tool_use block is still pending, so the refund is never actually executed. What is the correct way to drive the agentic loop?

- **A)** Terminate only when stop_reason is end_turn; whenever stop_reason is tool_use, execute the requested tool, append the tool_result to the message list, and continue the loop.
- **B)** Expand the regex allowlist of closing phrases so it recognizes every polite sign-off the model might emit before stopping.
- **C)** Add a max-iteration cap and stop the loop once the assistant produces two consecutive text-only responses.
- **D)** Run a lightweight intent classifier on each assistant message to decide whether the turn represents a genuine completion.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

The agentic loop must be driven by the API's stop_reason signal, not by parsing natural-language content. tool_use means the model wants a tool run, so you execute it, append the tool_result, and iterate; end_turn is the only correct termination signal.

- **Why B is wrong:** Matching closing phrases (B) is exactly the brittle text-parsing that caused the bug and will still misfire.
- **Why C is wrong:** An iteration cap plus consecutive text heuristic (C) can truncate legitimate multi-step work and still ignores stop_reason.
- **Why D is wrong:** A classifier (D) is over-engineering that reintroduces natural-language inference where a deterministic protocol field already gives the answer.

</details>

---

#### B2 · Multi-Agent Research System

*TS 1.1 · Agentic loops*

The coordinator's agentic loop sometimes stops before the report subagent has run: it ends whenever the latest assistant text contains the phrase 'research complete'. Intermittently the synthesis subagent uses that phrase mid-analysis, halting the workflow. How should loop termination be driven?

- **A)** Add a minimum iteration count before the coordinator is allowed to stop
- **B)** Use a stricter regex that also requires the word 'final'
- **C)** Continue the loop while stop_reason is tool_use (appending each tool result) and stop only on end_turn
- **D)** Have a classifier judge whether the report is truly finished

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

The agentic loop must be driven by the API's stop_reason — continue and append the tool result while stop_reason is tool_use, and terminate only on end_turn — never by parsing the model's natural-language text.

- **Why A is wrong:** A minimum iteration count (A) masks the bug with an arbitrary floor.
- **Why B is wrong:** A stricter regex (B) still parses assistant text and remains brittle.
- **Why D is wrong:** A classifier (D) is over-engineering for a deterministic termination signal the API already provides.

</details>

---

#### B3 · Multi-Agent Research System

*TS 1.2 · Coordinator-subagent orchestration*

Logs show the document-analysis subagent attempting to invoke the web-search subagent directly to fill gaps, producing routing errors and duplicated searches. How should the topology be corrected?

- **A)** Route all requests through the coordinator: subagents return gaps to it, and it dispatches any follow-up web searches
- **B)** Let subagents call each other but add a lock to prevent duplicate searches
- **C)** Merge the document-analysis and web-search subagents into one general subagent
- **D)** Give every subagent the full tool set so none needs to call another

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

The correct topology is hub-and-spoke: subagents report findings and gaps back to the coordinator, which dispatches any follow-up work — peer-to-peer subagent calls are the defect.

- **Why B is wrong:** Adding a lock (B) legitimizes and entrenches the wrong topology.
- **Why C is wrong:** Merging the two subagents (C) collapses the intended specialization and differentiation.
- **Why D is wrong:** Giving every subagent the full tool set (D) over-provisions tools, degrades tool selection, and still bypasses the coordinator's orchestration.

</details>

---

#### B4 · Customer Support Resolution Agent

*TS 1.3 · Subagent invocation & context passing*

The coordinator has already run get_customer and confirmed the account, then spawns subagents to check order history and refund eligibility in parallel. Those subagents return results that contradict the confirmed account — some even re-run get_customer and pick a different customer match. What is the correct fix?

- **A)** Enlarge the shared context window so the subagents can see the coordinator's earlier turns.
- **B)** Store the confirmed customer record in a global variable that the SDK automatically injects into every subagent.
- **C)** Instruct each subagent to call get_customer itself at the start so it independently establishes the account.
- **D)** Include the coordinator's already-verified findings — the confirmed customer record and identifiers — explicitly in each subagent's Task prompt, since subagents do not inherit the coordinator's conversation context.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Subagents run with their own fresh context and do not inherit the coordinator's conversation, so any prior findings they must use have to be passed explicitly in the Task prompt. Handing each subagent the confirmed customer record eliminates the contradictions and the redundant lookups.

- **Why A is wrong:** Enlarging a "shared" context window (A) misdescribes the architecture — there is no shared window to grow across the hub-and-spoke boundary.
- **Why B is wrong:** There is no SDK mechanism that auto-injects a global variable into subagents (B); it is a plausible-sounding non-feature.
- **Why C is wrong:** Having each subagent re-run get_customer (C) is exactly what produced the divergent matches and duplicates work.

</details>

---

#### B5 · Multi-Agent Research System

*TS 1.3 · Subagent invocation & context passing*

The synthesis subagent's reports read as generic overviews: in 30% of runs they omit the specific figures and named sources the web-search subagent surfaced. Tracing shows the Task prompt sent to the synthesis subagent contains only the report topic and formatting instructions. What is the best fix?

- **A)** Increase the coordinator's context window so more findings stay resident across delegations
- **B)** Have the coordinator embed the web-search and document-analysis findings verbatim in the synthesis subagent's Task prompt
- **C)** Add a 'recall the earlier research' instruction to the synthesis subagent's system prompt
- **D)** Give the synthesis subagent its own web-search tool so it can re-gather the missing figures

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Subagents do not inherit the coordinator's conversation context; in a hub-and-spoke topology the coordinator must pass prior findings explicitly in the Task prompt, so embedding the web-search and document-analysis results verbatim is the fix.

- **Why A is wrong:** A larger context window (A) does not cross the process boundary into a freshly spawned subagent.
- **Why C is wrong:** Instructing it to 'recall' earlier research (C) fails because the subagent starts with no prior turns to recall.
- **Why D is wrong:** Giving it its own search tool (D) is wrong scope and over-engineering — it duplicates work and diverges from the evidence already gathered.

</details>

---

#### B6 · Multi-Agent Research System

*TS 1.3 · Subagent invocation & context passing*

The coordinator dispatches three independent regional web-search subagents (EU, US, APAC), but issues each Task call only after the previous subagent returns. End-to-end research latency is about 3x the slowest single search. What change most reduces latency?

- **A)** Emit all three Task calls in a single assistant response so the subagents execute concurrently
- **B)** Merge the three regional searches into one subagent that loops over regions internally
- **C)** Add a fixed iteration cap so the coordinator stops waiting after a set number of rounds
- **D)** Set tool_choice to 'any' so the coordinator selects tools faster

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Parallel subagents are launched by issuing multiple Task calls in one assistant response; because the three regional searches are independent, doing so collapses their latency to roughly the slowest single search.

- **Why B is wrong:** Merging into one looping subagent (B) removes the parallelism and the per-region differentiation.
- **Why C is wrong:** An iteration cap (C) does not address latency and risks dropping regions entirely.
- **Why D is wrong:** tool_choice 'any' (D) governs which tool the model picks, not whether subagents run concurrently.

</details>

---

#### B7 · Multi-Agent Research System

*TS 1.3 · Subagent invocation & context passing*

A newly configured coordinator answers research questions directly from its own knowledge instead of delegating; its attempts to invoke subagents never appear in the trace. The subagent definitions exist and work when tested standalone. What is the root cause?

- **A)** The subagents were registered at user scope while the coordinator runs at project scope
- **B)** The coordinator's model is too small to plan multi-agent delegation
- **C)** The coordinator's allowedTools list omits 'Task', so it cannot spawn subagents
- **D)** tool_choice is forced to a specific tool, blocking delegation

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

To spawn subagents the coordinator's allowedTools must include 'Task'; when it is absent the coordinator has no delegation mechanism and silently answers on its own, exactly as observed.

- **Why A is wrong:** Scope mismatch (A) is not the gating mechanism for spawning and the subagents work standalone.
- **Why B is wrong:** Model size (B) does not explain a total absence of delegation attempts — the capability is simply not exposed.
- **Why D is wrong:** A forced tool_choice (D) would produce visible forced tool calls, not silent non-delegation.

</details>

---

#### B8 · Developer Productivity with Claude

*TS 1.3 · Subagent invocation & context passing*

The coordinator spawns a subagent with a single Task call to trace how a payment webhook propagates through the `orders` module. Logs show the subagent re-runs Grep to rediscover the entrypoint the coordinator had already located two turns earlier, and reaches a conclusion that contradicts the coordinator's established finding. What is the root cause?

- **A)** The subagent's allowedTools list is missing the Task tool, so it cannot delegate the trace further
- **B)** The Message Batches API strips prior turns, so the multi-turn context was silently dropped
- **C)** Subagents do not inherit the coordinator's conversation, so prior findings must be passed explicitly in the Task prompt
- **D)** tool_choice was set to "auto" instead of forcing the Grep tool for the trace

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

In a hub-and-spoke design each subagent starts with a clean context and sees only what the coordinator writes into its Task prompt, so unless prior findings are passed in, the subagent rediscovers and can contradict them — making C correct.

- **Why A is wrong:** A is wrong because the Task tool governs spawning further subagents, not inheriting context.
- **Why B is wrong:** B is irrelevant: interactive delegation does not run through the Batches API.
- **Why D is wrong:** D is wrong because tool_choice controls which tool is invoked, not whether earlier conclusions are carried over.

</details>

---

#### B9 · Customer Support Resolution Agent

*TS 1.4 · Workflow enforcement & handoff*

When the support agent invokes escalate_to_human, human agents report they must re-read the entire raw conversation to understand each case, and average handle time for escalated tickets has risen. What should the agent do at the moment of escalation to fix this?

- **A)** Attach the full untrimmed transcript of the conversation so the human has complete information.
- **B)** Enable extended thinking on the escalation turn so the model reasons longer before handing off.
- **C)** Escalate earlier in ambiguous cases so humans receive tickets before much conversation has accumulated.
- **D)** Produce a structured handoff summary — the customer's issue, verified identifiers, tool results and account state gathered, actions already attempted, and the specific recommended next step.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Effective human handoff requires a structured summary that front-loads exactly what the human needs: the problem, confirmed identifiers, what was tried, and a recommendation. That is what removes the re-reading burden and shortens handle time.

- **Why A is wrong:** Dumping the full transcript (A) is the current failure mode — it forces the human to reconstruct the case themselves.
- **Why B is wrong:** Extended thinking (B) improves the model's own reasoning but produces no artifact for the human.
- **Why C is wrong:** Escalating earlier (C) changes when tickets arrive, not the missing information problem, and would likely lower first-contact resolution.

</details>

---

#### B10 · Customer Support Resolution Agent

*TS 1.5 · Agent SDK hooks*

Company policy states that any refund above \$500 must be approved by a manager rather than auto-issued. The support agent, however, has autonomously called process_refund for amounts over \$500 in several audited cases. What is the best way to guarantee the policy holds?

- **A)** Add a PreToolUse interception hook on process_refund that blocks any call with an amount over \$500 and redirects the flow to escalate_to_human.
- **B)** Add a line to the system prompt instructing the agent to always escalate refunds over \$500 instead of processing them.
- **C)** Remove process_refund from the agent's toolset entirely so every refund is handled by a human.
- **D)** Fine-tune the model on historical tickets so it learns the \$500 threshold from past manager approvals.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

A policy violation that must never occur calls for an interception (PreToolUse) hook that deterministically blocks the disallowed process_refund call and redirects to escalation. The hook enforces the threshold regardless of what the model decides.

- **Why B is wrong:** A prompt instruction (B) is advisory and is precisely what the audited violations bypassed.
- **Why C is wrong:** Removing the tool (C) throws away all legitimate sub-\$500 automation and tanks first-contact resolution.
- **Why D is wrong:** Fine-tuning (D) is heavy, slow, and still probabilistic — it cannot promise the hard \$500 boundary is never crossed.

</details>

---

#### B11 · Customer Support Resolution Agent

*TS 1.5 · Agent SDK hooks*

get_customer and lookup_order call two different backend systems that return dates in different formats (ISO vs. US) and amounts with inconsistent currency notation. Logs show the model occasionally misreads these heterogeneous fields, e.g., treating a refund amount in the wrong units. What is the most robust remedy?

- **A)** Instruct the model in the system prompt to normalize all dates and currency values before reasoning about them.
- **B)** Add a PostToolUse hook that normalizes each tool's raw output into one canonical schema before it is appended to the context.
- **C)** Increase the context window so the model has more room to keep the varied formats straight.
- **D)** Spin up a dedicated formatting subagent that the coordinator calls after every backend lookup.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

A PostToolUse hook is the designed mechanism for normalizing heterogeneous tool output into a consistent schema before it ever reaches the model, eliminating the ambiguity deterministically.

- **Why A is wrong:** Asking the model to normalize in-prompt (A) leaves the same error-prone inference in place — the source of the current misreads.
- **Why C is wrong:** A larger context window (C) addresses volume, not format inconsistency, and does nothing for unit confusion.
- **Why D is wrong:** A separate formatting subagent (D) adds a full round-trip and orchestration overhead for a transformation a hook performs inline and reliably.

</details>

---

#### B12 · Multi-Agent Research System

*TS 1.6 · Task decomposition*

For open-ended briefs like 'assess the competitive landscape for X', the coordinator runs a fixed three-step chain (search then summarize then format). Reviewers note the reports consistently miss emerging subtopics that only surface mid-investigation. What orchestration change best fits this workload?

- **A)** Add few-shot examples of good reports to the formatting step
- **B)** Raise the temperature of the synthesis subagent to encourage broader coverage
- **C)** Keep the fixed chain but add a second formatting pass
- **D)** Switch to dynamic decomposition so the coordinator spawns follow-up subtasks based on what earlier findings reveal

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Open-ended investigation calls for dynamic decomposition, where the coordinator spawns follow-up subtasks in response to what earlier findings reveal; a fixed chain suits only predictable, well-defined multi-aspect work and structurally cannot chase subtopics it did not pre-plan.

- **Why A and C are wrong:** Few-shot formatting examples (A) and a second formatting pass (C) improve presentation, not investigative coverage.
- **Why B is wrong:** Raising temperature (B) adds randomness, not the adaptive depth that surfaces emerging subtopics.

</details>

---

#### B13 · Developer Productivity with Claude

*TS 1.6 · Task decomposition*

A coordinator agent is asked to produce a migration plan for a 400,000-line legacy billing monolith. The engineer hard-codes an upfront decomposition into five subagent tasks (billing, invoicing, tax, reporting, auth). Production runs show the final plan omits an entire 'dunning' subsystem and a currency-conversion layer that no subagent was ever assigned. What is the most effective decomposition strategy?

- **A)** Raise the coordinator's maximum iteration count so it has more turns to notice the missing subsystems on its own
- **B)** Run an initial mapping pass that enumerates the repository's actual modules, then dynamically spawn one subagent per discovered component
- **C)** Expand the fixed list from five to twelve subagent tasks based on the engineer's prior knowledge of the billing domain
- **D)** Give the coordinator a larger context window so the entire monolith fits inside a single prompt

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

For unfamiliar or legacy code you cannot know the true structure in advance, so any static decomposition guarantees blind spots — exactly the omitted 'dunning' and currency layers. The governing pattern is mapping-then-adaptive decomposition: discover the real module structure first, then decompose against what actually exists, which is why B is correct.

- **Why A is wrong:** A is wrong because more iterations do not change the coverage of a fixed assignment list.
- **Why C is wrong:** C is still a static decomposition bounded by incomplete prior knowledge — the missing subsystems were unknown, so listing more names from memory does not surface them.
- **Why D is wrong:** D confuses context size with decomposition; the failure is missed coverage, not insufficient window.

</details>

---

#### B14 · Multi-Agent Research System

*TS 1.7 · Session state, resumption & forking*

After the shared search and document-analysis phase completes, stakeholders want two report variants from the same evidence — a one-page executive brief and a technical deep-dive — without re-running the costly searches. What is the best way to produce both?

- **A)** Continue the same session, generate the brief, then ask for the deep-dive in the next turn
- **B)** Fork the session at the end of the research phase into two branches, one per report variant
- **C)** Start two fresh sessions and re-run the search subagents for each
- **D)** Increase the context window so both reports fit in one generation

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

fork_session creates divergent branches that share all prior state, so forking after the research phase lets each branch produce one variant from the same gathered evidence without re-searching.

- **Why A is wrong:** Continuing one session (A) entangles both outputs in a single history and biases the deep-dive on the already-generated brief.
- **Why C is wrong:** Two fresh sessions (C) wastefully re-run the expensive searches.
- **Why D is wrong:** A bigger context window (D) is unrelated to branching divergent outputs.

</details>

---

#### B15 · Multi-Agent Research System

*TS 1.7 · Session state, resumption & forking*

A weekly market report is produced by --resume of a named coordinator session. This week the report cites last week's prices as current, because the resumed session replays cached web-search tool results. What is the best fix?

- **A)** Fork the prior session so the stale results are copied into a new branch
- **B)** Keep using --resume but instruct the coordinator to ignore old tool outputs
- **C)** Increase the context window so both old and new results coexist
- **D)** Start a fresh session, inject a concise summary of the prior report's conclusions, and re-run the searches

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

When prior tool results are stale, the right move is to start fresh with an injected summary of the earlier conclusions and re-run the searches, so the report is grounded in current data rather than replayed cached results.

- **Why A is wrong:** Forking (A) copies the stale tool outputs into the new branch, preserving the problem.
- **Why B is wrong:** Prompt-based suppression under --resume (B) is unreliable and the outdated results still occupy context.
- **Why C is wrong:** A larger window (C) keeps the stale data resident, compounding the confusion.

</details>

---

#### B16 · Developer Productivity with Claude

*TS 1.7 · Session state, resumption & forking*

An engineer runs `--resume` on a named exploration session from three days ago to continue a refactor. Since then a teammate moved several files and renamed a package. The resumed agent replays cached Read and Grep results from the old history and proposes edits against paths that no longer exist. What is the best way to continue the work?

- **A)** Use `--resume` but manually enlarge the context window so more of the stale history is retained
- **B)** `fork_session` from the three-day-old session to branch the work onto a new line
- **C)** Re-run the same session with `/compact` to shrink the stale tool outputs before editing
- **D)** Start a fresh session and inject a short summary of the prior conclusions, discarding the stale tool results

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

When prior tool results are stale, replaying them drives edits against a state of the world that no longer holds; the fix is to keep the conclusions but re-ground on the current tree — start fresh with an injected summary, so D is correct.

- **Why A is wrong:** A is wrong because retaining more stale history is the problem, not a lack of window.
- **Why B is wrong:** B still carries the stale Read/Grep results into the fork.
- **Why C is wrong:** C compacts (summarizes) the stale outputs rather than replacing them with fresh reads of the moved files, so the wrong paths persist.

</details>

---

#### B17 · Customer Support Resolution Agent

*TS 2.1 · Tool descriptions & boundaries*

process_refund handles both full order refunds and small partial goodwill credits. Logs show the agent issues full refunds when only a partial credit was warranted in 17% of cases because it cannot tell from the single tool which behavior it is invoking. What is the best fix?

- **A)** Raise the agent's context window so the entire refund policy document fits alongside the conversation.
- **B)** Split process_refund into two tools, process_full_refund and process_partial_credit, each with a distinct description and explicit boundary on when to use it.
- **C)** Add three few-shot examples of partial-credit conversations to the system prompt while keeping the single tool.
- **D)** Set tool_choice to "any" so the model is compelled to select a tool on every refund turn.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

When one tool conflates two distinct behaviors, the reliable fix is to split it into separately named tools whose descriptions encode clear boundaries, making the choice explicit at selection time.

- **Why A is wrong:** Option A adds context but leaves the overloaded tool ambiguous, so the same misclassification persists.
- **Why C is wrong:** Option C helps at the margins but few-shot prompt examples cannot substitute for a tool interface that still exposes one undifferentiated action.
- **Why D is wrong:** Option D only guarantees that some tool is called, not that the correct refund behavior is chosen, and does nothing about the underlying overloading.

</details>

---

#### B18 · Multi-Agent Research System

*TS 2.1 · Tool descriptions & boundaries*

The web-search subagent is equipped with two tools, search_web and search_academic, each carrying a one-line description ("search the web" / "search academic sources"). It routes queries to the wrong tool in 31% of cases — for example, sending peer-reviewed-literature queries to search_web. What is the best first step?

- **A)** Consolidate the two tools into a single unified search tool
- **B)** Add a separate router model that classifies each query and selects the tool programmatically
- **C)** Set a forced tool_choice so the subagent always uses one designated search tool
- **D)** Expand and differentiate both tool descriptions — specifying input formats, example queries, source coverage, and explicit boundaries for when to use each

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Tool descriptions are the primary tool-selection mechanism, so the first move is to enrich them: state input formats, give example queries, spell out source coverage, and draw explicit boundaries between the two.

- **Why A is wrong:** Consolidating (A) destroys the useful distinction the two tools provide — here differentiating is the fix, not merging.
- **Why B is wrong:** A dedicated router model (B) over-engineers a problem that a description edit resolves.
- **Why C is wrong:** A static forced tool_choice (C) cannot work when the correct tool legitimately varies per query.

</details>

---

#### B19 · Customer Support Resolution Agent

*TS 2.2 · Structured error responses*

When the customer database is briefly unreachable, get_customer returns a plain empty result, and the agent tells the customer "I can't find any account with that information" and closes the case. Real accounts are being turned away during transient outages. What should the MCP tool return instead?

- **A)** Automatically retry get_customer in a tight loop until the database responds, however long that takes.
- **B)** Increase the get_customer connection timeout so slow database responses are no longer cut off.
- **C)** A structured error with an errorCategory (e.g. transient) and isRetryable metadata that lets the agent distinguish an access failure from a valid empty result and respond accordingly.
- **D)** Treat every non-successful get_customer response as an escalation trigger and route all of them to escalate_to_human.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

The root cause is that an access failure is being conveyed as a valid empty result, so the agent draws the wrong conclusion; the tool must emit structured error metadata (errorCategory, isRetryable) that separates 'lookup failed' from 'no such customer.'

- **Why A is wrong:** Option A retries blindly with no bound and still lacks the semantics to tell the two conditions apart.
- **Why B is wrong:** Option B may reduce some timeouts but does not fix the core ambiguity when a failure does occur.
- **Why D is wrong:** Option D over-escalates by treating validation and business outcomes identically to infrastructure faults, defeating the resolution target and burying real transient errors.

</details>

---

#### B20 · Customer Support Resolution Agent

*TS 2.2 · Structured error responses*

For refunds above the agent's \$500 authority, process_refund correctly declines, but it returns a generic isError response. Logs show the agent then retries the same \$640 refund three times before eventually giving a confusing reply. How should the tool signal this condition?

- **A)** Have the agent retry with exponential backoff, since repeated attempts eventually succeed for most declined refunds.
- **B)** Raise the agent's refund authority limit above \$640 so the call no longer fails.
- **C)** Instruct the agent in the system prompt to never retry a refund more than once.
- **D)** Return a structured error with errorCategory "business" and isRetryable set to false, so the agent stops retrying and routes the request to escalate_to_human.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

A policy-limit rejection is a business error that is deterministically non-retryable; encoding errorCategory "business" with isRetryable false tells the agent that retrying is pointless and that escalation is the correct next step.

- **Why A is wrong:** Option A applies retry logic meant for transient faults to a permanent business rule, wasting calls and never succeeding.
- **Why B is wrong:** Option B changes policy to dodge the control rather than handling the boundary the business intends to enforce.
- **Why C is wrong:** Option C is a prompt-only patch that leaves the error semantically indistinguishable from a retryable one, so behavior stays fragile compared with programmatic metadata the agent can act on reliably.

</details>

---

#### B21 · Multi-Agent Research System

*TS 2.3 · Tool distribution & tool_choice*

The synthesis subagent is initialized with the full registry of 18 tools shared across the system (web_search, fetch_url, parse_pdf, extract_tables, generate_chart, render_report, and others). Traces show that in 22% of runs it issues a fresh web_search or parse_pdf call instead of combining the findings the coordinator already passed in its prompt, adding 6-9 s of latency per run. What is the most effective fix?

- **A)** Restrict the synthesis subagent to the small set of tools its role actually needs, removing the search and parsing tools from its configuration
- **B)** Increase the synthesis subagent's context window so all prior findings fit and it stops re-fetching them
- **C)** Add a system-prompt line instructing the synthesis subagent to never call web_search or parse_pdf
- **D)** Force tool_choice to "any" so the model commits to exactly one deliberate tool call per turn

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Over-provisioning tools degrades selection: a role with 18 tools available will occasionally pick search or parse tools it should not touch. Scoping the synthesis subagent to only the tools its role needs removes the wrong options entirely and is the governing fix.

- **Why B is wrong:** A larger context window (B) is irrelevant — the findings already fit; the model is choosing to re-fetch, not overflowing.
- **Why C is wrong:** A prompt prohibition (C) is non-deterministic (the 22% proves prompt-only guardrails leak).
- **Why D is wrong:** Forcing tool_choice "any" (D) makes it worse by mandating a tool call every turn when synthesis often needs none.

</details>

---

#### B22 · Multi-Agent Research System

*TS 2.3 · Tool distribution & tool_choice*

The document-analysis subagent handles mixed inputs: some are PDFs requiring parse_pdf, and some are already plain-text snippets that need no tool at all. It is currently configured with tool_choice "any". Logs show it calling parse_pdf on plain-text inputs — returning garbage — in 18% of cases. What is the best fix?

- **A)** Force tool_choice to the parse_pdf tool so parsing behavior is deterministic
- **B)** Keep "any" but add a preprocessing stage that strips plain-text inputs out before dispatch
- **C)** Set tool_choice to "auto" so the model can respond directly when no tool is required
- **D)** Add more few-shot examples of plain-text inputs to the subagent's prompt

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

tool_choice "any" forces at least one tool call on every input, so plain-text snippets get needlessly routed through parse_pdf. "auto" lets the model choose to use no tool when none applies, which is exactly the flexibility this mixed-input role needs.

- **Why A is wrong:** Forcing parse_pdf (A) makes the wrong behavior mandatory.
- **Why B is wrong:** A preprocessing stage (B) addresses the symptom at the wrong layer and adds a component when a one-line config change suffices.
- **Why D is wrong:** Few-shot examples (D) cannot override the hard constraint that "any" mandates a tool call regardless.

</details>

---

#### B23 · Developer Productivity with Claude

*TS 2.4 · MCP server integration*

A security scan flags that the team's committed .mcp.json contains a hardcoded GitHub personal-access token used by an MCP server. The team wants the server config to stay shared and version-controlled, but each engineer (and CI) must supply their own credential without leaking it. What should they do?

- **A)** Encrypt the .mcp.json file and commit the encrypted blob, decrypting it at session start.
- **B)** Reference the credential as \${GITHUB_TOKEN} in .mcp.json so each environment supplies its own value from the environment at load time.
- **C)** Move the server to user-scoped \~/.claude.json so the token lives on a single trusted machine.
- **D)** Store the token in the project CLAUDE.md so it loads with memory and is easy to rotate.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

MCP configuration supports \${ENV_VAR} expansion precisely so secrets stay out of version control while the config itself remains shared: writing \${GITHUB_TOKEN} lets each machine and CI job inject its own value at load time (B).

- **Why A is wrong:** Committing an encrypted blob (A) still ships a secret in the repo and adds key-management burden.
- **Why C is wrong:** Moving to user scope (C) sacrifices the requirement that the config stay shared and version-controlled.
- **Why D is wrong:** Putting the token in CLAUDE.md (D) commits the plaintext secret into VCS — the same leak, in a different file.

</details>

---

#### B24 · Developer Productivity with Claude

*TS 2.4 · MCP server integration*

The agent connects to a database MCP server to help engineers query a legacy schema. Traces show that before every real query it fires 6-10 exploratory tool calls (list_tables, describe_table, sample_rows) to rediscover what exists, adding seconds of latency and thousands of tokens per session. The schema changes rarely. What is the best way to reduce this overhead?

- **A)** Force tool_choice to "any" so the model commits to a productive tool sooner.
- **B)** Raise the context window so more exploratory results fit without truncation.
- **C)** Add a short-lived cache in front of the query tool so repeated probes return faster.
- **D)** Expose the schema catalog as an MCP resource the agent reads once upfront, instead of probing with repeated exploratory tool calls.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

MCP resources exist to expose content catalogs (schemas, indexes, file listings) that the agent can read directly, eliminating the exploratory tool calls used just to discover what is available (D). Because the schema is stable, a resource is read once and reused.

- **Why A is wrong:** tool_choice "any" (A) forces a tool but does nothing to remove the discovery calls.
- **Why B is wrong:** A larger window (B) accommodates the bloat rather than preventing it.
- **Why C is wrong:** A cache (C) speeds up the redundant probes but still incurs the round trips and token cost — the discovery pattern itself is the problem.

</details>

---

#### B25 · Developer Productivity with Claude

*TS 2.4 · MCP server integration*

The team added a code-intelligence MCP server that does semantic, cross-file symbol search over an indexed monorepo. In practice the agent almost always reaches for the built-in Grep tool instead, so it misses semantically related call sites that don't share literal text. The MCP tool's description currently reads only "Search the code index." What is the most effective first step?

- **A)** Enrich the MCP tool's description with example queries and a note that it performs semantic, cross-file search that literal text matching misses, so the agent prefers it over Grep.
- **B)** Remove Grep from the agent's allowedTools so the MCP search tool is the only option.
- **C)** Add a CLAUDE.md rule instructing the agent to never use Grep for code search.
- **D)** Force tool_choice to the MCP search tool on every turn.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Tool descriptions are the primary tool-selection mechanism, so a thin description ("Search the code index") loses to a well-understood built-in like Grep; enriching it with example queries and an explicit boundary (semantic vs literal matching) makes the agent prefer it where appropriate (A).

- **Why B is wrong:** Removing Grep entirely (B) is over-scoped — literal search is still the right tool for many tasks.
- **Why C is wrong:** A CLAUDE.md prohibition (C) fights good behavior with a blanket rule instead of clarifying when each tool wins.
- **Why D is wrong:** Forcing tool_choice every turn (D) breaks legitimate Grep and Read use and does not teach correct selection.

</details>

---

#### B26 · Code Generation with Claude Code

*TS 3.1 · CLAUDE.md hierarchy & modularity*

Your team standardized naming and error-handling conventions by adding them to \~/.claude/CLAUDE.md on the tech lead's machine. Everything works for the lead, but three newly onboarded engineers who cloned the repo report that Claude Code ignores the conventions entirely, even though `git status` shows a clean checkout. What is the most effective fix?

- **A)** Add an \@import to the repo-root CLAUDE.md pointing at \~/.claude/CLAUDE.md so the personal file is pulled in for everyone
- **B)** Have each engineer run /memory after cloning to force-reload the user-scoped memory files
- **C)** Move the conventions into the project-scoped .claude/CLAUDE.md (or repo root) so they are committed to version control and load for every clone
- **D)** Convert the conventions into a skill under .claude/skills/ so they activate on demand for all users

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

The CLAUDE.md hierarchy loads user-scoped instructions from \~/.claude/CLAUDE.md, which is personal to a machine and is NOT shared through version control — the classic 'new teammate didn't get the instructions' symptom. Committing the conventions to the project-scoped .claude/CLAUDE.md (or repo root) makes them load for every clone, so C is correct.

- **Why A is wrong:** An \@import to \~/.claude/CLAUDE.md (A) fails because that file doesn't exist on the teammates' machines.
- **Why B is wrong:** Running /memory (B) only inspects already-loaded memory files; it cannot pull in instructions that were never in a shared scope.
- **Why D is wrong:** A skill (D) is on-demand rather than always-loaded, so it wouldn't provide the persistent conventions the team wants.

</details>

---

#### B27 · Developer Productivity with Claude

*TS 3.1 · CLAUDE.md hierarchy & modularity*

A senior engineer added the team's coding-convention instructions to `~/.claude/CLAUDE.md`, and the tool follows them perfectly on her machine. A newly onboarded teammate clones the same repository, but the tool ignores those conventions for him. What is the correct fix?

- **A)** Move the conventions into the project's `.claude/CLAUDE.md` (or root CLAUDE.md) checked into version control, since user-level memory is personal and not shared via VCS
- **B)** Add the conventions to the teammate's `~/.claude.json` MCP configuration so they load for him
- **C)** Convert the conventions into a Skill so they load on demand whenever they are relevant
- **D)** Register the conventions in a `.claude/config.json` commands array so they sync with the repository

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

`~/.claude/CLAUDE.md` is user-scoped memory that never travels with the repository, which is exactly why a fresh clone does not pick it up; project-level CLAUDE.md is shared through VCS, so A is correct.

- **Why B is wrong:** B misuses MCP config, which defines tool servers rather than conventions and is itself personal.
- **Why C is wrong:** C makes the always-on conventions on-demand and still requires committing something to be shared.
- **Why D is wrong:** D references a `.claude/config.json` commands array that does not exist.

</details>

---

#### B28 · Code Generation with Claude Code

*TS 3.2 · Slash commands & skills*

Your team's /investigate-flaky skill greps the entire test suite, reads dozens of files, and prints verbose exploration before it answers. Developers notice that after running it, the main session is cluttered with raw output and Claude's subsequent answers in that session degrade in quality. What is the best fix?

- **A)** Add a step to CLAUDE.md instructing Claude to summarize the skill's output before continuing
- **B)** Set context:fork in the skill's SKILL.md frontmatter so its verbose exploration runs in an isolated context and only the result returns to the main session
- **C)** Increase the context window so the accumulated exploration output no longer causes degradation
- **D)** Restrict the skill with an allowed-tools list so it reads fewer files during exploration

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Skills support context:fork in their SKILL.md frontmatter precisely to isolate verbose, exploratory work in a separate context and return only the distilled result, keeping the main session clean — so B is correct.

- **Why A is wrong:** A CLAUDE.md instruction to summarize (A) is a prompt-only measure that runs after the pollution already occurred and is unreliable.
- **Why C is wrong:** A larger context window (C) does not fix attention dilution from irrelevant accumulated output; it just delays it.
- **Why D is wrong:** An allowed-tools list (D) restricts which tools the skill may use but does not stop verbose output from landing in the main context, and it would also cripple the investigation the skill exists to perform.

</details>

---

#### B29 · Code Generation with Claude Code

*TS 3.2 · Slash commands & skills*

A shared `generate-docs` skill is supposed to read source and write Markdown into `docs/`, but twice it has run `Edit` and shell `rm` and overwritten source files. Adding the reminder "only write to docs/, never edit source" to the skill body has not reliably stopped it. What is the best way to guarantee the skill cannot touch source?

- **A)** Add a stronger system-prompt instruction forbidding edits to any file outside `docs/`
- **B)** Require the skill to run in plan mode every time so a human approves each action
- **C)** Declare `allowed-tools` in the skill's SKILL.md frontmatter, restricting it to Read/Glob/Write so the harness blocks Edit and Bash entirely
- **D)** Add a global PreToolUse hook that inspects every Edit across all sessions and rejects ones outside `docs/`

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

When deterministic tool restriction is required, prompt text is insufficient because the model can still choose a forbidden tool; the `allowed-tools` frontmatter enforces the restriction at the harness level, scoped exactly to this skill.

- **Why A is wrong:** Stronger prose (option A) is still an instruction the model may ignore, which is the observed failure.
- **Why B is wrong:** Plan mode (option B) adds friction to every run and relies on the human catching the bad action rather than preventing it.
- **Why D is wrong:** A global PreToolUse hook (option D) is the right idea at the wrong scope — it would police every session's edits when the problem is confined to one skill, and `allowed-tools` is the purpose-built, skill-scoped mechanism.

</details>

---

#### B30 · Code Generation with Claude Code

*TS 3.2 · Slash commands & skills*

A shared `/scaffold-component` skill expects a component-name argument, but telemetry shows teammates invoke it with no argument or a malformed one about a third of the time, producing empty or misnamed scaffolds. What is the most effective way to reduce these malformed invocations?

- **A)** Add an `argument-hint` field to the skill's SKILL.md frontmatter so the expected argument and format are shown at the invocation prompt
- **B)** Document the correct usage in the project CLAUDE.md so it is always loaded into context
- **C)** Rename the skill to `/scaffold-component-with-name` so the required input is implied by the name
- **D)** Have the skill body instruct Claude to ask the user for the name whenever it is missing

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

`argument-hint` is the frontmatter field designed precisely to surface the expected argument and format at the moment of invocation, guiding the caller before a malformed run happens.

- **Why B is wrong:** Documenting usage in CLAUDE.md (option B) helps the model but does not present the hint to the human at invocation time, where the mistake occurs.
- **Why C is wrong:** Renaming (option C) is a cosmetic nudge that neither shows the format nor prevents empty calls.
- **Why D is wrong:** A body instruction to ask when missing (option D) is a runtime fallback that adds a round-trip and still does not communicate the expected format up front — the direct, purpose-built fix is `argument-hint`.

</details>

---

#### B31 · Claude Code for Continuous Integration

*TS 3.2 · Slash commands & skills*

The project's always-loaded CLAUDE.md has grown to embed a long, detailed code-review rubric. That rubric is needed only by the review job, but it is now injected into every CI job — test generation, doc updates — inflating token cost and diluting the model's focus on those tasks. What is the best fix?

- **A)** Delete the rubric from CLAUDE.md and inline its full text into each review command invocation
- **B)** Split CLAUDE.md into two files and symlink the right one into place per job
- **C)** Keep the rubric in CLAUDE.md but wrap it in comment markers so it is visually separated
- **D)** Move the review rubric into an on-demand Skill under .claude/skills so only the review job loads it, keeping CLAUDE.md lean for always-on context

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

CLAUDE.md is always loaded, so anything job-specific placed there taxes every unrelated run. A Skill loads on demand only when its job invokes it, so moving the rubric into .claude/skills keeps it available to the review job while leaving CLAUDE.md lean for the rest — key D is correct.

- **Why A is wrong:** A loses versioned reuse and bloats the command definition.
- **Why B is wrong:** B is a fragile symlink hack that duplicates and desynchronizes the guidance.
- **Why C is wrong:** C changes formatting but the commented text is still loaded into context for every job, so the cost and dilution remain.

</details>

---

#### B32 · Developer Productivity with Claude

*TS 3.4 · Plan mode vs direct execution*

An engineer asks the tool to determine which of 30 services emits a particular Kafka event. Answering requires grepping and reading across dozens of files, producing thousands of lines of intermediate tool output. Done inline, this verbose discovery floods the main session's context and later steps degrade from lost-in-the-middle effects. What is the best approach?

- **A)** Increase the main session's context window so all of the intermediate tool output fits
- **B)** Delegate the discovery to an Explore subagent that returns only the concise finding, keeping the verbose output out of the main context
- **C)** Turn on plan mode so the tool plans the searches before it runs them
- **D)** Disable Grep and instead Read each candidate file directly in the main session

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Verbose, exploratory discovery belongs in an isolated Explore subagent that reports back only the concise answer, so the main context stays clean and free of attention dilution — making B correct.

- **Why A is wrong:** A is wrong because a larger window still accumulates and dilutes attention rather than preventing the flood.
- **Why C is wrong:** C plans the searches but does not reduce the volume of intermediate output they generate.
- **Why D is wrong:** D is slower and still pours every file's contents into the main context, worsening the problem.

</details>

---

#### B33 · Code Generation with Claude Code

*TS 3.4 · Plan mode vs direct execution*

To rename a widely-used internal API, Claude must first locate every call site across a large monorepo. Doing this discovery inline reads dozens of files and floods the main session with search output before any edits begin, and the later edits then start to wander. What is the best approach?

- **A)** Delegate the discovery to an Explore subagent that searches the monorepo and returns only a summary of call sites, keeping the main context focused on the edits
- **B)** Read every candidate file into the main session first so all call sites are visible before editing
- **C)** Run `/compact` after each search so the accumulated output is repeatedly condensed
- **D)** Move to a larger-context model so the exploration output no longer competes with the edits

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Verbose discovery over a large codebase should be delegated to a subagent so the exploration happens in an isolated context and only a distilled summary of call sites returns — the main session stays focused on performing the rename.

- **Why B is wrong:** Reading every file into the main session (option B) is exactly the flooding behavior causing the edits to wander.
- **Why C is wrong:** Repeated `/compact` (option C) is a lossy patch applied after the noise already accumulated and interrupts the flow.
- **Why D is wrong:** A bigger window (option D) still suffers attention dilution from the raw search output; isolating the discovery is what preserves the working context.

</details>

---

#### B34 · Code Generation with Claude Code

*TS 3.5 · Iterative refinement*

A developer needs three changes that all touch the shared `PaymentStatus` enum and the several functions that switch on it. They sent each change as a separate sequential message; after the third message the project no longer compiles because the earlier edits did not anticipate the later ones. What is the best way to structure the request?

- **A)** Run each of the three changes in its own fresh session so they stay cleanly isolated
- **B)** Enable plan mode and approve the three changes one at a time in sequence
- **C)** Keep them sequential but ask Claude to re-read the enum before each message
- **D)** Batch the three interacting changes into a single message so Claude edits the enum and all dependent switch sites together with full mutual context

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Fixes that interact — here all three touch one enum and its dependent switches — should be batched into a single message so the model reasons about them jointly and keeps the code internally consistent; only independent changes benefit from being sequenced separately.

- **Why A is wrong:** Separate fresh sessions (option A) make the interaction worse by discarding shared context each time.
- **Why B is wrong:** Plan mode approvals (option B) still process the edits one at a time, reproducing the same ordering problem.
- **Why C is wrong:** Re-reading the enum each message (option C) treats a coordination problem as a memory problem; the real fix is to let the model see all three interacting edits at once.

</details>

---

#### B35 · Code Generation with Claude Code

*TS 3.5 · Iterative refinement*

Asking Claude to "make the input validation more robust and idiomatic" yields inconsistent output across runs: sometimes it adds zod, sometimes hand-written checks, sometimes nothing meaningful. The team wants the validation style to come out consistent. What is the most effective refinement?

- **A)** Expand the request into a longer, more detailed prose description of what "robust" and "idiomatic" mean
- **B)** Provide 2-3 concrete input-to-output examples showing the exact validation style and library you want
- **C)** Fine-tune a model on the team's existing validation code so it internalizes the house style
- **D)** Add a standing CLAUDE.md rule stating that validation should always be robust and idiomatic

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

For ambiguous, style-sensitive tasks, concrete input/output examples communicate the target far more reliably than prose and generalize across cases — a few examples pin down the exact pattern the vague adjectives left open.

- **Why A is wrong:** More detailed prose (option A) still leaves "robust" and "idiomatic" open to interpretation, which is the source of the variance.
- **Why C is wrong:** Fine-tuning (option C) is heavy over-engineering for what a few examples in the prompt solve.
- **Why D is wrong:** A CLAUDE.md rule (option D) restates the same vague adjectives as an always-loaded instruction without demonstrating the concrete format, so the inconsistency persists.

</details>

---

#### B36 · Claude Code for Continuous Integration

*TS 3.6 · Claude Code in CI/CD*

A CI pipeline calls Claude Code to review each pull request, and a shell gate script greps stdout for the word "CRITICAL" to fail the build. The gate is flaky because Claude phrases severe findings differently each run — "severe issue," "major bug," "critical defect." What is the most effective fix?

- **A)** Add a post-processing library with a synonym dictionary that normalizes severity wording before the grep runs
- **B)** Invoke Claude with --output-format json plus a --json-schema so findings are emitted as machine-parseable structured fields the gate reads deterministically
- **C)** Instruct the prompt to "always use the exact word CRITICAL for severe issues" and rely on that wording
- **D)** Set temperature to 0 so the phrasing of findings stays stable between runs

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

CI consumers must read structured, machine-parseable output rather than parsing free-form natural language. Running with --output-format json and a --json-schema forces findings into typed fields (severity, file, line) the gate can evaluate deterministically, so key B is correct.

- **Why A is wrong:** A builds a brittle synonym-normalization layer that will always lag new phrasings and is unneeded once output is structured.
- **Why C is wrong:** C is a prompt-only instruction that offers no deterministic guarantee — the model can still deviate.
- **Why D is wrong:** D misunderstands temperature: even at 0 the output is prose, not a stable schema, and the gate remains fragile.

</details>

---

#### B37 · Claude Code for Continuous Integration

*TS 3.6 · Claude Code in CI/CD*

In the pipeline, the same Claude Code session that generates an implementation also performs the automated review step. It consistently rates its own output clean, yet human reviewers keep finding real bugs it missed. What is the most effective change?

- **A)** Run the review in a separate, independent Claude Code instance that shares no context with the generating session
- **B)** Enable extended thinking on the generating session before it reviews its own code
- **C)** Instruct the generating session to "review critically and assume the code contains bugs"
- **D)** Lower the pass threshold so more borderline findings surface in the same session

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

A session that produced code carries the same assumptions that caused its defects, so self-review systematically misses them. An independent instance with no shared context reviews the diff fresh and catches issues the author blind spots hid, so key A is correct.

- **Why B is wrong:** B adds reasoning depth but keeps the identical context and biases, so the blind spots persist.
- **Why C is wrong:** C is a prompt nudge that cannot overcome the structural problem of reviewing one's own reasoning.
- **Why D is wrong:** D changes sensitivity, not perspective — the same session still fails to see the bugs it never flagged.

</details>

---

#### B38 · Claude Code for Continuous Integration

*TS 3.6 · Claude Code in CI/CD*

Locally, the author's Claude generates tests that follow team conventions (naming, arrange-act-assert structure). In CI on a clean checkout, the generated tests ignore those conventions. The conventions are documented only in the author's \~/.claude/CLAUDE.md. What is the best fix?

- **A)** Pass the conventions as a --system-prompt argument in the CI command on every run
- **B)** Add the conventions to the CI build server's \~/.claude/CLAUDE.md so the runner picks them up
- **C)** Reproduce the author's local machine environment inside the CI job
- **D)** Move the conventions into the project's committed .claude/CLAUDE.md so every checkout — local and CI — loads them from version control

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

User-level \~/.claude/CLAUDE.md is personal and not shared through VCS, which is exactly why it is absent on a clean CI checkout. Moving the conventions into the project's committed .claude/CLAUDE.md makes them load for every clone, in CI and for every teammate, so key D is correct.

- **Why A is wrong:** A wires the guidance into a fragile command flag that lives outside the repo and drifts.
- **Why B is wrong:** B stashes instructions on one build server, still unversioned and invisible to teammates.
- **Why C is wrong:** C is disproportionate and does not address the real cause — the instructions simply are not in the shared project scope.

</details>

---

#### B39 · Claude Code for Continuous Integration

*TS 4.1 · Explicit criteria & false positives*

The CI reviewer posts naming-convention and style comments, but developers report that roughly 60% of these particular comments are wrong or trivial. They have started ignoring the bot entirely — including its genuine security findings. What is the most effective response?

- **A)** Add a single global confidence threshold so only comments the model rates above 0.8 are posted
- **B)** Temporarily disable the naming-convention category, then rewrite its prompt with explicit categorical rules before re-enabling it
- **C)** Route every comment through a human moderation queue before it is posted to the pull request
- **D)** Train a sentiment classifier on developer reactions and suppress whichever categories draw negative responses

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

A single high-false-positive category erodes trust in the entire reviewer, so the governing principle is to remove the offending category from production and fix its prompt rather than paper over it. Temporarily disabling naming-convention comments stops the trust bleed immediately, and rewriting the category with explicit categorical criteria addresses the root cause before re-enabling it.

- **Why A is wrong:** A global confidence threshold (A) is the wrong instrument — self-reported confidence is poorly calibrated and would also suppress good findings from healthy categories.
- **Why C is wrong:** A human moderation queue (C) defeats the point of automation and does not fix the noisy prompt.
- **Why D is wrong:** A sentiment classifier (D) is heavy over-engineering that treats symptoms (developer annoyance) rather than the miscalibrated category definition.

</details>

---

#### B40 · Claude Code for Continuous Integration

*TS 4.2 · Few-shot prompting*

The CI reviewer handles clear-cut defects well but is inconsistent on genuinely ambiguous cases — for example, whether a bare `except:` is acceptable in a one-off script versus a shared library. Enumerating every possible scenario in the prompt is impractical. What is the best approach?

- **A)** Add two to four few-shot examples of representative ambiguous cases, each with the reasoning behind its verdict
- **B)** Write an exhaustive list in CLAUDE.md of every code pattern and its correct verdict
- **C)** Fine-tune a dedicated model on the team's full history of past review decisions
- **D)** Lower the bar so every borderline case is flagged and pushed to a human reviewer

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Few-shot examples are the right tool for ambiguous judgment because a small set of well-chosen cases with their reasoning lets the model generalize the underlying principle rather than memorize a lookup table. Two to four exemplars covering the representative ambiguous patterns teaches the script-versus-library distinction without enumerating everything.

- **Why B is wrong:** An exhaustive list (B) is the enumeration approach the stem already rejects as impractical and it still will not cover unseen cases.
- **Why C is wrong:** Fine-tuning (C) is disproportionate engineering for a prompt-level problem and is slow to iterate.
- **Why D is wrong:** Flagging every borderline case to humans (D) abandons the goal of useful automated feedback and floods reviewers with noise.

</details>

---

#### B41 · Structured Data Extraction

*TS 4.3 · Structured output via tool use & schemas*

Production logs show that 8% of extraction responses fail the downstream JSON.parse() step because Claude wraps its output in markdown code fences or prepends a sentence like "Here is the extracted data:". Prompt tweaks reduced but did not eliminate the failures. What is the most reliable fix?

- **A)** Add a stern system-prompt line ("Return only valid JSON, no markdown, no commentary") and set temperature to 0 to make formatting deterministic.
- **B)** Define the target structure as a tool input_schema, force that tool via tool_choice, and read the structured JSON directly from the tool_use block.
- **C)** Post-process every response with a regex that strips markdown fences and leading prose before calling JSON.parse().
- **D)** Lower max_tokens so the model has no room to add explanatory prose around the JSON object.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

The governing principle is that tool_use with a JSON schema is the most reliable way to obtain structured output because the model returns a validated JSON object in the tool_use block, eliminating syntax and formatting errors entirely. Forcing the tool guarantees the structured path is taken.

- **Why A is wrong:** Prompt instructions plus temperature 0 (A) still rely on the model formatting free text and leave residual failures.
- **Why C is wrong:** Regex stripping (C) is a brittle patch that breaks on unexpected wrappers and does not address the root cause.
- **Why D is wrong:** Capping max_tokens (D) can truncate valid JSON and does not stop fence-wrapping.

</details>

---

#### B42 · Structured Data Extraction

*TS 4.3 · Structured output via tool use & schemas*

After migrating to tool_use with a strict JSON schema, JSON parse failures dropped to zero. However, reconciliation shows 6% of extracted invoices have line-item amounts that do not sum to the stated invoice total. What does this reveal about schema-based structured output?

- **A)** The schema is misconfigured; adding "additionalProperties": false will force the line items to reconcile with the total.
- **B)** tool_choice should be "any" rather than forced so the model can self-correct the arithmetic before returning.
- **C)** Schema enforcement removes syntax and type errors but not semantic errors, so a downstream arithmetic validation check must flag or trigger a retry on mismatches.
- **D)** max_tokens is too low, so the model omits line items; raising it will make the totals add up.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

The principle is that a JSON schema guarantees structural and type validity but cannot enforce semantic correctness such as line items summing to a total. The correct response is a downstream business-rule validation step that recomputes the sum and flags or retries mismatches.

- **Why A is wrong:** Option A confuses a structural constraint (additionalProperties) with an arithmetic invariant it cannot express.
- **Why B is wrong:** Option B misuses tool_choice, which controls tool selection, not numeric correctness.
- **Why D is wrong:** Option D misattributes a semantic discrepancy to a token limit; truncation would cause parse or completeness failures, not silent miscalculation.

</details>

---

#### B43 · Structured Data Extraction

*TS 4.3 · Structured output via tool use & schemas*

For documents that lack a purchase-order number, audit logs show Claude inventing plausible-looking PO numbers about 11% of the time instead of signaling absence. Downstream systems then treat the fabricated values as real. What is the best schema-level fix?

- **A)** Add a required confidence float to every field so downstream code can discard PO numbers below a threshold.
- **B)** Switch tool_choice to "auto" so the model can decline to call the extraction tool when the PO number is missing.
- **C)** Add a few-shot example containing a correctly formatted PO number to anchor the expected pattern.
- **D)** Make the po_number field nullable in the schema and instruct the model to return null when the value is not present in the source.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

The principle is that making the field nullable gives the model a sanctioned way to represent absence, so it returns null instead of fabricating a value to satisfy a required field. The key can stay required while its value may be null; an optional field would also stop the fabrication, but the key would then be omitted rather than set to null. Pairing this with an explicit instruction to emit null when the data is absent directly addresses the hallucination.

- **Why A is wrong:** A confidence score (A) is poorly calibrated and does not stop the fabrication, only tries to filter it afterward.
- **Why B is wrong:** Setting tool_choice to "auto" (B) risks skipping extraction entirely for otherwise-complete documents.
- **Why C is wrong:** A few-shot example of a valid PO (C) reinforces the format but gives no path for the missing case, which is the actual problem.

</details>

---

#### B44 · Structured Data Extraction

*TS 4.3 · Structured output via tool use & schemas*

The extraction schema uses an enum with six known values for document_type. A new vendor begins sending "credit memo" documents, which are not in the enum, and the model force-fits them to "invoice", corrupting downstream routing. What is the best design change?

- **A)** Add an "other" value to the enum plus an optional document_type_detail string so novel types are captured verbatim instead of misclassified.
- **B)** Remove the enum constraint entirely and let the model emit any free-text string for document_type.
- **C)** Fine-tune a dedicated document-type classifier and run it before extraction to assign the type.
- **D)** Brainstorm and add every conceivable document type to the enum so no future document falls outside it.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

The principle is designing enums for extensibility with an "other" (or "unclear") escape value plus a free-text detail field, so unanticipated categories are surfaced rather than silently forced into a wrong bucket. This keeps the controlled vocabulary intact while capturing novel cases for review.

- **Why B is wrong:** Dropping the enum (B) sacrifices the validation and normalization that make the field useful downstream.
- **Why C is wrong:** A fine-tuned classifier (C) is heavy over-engineering when a schema change solves the problem.
- **Why D is wrong:** Enumerating every possible type (D) is impossible to complete and still breaks on the next truly novel document.

</details>

---

#### B45 · Structured Data Extraction

*TS 4.3 · Structured output via tool use & schemas*

The pipeline handles six document types, each with its own extraction tool. Even when the type is already known from the intake folder that routed the document, the caller still sends all six tools with tool_choice "auto", and in 9% of cases the model invokes the wrong extractor. What is the best fix?

- **A)** Keep tool_choice "auto" but add sentiment analysis of the document to help the model decide which extractor fits.
- **B)** When the document type is already known from routing, force the specific extractor with tool_choice {"type": "tool", "name": "extract_invoice"} rather than leaving selection to "auto".
- **C)** Set tool_choice to "any" so the model is at least required to call some extraction tool.
- **D)** Merge all six extraction tools into one large tool with a combined schema covering every document type.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

The principle is that when the correct tool is already determined by upstream logic, you should force it with tool_choice {"type":"tool","name":...} instead of asking the model to re-decide, which removes the selection error entirely.

- **Why A is wrong:** "auto" (A) reintroduces the very choice that is failing, and sentiment analysis is irrelevant over-engineering.
- **Why C is wrong:** "any" (C) still lets the model pick the wrong extractor among the six.
- **Why D is wrong:** Merging into one mega-tool (D) creates a bloated schema and does not use the routing information you already have, likely degrading extraction quality.

</details>

---

#### B46 · Structured Data Extraction

*TS 4.4 · Validation, retry & feedback loops*

Extractions pass the JSON schema but sometimes violate a business rule (invoice_date later than due_date). The current recovery discards the result and re-runs the identical prompt from scratch; 40% of retries still fail the rule. What is the most effective retry strategy?

- **A)** Re-run the identical prompt up to five times and accept the first result that happens to pass the rule.
- **B)** Append the previous failed extraction and the specific validation error messages to a follow-up message, asking the model to correct only the offending fields.
- **C)** Lower temperature to 0 and re-run the original prompt exactly once.
- **D)** Move all rule violations to the Message Batches API and reprocess them overnight.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

The validation-retry principle is to feed the model its own failed output plus the exact validation errors, so it can correct the specific fields rather than re-deriving everything blindly. This targeted feedback is far more likely to resolve a rule violation than repetition.

- **Why A and C are wrong:** Re-running the same prompt (A, C) provides no new information and mostly reproduces the same error.
- **Why D is wrong:** The Batch API (D) is for high-volume latency-tolerant work, not a corrective loop, and it does nothing to give the model the failure context it needs to fix the fields.

</details>

---

#### B47 · Structured Data Extraction

*TS 4.4 · Validation, retry & feedback loops*

For scanned documents where the tax ID is physically cut off at the page edge and simply absent from the source text, the validation-retry loop retries five times and still fails, burning tokens on every attempt. What is the correct understanding of this situation?

- **A)** Raise the retry ceiling to ten; with enough attempts the model will eventually recover the tax ID.
- **B)** Enable extended thinking so the model reasons more deeply and reconstructs the missing tax ID.
- **C)** Retries can fix format and structural errors but cannot recover information absent from the source, so detect the absence, return null, and route the document to human review instead of retrying.
- **D)** Switch tool_choice to "any" so the model can select a different extraction tool that will locate the tax ID.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

The principle is that a validation-retry loop repairs formatting and structural mistakes but cannot conjure information that is not present in the source document. Recognizing absence, returning null, and escalating to a human is the correct terminal behavior, and it stops the wasteful token spend.

- **Why A is wrong:** Increasing retries (A) cannot recover data that was never in the input.
- **Why B is wrong:** Extended thinking (B) improves reasoning over available evidence but cannot invent an absent value without fabricating it.
- **Why D is wrong:** Changing tool_choice (D) alters which tool runs, not what text exists on the truncated page.

</details>

---

#### B48 · Structured Data Extraction

*TS 4.5 · Batch processing*

A 50,000-document extraction batch completes and 300 requests returned errors. The team needs to reprocess only the failures and reliably match every result back to its source document. What is the correct approach?

- **A)** Assign each request a custom_id mapping to its source document, correlate results by custom_id, and submit a new batch containing only the 300 failed custom_ids.
- **B)** Resubmit the entire 50,000-document batch, since batches are atomic and cannot be reprocessed partially.
- **C)** Rely on result ordering, since the Batch API guarantees that output order matches input order.
- **D)** Convert the 300 failures to synchronous calls and use response timestamps to align each result with its source document.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

The principle is that each Batch API request carries a caller-supplied custom_id, which is the correlation key for matching results back to sources and for selectively resubmitting only the failed items. Reprocessing the 300 failed custom_ids in a fresh batch is both cheap and precise.

- **Why B is wrong:** Resubmitting all 50,000 (B) wastes cost and time; batches are not atomic in that sense.
- **Why C is wrong:** Result ordering is not guaranteed (C), which is exactly why custom_id exists.
- **Why D is wrong:** Timestamp matching (D) is fragile and unnecessary when a deterministic custom_id mapping is available.

</details>

---

#### B49 · Claude Code for Continuous Integration

*TS 4.6 · Multi-instance & multi-pass review*

In the pipeline, Claude Code generates test cases and then, within the same session, reviews those tests for quality. The review almost never finds problems, yet audits show 20% of the generated tests have weak or tautological assertions. What explains this and what is the fix?

- **A)** Enable extended thinking in the same session so the self-review reasons more carefully before approving
- **B)** Have the same session review the tests three times and take a majority vote on each
- **C)** Add more detailed quality criteria to the existing session's review prompt
- **D)** A session reviewing its own output is biased toward confirming it, so have a separate, independent Claude instance review the generated tests

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

A session evaluating work it just produced is anchored to its own reasoning and tends to confirm rather than challenge it, which is why the self-review misses weak assertions the audit catches. An independent instance with no stake in the original generation reviews far more critically, so routing the tests to a fresh reviewer is the correct fix.

- **Why A is wrong:** Extended thinking (A) makes the same biased session think longer about the same conclusion; it does not remove the conflict of interest.
- **Why B is wrong:** Voting across three passes of the same session (B) compounds the shared bias instead of countering it.
- **Why C is wrong:** More detailed criteria (C) help at the margin but cannot overcome a reviewer predisposed to approve its own output.

</details>

---

#### B50 · Customer Support Resolution Agent

*TS 5.1 · Context preservation*

In longer billing-dispute conversations, the agent uses progressive summarization to keep the history compact. Testers find that after several turns it quotes the wrong refund amount and an incorrect order date, even though those values appeared earlier. What is the best remedy?

- **A)** Maintain a persistent 'case facts' block (order id, amount, dates, customer expectation) stored outside the summarized history and re-injected verbatim each turn.
- **B)** Switch to a model with a larger context window so no summarization is needed.
- **C)** Call get_customer and lookup_order again on every turn to refetch the numbers.
- **D)** Enable extended thinking so the model reasons more carefully before quoting figures.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Progressive summarization compresses away precise numbers, dates, and stated expectations, so the fix is to keep those exact values in a persistent case-facts block that lives outside the summarized transcript and is re-supplied unchanged.

- **Why B is wrong:** Option B only delays the same information loss once conversations grow, and does not protect the specific fields.
- **Why C is wrong:** Option C is wasteful and can still be re-summarized away, plus it re-incurs latency and error surface each turn.
- **Why D is wrong:** Option D improves reasoning but cannot recover facts that summarization has already discarded from context.

</details>

---

#### B51 · Code Generation with Claude Code

*TS 5.1 · Context preservation*

In a long debugging session Claude repeatedly runs the full test suite; each run dumps roughly 2,000 lines of passing-test noise into the context. Over time Claude starts overlooking the failing-test details and the constraints stated at the start of the session. What is the best fix?

- **A)** Switch to a larger-context model so all the test output fits comfortably
- **B)** Trim each test run's output to the relevant failing-test lines before it accumulates, and keep the key constraints in a summary placed at the top of the context
- **C)** Enable extended thinking so Claude reasons harder over the full accumulated output
- **D)** Instruct Claude to re-read the entire session history at the start of every turn

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

The failure is context degradation from unbounded verbose tool output plus lost-in-the-middle placement of the key constraints; the fix is to trim tool outputs to the relevant fields before they accumulate and to keep the critical summary up front where attention is strongest.

- **Why A is wrong:** A larger window (option A) still lets 2,000 lines per run bury the failing details and dilute attention — capacity is not the problem, signal-to-noise is.
- **Why C is wrong:** Extended thinking (option C) makes the model reason harder over the same noisy context without removing the noise.
- **Why D is wrong:** Re-reading the whole history each turn (option D) amplifies the very clutter that is drowning the failing-test details.

</details>

---

#### B52 · Multi-Agent Research System

*TS 5.1 · Context preservation*

The coordinator passes each subagent's output forward verbatim: the web-search subagent's full scraped page bodies — roughly 38k tokens — flow straight into the synthesis subagent's prompt. Synthesis quality has dropped and it now misses findings buried in the middle of the passed-in content (lost-in-the-middle). What is the best fix?

- **A)** Trim each subagent's output to its relevant extracted findings before passing it downstream, and lead the synthesis prompt with a key-facts summary under section headers
- **B)** Move synthesis to a larger-context model so all 38k tokens fit comfortably
- **C)** Keep passing the raw pages but add a prompt instruction telling synthesis to focus on the important parts
- **D)** Have synthesis re-fetch and re-summarize each page itself before combining them

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Downstream context should be budgeted: trim verbose tool outputs to the relevant extracted fields before they accumulate, and counter lost-in-the-middle by placing the key summary first under clear section headers.

- **Why B is wrong:** A larger-context model (B) does not fix attention dilution — lost-in-the-middle persists at any window size when the prompt is full of low-signal bulk.
- **Why C is wrong:** A "focus on important parts" instruction (C) does not reduce the noise the model must still attend to.
- **Why D is wrong:** Re-fetching (D) duplicates work and re-introduces the same raw volume the trimming was meant to remove.

</details>

---

#### B53 · Customer Support Resolution Agent

*TS 5.2 · Escalation & ambiguity resolution*

A customer gives only the name "Maria Garcia." get_customer returns three distinct matching accounts. Logs show the agent proceeds with the most recently active match, and in several cases it has processed refunds against the wrong customer's order. What is the correct behavior?

- **A)** Continue with the most recently active of the three accounts, since it is the most likely current customer.
- **B)** Pick the match with the highest fuzzy name-similarity score and proceed.
- **C)** Escalate every multiple-match case straight to a human agent.
- **D)** Ask the customer for an additional identifier (order number, email, or ZIP) to disambiguate before taking any account action.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

When multiple customer records match, the agent must not guess; requesting an additional identifier resolves the ambiguity and prevents acting on the wrong account.

- **Why A is wrong:** Option A guesses on recency and is exactly the behavior producing misdirected refunds.
- **Why B is wrong:** Option B guesses on name similarity, which cannot distinguish two people with the same name.
- **Why C is wrong:** Option C needlessly hands off a case the agent can resolve itself with one clarifying question, undermining the first-contact-resolution target when disambiguation is straightforward.

</details>

---

#### B54 · Code Generation with Claude Code

*TS 5.4 · Large-codebase context management*

During a 200-file framework migration carried out in one long session, logs show Claude re-migrating files it already converted and skipping others as the conversation grows. What technique best keeps the migration on track?

- **A)** Switch to a larger-context model so the whole file list stays in the window
- **B)** Run `/compact` frequently to keep the history short
- **C)** Maintain a scratchpad manifest file that records migrated vs pending files and update it after each file, so progress survives context growth
- **D)** Do the entire migration in a single large message to avoid state loss between turns

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

For large, long-running tasks the durable pattern is an external scratchpad or manifest that tracks state on disk, so progress is authoritative regardless of what falls out of the conversation window.

- **Why A is wrong:** A larger window (option A) still degrades over a 200-file span and does not give a reliable record of what is done.
- **Why B is wrong:** `/compact` (option B) shrinks history but can summarize away exactly the per-file progress you need, and it does not create a checkable record.
- **Why D is wrong:** A single massive message (option D) is impractical at this scale and still offers no crash-safe record of completed work — the manifest provides both tracking and recovery.

</details>

---

#### B55 · Code Generation with Claude Code

*TS 5.4 · Large-codebase context management*

Midway through a coherent multi-hour refactor, the session approaches its context limit. The history is full of superseded intermediate diffs, but the current task and goal are unchanged and the latest file states are still valid. What should the developer do to keep going without losing the thread?

- **A)** Start a brand-new session with no prior context and re-describe the task
- **B)** Fork the session to branch off a divergent line of work
- **C)** Request a larger context window and continue as-is
- **D)** Run `/compact` to condense the accumulated history into a summary while continuing the same task

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

When the task is a single coherent thread and only the accumulated intermediate history is bloating the context, `/compact` condenses that history into a summary so work can continue seamlessly on the same task.

- **Why A is wrong:** Starting fresh (option A) is the move when prior tool results are stale or the task changes — neither applies here, and it discards still-valid state.
- **Why B is wrong:** Forking (option B) is for pursuing a divergent branch, not for continuing one coherent line.
- **Why C is wrong:** A larger window (option C) postpones the limit without addressing the superseded-diff clutter; compaction removes the dead weight while preserving the goal.

</details>

---

#### B56 · Multi-Agent Research System

*TS 5.6 · Provenance & multi-source synthesis*

The synthesis subagent merges findings from the web-search and document-analysis subagents into a unified narrative but drops the record of which source each claim came from. The report-generation subagent can then no longer attach citations, and 40% of the report's sentences ship uncited. What is the best fix?

- **A)** Have the report-generation subagent re-run web_search to find a supporting source for each sentence
- **B)** Add a citation post-processor that guesses the most likely source for each uncited sentence
- **C)** Preserve claim-to-source mappings through the synthesis step so each synthesized claim carries its originating source(s) into report generation
- **D)** Instruct synthesis to insert "(source)" placeholders that a human fills in after the report is generated

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Provenance must be preserved through synthesis, not reconstructed afterward: each synthesized claim should carry its originating source(s) forward so report generation can cite the actual origin.

- **Why A is wrong:** Re-running web_search (A) invents new sources that may not be what the claim was actually based on.
- **Why B is wrong:** A guessing post-processor (B) fabricates provenance.
- **Why D is wrong:** Placeholder markers (D) defer the problem to humans and leave the report uncited by default.
- Carrying claim-source mappings end to end is the governing principle.

</details>

---

#### B57 · Multi-Agent Research System

*TS 5.6 · Provenance & multi-source synthesis*

Two sources report different market-size figures for the same market — \$4.2B (2021 data) versus \$6.8B (2023 data). The synthesis subagent silently picks \$6.8B and presents it as the single authoritative fact. Reviewers, seeing the discarded figure elsewhere, flag the report as unreliable. What is the best fix?

- **A)** Always keep the larger or more recent figure and drop the other before synthesis
- **B)** Average the two figures into a single blended number
- **C)** Route every conflicting statistic to a human reviewer before synthesis can continue
- **D)** Annotate both statistics with their source and collection/publication date, so the temporal difference is surfaced rather than misread as a contradiction

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

These figures are not actually contradictory — they are different years — so the fix is to annotate both with their source and date, which requires the pipeline to preserve publication/collection dates precisely so temporal change is not mistaken for conflict.

- **Why A and B are wrong:** Silently keeping one figure, even the more recent one (A), or averaging them (B) destroys provenance and can be flatly wrong.
- **Why C is wrong:** Human routing (C) is unnecessary overhead when the discrepancy is fully explained by date; escalation should be reserved for genuinely ambiguous conflicts, not every differing number.

</details>

---

## Part C — Community bank (457)

Grouped by the official domain of the task statement each question tests, then ordered by task statement.

<a id="part-c-domain-1"></a>

### Part C · Domain 1 · Agentic Architecture & Orchestration (27%)

#### C1

*TS 1.1 · Agentic loops*

**Situation:** You are building an agentic loop in Python using the Anthropic SDK. After calling `client.messages.create(...)`, you receive a response object and need to decide whether to keep looping or stop.

Which field on the API response object should your loop inspect to determine whether the model has finished or is requesting a tool call?

- **A)** response.content[0].type
- **B)** response.model
- **C)** response.usage.output_tokens
- **D)** response.stop_reason

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

The `stop_reason` field signals why the model stopped generating. A value of `"tool_use"` means the model is requesting one or more tool calls and the loop should continue; `"end_turn"` means the model has finished and the loop should exit.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use

**Why the others are wrong:**

- **A:** `response.content[0].type` tells you the type of the first content block (e.g., `text` or `tool_use`), but it is not the canonical termination signal — `stop_reason` is the authoritative field for loop control.
- **B:** `response.model` echoes which model version handled the request and has no bearing on agentic loop termination.
- **C:** `usage.output_tokens` is a billing and telemetry field; it carries no semantic information about whether the model wants to call a tool or has completed its response.

</details>

---

#### C2

*TS 1.1 · Agentic loops*

**Situation:** A developer is reading the Anthropic documentation on tool use. They encounter two `stop_reason` values — `"tool_use"` and `"end_turn"` — and need to understand what each means for their agentic loop.

What does a `stop_reason` of `"end_turn"` indicate in an agentic loop?

- **A)** An error occurred inside the model and the request must be retried.
- **B)** The model is requesting that the orchestrator execute one or more tools.
- **C)** The model ran out of tokens and was cut off before finishing.
- **D)** The model has completed its response and the loop should terminate.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

`"end_turn"` is the normal completion signal: the model has generated its full response and is not requesting any further tool execution. Upon receiving this value, the agentic loop should stop iterating and return the final result to the user.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use

**Why the others are wrong:**

- **A:** API errors surface as HTTP error responses or SDK exceptions, not as a special `stop_reason` value on a successful response object.
- **B:** A tool-call request produces `stop_reason: "tool_use"`. The presence of `tool_use` content blocks and that specific stop reason together signal that the model wants tools executed.
- **C:** A token-limit truncation produces `stop_reason: "max_tokens"`, not `"end_turn"`. Those are distinct values with different handling requirements.

</details>

---

#### C3

*TS 1.1 · Agentic loops*

**Situation:** An engineer is reviewing a colleague's agentic loop implementation. The colleague's loop adds the assistant's response to the message history and then appends a new user message containing the tool results before calling the API again.

Why must the assistant's full response message be added to the conversation history before submitting tool results?

- **A)** Because the API caches assistant messages and will reject duplicate user messages otherwise.
- **B)** So the model can recalculate its token usage from the previous turn.
- **C)** To maintain the required alternating user/assistant message structure that the Messages API enforces.
- **D)** So that the tool result is associated with the correct tool by name rather than by ID.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

The Messages API requires that messages alternate between `user` and `assistant` roles. Including the assistant's full content in the history before appending the new user message with tool results preserves this alternating structure and provides the `tool_use_id` references the model needs to match results to calls.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use

**Why the others are wrong:**

- **A:** The API does not perform cross-turn message deduplication caching of this kind; the structural requirement is about conversation turn ordering.
- **B:** Token usage recalculation is handled internally by the API; it is not a reason to retain the assistant message in history.
- **D:** Tool results are matched to their originating calls via `tool_use_id`, which is a UUID generated by the model — not the tool's name. The assistant message must be retained because it contains those IDs.

</details>

---

#### C4

*TS 1.1 · Agentic loops*

**Situation:** A developer is inspecting a model response and finds a content block with `"type": "tool_use"`. They need to extract the information required to execute the tool and return results.

Which fields inside a `tool_use` content block must the orchestrator read to execute the tool correctly?

- **A)** `name` and `stop_sequence`
- **B)** `type` and `text`
- **C)** `id` and `stop_reason`
- **D)** `id`, `name`, and `input`

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

A `tool_use` block contains `id` (a unique identifier used to correlate the result back to this specific call), `name` (the tool to invoke), and `input` (a JSON object of arguments). All three are required: `name` and `input` to execute the tool, and `id` to construct the matching `tool_result` block.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use

**Why the others are wrong:**

- **A:** `stop_sequence` is a top-level response field that indicates if a custom stop sequence was triggered; it is not part of a `tool_use` content block.
- **B:** `tool_use` blocks do not have a `text` field — that belongs to `text` content blocks. The execution fields are `id`, `name`, and `input`.
- **C:** `stop_reason` is a top-level response field, not part of an individual content block. The `id` alone is insufficient to execute a tool without `name` and `input`.

</details>

---

#### C5

*TS 1.1 · Agentic loops*

**Situation:** You are auditing an agentic loop that processes financial transactions. The loop has been running in production for two weeks and engineers notice it occasionally cycles for 30+ iterations before terminating, consuming thousands of tokens on tasks that should complete in 3–5 iterations. The logs show `stop_reason: "tool_use"` on every iteration of the runaway sessions.

Which combination of root causes and mitigations best addresses this runaway loop problem?

- **A)** Root cause: Parallel tool execution is causing race conditions that produce stale results. Mitigation: Switch all tool calls to sequential execution to eliminate concurrency.
- **B)** Root cause: Tool results are likely returning ambiguous or error states that the model tries to resolve in a loop without ever converging. Mitigation: Add structured error responses with `"is_error": true`, implement an iteration cap, and add observability to log the content of each tool result.
- **C)** Root cause: The model is ignoring `end_turn` signals. Mitigation: Switch to a newer model version that has better instruction-following.
- **D)** Root cause: The agentic loop is not including the assistant message in conversation history, causing the model to re-request the same tools. Mitigation: Fix message history construction to always include the previous assistant turn.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Runaway loops where `stop_reason` stays `"tool_use"` for many iterations typically indicate the model is caught in a resolution cycle — it receives an ambiguous or error-bearing tool result, attempts to recover, and the next result is similarly unresolved. The correct mitigations are: returning structured errors with `is_error: true` so the model can reason about failure rather than retry blindly; adding a hard iteration cap as a safety backstop; and adding observability to diagnose the specific cycle pattern. These address both the symptom and the root cause.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use
- https://docs.anthropic.com/en/docs/build-with-claude/agents/build-effective-agents

**Why the others are wrong:**

- **A:** Race conditions in parallel tool execution would produce inconsistent individual results but would not cause the loop itself to cycle indefinitely; the termination condition depends on `stop_reason`, not on execution ordering.
- **C:** Runaway loops are almost never caused by the model 'ignoring' `end_turn` signals — `end_turn` is issued by the model itself when it is satisfied. Switching model versions does not fix the underlying protocol or result-quality issue.
- **D:** Missing assistant messages in history would cause an API validation error on the next call, not a silent runaway loop. The symptom described — 30+ successful iterations — rules out a history-construction bug.

</details>

---

#### C6

*TS 1.1 · Agentic loops*

**Situation:** A team is comparing two agentic loop designs. Design A executes all tool calls in the order the model lists them and waits for each to complete before starting the next. Design B detects which tool calls are independent (no data dependencies between them) and fans them out in parallel, sequencing only the ones with dependencies. The task involves retrieving data from three external APIs and then synthesizing the results.

What is the primary trade-off when choosing between Design A and Design B for this task?

- **A)** Design B is always superior because parallel execution is faster; Design A should never be used in production.
- **B)** Design A produces higher-quality model outputs because the model receives each tool result before the next call is made, enabling incremental reasoning.
- **C)** Design B reduces wall-clock latency for independent calls but requires the orchestrator to implement dependency analysis and manage concurrent execution, adding implementation complexity and the risk of incorrect dependency classification.
- **D)** Design A is safer because sequential execution avoids race conditions; Design B introduces correctness risks that outweigh the latency benefit.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

The core trade-off is latency versus implementation complexity. Design B is faster for independent calls but shifts the dependency-analysis responsibility onto the orchestrator — an error in classifying a dependent call as independent produces incorrect results. Design A is simpler and always correct, but pays the latency cost of serializing calls that could safely run in parallel. The right choice depends on whether the latency gain justifies the additional orchestration complexity and the risk surface of misclassification.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use
- https://docs.anthropic.com/en/docs/build-with-claude/agents/build-effective-agents

**Why the others are wrong:**

- **A:** Design B is not universally superior — it is faster for independent calls but adds complexity and risk. For tasks with true sequential dependencies, Design B provides no latency benefit and adds overhead.
- **B:** All tool results for a turn are returned to the model in a single user message regardless of execution strategy; the model does not receive incremental results between parallel calls. Design A does not enable 'incremental reasoning' within a turn.
- **D:** Design B does not inherently introduce correctness risks if dependency analysis is done correctly; the risk is bounded to misclassification. For genuinely independent API calls, parallel execution is both safe and faster.

</details>

---

#### C7

*TS 1.1 · Agentic loops*

**Situation:** An engineer implements an agentic loop that executes tool calls and feeds results back. During a long-running task the model returns `stop_reason: "tool_use"` with two `tool_use` content blocks in a single response.

What is the correct way to return the results of both tool calls to the model in the next API request?

- **A)** Add each `tool_result` as a separate `user` turn so the model sees alternating user/assistant messages.
- **B)** Replace the last user message with a combined message containing both `tool_result` blocks, discarding the assistant turn.
- **C)** Make two separate API calls, each with one `tool_result` block, so the model processes them individually.
- **D)** Append the model's full assistant message to the conversation, then add a single `user` message containing both `tool_result` content blocks.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

When the model requests multiple tool calls in one turn, all results must be returned together in a single `user` message containing one `tool_result` block per tool call, each keyed to its corresponding `tool_use_id`. The assistant turn that triggered the calls must remain in the conversation history to preserve the alternating structure.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use

**Why the others are wrong:**

- **A:** Inserting multiple `user` turns breaks the required alternating structure; the API expects one `user` message containing all `tool_result` blocks for the current round.
- **B:** Discarding the assistant turn violates the alternating user/assistant message structure required by the Messages API and would cause a validation error.
- **C:** Splitting results across two API calls creates separate, unrelated conversation threads; the model would lack full context and the `tool_use_id` linkage between call and result would be broken.

</details>

---

#### C8

*TS 1.1 · Agentic loops*

**Situation:** You are building an agentic loop and need to define a safe termination policy. The loop should stop when the model signals completion, but you also need a guard against runaway loops caused by bugs or unexpected model behavior.

Which combination of termination conditions is considered best practice for a production agentic loop?

- **A)** Stop when the model's response contains no `text` content blocks, regardless of `stop_reason`.
- **B)** Stop after a fixed number of seconds regardless of `stop_reason`.
- **C)** Stop only when `stop_reason == "end_turn"`; trust the model to always reach that state.
- **D)** Stop when `stop_reason == "end_turn"` OR a maximum iteration count is reached, whichever comes first.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

A robust loop checks the semantic termination signal (`stop_reason == "end_turn"`) as the primary exit condition and a hard iteration cap as a safety backstop. The iteration cap prevents infinite loops caused by bugs, unexpected model behavior, or tool failures that never resolve to `end_turn`.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use
- https://docs.anthropic.com/en/docs/build-with-claude/agents/build-effective-agents

**Why the others are wrong:**

- **A:** The absence of `text` content blocks is not a reliable termination signal; the model may return only `tool_use` blocks legitimately for many consecutive turns and still be making progress.
- **B:** A time-based cutoff is fragile because tool execution latency varies. A slow but valid tool call could be interrupted mid-execution, leaving state inconsistent. Iteration-based limits are more deterministic.
- **C:** Relying solely on the model to signal `end_turn` is unsafe in production; a bug or malformed tool result could cause the loop to cycle indefinitely, consuming tokens and potentially causing cascading failures.

</details>

---

#### C9

*TS 1.1 · Agentic loops*

**Situation:** A developer has built an agentic loop where the model calls a `search_web` tool and a `read_url` tool in the same turn. The developer's code iterates over `response.content` to find all `tool_use` blocks and runs each tool, then collects all results into a single user message.

When building the `tool_result` blocks to return to the model, what is the critical linking field that must be preserved for each result?

- **A)** The index position of the `tool_use` block in `response.content`, so results are returned in order.
- **B)** The `name` of the tool that was called, so the model knows which tool produced the result.
- **C)** The `input` object from the `tool_use` block, echoed back alongside the result.
- **D)** The `tool_use_id` from the originating `tool_use` block, so the model can correlate each result with the correct call.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Each `tool_result` block must include a `tool_use_id` that matches the `id` field of the `tool_use` block it answers. The model uses this ID to correctly associate each result with the right reasoning step; without it, the model cannot determine which output corresponds to which tool call.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use

**Why the others are wrong:**

- **A:** The API does not use positional index for result correlation; results can be returned in any order as long as each carries the correct `tool_use_id`.
- **B:** The tool name is useful for human readability but is not the field the model uses to correlate results. Two calls to the same tool in one turn would be ambiguous by name alone — they share a name but have distinct `id` values.
- **C:** Echoing the `input` object is not required by the protocol and does not serve as the correlation mechanism; `tool_use_id` is the sole required linking field.

</details>

---

#### C10

*TS 1.1 · Agentic loops*

**Situation:** A developer wants their agentic loop to execute multiple independent tool calls — such as three separate web searches — as fast as possible. The model returns all three `tool_use` blocks in a single response.

What execution strategy should the orchestrator use for the three tool calls to minimize total latency before feeding results back to the model?

- **A)** Execute each tool call sequentially in the order they appear in `response.content` and return results after all three complete.
- **B)** Execute the tool calls in reverse order of their appearance to give the model the most recently requested result first.
- **C)** Execute all three tool calls in parallel (e.g., using `asyncio.gather` or a thread pool) and collect all results before constructing the single user message.
- **D)** Execute the first tool call, return its result to the model immediately, wait for a new response, then execute the second call.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

When multiple tool calls in a single response are independent, executing them in parallel minimizes wall-clock latency — all three calls run concurrently and all results are collected before the next API call. The results are then bundled into the single required user message, respecting the protocol while maximizing throughput.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use
- https://docs.anthropic.com/en/docs/build-with-claude/agents/build-effective-agents

**Why the others are wrong:**

- **A:** Sequential execution is correct for ordering but suboptimal for latency; three serial tool calls take 3x as long as parallel execution when the calls are independent.
- **B:** Execution order does not affect correctness since all results must be collected before the next API call, but reverse order provides no benefit and may confuse readers of the code.
- **D:** Returning partial results mid-turn by making an API call after each tool violates the protocol: all tool results for a given assistant turn must be returned in a single user message, not incrementally.

</details>

---

#### C11

*TS 1.1 · Agentic loops*

Your agentic loop implementation uses the following termination logic:


```python
if "I've completed" in response.content[0].text:
    break
```


The agent occasionally runs indefinitely or terminates mid-investigation. What is the correct approach?

- **A)** Check whether the response contains any `TextBlock` content, as this indicates the model has produced its final answer.
- **B)** Parse the response for keywords like "done," "complete," or "finished" to determine when the agent has concluded its work.
- **C)** Add a maximum iteration counter (e.g., `max_iterations=10`) as the primary stopping mechanism to prevent infinite loops.
- **D)** Check `response.stop_reason == "end_turn"` to terminate the loop, and continue when `stop_reason == "tool_use"`.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

The agentic loop must check `stop_reason`. `"tool_use"` → continue. `"end_turn"` → stop. Iteration caps are fine as a secondary safety net but not as the primary stopping mechanism.

**Why the others are wrong:**

- **A:** Checking for TextBlock is unreliable — responses can contain both text and tool calls.
- **B:** Parsing text is an anti-pattern.
- **C:** Parsing text is an anti-pattern.

</details>

---

#### C12

*TS 1.1 · Agentic loops*

Your development agent has an agentic loop that processes tool calls. Currently, the loop checks whether the response contains a `TextBlock` to determine if Claude is done. During complex tasks with interleaved text and tool calls, the agent sometimes stops prematurely. Why?

- **A)** The temperature is too low, causing the model to generate text too early.
- **B)** Checking for `TextBlock` is an anti-pattern. Claude responses can contain both text AND tool calls simultaneously. The correct signal is `stop_reason == "end_turn"`.
- **C)** The model's `max_tokens` is too low, causing it to end the response before finishing its tool calls.
- **D)** The system prompt needs to instruct the model to never produce text until all tool calls are complete.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Checking for `TextBlock` is an anti-pattern — responses can contain both text and tool calls. The correct signal is `stop_reason == "end_turn"`. Low temperature (A), max_tokens (C), and prompt instructions (D) don't address the fundamental issue.

</details>

---

#### C13

*TS 1.1 · Agentic loops*

Your agent's agentic loop has been running tool calls for a code exploration task. The loop condition is:


```python
for i in range(20):
    response = client.messages.create(...)
    if response.stop_reason == "end_turn":
        break
    # ... process tool calls
```


What's wrong with this implementation?

- **A)** Nothing — this is the correct pattern with a safety limit.
- **B)** The iteration cap (`range(20)`) as a primary stopping mechanism is an anti-pattern. If the model needs 21 iterations to complete, it will stop prematurely. Use `stop_reason` as the primary mechanism; if you must have a safety limit, make it very generous and log a warning when hit.
- **C)** The loop should check for tool calls in the content blocks, not the `stop_reason`.
- **D)** `"end_turn"` is the wrong signal. It should check for `"stop"`.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

The code already uses `stop_reason == "end_turn"` as the primary mechanism, which is correct. The issue is that `range(20)` acts as a tight cap that could prematurely terminate a legitimate exploration needing 21+ iterations. Safety limits are valid as a secondary safeguard but should be generous (e.g., 50+) with a warning logged when hit.

</details>

---

#### C14

*TS 1.1 · Agentic loops*

Your agent loop appends each tool result to the conversation messages and sends them back to Claude for the next iteration. A colleague suggests optimizing by only sending the latest tool result instead of the full conversation history. What's wrong with this optimization?

- **A)** Only the system prompt and latest result are needed — the model's internal memory handles the rest.
- **B)** Nothing — sending only the latest result reduces token costs.
- **C)** The full conversation history must be sent in subsequent API requests so Claude can reason about accumulated context. Without prior results, the model loses track of what's already been investigated and may repeat actions.
- **D)** You can omit tool results but must keep assistant messages.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Full conversation history must be sent so Claude reasons about accumulated context. Without prior results, the model repeats investigations and loses track of the case. This is how the Messages API works — it's stateless.

</details>

---

#### C15

*TS 1.1 · Agentic loops*

Your agent's agentic loop has this structure:


```python
for i in range(15):
    response = client.messages.create(...)
    if response.stop_reason == "end_turn":
        break
    # ... process tool calls
```


What's the concern with this implementation?

- **A)** The loop should check for tool calls in content blocks, not the `stop_reason`.
- **B)** `"end_turn"` is the wrong signal — it should check for `"complete"`.
- **C)** Nothing — this is the correct pattern with appropriate safety limits.
- **D)** The iteration cap (`range(15)`) could prematurely terminate legitimate explorations needing 16+ iterations. Use `stop_reason` as primary; if a safety limit is needed, make it generous (50+) and log warnings when hit.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

The `range(15)` cap could terminate legitimate explorations needing 16+ iterations. Safety limits are valid as secondary safeguards but should be generous (50+) with warnings.

**Why the others are wrong:**

- **A:** Content block checking is an anti-pattern.
- **B:** `"complete"` isn't valid.

</details>

---

#### C16

*TS 1.2 · Coordinator-subagent orchestration*

**Situation:** A team is building a multi-agent research pipeline. The architect explains that the system follows a hub-and-spoke topology where one agent manages all the others.

In a hub-and-spoke multi-agent architecture, which agent is responsible for spawning subagents, assigning tasks, and aggregating their results?

- **A)** The subagent with the largest context window.
- **B)** The coordinator agent, which acts as the single authority over the entire workflow.
- **C)** A designated peer subagent elected by majority vote among all agents.
- **D)** Whichever subagent completes its task first and becomes available.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

In hub-and-spoke architecture the coordinator is the single authority. It decomposes the overall goal into subtasks, assigns them to subagents, and aggregates results. Subagents do not elect leaders, swap roles, or communicate with each other directly — all orchestration flows through the coordinator.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems

**Why the others are wrong:**

- **A:** Context window size is a resource constraint, not an authority indicator; the coordinator role is an architectural assignment, not something earned by having more tokens available.
- **C:** Peer election is a distributed consensus pattern, not hub-and-spoke. It introduces coordination overhead and eliminates the clear authority boundary that simplifies error recovery.
- **D:** Allowing the first available subagent to assume control is ad-hoc and removes the single-authority guarantee that makes hub-and-spoke predictable and auditable.

</details>

---

#### C17

*TS 1.2 · Coordinator-subagent orchestration*

**Situation:** A junior engineer is reading the team's multi-agent design doc. She notices that subagent A produces output that subagent B needs, but the diagram shows no direct arrow between them — only arrows to and from the coordinator.

Why do subagents in a hub-and-spoke architecture route results through the coordinator rather than sending them directly to peer subagents?

- **A)** Routing through the coordinator keeps the coordinator as the single decision-making authority, simplifies error recovery, and prevents subagents from building hidden dependencies on each other.
- **B)** Direct subagent-to-subagent calls are technically impossible in the Anthropic SDK.
- **C)** It reduces total latency because the coordinator batches all messages before forwarding them.
- **D)** Subagents lack tool access and cannot initiate network calls to other agents.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

The hub-and-spoke pattern deliberately routes all inter-agent communication through the coordinator. This preserves the coordinator's role as the single authority that can retry, reorder, or abort subtasks, and it prevents the tangled peer dependencies that make distributed debugging extremely difficult.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems

**Why the others are wrong:**

- **B:** The SDK does not technically prevent agents from calling other agents; the constraint is an architectural choice, not a platform limitation.
- **C:** Routing through a coordinator typically adds latency compared to direct peer calls; the pattern is chosen for correctness and maintainability, not speed.
- **D:** Subagents can be granted tool access including tools that communicate with external systems; lacking tools is not the reason for coordinator routing.

</details>

---

#### C18

*TS 1.2 · Coordinator-subagent orchestration*

**Situation:** A coordinator agent is given a large research task and must divide it into independent subtopics, each handled by a dedicated subagent. The coordinator plans to run all subagents at the same time.

When subtasks are fully independent of each other, what execution strategy should the coordinator use to minimize total wall-clock time?

- **A)** Delegated execution — let each subagent decide when to start based on its own availability.
- **B)** Sequential execution — run each subagent one at a time to avoid API rate limits.
- **C)** Random execution order — the coordinator picks a random subagent to run next until all are done.
- **D)** Parallel execution — invoke all subagents concurrently so their work overlaps.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

When subtasks have no dependencies on each other, parallel execution is the correct strategy. Running subagents concurrently means total wall-clock time approaches the duration of the longest subtask rather than the sum of all subtask durations.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems

**Why the others are wrong:**

- **A:** Delegating scheduling authority to subagents violates the single-authority principle; the coordinator, not subagents, decides when work begins.
- **B:** Sequential execution is appropriate only when tasks have ordering dependencies. Running independent tasks sequentially wastes time and is the wrong default for independent workloads.
- **C:** Random ordering provides no deterministic benefit and does not exploit concurrency; it is not a recognized coordination strategy.

</details>

---

#### C19

*TS 1.2 · Coordinator-subagent orchestration*

**Situation:** A coordinator agent receives results back from three subagents that each processed a different data segment. The coordinator must now combine these results into a single coherent output for the user.

Which term best describes the coordinator's responsibility of combining subagent outputs into a unified result?

- **A)** Result aggregation
- **B)** Parallel dispatch
- **C)** Context isolation
- **D)** Task decomposition

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Result aggregation is the coordinator's responsibility of merging, reconciling, or synthesizing the outputs returned by subagents into the final unified answer. Task decomposition is the complementary step of splitting the work before dispatch; context isolation and parallel dispatch describe other aspects of multi-agent design.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems

**Why the others are wrong:**

- **B:** Parallel dispatch is the act of invoking multiple subagents concurrently, which happens before results are available, not after.
- **C:** Context isolation refers to giving each subagent its own conversation context so subagents do not share or pollute each other's state — it is not about combining results.
- **D:** Task decomposition is the act of breaking the original task into subtasks before sending them to subagents — the inverse operation of aggregation.

</details>

---

#### C20

*TS 1.2 · Coordinator-subagent orchestration*

**Situation:** A coordinator orchestrates a data pipeline with six subagents. Subagents A and B run in parallel in phase 1. Subagent C depends on A's output. Subagent D depends on both A and B. Subagents E and F are independent and can run at any time. The team wants to minimize total latency while respecting all dependencies.

What is the optimal execution schedule that minimizes wall-clock time while satisfying all dependency constraints?

- **A)** Run A, B, E, F in parallel first; after A completes start C; after both A and B complete start D. This yields three sequential phases with maximum concurrency within each.
- **B)** Run all six agents sequentially in the order A → B → C → D → E → F to ensure no dependency is violated.
- **C)** Run A, B, E, F in parallel; then run C, D, E-continuation, F-continuation sequentially to avoid any race conditions.
- **D)** Run A and B in parallel first; after both complete, run C, D, E, F all in parallel.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

The optimal schedule starts A, B, E, and F in parallel since none depend on others. As soon as A completes, C can start immediately without waiting for B. D must wait for both A and B. E and F are independent and run from the start. This overlaps C's execution with B's remaining runtime, minimizing idle time and total latency compared to any strategy that waits for all of phase 1 to finish before starting downstream agents.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems

**Why the others are wrong:**

- **B:** Full sequential execution ignores all parallelism opportunities and maximizes total wall-clock time to the sum of all agent durations — the worst possible schedule.
- **C:** Forcing C and D into a sequential phase after A completes, rather than starting C as soon as A finishes, introduces unnecessary idle time for C while B is still running.
- **D:** Waiting for both A and B before starting C is suboptimal: C only depends on A. Starting C the moment A finishes, while B is still in progress, reduces total latency for the C path.

</details>

---

#### C21

*TS 1.2 · Coordinator-subagent orchestration*

**Situation:** An architect designs a pipeline where a coordinator first runs a planning subagent to produce a task list, then dispatches worker subagents for each task, and finally runs a review subagent that validates the combined outputs. The planning step must complete before workers start, and workers must all finish before the review step begins.

Which execution model correctly describes this pipeline?

- **A)** Fully sequential: each agent runs one at a time in a fixed order, including the worker agents.
- **B)** Recursive: each worker subagent spawns the next worker subagent in the chain until the list is exhausted.
- **C)** Mixed: sequential phases where phases 1 and 3 are single-agent steps, and phase 2 is a parallel fan-out of worker subagents.
- **D)** Fully parallel: all four agent types run simultaneously to maximize throughput.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

This pipeline has hard ordering dependencies between phases but independence within the worker phase. The correct model is sequential at the phase level (plan → work → review) with parallel fan-out for the independent worker subagents within the work phase. This is the standard coordinator-managed mixed execution pattern.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems

**Why the others are wrong:**

- **A:** Running worker subagents sequentially when they are independent wastes concurrency; the mixed model correctly parallelizes only where it is safe to do so.
- **B:** Chaining workers so each spawns the next creates a sequential peer-to-peer chain that bypasses the coordinator's authority and eliminates the parallelism benefit.
- **D:** Fully parallel execution ignores the data dependencies: worker subagents need the plan before they can begin, and the review subagent needs worker results before it can validate.

</details>

---

#### C22

*TS 1.2 · Coordinator-subagent orchestration*

**Situation:** A coordinator agent needs to split a 10,000-word legal document into five sections and send each section to a separate summarization subagent. The team debates how much context each subagent should receive.

What is the primary reason for giving each subagent only the section it needs to summarize rather than the full document?

- **A)** The API will reject requests that include the full document in a subagent's system prompt.
- **B)** Subagents cannot process text longer than 2,000 tokens, so the document must be split for technical reasons.
- **C)** Sending the full document to each subagent duplicates cost but has no effect on output quality.
- **D)** Isolated context per subagent reduces noise, prevents cross-section confusion, and keeps each subagent's context window small and focused.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Context isolation is a core principle of multi-agent design. When each subagent receives only the data relevant to its task, it produces more focused results, avoids confusion caused by irrelevant content from other sections, and consumes a smaller context window — which improves both quality and efficiency.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems

**Why the others are wrong:**

- **A:** The API does not enforce a rule against passing full documents in subagent prompts; the constraint is a quality and efficiency design principle, not a platform restriction.
- **B:** Claude models support context windows of 200,000 tokens; a 2,000-token limit does not exist. Splitting is an architectural choice for quality, not a hard technical ceiling.
- **C:** Sending the full document to every subagent does affect quality: extraneous context increases the chance the model attends to irrelevant sections and produces lower-quality targeted summaries.

</details>

---

#### C23

*TS 1.2 · Coordinator-subagent orchestration*

**Situation:** A coordinator agent is designing a workflow where subagent X must produce a classification label that subagent Y then uses to select a processing strategy. The coordinator considers whether to run X and Y in parallel or in sequence.

Which execution model is required when one subagent's output is a direct input to another subagent's task?

- **A)** Parallel execution with a merge step: both run at the same time and the coordinator reconciles any inconsistencies after both finish.
- **B)** Sequential execution: subagent X must complete and return its output to the coordinator before the coordinator invokes subagent Y with that output.
- **C)** The subagents can self-coordinate: X pushes its result directly to Y over a side channel without involving the coordinator.
- **D)** Parallel execution: both subagents are invoked simultaneously and Y polls a shared store until X writes its result.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

When a data dependency exists between subagents — Y's input is X's output — the coordinator must execute them sequentially. The coordinator waits for X to return its result, then uses that result when constructing Y's invocation. This is the defining case where parallel execution is incorrect and sequential execution is mandatory.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems

**Why the others are wrong:**

- **A:** Running both in parallel when Y depends on X's output means Y will either block waiting for data that does not yet exist or operate on a missing or default value, producing incorrect results. Parallel execution is only valid for independent tasks.
- **C:** Direct side-channel communication between subagents violates the hub-and-spoke principle. The coordinator must remain the intermediary for all inter-agent data flow so it retains visibility, control, and error recovery authority.
- **D:** Having Y poll a shared store introduces a busy-wait coupling between subagents and a race condition if X fails; it also bypasses the coordinator's authority by letting subagents coordinate through a side channel rather than through the coordinator.

</details>

---

#### C24

*TS 1.2 · Coordinator-subagent orchestration*

After researching "global food security challenges," the final report discusses only crop production, omitting supply chain disruption, climate adaptation, and nutritional equity. Coordinator logs show it decomposed the topic into: "wheat production trends," "rice yield forecasting," and "corn production technology." What is the root cause?

- **A)** The coordinator's task decomposition was too narrow, assigning subtasks that only covered one dimension of the topic.
- **B)** The document analysis subagent filtered out non-production-related sources.
- **C)** The synthesis subagent failed to identify coverage gaps and request additional research on missing areas.
- **D)** The web search subagent's queries were too narrow and only returned crop-specific results.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

The coordinator decomposed a broad topic ("food security") into narrow subtasks (only crop production). Subagents worked correctly within their assigned scope.

**Why the others are wrong:**

- **B:** The root cause is the coordinator's decomposition, not downstream agents.
- **C:** The root cause is the coordinator's decomposition, not downstream agents.
- **D:** The root cause is the coordinator's decomposition, not downstream agents.

</details>

---

#### C25

*TS 1.3 · Subagent invocation & context passing*

**Situation:** A developer is explaining subagent context isolation to a new team member. The coordinator has accumulated a long conversation history including sensitive user credentials it received earlier in the session.

When the coordinator spawns a subagent using the Task tool, which statement about context sharing is correct?

- **A)** The subagent shares the coordinator's system prompt but not its message history.
- **B)** The subagent inherits all tool results from the coordinator's current turn.
- **C)** The subagent automatically inherits the coordinator's full conversation history.
- **D)** The subagent starts with an isolated context and receives only what the coordinator explicitly passes in the Task invocation.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Subagents have fully isolated contexts. They do not share memory, conversation history, or tool results with the coordinator. The only information a subagent receives is what is serialized and passed explicitly in the Task invocation — typically in the prompt or user message field.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems
- https://docs.anthropic.com/en/docs/claude-code/sdk

**Why the others are wrong:**

- **A:** Subagents do not share the coordinator's system prompt by default. They receive their own system prompt (if any) specified at invocation time.
- **B:** Tool results in the coordinator's context are not transmitted to subagents. Subagents only see data the coordinator explicitly serializes and passes as part of the Task call.
- **C:** Subagents never inherit the coordinator's conversation history automatically. Each subagent starts a fresh context, which is a core safety and isolation property of the multi-agent architecture.

</details>

---

#### C26

*TS 1.3 · Subagent invocation & context passing*

**Situation:** An engineering team is adopting multi-agent patterns. A senior engineer states: 'We should give each subagent only the tools it absolutely needs for its specific subtask.' A junior engineer asks why this matters if all subagents are trusted internal components.

Which principle does the senior engineer's recommendation reflect, and what is its primary benefit in a multi-agent system?

- **A)** Fail-fast — it causes subagents to error early when tools are misconfigured.
- **B)** Separation of concerns — it keeps each agent's code simpler.
- **C)** Least-privilege — it limits the blast radius if a subagent is compromised or behaves unexpectedly.
- **D)** Single responsibility — it ensures each subagent has exactly one purpose.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

The least-privilege principle dictates that each subagent should only have access to the tools it genuinely needs. In a multi-agent system this is critical because a compromised or prompt-injected subagent with broad tool access could cause far greater damage than one with narrowly scoped tools. The `allowed_tools` parameter enforces this deterministically.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems
- https://docs.anthropic.com/en/docs/build-with-claude/agents/agent-security

**Why the others are wrong:**

- **A:** Fail-fast is about detecting errors early in a process, not about restricting tool permissions for security or blast-radius control.
- **B:** Separation of concerns is about dividing responsibilities across components for maintainability, not about restricting tool access for safety. While related in spirit, it does not capture the security motivation here.
- **D:** Single responsibility is a design principle about an agent's purpose and role, not about controlling which tools it can invoke at runtime.

</details>

---

#### C27

*TS 1.3 · Subagent invocation & context passing*

**Situation:** A coordinator agent must pass a structured JSON payload — a list of customer records — to a subagent so the subagent can perform data validation. The coordinator serializes the records as a JSON string.

Where should the coordinator embed this serialized data when invoking the subagent via the Task tool?

- **A)** In a shared in-memory cache that both agents access.
- **B)** As an environment variable injected into the subagent's runtime.
- **C)** Explicitly in the prompt or user message field of the Task invocation.
- **D)** In the subagent's system prompt, which is pre-loaded at startup.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Because subagents have isolated contexts and do not share memory with the coordinator, all data must be passed explicitly. The canonical pattern is to serialize the data (e.g., as JSON) and embed it directly in the user message field of the Task invocation. The subagent then has full access to that data within its own context.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems
- https://docs.anthropic.com/en/docs/claude-code/sdk

**Why the others are wrong:**

- **A:** A shared in-memory cache is an external side-channel and not part of the Agent SDK's context-passing model. Relying on shared state undermines isolation and introduces race conditions.
- **B:** Environment variables are set at process startup and are not a dynamic context-passing mechanism for inter-agent communication at invocation time.
- **D:** The subagent's system prompt is defined statically at invocation and is not appropriate for passing dynamic runtime data like a variable list of customer records.

</details>

---

#### C28

*TS 1.3 · Subagent invocation & context passing*

**Situation:** A coordinator agent is asked to process 1,000 product descriptions and categorise each one. The coordinator spawns subagents in batches of 50, passing each batch as a serialised JSON array in the user message. After the first batch completes, a developer notices the subagents' outputs are inconsistent — some use different category taxonomies than others.

Which root cause and fix best explains and resolves the taxonomy inconsistency across subagents?

- **A)** Each subagent has an isolated context and does not share the coordinator's system prompt or any shared taxonomy definition unless it is explicitly passed. Fix: include the canonical taxonomy definition in every Task invocation.
- **B)** The model used for subagents is non-deterministic by default. Fix: set `temperature: 0` on the coordinator to stabilise outputs.
- **C)** The subagents have different context window sizes, so some truncate the taxonomy definition. Fix: reduce batch size.
- **D)** The JSON serialisation of the batch is inconsistent. Fix: switch to CSV encoding for the product descriptions.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Because subagents have isolated contexts, they receive only what is explicitly passed in the Task invocation. If the taxonomy is defined in the coordinator's system prompt or memory, subagents are unaware of it — they may infer different categories independently. The fix is to include the canonical taxonomy in every Task invocation, either in the subagent's system prompt or the user message, so all subagents operate from the same definition.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems
- https://docs.anthropic.com/en/docs/claude-code/sdk

**Why the others are wrong:**

- **B:** `temperature: 0` applies to the coordinator's own generation, not to subagents' generation. Even at `temperature: 0` the coordinator cannot force subagents to use the same taxonomy if they have not been given one; and the subagent temperature must be set independently on each Task invocation.
- **C:** Context window truncation would manifest as missing product descriptions at the end of a batch, not as taxonomy inconsistency. A taxonomy definition is typically short and would not be truncated.
- **D:** The encoding format (JSON vs CSV) does not affect category taxonomy. The inconsistency is a semantic issue — subagents not having a shared definition — not a parsing issue.

</details>

---

#### C29

*TS 1.3 · Subagent invocation & context passing*

**Situation:** A coordinator agent spawns three subagents in parallel: one to query a database, one to call an external REST API, and one to read from a local file. All three tasks are independent. After all three complete, the coordinator aggregates their outputs.

What is the correct Agent SDK pattern to spawn all three subagents simultaneously rather than sequentially?

- **A)** Set `parallel: true` on each Task invocation to signal concurrent execution.
- **B)** Invoke the Task tool for the first subagent, wait for its result, then invoke the second, and so on.
- **C)** Invoke the Task tool three times in the same model turn, placing all three Task calls in the same response, so the runtime executes them concurrently.
- **D)** Use separate API clients for each subagent and call them from the coordinator's system prompt.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

In the Agent SDK, a coordinator can request parallel subagent execution by emitting multiple Task tool calls within a single model turn. The SDK runtime detects these concurrent tool calls and executes them in parallel, then returns all results to the coordinator in a single follow-up turn. This is identical in structure to how the Messages API handles multiple tool calls in one response.

References:
- https://docs.anthropic.com/en/docs/claude-code/sdk
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems

**Why the others are wrong:**

- **A:** There is no `parallel: true` flag on Task invocations. Parallelism is achieved structurally by emitting multiple Task calls in the same model turn.
- **B:** Sequential invocation defeats the purpose of parallelism. Waiting for each subagent to complete before starting the next serialises latency instead of overlapping it.
- **D:** Using separate API clients from within the system prompt is not a valid execution model; system prompts are static text, not executable code.

</details>

---

#### C30

*TS 1.3 · Subagent invocation & context passing*

**Situation:** A coordinator agent processes a user's financial report request. It needs a subagent to calculate tax liabilities from raw transaction data. The coordinator has already fetched and validated 200 transaction records in its own context.

Which approach correctly passes the transaction data to the subagent while respecting context isolation?

- **A)** Serialize the 200 validated transaction records as JSON and include them in the user message of the Task invocation.
- **B)** Instruct the subagent to call the same data-fetch tool that the coordinator used, so it retrieves the records independently.
- **C)** Store the records in a global variable and tell the subagent the variable name in its prompt.
- **D)** Pass only the user's original query to the subagent and let it re-fetch and re-validate the data on its own.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Since subagents have isolated contexts and cannot access the coordinator's memory, the coordinator must serialize the already-validated records and include them explicitly in the Task invocation's user message. This avoids redundant I/O (re-fetching) and ensures the subagent operates on the same validated dataset the coordinator prepared.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems
- https://docs.anthropic.com/en/docs/claude-code/sdk

**Why the others are wrong:**

- **B:** Having the subagent re-fetch the data duplicates network I/O, risks fetching a different version of the data, and discards the coordinator's validation work.
- **C:** Global variables are not accessible across isolated subagent contexts. The subagent runs in a separate process or context and cannot read the coordinator's runtime variables.
- **D:** Letting the subagent re-fetch and re-validate independently is inefficient, potentially inconsistent, and may exceed the subagent's tool permissions if it was not granted the data-fetch tool.

</details>

---

#### C31

*TS 1.3 · Subagent invocation & context passing*

**Situation:** A developer writes the following pseudocode for a coordinator agent:


```
result1 = Task(prompt='Summarise document A', allowed_tools=['file_read'])
result2 = Task(prompt='Summarise document B', allowed_tools=['file_read'])
aggregated = Task(prompt=f'Combine: {result1} and {result2}', allowed_tools=[])
```


The first two tasks are independent. The third depends on both results.

Which statement best describes a flaw in this implementation?

- **A)** The `allowed_tools=[]` on the aggregation task will cause an error; at least one tool must always be specified.
- **B)** The first two tasks are invoked sequentially rather than in parallel, missing an opportunity to reduce latency.
- **C)** The aggregation task cannot receive data from the first two tasks because subagents have isolated contexts.
- **D)** The `file_read` tool cannot be listed in `allowed_tools` for more than one subagent at a time.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Because the summaries of documents A and B are independent, they could be requested in the same model turn as parallel Task calls. The sequential pseudocode (awaiting result1 before starting result2) serialises two latencies that could be overlapped. The aggregation task correctly depends on both results and must remain sequential.

References:
- https://docs.anthropic.com/en/docs/claude-code/sdk
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems

**Why the others are wrong:**

- **A:** `allowed_tools=[]` is a valid and intentional configuration meaning the subagent has no tool access. For a pure text-aggregation task that only needs to reason over passed-in strings, this is correct least-privilege behaviour.
- **C:** The aggregation task receives data correctly because the coordinator serialises `result1` and `result2` and passes them explicitly in the aggregation prompt. Context isolation prevents implicit sharing, but explicit passing via the prompt is the correct pattern.
- **D:** There is no restriction on multiple subagents sharing the same tool name in their `allowed_tools` list. Each subagent's tool access is independent.

</details>

---

#### C32

*TS 1.3 · Subagent invocation & context passing*

**Situation:** A research assistant workflow has a `web_search` step followed by a `synthesise_findings` step. The architect wants the synthesis prompt to include the exact URLs returned by the search so the model can cite sources. Currently, the workflow passes only the search query string as context to the synthesis step.

Which change to the handoff protocol best satisfies this requirement?

- **A)** Store the structured output of `web_search` — including the list of URLs and snippets — in the workflow state, and inject the relevant fields into the synthesis step's user message.
- **B)** Re-run `web_search` at the start of the synthesis step to retrieve the URLs again.
- **C)** Add a PostToolUse hook to `web_search` that sends the URLs to an external database and instructs the model to query it during synthesis.
- **D)** Ask the model to remember the URLs from the search step by including a reminder in the system prompt.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Persisting the structured output of `web_search` in the workflow state and explicitly injecting it into the next step's context is the canonical structured handoff pattern. It ensures the synthesis step receives exactly the data it needs (URLs and snippets) without relying on the model's memory, re-execution, or external lookups.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems

**Why the others are wrong:**

- **B:** Re-running `web_search` is wasteful and may return different results, making the synthesis non-reproducible. The correct pattern is to pass forward the outputs already obtained.
- **C:** Routing URLs through an external database adds unnecessary infrastructure complexity and still requires the model to decide when and how to query it, reintroducing probabilistic behaviour.
- **D:** Asking the model to 'remember' URLs relies on in-context retention, which is unreliable across long workflows. The model does not have persistent memory between turns unless data is explicitly passed through the conversation.

</details>

---

#### C33

*TS 1.3 · Subagent invocation & context passing*

Your coordinator agent's `AgentDefinition` has `allowedTools: ["web_search", "analyze_document"]`. When the coordinator tries to spawn a subagent using the `Task` tool, it fails silently. What is the problem?

- **A)** The coordinator needs `allowedTools: ["Task"]` (or a list that includes `"Task"`) to be able to spawn subagents.
- **B)** The `Task` tool must be explicitly registered in the `.mcp.json` configuration file.
- **C)** Subagent definitions must be loaded into the coordinator's system prompt before they can be invoked.
- **D)** The coordinator needs a `spawn_subagent` flag set to `true` in its `AgentDefinition`.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

The coordinator's `allowedTools` must include `"Task"` to spawn subagents. Without it, the Task tool call fails. `.mcp.json` (B), system prompts (C), and flags (D) are not the mechanism for enabling subagent spawning.

</details>

---

#### C34

*TS 1.3 · Subagent invocation & context passing*

The web search subagent finds 12 relevant articles about renewable energy policy. You need to pass these results to the synthesis subagent. How should you transfer this context?

- **A)** Store the results in a shared database and give the synthesis subagent the database query to retrieve them.
- **B)** Include the complete search results directly in the synthesis subagent's `Task` prompt, with content separated from metadata (source URLs, dates, document names).
- **C)** The synthesis subagent automatically inherits the coordinator's context, which includes the web search results.
- **D)** Save the results to a file and instruct the synthesis subagent to read the file using the Read tool.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Subagents have isolated context — they don't inherit the coordinator's history. Findings must be passed explicitly in the Task prompt. Shared databases (A) and files (D) add unnecessary complexity.

**Why the others are wrong:**

- **C:** Automatic inheritance is incorrect — this is a key exam concept.

</details>

---

#### C35

*TS 1.3 · Subagent invocation & context passing*

You want the web search and document analysis subagents to execute simultaneously on different aspects of a research topic. How should you implement parallel execution?

- **A)** Have the coordinator send the first Task call, wait for it to complete, then send the second.
- **B)** Have the coordinator emit multiple `Task` tool calls in a single response — one for each subagent.
- **C)** Configure `parallel: true` in each subagent's `AgentDefinition`.
- **D)** Use Python threading to spawn both subagents in separate threads from your application code.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Parallel execution is achieved by emitting multiple Task calls in a single coordinator response.

**Why the others are wrong:**

- **A:** Sequential calls are by definition not parallel.
- **C:** `parallel: true` doesn't exist.
- **D:** Python threading is application-level, not SDK-level.

</details>

---

#### C36

*TS 1.3 · Subagent invocation & context passing*

The coordinator spawns a synthesis subagent with the prompt: "Synthesize the research findings." The subagent produces a vague, generic synthesis that doesn't reference the specific findings from the search and analysis phases. What's wrong?

- **A)** The synthesis subagent needs a more detailed system prompt with specific synthesis instructions.
- **B)** The subagent has isolated context — it did not receive the search and analysis findings. The coordinator must explicitly pass the findings directly in the Task prompt.
- **C)** The synthesis subagent needs access to the Read tool to load the findings from a shared file.
- **D)** The coordinator should re-run the search and analysis subagents within the synthesis context.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Subagent context isolation. The coordinator must pass findings explicitly in the Task prompt. File reading (C) and re-running (D) add unnecessary complexity.

**Why the others are wrong:**

- **A:** Better system prompts won't help if the findings data isn't there.

</details>

---

#### C37

*TS 1.3 · Subagent invocation & context passing*

Your coordinator agent needs to pass a verified customer record to a billing subagent. Which approach correctly preserves the information?

- **A)** Tell the billing subagent: "The customer has been verified. Look up their billing history."
- **B)** Include the complete verified customer data in the billing subagent's Task prompt: `"Verified customer: ID C-1001, name Alex Chen, email alex@work.com. Investigate billing dispute for order #5582, amount $284.50, charged on 2025-02-15."`
- **C)** Store the customer data in a shared memory object that both agents can access.
- **D)** Have the billing subagent re-call `get_customer` to verify independently.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Subagents have isolated context. Pass complete verified data in the Task prompt.

**Why the others are wrong:**

- **A:** Vague instructions leave the subagent without critical data.
- **C:** Shared memory doesn't exist in the SDK pattern.
- **D:** Re-verification wastes a tool call.

</details>

---

#### C38

*TS 1.3 · Subagent invocation & context passing*

Your coordinator agent needs to pass collected patient information to a scheduling subagent. Which approach correctly preserves the information?

- **A)** Tell the scheduling subagent: "The patient has been verified. Schedule their appointment."
- **B)** Include the complete verified patient data in the scheduling subagent's Task prompt: `"Verified patient: ID P-5678, name Jordan Smith, DOB 1985-06-15, insurance verified. Schedule follow-up for Dr. Chen, preferred times: weekday afternoons."`
- **C)** Store the patient data in a shared memory object that both agents can access.
- **D)** Have the scheduling subagent re-call `lookup_patient_record` to verify independently.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Subagents have isolated context. Pass complete verified data in the Task prompt.

**Why the others are wrong:**

- **A:** Vague instructions leave the subagent without critical data.
- **C:** Shared memory doesn't exist in the SDK pattern.
- **D:** Re-verification wastes a tool call.

</details>

---

#### C39

*TS 1.3 · Subagent invocation & context passing*

Your coordinator agent needs to pass verified inventory data to a procurement subagent. Which approach correctly preserves the information?

- **A)** Tell the procurement subagent: "Inventory is low. Create purchase orders."
- **B)** Include the complete verified inventory data in the procurement subagent's Task prompt: `"Verified low-stock: SKU-1234 (current: 50, reorder: 200), SKU-5678 (current: 30, reorder: 150). Preferred suppliers: Vendor A, Vendor B."`
- **C)** Store the inventory data in a shared memory object that both agents can access.
- **D)** Have the procurement subagent re-call `check_inventory` to verify independently.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Subagents have isolated context. Pass complete verified data in the Task prompt.

</details>

---

#### C40

*TS 1.3 · Subagent invocation & context passing*

Your coordinator agent needs to pass verified standards alignment to a content generation subagent. Which approach correctly preserves the information?

- **A)** Tell the content subagent: "Standards are verified. Generate the lesson."
- **B)** Include the complete verified standards data in the content subagent's Task prompt: `"Verified standards: CCSS.MATH.7.NS.A.1-3, Grade 7 Number Systems. Generate 5-day unit with assessments."`
- **C)** Store the standards data in a shared memory object that both agents can access.
- **D)** Have the content subagent re-call `search_standards` to verify independently.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Subagents have isolated context. Pass complete verified data in the Task prompt.

</details>

---

#### C41

*TS 1.4 · Workflow enforcement & handoff*

**Situation:** An engineer is designing a data-enrichment pipeline with three steps: fetch, transform, and store. Each step produces a structured output that becomes the input for the next step. The team wants the next step to know exactly what the previous step produced.

What is the primary purpose of a structured handoff protocol between workflow steps?

- **A)** To compress the conversation history so it fits within the context window.
- **B)** To reduce the number of API calls by batching all steps into a single model turn.
- **C)** To allow the model to decide on its own which step to execute next without developer intervention.
- **D)** To ensure each step receives a well-defined, validated payload from the preceding step, preventing ambiguity and silent failures.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Structured handoff protocols define the shape of data passed between workflow steps. By validating and explicitly passing structured outputs (e.g., typed schemas or serialised state), each downstream step receives unambiguous input, which eliminates silent failures caused by missing or malformed data and makes the pipeline auditable.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems

**Why the others are wrong:**

- **A:** Context window compression is a separate concern about token management; structured handoffs carry forward only the data needed for the next step, but their primary purpose is correctness, not token reduction.
- **B:** Batching steps into a single model turn is an optimisation concern unrelated to the purpose of handoff protocols; handoffs are specifically about data contracts between steps, not API call count.
- **C:** Allowing the model to decide autonomously which step runs next is a routing concern and does not describe the purpose of a structured handoff — handoffs define what data flows, not what logic runs.

</details>

---

#### C42

*TS 1.4 · Workflow enforcement & handoff*

**Situation:** A financial services firm deploys a multi-step wire transfer workflow: (1) validate_beneficiary, (2) check_limits, (3) execute_transfer. A security audit requires that `execute_transfer` can only be called if both prior steps completed successfully in the same workflow session, the transfer amount is within the approved limit stored in state, and the beneficiary ID has not changed since validation.

How should the architect design the enforcement layer for `execute_transfer`?

- **A)** A PostToolUse hook on `check_limits` that calls `execute_transfer` directly if limits are met, bypassing the normal tool call flow.
- **B)** A PreToolUse hook on `execute_transfer` that reads the workflow state, verifies that `validate_beneficiary` and `check_limits` have both recorded success flags for the current session, confirms the transfer amount is within the stored limit, and checks that the beneficiary ID matches the validated one — aborting with a structured error if any condition fails.
- **C)** A system prompt listing all three preconditions, relying on the model to verify them before calling `execute_transfer`.
- **D)** A validator that runs after the workflow completes and flags non-compliant executions for manual review.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

All three preconditions are hard compliance requirements with irreversible financial consequences, making deterministic enforcement mandatory. A PreToolUse hook on `execute_transfer` is the correct architectural pattern: it runs before any execution, can atomically read and validate all required state fields for the current session, and aborts with a structured error if any check fails. This prevents the transfer from executing under any non-compliant condition, regardless of model reasoning.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/claude-code/hooks
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems

**Why the others are wrong:**

- **A:** Calling `execute_transfer` from inside a PostToolUse hook creates an out-of-band execution path that bypasses the standard tool call flow, making it invisible to auditors and orchestrators. It also inverts the responsibility boundary — hooks should validate or transform, not initiate new operations.
- **C:** System prompt preconditions are probabilistic. In a high-stakes financial workflow, the model may reason incorrectly about state, misinterpret flags, or be manipulated through prompt injection. Financial compliance controls must be implemented in code, not in prompts.
- **D:** Post-execution review is reactive and cannot prevent an illegal transfer. In regulated financial systems, controls must be preventive (pre-execution). After-the-fact flagging may satisfy audit logging requirements but does not constitute enforcement.

</details>

---

#### C43

*TS 1.4 · Workflow enforcement & handoff*

**Situation:** A multi-agent pipeline has an orchestrator and three specialist subagents. The orchestrator invokes each subagent in sequence and passes the output of one as the input of the next. During a production incident, the team discovers that the second subagent silently accepted a malformed handoff payload (missing required fields), produced nonsensical output, and passed it downstream — causing the third subagent to fail with an opaque error.

Which combination of architectural changes would most effectively prevent this failure mode from recurring?

- **A)** Add a system prompt to each subagent instructing it to validate its input before processing.
- **B)** Add JSON schema validation at each handoff boundary (enforced by code before the subagent receives input), and implement a PostToolUse hook on each subagent invocation that validates the output schema before passing it to the next stage.
- **C)** Use a single large context window for all three subagents so they share state and can detect inconsistencies themselves.
- **D)** Wrap each subagent call in a try/except block and log the error, then retry the failed subagent up to three times.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

The root cause is that malformed data passed through two handoff boundaries undetected. The fix requires deterministic schema enforcement at every boundary: (1) validate inbound payload before the subagent runs (PreToolUse or orchestrator-side check) and (2) validate outbound payload after the subagent produces output (PostToolUse hook) before passing it downstream. This creates a two-sided validation gate at every handoff, ensuring malformed data is caught at the source rather than propagating silently.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/claude-code/hooks
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems

**Why the others are wrong:**

- **A:** System prompt validation instructions are probabilistic. A subagent instructed to validate its input may still process malformed data, especially when the model cannot distinguish valid from invalid schemas without programmatic assistance.
- **C:** Sharing a single context window between all subagents eliminates isolation and creates a different class of correctness problems (context pollution, cross-contamination). It does not provide structured schema validation at handoff boundaries.
- **D:** Retrying a subagent that received malformed input will produce the same malformed output on each attempt. The retry logic does not address the root cause (malformed handoff) and wastes resources without improving correctness.

</details>

---

#### C44

*TS 1.4 · Workflow enforcement & handoff*

**Situation:** A legal document workflow has two enforcement layers: (1) a PreToolUse hook that blocks `file_document` if the document status is not `'APPROVED'`, and (2) a system prompt instruction to always request human approval before filing. During testing, a tester notes that under a prompt injection attack the model was manipulated into claiming the document was approved and attempting `file_document`, but the hook correctly blocked it.

What does this test result reveal about the defense-in-depth design of this workflow?

- **A)** The system prompt instruction is the primary control; the hook is redundant and should be removed to simplify the codebase.
- **B)** The test proves that prompt injection makes system prompt instructions useless and they should never be relied upon in any workflow.
- **C)** Both controls are complementary: the prompt instruction reduces the frequency of unsafe attempts under normal conditions, while the hook provides the deterministic backstop that catches attempts that bypass the prompt — including prompt injection attacks.
- **D)** The hook is the primary control; the system prompt instruction is redundant and can be removed.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Defense-in-depth in agentic workflows uses both probabilistic and deterministic controls as complementary layers. The system prompt instruction reduces unsafe attempts under normal operation by shaping model behaviour, but it cannot prevent all adversarial manipulation. The PreToolUse hook provides the deterministic backstop: even if the model is manipulated into attempting an unsafe action, the hook blocks it based on code-verified state, not model reasoning. The test result validates that the architecture is correctly layered — neither control alone is sufficient.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/claude-code/hooks
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems

**Why the others are wrong:**

- **A:** Removing the hook would leave only probabilistic control. The test result demonstrates exactly why this is insufficient: prompt injection bypassed the prompt instruction but was caught by the hook. Removing the hook eliminates the only deterministic safeguard.
- **B:** The test does not prove system prompt instructions are useless — it proves they are probabilistic and can be bypassed under adversarial conditions. The right conclusion is that prompt instructions must be backed by deterministic enforcement for high-stakes actions, not eliminated.
- **D:** Removing the system prompt instruction would increase the frequency of unsafe attempts reaching the hook. While the hook would still block them, adding unnecessary load to the enforcement layer and increasing attack surface is bad practice. Both layers serve distinct purposes.

</details>

---

#### C45

*TS 1.4 · Workflow enforcement & handoff*

**Situation:** A compliance pipeline has four sequential steps: validate_input, classify_document, apply_redaction, and archive. The team must guarantee that `apply_redaction` is never called unless `classify_document` has run successfully and produced a `sensitivity_level` field in the workflow state.

What is the most reliable design for enforcing this step-ordering requirement?

- **A)** Implement a PreToolUse hook for `apply_redaction` that checks the workflow state object for the presence and validity of `sensitivity_level` and aborts with an error if it is missing.
- **B)** Use a try/except block around `apply_redaction` to catch the KeyError if `sensitivity_level` is absent and log it.
- **C)** Write a system prompt that explicitly lists the required step order and instructs the model to follow it.
- **D)** Add a PostToolUse hook to `classify_document` that sets a global flag, and rely on the model to check the flag before calling `apply_redaction`.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

A PreToolUse hook on `apply_redaction` that inspects the workflow state provides deterministic enforcement: the tool call is gated by code, not by model intent. If `classify_document` has not run or failed to produce `sensitivity_level`, the hook aborts execution before any side-effects occur. This is the correct pattern for step-ordering enforcement in multi-step workflows.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/claude-code/hooks
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems

**Why the others are wrong:**

- **B:** Catching a KeyError is reactive error handling — the tool has already started executing. The goal is to abort before execution begins. This pattern also swallows the error silently, which violates error-handling best practices.
- **C:** System prompt ordering instructions are probabilistic. Under adversarial prompting, tool-use planning errors, or context pressure, the model may invoke `apply_redaction` prematurely. Code-level gating is required for compliance guarantees.
- **D:** Setting a flag in PostToolUse is correct, but relying on the model to check the flag introduces probabilistic behaviour. The model may not check the flag or may misinterpret it. The gate must be in code (PreToolUse), not in model reasoning.

</details>

---

#### C46

*TS 1.4 · Workflow enforcement & handoff*

**Situation:** A developer is choosing between two enforcement strategies for a content moderation step in their pipeline. Option X: a system prompt rule stating 'never pass content rated UNSAFE to the publish tool'. Option Y: a PreToolUse hook that reads the moderation score from state and blocks the publish tool call if score > 0.8.

In which scenario would you prefer Option X (prompt-based guidance) over Option Y (programmatic hook)?

- **A)** When the moderation score is computed by the model itself as a heuristic judgment, and exact thresholds are less important than nuanced contextual reasoning.
- **B)** When you need to audit every tool call for a security review board.
- **C)** When regulatory compliance requires a hard guarantee that unsafe content is never published.
- **D)** When the publish action has irreversible consequences such as sending emails or charging a payment method.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Prompt-based guidance is preferred when the decision requires nuanced contextual reasoning that is difficult to encode as a deterministic rule. If the model is generating the moderation signal itself and the threshold is a soft heuristic rather than a hard compliance boundary, a prompt instruction allows the model to exercise judgment. For hard boundaries with irreversible consequences or compliance requirements, programmatic hooks are mandatory.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/claude-code/hooks

**Why the others are wrong:**

- **B:** Audit requirements are best served by hooks (PostToolUse logging) that fire deterministically on every tool call. A system prompt cannot guarantee that every call is audited.
- **C:** Regulatory compliance requires deterministic enforcement. A prompt-based rule is probabilistic and cannot satisfy a hard compliance guarantee — a PreToolUse hook that blocks execution based on a computed score is the correct choice here.
- **D:** Irreversible consequences (email, payments) demand deterministic prevention. A prompt instruction is insufficient because the model may still trigger the action under certain conditions.

</details>

---

#### C47

*TS 1.4 · Workflow enforcement & handoff*

**Situation:** An agent is being built to assist customer support representatives. It has access to five tools: `lookup_order`, `issue_refund`, `escalate_to_human`, `send_email`, and `update_shipping_address`. A security review flags that the model should never be able to issue a refund without an explicit human confirmation step.

Which tool distribution and `tool_choice` design best enforces the human-in-the-loop requirement for refunds?

- **A)** Set `tool_choice` to `{"type": "tool", "name": "issue_refund"}` so the model explicitly calls the refund tool when needed
- **B)** Keep all five tools on the agent and add a system prompt instruction that says 'always confirm before refunding'
- **C)** Remove `issue_refund` from the agent's tool list and route all refund requests through a separate confirmation workflow before calling a restricted refund agent
- **D)** Add a `confirm_action` boolean parameter to `issue_refund` and instruct the agent to always set it to `false` first

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Removing a high-risk tool from an agent's tool list is the strongest architectural control. System prompt instructions can be overridden by adversarial inputs or model drift. The correct pattern per Anthropic's safety guidance is to restrict access at the tool provisioning level: the agent physically cannot call `issue_refund` because it is not in its tool list. A separate, human-gated workflow handles the confirmation before the restricted refund agent receives the call.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents#security-and-trust
- https://docs.anthropic.com/en/docs/build-with-claude/agents#human-in-the-loop

**Why the others are wrong:**

- **A:** Forcing `tool_choice` to `issue_refund` would cause the model to call the refund tool on every turn, which is the opposite of the desired behavior and would create uncontrolled refund issuance.
- **B:** System prompt instructions are a soft control. Prompt injection attacks or sufficiently unusual inputs can cause the model to bypass them. This approach does not provide a hard architectural guarantee.
- **D:** Adding a boolean parameter that the agent controls does not enforce human confirmation. The model can set `confirm_action: true` without any human involvement, providing only an illusion of safety.

</details>

---

#### C48

*TS 1.4 · Workflow enforcement & handoff*

After deploying the agent, logs reveal that 15% of refunds are processed without customer identity verification. The system prompt clearly states: "Always verify customer identity using `get_customer` before processing any refund." What change provides the strongest reliability guarantee?

- **A)** Implement a `PreToolUse` hook that blocks `process_refund` unless a verified customer ID from `get_customer` exists in the session state.
- **B)** Add a validation layer that checks tool call sequences after each turn and sends a corrective prompt if `get_customer` was skipped.
- **C)** Rewrite the system prompt with stronger language: "CRITICAL: You MUST call get_customer before process_refund. Failure to do so is a policy violation."
- **D)** Add 5 few-shot examples showing the agent calling `get_customer` before every refund, covering various customer request patterns.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Hooks provide 100% deterministic enforcement.

**Why the others are wrong:**

- **B:** Post-hoc validation catches errors after the fact but doesn't prevent them.
- **C:** Prompt-based approaches have a non-zero failure rate — 15% failure is proof.
- **D:** Prompt-based approaches have a non-zero failure rate — 15% failure is proof.

</details>

---

#### C49

*TS 1.4 · Workflow enforcement & handoff*

A customer writes: "I need to return order #8842 because the screen is cracked, AND I want to dispute the charge on order #7791 because it was unauthorized." How should the agent handle this?

- **A)** Ask the customer which issue they'd like to address first, then handle them one at a time across separate turns.
- **B)** Decompose the message into two distinct issues, investigate each in parallel using shared customer context, then synthesize a unified response.
- **C)** Escalate immediately because multi-issue requests exceed the agent's complexity threshold.
- **D)** Process both issues sequentially in the same turn, completing the return before starting the billing dispute.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Multi-concern requests should be decomposed into distinct items, investigated in parallel using shared context, then synthesized.

**Why the others are wrong:**

- **A:** Asking adds unnecessary friction.
- **C:** Escalating contradicts the 80% resolution target.
- **D:** Sequential is slower than needed.

</details>

---

#### C50

*TS 1.4 · Workflow enforcement & handoff*

Your agent needs to: (1) verify customer identity, (2) look up order details, (3) check return eligibility, and (4) process the refund. Steps must happen in this exact order for compliance. Currently, steps are listed in the system prompt instructions. What architecture ensures 100% compliance with this sequence?

- **A)** Implement programmatic prerequisite gates that block each downstream tool call until its prerequisite has completed successfully.
- **B)** Use `tool_choice` forced selection to call tools in the correct order across sequential turns.
- **C)** Number the steps clearly in the system prompt and add "CRITICAL: Follow this exact sequence" at the top.
- **D)** Use few-shot examples demonstrating the correct step sequence across multiple scenarios.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Critical compliance sequences require programmatic enforcement — prerequisite gates block downstream tools until prerequisites complete.

**Why the others are wrong:**

- **B:** `tool_choice` only controls the first tool per turn, not multi-step sequences.
- **C:** Prompt instructions have non-zero failure rates.
- **D:** Prompt instructions have non-zero failure rates.

</details>

---

#### C51

*TS 1.4 · Workflow enforcement & handoff*

When the agent must escalate to a human, the human agent will handle the case in a separate system without access to the AI conversation transcript. What information must the escalation include?

- **A)** A link to the full AI conversation transcript so the human agent can review the discussion.
- **B)** A structured handoff summary including customer ID, order details, root cause analysis, refund amount, and recommended action.
- **C)** The customer's last message and the AI agent's last response.
- **D)** Only the customer ID — the human agent can look up everything else in the CRM.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Human agents without transcript access need a self-contained structured handoff: customer ID, root cause, amounts, recommended action. Last message (C) and ID only (D) are insufficient.

**Why the others are wrong:**

- **A:** Transcript links assume access.

</details>

---

#### C52

*TS 1.4 · Workflow enforcement & handoff*

Your agent frequently recommends out-of-stock items because it searches products before checking inventory. Customers get frustrated when their selected items are unavailable. What's the fix?

- **A)** Restructure the tool flow: use a `search_and_check_inventory` combined tool, or enforce inventory checks via `tool_choice` sequencing before any recommendations are presented.
- **B)** Display inventory status prominently in the UI so customers can see availability themselves.
- **C)** Add "Check inventory before recommending" to the prompt — this will ensure the agent remembers.
- **D)** Only recommend items that have been in stock for the past 30 days based on historical data.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Restructure tool flow to enforce inventory checks before recommendations.

**Why the others are wrong:**

- **B:** UI display doesn't prevent the error.
- **C:** Prompt instructions are probabilistic.
- **D:** Historical data doesn't reflect current stock.

</details>

---

#### C53

*TS 1.4 · Workflow enforcement & handoff*

Your agent frequently recommends reordering items that are already on order because it checks inventory before checking pending purchase orders. What's the fix?

- **A)** Restructure the tool flow: use a combined `check_inventory_and_pending` tool, or enforce pending order checks via `tool_choice` sequencing before any reorder recommendations.
- **B)** Display pending orders prominently in the UI so operators can see them.
- **C)** Add "Check pending orders before recommending" to the prompt.
- **D)** Only recommend reorders for items that have been out of stock for 7+ days.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Restructure tool flow to enforce pending order checks before recommendations.

**Why the others are wrong:**

- **C:** Prompt instructions are probabilistic.

</details>

---

#### C54

*TS 1.5 · Agent SDK hooks*

**Situation:** A developer is building a multi-step document approval workflow. At the file-write step, they want to guarantee that no file is ever written outside of a designated `/approved` directory, regardless of what the model generates as arguments.

Which enforcement mechanism provides the strongest guarantee that the path restriction will always be respected?

- **A)** Add a sentence to the system prompt instructing the model to only write files in `/approved`.
- **B)** Ask the model to confirm the file path before each write by outputting it as plain text.
- **C)** Implement a PreToolUse hook that inspects the `path` argument and aborts the tool call if the path does not start with `/approved`.
- **D)** Use a PostToolUse hook to delete any file written outside `/approved` after the fact.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

A PreToolUse hook runs programmatic code before the tool executes, giving it the ability to inspect arguments and abort execution deterministically. This is the only option that prevents the out-of-bounds write from ever happening. Prompt instructions are probabilistic — the model may deviate. Asking for confirmation relies on another model turn. A PostToolUse hook is reactive and cannot undo all side-effects of a write.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/claude-code/hooks

**Why the others are wrong:**

- **A:** System prompt instructions are probabilistic guidance: the model is likely but not guaranteed to follow them, especially under adversarial input or prompt injection. Code-level enforcement is required for hard security boundaries.
- **B:** Requesting a confirmation in the conversation relies on another model inference step, which is again probabilistic and adds latency without providing a true enforcement guarantee.
- **D:** A PostToolUse hook fires after the tool has already executed. The file has already been written (and potentially read by other processes), so deletion is reactive cleanup, not prevention.

</details>

---

#### C55

*TS 1.5 · Agent SDK hooks*

**Situation:** A team is documenting their multi-agent pipeline and needs to classify each control mechanism. They have: (1) a JSON schema that rejects tool calls with missing required fields, (2) a system prompt telling the model to always summarise before proceeding, and (3) a PostToolUse hook that logs every tool result.

Which of the three mechanisms is deterministic rather than probabilistic?

- **A)** Only (2) — the system prompt.
- **B)** Only (1) and (3) — the schema validation and the logging hook.
- **C)** Only (2) and (3) — the system prompt and the logging hook.
- **D)** All three are deterministic because they all execute on every turn.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

JSON schema validation (1) and a PostToolUse hook (3) are both programmatic code that executes deterministically on every applicable event — they do not depend on the model's inference. The system prompt (2) is probabilistic guidance: the model will usually follow it but may deviate, so it cannot be classified as deterministic enforcement.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/claude-code/hooks

**Why the others are wrong:**

- **A:** The system prompt is the probabilistic mechanism, not a deterministic one. Model inference introduces variance regardless of how clearly the instruction is phrased.
- **C:** The system prompt is probabilistic, not deterministic. Including it with the logging hook misclassifies the enforcement model.
- **D:** Executing on every turn does not make something deterministic. The system prompt runs on every turn but still produces probabilistic outputs because the model may choose not to follow it.

</details>

---

#### C56

*TS 1.5 · Agent SDK hooks*

**Situation:** An agentic coding assistant uses a `run_tests` tool. A PostToolUse hook currently logs every test result to a monitoring service. The team now wants to add logic so that if the test run fails, the hook automatically injects a structured failure summary into the tool result before the model sees it, prompting the model to fix the issues.

What is the correct description of how a PostToolUse hook achieves this result injection?

- **A)** The hook sends a separate API request to Claude with the failure summary to get a correction plan, then discards the original tool result.
- **B)** The hook appends a new user message to the conversation with the failure summary, bypassing the tool result entirely.
- **C)** The hook writes the failure summary to a file and updates the system prompt to reference the file path.
- **D)** The hook modifies or replaces the tool result content returned to the model, so the model's next turn sees the enriched result as if the tool itself produced it.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

PostToolUse hooks can intercept the tool result before it is injected into the conversation and modify or replace its content. The model then receives the enriched result as part of the normal tool result flow. This is the canonical pattern for PostToolUse result transformation: augment or replace the raw tool output so the model's next reasoning step operates on richer, structured data.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/claude-code/hooks

**Why the others are wrong:**

- **A:** Sending a separate API request from the hook creates a side-channel inference loop that bypasses the main workflow and produces results the orchestrator cannot track. This pattern also incurs unnecessary latency and cost.
- **B:** Appending a separate user message would break the expected tool-result message structure and could confuse the model about which `tool_use_id` the result belongs to. Tool results must be returned as `tool_result` content blocks, not free-form user messages.
- **C:** Updating the system prompt mid-workflow requires a new API call and the model must re-read the file in a subsequent tool call. This is indirect and unreliable compared to direct result injection.

</details>

---

#### C57

*TS 1.5 · Agent SDK hooks*

**Situation:** A developer is adding observability to an agent built with the Claude Agent SDK. They want every tool call to be written to an audit log before the tool executes, without changing the tool's behavior.

Which hook type should the developer implement to achieve this?

- **A)** PreToolUse hook, because it fires before tool execution and can record the intended call without modifying behavior.
- **B)** A custom middleware layer wrapping the model client, because the SDK does not support hooks.
- **C)** PostToolUse hook, because it fires after the result is available and provides the full picture.
- **D)** Stop hook, because it fires at session end and can flush all accumulated calls at once.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

PreToolUse hooks execute before the tool runs, making them the correct place to log the incoming tool name and parameters for an immutable audit trail. They can be configured to pass through without modifying parameters or aborting the call.

References:
- https://docs.anthropic.com/en/docs/claude-code/hooks

**Why the others are wrong:**

- **B:** The Claude Agent SDK provides a first-class hooks system; wrapping the model client is an unnecessary workaround that would break on SDK updates and duplicate telemetry infrastructure.
- **C:** A PostToolUse hook fires after execution and would miss logging calls that were aborted mid-flight or that raised exceptions, leaving gaps in the audit trail.
- **D:** Stop hooks fire when the agent session ends, not per tool call. Deferring logging to session end would lose per-call ordering and fail to capture calls from sessions that crash.

</details>

---

#### C58

*TS 1.5 · Agent SDK hooks*

**Situation:** An agent calls a database query tool. Before the query executes, the platform team wants to reject any call whose `sql` parameter contains a `DROP` statement to prevent accidental data loss.

Which capability of a PreToolUse hook enables this enforcement?

- **A)** The hook can increase the model's temperature to make it less likely to generate DROP statements.
- **B)** The hook can rewrite the SQL after execution to undo DROP effects.
- **C)** The hook can replace the tool's implementation with a no-op function.
- **D)** The hook can inspect parameters and return an abort signal, preventing the tool from executing.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

PreToolUse hooks receive the tool name and parameters before execution. They can validate those parameters and return an abort result, which prevents the SDK from invoking the tool at all and returns an error to the model instead.

References:
- https://docs.anthropic.com/en/docs/claude-code/hooks

**Why the others are wrong:**

- **A:** Hooks operate on tool call parameters, not on model inference settings. Temperature is set at request time and has no effect on which tool arguments the model has already committed to.
- **B:** A PreToolUse hook runs before execution, so there are no effects to undo. Post-execution SQL rollback would require a different mechanism entirely.
- **C:** Hooks intercept calls at the SDK layer; they do not replace or monkey-patch the underlying tool implementation.

</details>

---

#### C59

*TS 1.5 · Agent SDK hooks*

**Situation:** A compliance team requires that all tool outputs containing email addresses be redacted before the model sees them, so PII never enters the conversation context.

Which hook type is the correct place to implement this PII redaction?

- **A)** PostToolUse hook, because it receives the tool result and can transform it before the model processes it.
- **B)** Stop hook, because it has access to the full session transcript and can redact it at the end.
- **C)** No hook is needed; the system prompt should instruct the model to ignore email addresses.
- **D)** PreToolUse hook, because it can modify the tool parameters to remove email addresses before execution.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

PostToolUse hooks fire after a tool returns its result and before the SDK passes that result back into the model context. This is the correct interception point to scan the output and replace email addresses with redacted placeholders.

References:
- https://docs.anthropic.com/en/docs/claude-code/hooks

**Why the others are wrong:**

- **B:** Stop hooks run at session end. By that point the unredacted email addresses would already have been fed into the model context, defeating the compliance goal.
- **C:** Instructing the model via prompts is probabilistic; the model may still reference, repeat, or reason over PII in its internal context. Deterministic redaction requires a hook that transforms the data before it reaches the model.
- **D:** A PreToolUse hook runs before execution and only has access to input parameters, not the tool output. Email addresses that appear in results cannot be intercepted there.

</details>

---

#### C60

*TS 1.5 · Agent SDK hooks*

**Situation:** A team debates whether to enforce a data-format policy (all tool outputs must be valid JSON) via a system prompt instruction or via a PostToolUse hook.

What is the primary advantage of enforcing this policy with a PostToolUse hook rather than a system prompt?

- **A)** Hooks can access the internet to validate JSON schemas, whereas system prompts cannot.
- **B)** Hooks execute faster than prompt instructions because they bypass the model's tokeniser.
- **C)** System prompts increase cost per token, so hooks are always cheaper.
- **D)** Hooks provide deterministic enforcement; the transformation always runs regardless of how the model interprets the instruction.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

System prompt instructions are interpreted probabilistically by the model and may be ignored or misapplied. A PostToolUse hook is deterministic code: it runs unconditionally on every tool result, guaranteeing the JSON normalization happens on every call regardless of model behavior.

References:
- https://docs.anthropic.com/en/docs/claude-code/hooks

**Why the others are wrong:**

- **A:** Neither hooks nor system prompts have inherent network access. A hook can call external services only if the developer explicitly codes that call.
- **B:** Hooks are synchronous Python/TypeScript code that runs in the SDK runtime. They do not bypass tokenisation; they simply run outside the model inference step.
- **C:** Hook execution cost is a compute concern in the application layer, not an API token cost. System prompt length does affect token billing, but that is a separate tradeoff unrelated to determinism.

</details>

---

#### C61

*TS 1.5 · Agent SDK hooks*

**Situation:** An agent processes medical records. A PostToolUse hook is designed to redact HIPAA-protected fields from tool outputs. A code reviewer notices that the hook raises a `KeyError` when a tool returns a result that lacks the expected `patient` key, and the developer has not added error handling. In production, an unhandled `KeyError` causes the session to crash rather than continuing with redacted output.

Which refactoring strategy correctly addresses both the crash risk and the compliance requirement?

- **A)** Add input validation at the start of the hook; if the result lacks the `patient` key, return an empty dict so the model receives no output from this tool.
- **B)** Add a try/except around the entire hook body; on `KeyError`, return the original unmodified result so the agent continues without crashing.
- **C)** Add a try/except around only the field access; on `KeyError`, skip redacting that field and return the partially redacted result.
- **D)** Add a try/except around the entire hook body; on `KeyError`, abort the tool call with an error so the model is notified and the unredacted result is never passed to the model.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

When a PostToolUse hook cannot guarantee full redaction (e.g., due to an unexpected result shape), passing any unredacted data to the model violates the compliance requirement. The correct behavior is to catch the error and abort, preventing the result from reaching the model and surfacing a structured error for investigation. Aborting from a PostToolUse hook is supported and returns an error result to the model.

References:
- https://docs.anthropic.com/en/docs/claude-code/hooks

**Why the others are wrong:**

- **A:** Returning an empty dict avoids the crash and prevents PHI exposure, but silently discarding tool output could cause the agent to make incorrect decisions without understanding why the data is missing. An explicit abort with an error message is more correct and auditable.
- **B:** Returning the original unmodified result on error would pass unredacted HIPAA-protected data to the model, which violates the compliance requirement even though the crash is avoided.
- **C:** Partial redaction is insufficient for HIPAA compliance. If any protected field is skipped because of an exception, the result contains PHI that should not be in the model's context.

</details>

---

#### C62

*TS 1.5 · Agent SDK hooks*

**Situation:** A developer registers a PreToolUse hook that validates tool parameters against a JSON Schema. During a test run, the hook raises an unhandled Python exception because an unexpected parameter shape causes the schema validator to crash.

What is the most appropriate error handling strategy for this hook to maintain agent reliability?

- **A)** Log the exception and replace all parameters with empty strings to prevent the tool from receiving unexpected input.
- **B)** Wrap the validator call in a try/except block; on exception, abort the tool call and return a structured error message to the model describing the validation failure.
- **C)** Wrap the validator call in a try/except block; on exception, silently pass through and allow the tool to execute with the original parameters.
- **D)** Let the exception propagate so the agent framework's top-level handler can decide whether to retry.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Hooks should catch exceptions internally and return a well-formed abort result with a descriptive error when validation fails unexpectedly. This keeps the agent loop intact, surfaces a meaningful error to the model (which can then decide to retry with corrected parameters), and avoids crashing the entire session.

References:
- https://docs.anthropic.com/en/docs/claude-code/hooks

**Why the others are wrong:**

- **A:** Replacing parameters with empty strings would cause the tool to execute in an undefined or broken state, potentially producing wrong results or raising further errors downstream.
- **C:** Silently passing through on a validation crash defeats the purpose of the hook. If the validator crashes on an unexpected shape, that shape may be dangerous or malformed, and allowing execution hides the bug.
- **D:** Unhandled exceptions propagating out of a hook will typically crash the agent session, not trigger a clean retry. The top-level handler has no tool-level context to perform an intelligent recovery.

</details>

---

#### C63

*TS 1.5 · Agent SDK hooks*

**Situation:** An agent calls an external CRM API tool that returns customer records in a legacy XML format. The rest of the agent pipeline expects all tool outputs to be JSON objects. A PostToolUse hook has been implemented to parse the XML and return a JSON equivalent.

Which statement best describes what the hook must return for the normalization to take effect?

- **A)** The hook must return a replacement result object; the SDK will substitute this for the original tool output before passing it to the model.
- **B)** The hook must re-invoke the tool with the JSON as a new parameter so the tool can return the correct format itself.
- **C)** The hook must return the JSON string and also call `tool.set_output()` to update the tool's internal state.
- **D)** The hook must write the JSON to a shared memory store; the model reads from that store automatically on the next turn.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

A PostToolUse hook that returns a replacement result causes the SDK to use that value instead of the original tool output when constructing the tool_result message for the model. This is the designed interception point for output normalization.

References:
- https://docs.anthropic.com/en/docs/claude-code/hooks

**Why the others are wrong:**

- **B:** Re-invoking the tool would start a new tool execution cycle, potentially causing infinite loops or duplicate side effects. Normalization should happen in the hook layer, not by re-running the original tool.
- **C:** There is no `tool.set_output()` API in the Claude Agent SDK. Hook result interception is achieved purely by the hook's return value.
- **D:** The model does not poll a shared memory store between turns. All data flows through the messages API; the hook return value is the correct and only mechanism to substitute output.

</details>

---

#### C64

*TS 1.5 · Agent SDK hooks*

**Situation:** A security-conscious team wants to prevent the model from calling a `delete_record` tool unless the current session has an `admin` role set in the session context. They are choosing between encoding this rule in the system prompt versus a PreToolUse hook.

Why is a PreToolUse hook the architecturally superior choice for this access-control requirement?

- **A)** Hooks are faster to evaluate than the model parsing a system prompt instruction.
- **B)** A hook can read structured session context and abort the call deterministically, whereas a system prompt instruction relies on the model correctly interpreting and obeying it every time.
- **C)** System prompts cannot reference session context variables, so hooks are the only option.
- **D)** Hooks are versioned alongside the tool definitions, making them easier to audit than system prompts.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Access control is a security invariant that must hold unconditionally. A PreToolUse hook is code: it reads the session context, checks the role, and aborts if the condition is not met — every single time, without exception. A system prompt instruction is interpreted by a language model, which could theoretically be manipulated via prompt injection or produce an incorrect judgment.

References:
- https://docs.anthropic.com/en/docs/claude-code/hooks

**Why the others are wrong:**

- **A:** Latency is not the primary concern for access control. Correctness and determinism are. Even if hooks were slower, the security guarantee would still make them preferable.
- **C:** System prompts can reference session metadata if it is injected as text, but this makes the check probabilistic. The architectural issue is reliability, not the technical inability to reference variables.
- **D:** While co-locating hooks and tool definitions can aid auditability, this is a secondary benefit. The core reason to prefer hooks for access control is deterministic enforcement, not version management.

</details>

---

#### C65

*TS 1.5 · Agent SDK hooks*

Your `PostToolUse` hook receives order data from the `lookup_order` tool. The tool returns timestamps as Unix integers, status as numeric codes (1=pending, 2=shipped, 3=delivered), and amounts without currency indicators. What should the hook do?

- **A)** Convert the data into a natural language summary paragraph to make it easier for the model to understand.
- **B)** Store the raw data in a side channel and provide only the fields the model needs for the current query.
- **C)** Pass the raw data through unchanged — the model can interpret numeric codes and Unix timestamps on its own.
- **D)** Normalize timestamps to ISO 8601, map status codes to human-readable strings, and add the currency symbol from the customer's locale.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

PostToolUse hooks normalize heterogeneous data formats (timestamps, status codes, currencies) before the model processes them.

**Why the others are wrong:**

- **A:** Natural language loses structure.
- **B:** Side channels aren't how hooks work.
- **C:** Raw data forces the model to interpret ambiguous formats.

</details>

---

#### C66

*TS 1.5 · Agent SDK hooks*

You want to add the `process_refund` tool to the agent but ensure refunds above $500 are automatically routed to a human approver. The compliance team requires 100% enforcement. Which approach is correct?

- **A)** Add the policy to the system prompt: "For refunds above $500, always use `escalate_to_human` instead of `process_refund`."
- **B)** Implement a `PreToolUse` hook that intercepts `process_refund` calls, checks the amount, and redirects amounts over $500 to human escalation.
- **C)** Configure two versions of the tool: `process_refund_small` (≤$500) and `process_refund_large` (>$500, requires human approval).
- **D)** Use few-shot examples showing the agent escalating high-value refunds while processing low-value ones autonomously.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

100% enforcement requires a PreToolUse hook. Prompts (A) and examples (D) are probabilistic.

**Why the others are wrong:**

- **C:** Two tool variants don't enforce the policy — the agent could still call the wrong one.

</details>

---

#### C67

*TS 1.5 · Agent SDK hooks*

Your agent now uses hooks to enforce a $500 refund limit. A customer is owed $1,200 for three defective items in a single order ($400, $350, $450). The agent attempts to process three separate refunds of $400, $350, and $450 — all below the $500 limit. Is this a problem?

- **A)** No — each individual refund is below the threshold, so the hooks correctly allow them.
- **B)** No — the $500 limit applies per-transaction only; cumulative tracking is the responsibility of the business system.
- **C)** Yes — the agent should process a single $1,200 refund, which the hook would correctly block.
- **D)** Yes — the hook should track cumulative refund amounts per order/customer, not just per-transaction amounts. $1,200 total for one order likely exceeds the policy intent even if individual items don't.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Cumulative refund tracking is needed. Individual-transaction hooks miss split-refund circumvention. The $500 policy likely applies to total exposure, not per-item. The hook should track cumulative amounts per order or customer session.

</details>

---

#### C68

*TS 1.5 · Agent SDK hooks*

Your agent uses hooks to enforce that only licensed clinicians can book certain appointment types. A patient requests a "specialist consultation" which requires clinician approval. The agent attempts to book it directly three times — each attempt is blocked by the hook. Is this a problem?

- **A)** No — the hook should allow the booking after 3 attempts as a fallback.
- **B)** Yes — the agent should bypass the hook for patient-requested appointments.
- **C)** Yes — after the first hook rejection, the agent should recognize the restriction and either escalate to a clinician or explain the approval requirement to the patient. Repeated failed attempts waste resources and frustrate patients.
- **D)** No — the agent is correctly attempting to fulfill the patient's request.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

After the first hook rejection, the agent should recognize the restriction and either escalate or explain the requirement. Repeated failed attempts waste resources and frustrate patients.

</details>

---

#### C69

*TS 1.5 · Agent SDK hooks*

Your agent uses hooks to enforce that only certified educators can access certain advanced content templates. An uncertified user attempts to access an "AP Biology Lab" template three times — each attempt is blocked by the hook. Is this a problem?

- **A)** No — the hook should allow access after 3 attempts as a fallback.
- **B)** No — the agent is correctly attempting to fulfill the user's request.
- **C)** Yes — the agent should bypass the hook for educator-requested content.
- **D)** Yes — after the first hook rejection, the agent should recognize the restriction and either explain the certification requirement or suggest alternative accessible templates. Repeated failed attempts waste resources and frustrate users.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

After the first hook rejection, the agent should recognize the restriction and either explain or suggest alternatives. Repeated failed attempts waste resources.

</details>

---

#### C70

*TS 1.6 · Task decomposition*

**Situation:** A developer needs to build a pipeline that always extracts data from a file, then validates it, then formats the output as JSON. The steps are fixed and the output of each step feeds directly into the next.

Which decomposition strategy is most appropriate for this workflow?

- **A)** Dynamic decomposition, because the model should decide each step at runtime
- **B)** Prompt chaining, because the steps are predetermined and execute in a fixed sequence
- **C)** Two-phase decomposition, because results must be aggregated at the end
- **D)** Parallel per-item analysis, because each step is independent

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Prompt chaining is the correct choice when the workflow has a fixed, predetermined sequence of steps where each step's output feeds into the next. The pipeline here — extract, validate, format — has no branching or dynamic routing, making a static chain the simplest and most predictable solution.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/build-with-claude-overview
- https://docs.anthropic.com/en/docs/about-claude/models/overview

**Why the others are wrong:**

- **A:** Dynamic decomposition introduces model-driven routing overhead that is unnecessary when the steps are already known. It also reduces predictability and increases token cost.
- **C:** Two-phase decomposition is designed for a fan-out of independent items followed by a single aggregation step, not for a linear multi-step pipeline on a single item.
- **D:** Parallel per-item analysis applies when many independent items must each be processed in isolation (e.g., 50 separate files). Here there is only one item moving through sequential stages.

</details>

---

#### C71

*TS 1.6 · Task decomposition*

**Situation:** A team is reviewing a design where a single agent is asked to analyze an entire 10,000-file codebase in one prompt. The agent frequently loses track of earlier files and produces inconsistent results.

What fundamental decomposition problem does this design illustrate?

- **A)** Granularity that is too fine, causing excessive orchestration overhead
- **B)** Missing a two-phase aggregation step after parallel analysis
- **C)** Granularity that is too coarse, causing context overload
- **D)** Using dynamic decomposition when a static chain would suffice

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Feeding an entire 10,000-file codebase into a single prompt is an example of decomposition granularity that is too coarse. The model's context window is overwhelmed, degrading recall and consistency. The correct approach is to break the work into smaller, manageable units.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/build-with-claude-overview

**Why the others are wrong:**

- **A:** Too-fine granularity means splitting work into so many tiny tasks that scheduling and token overhead dominates. The described problem is the opposite — everything is crammed into one enormous task.
- **B:** While a two-phase pattern might ultimately be part of the solution, the root problem being illustrated is context overload from overly coarse decomposition, not a missing aggregation phase.
- **D:** The choice between static and dynamic decomposition concerns routing logic, not the size of individual tasks. This problem is specifically about task size overwhelming the context window.

</details>

---

#### C72

*TS 1.6 · Task decomposition*

**Situation:** An architect is designing a system to summarize 200 independent customer support tickets. Each ticket has no dependency on any other ticket, and all summaries must eventually be combined into a single executive report.

Which pattern most directly fits this workload?

- **A)** Cross-file integration using a serial, aggregated approach for all 200 tickets
- **B)** Dynamic decomposition where the model selects which ticket to process next
- **C)** Prompt chaining with all 200 tickets passed sequentially to the same context
- **D)** Two-phase pattern: Phase 1 parallel per-ticket summarization, Phase 2 single integration

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

The two-phase pattern is purpose-built for this shape of work: many independent items that can be processed in parallel (Phase 1), followed by a single step that integrates all outputs into a final artifact (Phase 2). It maximizes throughput while producing a unified result.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/build-with-claude-overview

**Why the others are wrong:**

- **A:** Serial aggregation processes items one by one and accumulates context, which is slower and risks context overload. The tickets are independent and can be parallelized instead.
- **B:** Dynamic decomposition adds model-driven routing complexity that is not needed here. The structure of the work — all items independent, one final aggregation — is fully known in advance.
- **C:** Passing all 200 tickets sequentially to the same context would exhaust the context window and make the model's attention degrade over the sequence. It also prevents parallelism.

</details>

---

#### C73

*TS 1.6 · Task decomposition*

**Situation:** A workflow automation tool is being built to handle open-ended research tasks. At each step, the model must evaluate what it has learned so far and decide whether to search for more information, synthesize findings, or request human clarification.

Which decomposition strategy is most appropriate here?

- **A)** Two-phase decomposition, because all research items need a final aggregation
- **B)** Prompt chaining, because the steps are well-defined in advance
- **C)** Parallel per-item analysis, because the research topics are independent
- **D)** Dynamic decomposition, because the model determines the next step based on intermediate results

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Dynamic decomposition lets the model inspect intermediate results and decide the next action at runtime — searching, synthesizing, or escalating. This is the correct pattern when the workflow cannot be fully specified in advance and branches depend on what the model discovers.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/build-with-claude-overview

**Why the others are wrong:**

- **A:** Two-phase decomposition assumes a known set of independent items to fan out over, then aggregate. An exploratory research task has unknown scope and adaptive branching, which two-phase does not model well.
- **B:** Prompt chaining requires that every step be predetermined. Open-ended research tasks inherently have unknown branching points, making a fixed chain insufficient.
- **C:** Parallel per-item analysis applies to batches of isolated items processed independently. An open-ended research task is a single evolving thread, not a batch.

</details>

---

#### C74

*TS 1.6 · Task decomposition*

**Situation:** A software architect is designing a multi-repository refactoring system. The system must: (1) identify all files that use a deprecated API across 20 repositories, (2) rewrite each file independently to use the new API, (3) detect and resolve cross-repository type conflicts introduced by the rewrites, and (4) generate a migration report. The architect is debating whether to use per-file parallel agents or cross-file serial aggregation for each phase.

Which decomposition design correctly maps each phase to the appropriate processing strategy?

- **A)** Phase 1: parallel per-file scan; Phase 2: serial rewrite of all files in sequence; Phase 3: parallel per-repo conflict agents; Phase 4: parallel per-repo report agents
- **B)** All four phases should use a single serial agent to maintain a consistent global view of the entire codebase throughout
- **C)** Phase 1: serial scan across all repos; Phase 2: parallel per-file rewrite agents (isolated); Phase 3: serial cross-repo conflict resolution (aggregated); Phase 4: serial report generation from aggregated Phase 3 output
- **D)** All four phases should use parallel per-file agents to maximize throughput across all repositories

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

The correct mapping uses the isolation and dependency structure of each phase to select the strategy. Phase 1 needs a unified scan (serial or coordinated) to build a complete file list. Phase 2 rewrites are per-file independent operations, ideal for parallel isolated agents. Phase 3 requires seeing all rewritten files together to detect cross-repository conflicts, requiring serial aggregated processing. Phase 4 generates a report from the already-aggregated Phase 3 output, which is inherently serial.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/build-with-claude-overview

**Why the others are wrong:**

- **A:** Serial rewriting in Phase 2 is unnecessarily slow — the rewrites are per-file independent and should be parallelized. Parallel per-repo conflict agents in Phase 3 still fail to catch conflicts that span repository boundaries.
- **B:** A single serial agent across all phases would create a massive context window burden, lose the parallelism benefit in Phase 2, and likely exceed context limits for large codebases. It also cannot scale to 20 repositories.
- **D:** Parallel per-file agents for Phase 3 cannot detect cross-repository type conflicts because each agent sees only its own file. Cross-file dependency resolution is definitionally a serial, aggregated operation requiring global context.

</details>

---

#### C75

*TS 1.6 · Task decomposition*

**Situation:** A research assistant system uses dynamic decomposition to conduct literature reviews. During a review, the model has accumulated 18,000 tokens of intermediate findings in its context. An engineer notices the model begins making contradictory statements and losing track of earlier conclusions. The engineer considers three fixes: (A) summarize and compress intermediate findings periodically, (B) switch to a static prompt chain, (C) increase the model's context window by upgrading to a larger model.

Which fix, or combination of fixes, most directly addresses the root cause of the problem while preserving the dynamic routing capability needed for open-ended research?

- **A)** Fix A combined with Fix C — summarization manages context growth and a larger context window provides headroom, together preserving dynamic decomposition for cases where compression is insufficient
- **B)** Fix B alone — static prompt chaining eliminates the context growth problem by design
- **C)** Fix C alone — a larger context window resolves the overload without changing the architecture
- **D)** Fix A alone — periodic summarization and context compression directly addresses the context overload root cause while preserving dynamic decomposition

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

The root cause is context overload as intermediate findings accumulate. Fix A — periodic summarization and compression of the working context — directly addresses this by keeping the active context within manageable bounds without changing the decomposition strategy. Dynamic routing is preserved because the model continues to decide its next step; it simply operates on a compressed representation of prior findings rather than the raw accumulation.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/build-with-claude-overview

**Why the others are wrong:**

- **A:** Summarization plus a larger context window is belt-and-suspenders protection, but Fix A alone addresses the root cause and Fix C adds cost without solving the fundamental problem. The question asks for the fix that most directly addresses the root cause while preserving dynamic decomposition.
- **B:** Switching to a static chain eliminates the dynamic routing capability that is explicitly required for open-ended research. This solves context growth by changing the problem specification, not by addressing the architectural challenge.
- **C:** Upgrading to a larger context window defers the problem rather than solving it. For sufficiently long research tasks, any context window will eventually be exceeded. It also adds cost without addressing the architectural root cause.

</details>

---

#### C76

*TS 1.6 · Task decomposition*

**Situation:** A security scanning system processes 500 source files. An architect proposes spawning one agent per line of code to maximize parallelism. The system is later observed to spend 95% of its time on scheduling and inter-agent communication rather than actual analysis.

Which decomposition principle does this scenario violate, and what is the correct corrective action?

- **A)** Too-fine granularity; the fix is to increase task size so each agent analyzes a whole file or logical module
- **B)** Too-coarse granularity; the fix is to switch to dynamic decomposition so the model batches lines automatically
- **C)** Missing two-phase pattern; the fix is to add a Phase 2 aggregation step after per-line agents complete
- **D)** Incorrect use of serial processing; the fix is to process all lines in a single sequential chain

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Decomposing at line-of-code granularity is far too fine. The overhead of spawning, scheduling, and coordinating hundreds of thousands of micro-agents dwarfs the actual analysis work. The correct fix is to raise the granularity to the file or module level so each agent does a meaningful, self-contained unit of work.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/build-with-claude-overview

**Why the others are wrong:**

- **B:** The problem is too-fine granularity, not too-coarse. Switching to dynamic decomposition does not inherently change granularity; it only changes who decides the next step, and it would still incur massive overhead if the model chose line-level tasks.
- **C:** Adding a Phase 2 aggregation step is a valid architectural addition but does not address the root cause: the Phase 1 tasks are themselves too small, causing scheduling overhead to dominate.
- **D:** Processing all lines in a single sequential chain would eliminate parallelism and create a context-overload problem. It addresses neither the granularity issue nor the overhead.

</details>

---

#### C77

*TS 1.6 · Task decomposition*

**Situation:** An engineer is building a code migration tool. Phase 1 spawns one agent per file to translate syntax from Python 2 to Python 3. Phase 2 runs a single agent to review all translated files together and fix import conflicts that span multiple files.

Why is serial processing used in Phase 2 rather than another round of parallel agents?

- **A)** The model cannot process more than one file at a time in a single turn
- **B)** Phase 2 requires cross-file context to detect and resolve dependencies that span file boundaries, which parallel isolated agents cannot see
- **C)** Parallel agents in Phase 2 would violate the two-phase pattern's requirement for a single integration pass
- **D)** Serial processing is always cheaper than parallel processing in terms of API cost

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Cross-file integration tasks require a view of multiple files simultaneously to detect conflicts and dependencies that exist across file boundaries. Parallel isolated agents each see only their own file and cannot reason about inter-file relationships. Phase 2 must be serial (or at least aggregated) so the integration agent has the full picture.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/build-with-claude-overview

**Why the others are wrong:**

- **A:** A model can process multiple files in a single turn if they fit in the context window. The constraint is logical, not technical: inter-file dependency resolution requires seeing multiple files together.
- **C:** The two-phase pattern does not mandate any specific internal structure for Phase 2. Phase 2 could in principle use further parallelism if the sub-tasks were independent. The reason for serial integration here is the nature of the cross-file dependency problem.
- **D:** Cost is not the determining factor here. Serial processing in Phase 2 is chosen for correctness — the task requires global context — not for cost optimization.

</details>

---

#### C78

*TS 1.6 · Task decomposition*

**Situation:** A data engineering team must generate weekly reports for 1,000 regional stores. Each store's report is computed independently from that store's own sales data. After all reports are generated, a global summary comparing all regions is produced for the executive team.

An architect proposes the following: (1) send all 1,000 stores' data to a single agent to generate all reports at once, then (2) pass those reports to a second agent for the global summary. What is the primary flaw in this design?

- **A)** The design incorrectly uses two phases; a single phase would be more efficient
- **B)** Phase 1 processes all stores in a single agent context, causing context overload and defeating the purpose of decomposition
- **C)** Dynamic decomposition should be used in Phase 1 so the model chooses which stores to process
- **D)** The global summary in Phase 2 should also be parallelized across multiple summary agents

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

The flaw is that Phase 1 is not actually decomposed: all 1,000 stores are sent to one agent in one context. This causes context overload and negates the benefit of the two-phase pattern. Phase 1 should fan out to 1,000 parallel isolated agents, each handling one store, then Phase 2 aggregates the results.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/build-with-claude-overview

**Why the others are wrong:**

- **A:** Two phases are exactly the right structure for this workload. The problem is the internal implementation of Phase 1 (no fan-out), not the existence of two phases.
- **C:** Dynamic decomposition in Phase 1 adds unnecessary runtime model decision-making overhead. The set of stores to process is fully known in advance, making static parallel fan-out more efficient and predictable.
- **D:** The global summary inherently requires seeing all regional data together to make comparisons. Parallelizing Phase 2 across multiple agents would prevent each agent from having the full cross-region view needed for the executive summary.

</details>

---

#### C79

*TS 1.6 · Task decomposition*

**Situation:** Two architects debate task decomposition strategies. Architect A argues that static decomposition (prompt chaining) should be the default and dynamic decomposition used only as an exception. Architect B argues the opposite.

Which architect's position is better supported by engineering principles, and why?

- **A)** Both positions are equally valid because the choice depends entirely on the specific use case with no general default
- **B)** Architect B, because dynamic decomposition is more powerful and can always replicate what a static chain does
- **C)** Architect A, because static decomposition is more predictable, easier to test, and has lower token overhead; dynamic decomposition should be added only when the workflow cannot be fully specified in advance
- **D)** Architect B, because modern models have enough reasoning capability to always determine the optimal next step without a predefined sequence

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Static decomposition (prompt chaining) is the correct engineering default because it is deterministic, easier to test and debug, has lower latency and token cost, and produces more predictable outputs. Dynamic decomposition introduces model-driven branching that is harder to test exhaustively and consumes more tokens. Dynamic decomposition should be reserved for workflows that genuinely cannot be fully specified in advance.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/build-with-claude-overview

**Why the others are wrong:**

- **A:** While context matters, there is a principled default: prefer static over dynamic unless runtime adaptivity is genuinely required. This is not an arbitrary preference — it follows from testability, cost, and predictability considerations.
- **B:** While dynamic decomposition is more powerful in theory, power is not the same as appropriateness. Using a more powerful but less predictable mechanism when a simpler one suffices violates the principle of minimizing unnecessary complexity.
- **D:** Model reasoning capability is not the limiting factor. The issue is testability and predictability: even a perfectly capable model introduces non-determinism in routing decisions that complicates debugging and increases cost without benefit when the steps are known.

</details>

---

#### C80

*TS 1.7 · Session state, resumption & forking*

**Situation:** A developer is building a customer-support chatbot and wants users to be able to close their browser and return to the same conversation hours later. The conversation history must persist across browser sessions.

Which Claude SDK feature most directly enables persistent, resumable conversations that survive client disconnects?

- **A)** Named sessions that store conversation state server-side under a stable identifier
- **B)** Increasing the `max_tokens` parameter on each request
- **C)** Setting a high `temperature` value to improve response consistency
- **D)** Storing the system prompt in a separate file and reloading it on each request

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Named sessions attach a stable identifier to a conversation so the SDK can persist and reload full context server-side. This allows a user to disconnect and resume seamlessly without the client needing to manage or replay history.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/sessions

**Why the others are wrong:**

- **B:** `max_tokens` controls the length of a single response, not persistence across disconnections; it has no effect on whether a session can be resumed.
- **C:** `temperature` affects response randomness, not state persistence; a high temperature would actually reduce consistency rather than help resumption.
- **D:** Reloading a system prompt on each request does not restore conversation history; the model would have no memory of prior turns without the full message array.

</details>

---

#### C81

*TS 1.7 · Session state, resumption & forking*

**Situation:** A team has a long-running research session in Claude Code. They want to explore an alternative implementation approach without losing their current progress.

Which action creates an independent branch of the conversation so the original session remains intact?

- **A)** Open a second terminal window and start a new unnamed session
- **B)** Call `fork_session` to create a copy of the current session state
- **C)** Use `/compact` followed by `/reset` to clear the context
- **D)** Increase `max_tokens` so both approaches fit in one context

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

`fork_session` duplicates the current session at that exact point in time, producing two independent branches that can diverge freely. The original session is untouched, so the team can return to it if the alternative approach fails.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/sessions

**Why the others are wrong:**

- **A:** Opening a new unnamed session starts completely fresh with no shared history; it does not branch from the current state.
- **C:** `/compact` summarizes context and `/reset` clears it; neither operation creates a branch — they modify or destroy the existing session.
- **D:** Increasing `max_tokens` only extends response length; it cannot fork session state or preserve two independent lines of work.

</details>

---

#### C82

*TS 1.7 · Session state, resumption & forking*

**Situation:** A developer resumed a Claude session that had been idle for three weeks. The codebase the session was referencing has since undergone significant refactoring.

What is the primary risk of immediately continuing work in a resumed session after a long gap without re-validating context?

- **A)** Token costs will be higher because the model re-processes the entire history on every request
- **B)** The system prompt is permanently lost after a session is idle for more than 24 hours
- **C)** The model may act on stale assumptions about code structure, APIs, or decisions that no longer reflect the current state
- **D)** The session will automatically expire and return an authentication error

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Stale context risk is the key hazard after a long gap: the session still contains facts, assumptions, and code snippets from before the refactoring. The model will reason from this outdated information unless the architect explicitly re-validates or updates the context.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/sessions

**Why the others are wrong:**

- **A:** Token costs are based on the tokens sent in each request; re-processing of history does happen but this is a cost concern, not the primary correctness risk described.
- **B:** System prompts are not time-limited; they persist as configured regardless of session idle time.
- **D:** Sessions do not automatically expire with a hard error on resume; stale context is a semantic problem, not an authentication or lifecycle error.

</details>

---

#### C83

*TS 1.7 · Session state, resumption & forking*

**Situation:** An architect is designing a multi-agent system where a coordinator spawns worker sessions to complete sub-tasks. The coordinator must merge results from forked sessions back into a coherent master plan. Three forks have been running in parallel for two hours and have made contradictory decisions about the shared data model.

Which strategy best handles the merge of contradictory forked session states back into the coordinator session?

- **A)** Discard all forks and start fresh, since contradictory states cannot be safely merged
- **B)** Take the output of the fork with the longest message history, as more conversation implies more refinement
- **C)** Ask each fork to produce a structured decision log; surface contradictions explicitly to the coordinator, which then resolves them using the original session's constraints before synthesizing a unified plan
- **D)** Concatenate all three forks' assistant messages into a single user message for the coordinator to process

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

When forks diverge, the coordinator must act as a deliberate merge agent. Each fork should surface its decisions in a structured log (not raw transcripts). The coordinator then sees the contradictions explicitly, applies the original constraints from the parent session to adjudicate them, and synthesizes a unified outcome. This mirrors a structured rebase: understand divergence, resolve conflicts with authoritative context, produce a clean result.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/sessions
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems

**Why the others are wrong:**

- **A:** Discarding all parallel work wastes the valuable progress made in each fork; structured merging is always preferable to abandonment when forks have produced substantive output.
- **B:** Message count is a poor proxy for quality; a long fork may have explored many dead ends, while a shorter fork may have reached a cleaner conclusion. Length does not resolve contradictions.
- **D:** Concatenating raw message streams creates a massive, unstructured context full of contradictions; the coordinator has no principled way to adjudicate conflicts from an undifferentiated blob of text.

</details>

---

#### C84

*TS 1.7 · Session state, resumption & forking*

**Situation:** A developer is designing a session management library for a team of Claude-powered coding agents. Sessions may run for days, be forked for experiments, and be resumed after variable idle periods. The library must implement a policy for when to fork versus when to start fresh.

Which policy logic most correctly captures the trade-off between forking and starting fresh?

- **A)** Always fork; a fresh session should never be started because historical context always adds value
- **B)** Fork when the current session contains foundational context that the new work depends on; start fresh when the accumulated history is net-negative — either because it is so large that it degrades inference quality, so stale that it would actively mislead, or so cluttered with contradicted paths that a clean slate with an explicit context injection is more accurate
- **C)** Start fresh whenever the session exceeds 30k tokens to keep latency low; use forking only for sessions under 30k tokens
- **D)** Fork for all experiments and start fresh only for production tasks, regardless of context state

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

The fork-vs-fresh decision is a function of net context value. Fork when history is foundational: the new branch needs the accumulated design decisions, constraints, or domain knowledge established in the parent. Start fresh when history is net-negative: a context so large it increases latency and noise, so stale it carries wrong facts, or so cluttered with abandoned paths that an explicit, curated injection of only the relevant truths is more accurate than the raw history. Token threshold alone (option C) is too blunt; quality and relevance determine net value.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/sessions

**Why the others are wrong:**

- **A:** Historical context is not always additive; stale facts, overruled decisions, and accumulated noise actively mislead the model. Blanket forking ignores this degradation and produces worse outcomes than a well-scoped fresh start.
- **C:** A fixed token threshold ignores context quality entirely; a 40k-token session of high-quality, current decisions may be worth forking, while a 25k-token session of stale contradictions may warrant starting fresh.
- **D:** Tying the fork/fresh decision to production status rather than context quality is an arbitrary policy that does not address the actual risk — misleading or degraded context — in either environment.

</details>

---

#### C85

*TS 1.7 · Session state, resumption & forking*

**Situation:** An engineering team uses a shared Claude session to design a new microservice. Halfway through the session, one developer wants to prototype a GraphQL API while another wants to prototype a REST API, comparing both options before committing.

What is the most architecturally sound way to explore both approaches without contaminating each other or losing the shared foundation?

- **A)** Fork the session at the current state to create two independent branches, one for each API style
- **B)** Continue in the existing session, exploring GraphQL first and then restarting the conversation to explore REST
- **C)** Open two completely fresh sessions and manually copy-paste the shared design decisions into each one
- **D)** Ask Claude to keep two simultaneous design tracks in memory within the same session

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

`fork_session` at the current point creates two branches that share the same foundation of design decisions. Each branch can explore its API style independently without affecting the other, and the team can compare outcomes before deciding which branch to promote.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/sessions

**Why the others are wrong:**

- **B:** Exploring sequentially in one session means the REST exploration is contaminated by the GraphQL decisions already made; restarting loses the shared foundation entirely.
- **C:** Manually copying context is error-prone, tedious, and will inevitably diverge from the live session state; forking is both safer and more accurate.
- **D:** A single session cannot maintain two truly independent lines of thought; the model's responses will blend both approaches, making comparison unreliable.

</details>

---

#### C86

*TS 1.7 · Session state, resumption & forking*

**Situation:** An architect resumes a named session that was last active two months ago. The session contains extensive context about a system that has since been partially rewritten.

Which combination of actions best mitigates stale context risk before continuing substantive work?

- **A)** Immediately continue work; Claude will automatically detect outdated information and flag it
- **B)** Run `/compact` to clear the old context and start fresh from the current codebase
- **C)** Ask Claude to summarize its current understanding, review the summary for stale facts, then inject corrections as updated context before proceeding
- **D)** Increase the context window size so all historical context plus new changes fit simultaneously

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Asking Claude to surface its current understanding makes stale assumptions visible. The architect can then identify and correct outdated facts — decisions overturned, APIs changed, modules renamed — and inject the corrections before any substantive work begins. This is a deliberate re-validation step.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/sessions

**Why the others are wrong:**

- **A:** Claude does not automatically detect that its session context is outdated relative to an external codebase; it will confidently reason from stale facts unless corrected.
- **B:** `/compact` summarizes the existing (potentially stale) history rather than replacing it with current truth; running it alone does not resolve the staleness problem.
- **D:** A larger context window accommodates more tokens but does not make outdated information accurate; the stale facts remain in the history regardless of window size.

</details>

---

#### C87

*TS 1.7 · Session state, resumption & forking*

**Situation:** A developer has two active Claude sessions: Session A is a long-running design session for a core library, and Session B is an exploratory spike that has grown cluttered with dead ends. The spike produced one valuable insight that must be carried forward.

When is it most appropriate to start Session B completely fresh rather than continuing or forking it?

- **A)** When Session A and Session B share any overlapping topics
- **B)** Whenever Session B exceeds 50% of the context window
- **C)** When the session's accumulated noise, dead ends, and abandoned paths would mislead future reasoning more than the remaining useful context would help
- **D)** When the session has been idle for more than 48 hours

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

The decision to start fresh should be driven by signal-to-noise ratio. If a session is so cluttered with contradicted directions, failed experiments, and stale assumptions that continuing it would actively mislead the model, a fresh start — carrying only the one valuable insight explicitly — produces better outcomes than trying to salvage a polluted context.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/sessions

**Why the others are wrong:**

- **A:** Topic overlap between sessions is not a criterion for starting fresh; topics can legitimately span sessions without requiring either to be abandoned.
- **B:** Context window percentage is a mechanical metric; a session at 45% could be perfectly clean while one at 20% could be badly polluted. Quality of context, not quantity, drives the decision.
- **D:** Idle time alone does not determine whether a session's context is useful; a well-structured session paused for a week may resume better than a chaotic session paused for an hour.

</details>

---

#### C88

*TS 1.7 · Session state, resumption & forking*

**Situation:** A platform engineering team is building a system that programmatically manages dozens of Claude sessions for different projects. They need to resume any session on demand and ensure each session has the correct context restored.

What must an architect ensure is reliably stored and restored to achieve correct session resumption in a programmatic system?

- **A)** The session ID and the current date, which are sufficient for Claude to reconstruct context
- **B)** The full ordered messages array and the system prompt, so the model receives the same context it had when last active
- **C)** Only the last assistant message, to avoid sending redundant tokens
- **D)** Only the system prompt, since Claude can infer conversation history from it

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Correct resumption requires restoring the complete ordered messages array (all prior user and assistant turns) plus the system prompt. The model has no persistent memory between API calls; every request must supply the full context it needs. Partial restoration leads to incoherent or contradictory responses.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/sessions
- https://docs.anthropic.com/en/api/getting-started

**Why the others are wrong:**

- **A:** A session ID is a storage key, not a context payload; Claude's API is stateless and does not use an ID to recall history — the caller must supply the messages array explicitly.
- **C:** Sending only the last assistant message strips all prior context; the model cannot reconstruct the decision trail, open questions, or intermediate steps from a single message.
- **D:** The system prompt sets instructions and persona but contains no record of what was actually discussed; conversation history is irreplaceable for coherent resumption.

</details>

---

#### C89

*TS 1.7 · Session state, resumption & forking*

You want to explore two different synthesis approaches (narrative-style vs data-driven) from the same set of search and analysis results. What feature enables this?

- **A)** Run the synthesis subagent twice with different system prompts.
- **B)** Use `fork_session` to create two independent branches from the shared analysis baseline, each exploring a different approach.
- **C)** Create two separate synthesis subagents with different `AgentDefinition` configurations.
- **D)** Use `--resume` to go back to the pre-synthesis point and try a different approach.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

`fork_session` creates independent branches from a shared analysis baseline — each fork has a full copy of the context at the fork point.

**Why the others are wrong:**

- **A:** Running twice can work if you pass the same findings explicitly, but doesn't preserve the full conversational context.
- **C:** Different AgentDefinitions create independent agents without shared history.
- **D:** `--resume` continues linearly, it doesn't branch.

</details>

---

#### C90

*TS 1.7 · Session state, resumption & forking*

You want to compare two refactoring approaches — one using the Strategy pattern and one using the Builder pattern — from the same codebase analysis baseline. What feature enables this?

- **A)** Use `--resume` to return to the analysis point and try the second approach.
- **B)** Use `fork_session` to create independent branches from the shared analysis baseline, exploring each pattern in its own branch.
- **C)** Run the agent twice with different system prompts.
- **D)** Create two separate agents with different `AgentDefinition` configurations.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

`fork_session` creates independent branches from a shared baseline.

**Why the others are wrong:**

- **A:** `--resume` doesn't create branches — it continues linearly.
- **C:** Running twice lacks shared context.
- **D:** Different agents don't share the existing analysis.

</details>

---

#### C91

*TS 1.7 · Session state, resumption & forking*

A developer resumes a session from yesterday using `--resume investigation`. Since yesterday, three critical files in the codebase have been refactored. The agent continues analyzing based on stale information. What's the solution?

- **A)** Use `/compact` after resuming to force the agent to re-read everything.
- **B)** Always start a new session instead of resuming.
- **C)** Run Bash to `git diff` and pipe the results into the agent context.
- **D)** After resuming, inform the agent about the specific files that changed so it can re-analyze those files rather than re-exploring the entire codebase.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

After resuming, inform the agent about specific file changes for targeted re-analysis.

**Why the others are wrong:**

- **A:** `/compact` doesn't trigger re-reading.
- **B:** Always starting fresh wastes prior work.
- **C:** Git diff helps but should be combined with targeted guidance.

</details>

---

#### C92

*TS 1.7 · Session state, resumption & forking*

An attorney resumes a session from yesterday using `--resume contract-review`. Since yesterday, the counterparty has sent revised contract terms. The agent continues analyzing based on stale information. What's the solution?

- **A)** Always start a new session instead of resuming.
- **B)** Run Bash to compare file versions and pipe the results into the agent context.
- **C)** After resuming, inform the agent about the specific sections that changed so it can re-analyze those rather than re-reviewing the entire contract.
- **D)** Use `/compact` after resuming to force the agent to re-read everything.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

After resuming, inform the agent about specific sections that changed for targeted re-analysis.

**Why the others are wrong:**

- **A:** Always starting fresh wastes prior work.

</details>

---

<a id="part-c-domain-2"></a>

### Part C · Domain 2 · Tool Design & MCP Integration (18%)

#### C93

*TS 2.1 · Tool descriptions & boundaries*

**Situation:** A developer is registering a tool called `get_user` with no description field. When tested, Claude sometimes calls this tool when it should be calling `get_account` instead, and vice versa.

What is the PRIMARY reason Claude confuses these two tools?

- **A)** The Anthropic API requires a minimum of three tools before disambiguation logic activates.
- **B)** Claude always calls the first tool listed when unsure, so `get_user` should be moved later in the list.
- **C)** Tool descriptions are the primary signal Claude uses to decide which tool to call; without them, Claude can only guess from the name alone.
- **D)** The tool names are too short and should be more verbose.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Tool descriptions are the primary mechanism by which Claude decides which tool is appropriate for a given situation. Without a description, Claude has only the tool name as signal, which is insufficient to reliably distinguish between semantically similar tools like `get_user` and `get_account`.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/overview
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use

**Why the others are wrong:**

- **A:** There is no minimum tool count requirement in the Anthropic API before disambiguation logic applies; descriptions matter regardless of how many tools are registered.
- **B:** Claude does not use list position as a tiebreaker; it evaluates all available tools against the description and context of the request.
- **D:** Tool name length has no direct bearing on disambiguation quality; a descriptive description on a short-named tool will outperform a long name with no description.

</details>

---

#### C94

*TS 2.1 · Tool descriptions & boundaries*

**Situation:** You are writing a tool description for a function that queries a read-only product catalog. The tool takes a `query` string and returns a list of matching product objects including name, SKU, and price.

Which description best follows tool-description best practices?

- **A)** "Searches the catalog."
- **B)** "A tool for products. Input: query. Output: list."
- **C)** "Searches the read-only product catalog by keyword. Returns a list of matching products, each containing name, SKU, and price. Use this tool when the user asks about available products, pricing, or inventory lookup. Does not modify catalog data."
- **D)** "Call this function with a string. It will return some objects. Please read the source code for full details."

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

A good tool description covers what the tool does, when to use it, what it returns, and any important constraints or edge cases. Option C states the action (searches), the data source (read-only product catalog), the input type (keyword), the return structure (list with name, SKU, price), a usage hint, and a safety constraint (does not modify data).

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/overview

**Why the others are wrong:**

- **A:** Too vague — it tells Claude nothing about what catalog is being searched, what the return value looks like, or when this tool is preferred over alternatives.
- **B:** Restates the parameter names without adding semantic meaning; Claude cannot infer use cases, return shape, or constraints from this description.
- **D:** Referring Claude to source code is not actionable; Claude cannot read source files at inference time and this provides no useful disambiguation signal.

</details>

---

#### C95

*TS 2.1 · Tool descriptions & boundaries*

**Situation:** A team registers a single tool called `file_manager` that accepts a `mode` parameter with values `read`, `write`, `delete`, and `list`. They notice Claude sometimes picks the wrong mode or calls the tool unnecessarily.

What is the recommended refactoring approach for this tool design?

- **A)** Keep the single tool but add more allowed values to the `mode` parameter to cover additional operations.
- **B)** Split the tool into separate focused tools — `read_file`, `write_file`, `delete_file`, `list_files` — each with its own description.
- **C)** Rename the tool to `advanced_file_manager` so Claude treats it with higher priority.
- **D)** Add a `description` parameter to the tool call so Claude can explain why it chose that mode.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Splitting a mega-tool with a mode parameter into focused, single-purpose tools is a key best practice. Each focused tool can have a precise description covering its specific behavior, preconditions, and return value, giving Claude far richer signal for selecting the correct operation than a shared tool with a mode switch.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/overview

**Why the others are wrong:**

- **A:** Adding more modes to an already overloaded tool compounds the disambiguation problem; the description becomes harder to write clearly and Claude must make an additional disambiguation decision for the mode value.
- **C:** Tool naming conventions influence readability and clarity but renaming alone does not improve Claude's ability to select the right operation; descriptions drive selection.
- **D:** Parameters are inputs to the tool, not meta-instructions to the model; adding a `description` parameter would be passed to the tool function, not used by Claude for tool selection.

</details>

---

#### C96

*TS 2.1 · Tool descriptions & boundaries*

**Situation:** An architect is naming a set of tools for a calendar integration. They are considering the names: `doCalendarStuff`, `CalendarTool`, `create_calendar_event`, and `c`.

Which naming convention best follows tool-naming best practices for Claude tool interfaces?

- **A)** `CalendarTool` — PascalCase is the standard for tools in the Anthropic API.
- **B)** `create_calendar_event` — snake_case with a verb-noun pattern that clearly states the action and object.
- **C)** `c` — short names reduce token usage and improve performance.
- **D)** `doCalendarStuff` — verb-first camelCase communicates action intent.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

The recommended convention is snake_case with a clear verb-noun pattern (e.g., `create_calendar_event`). This makes the tool's purpose immediately scannable, pairs naturally with an equally clear description, and is consistent with the naming style used throughout Anthropic's documentation and examples.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/overview

**Why the others are wrong:**

- **A:** There is no PascalCase requirement in the Anthropic API; snake_case is the idiomatic style shown in official documentation and examples.
- **C:** Ultra-short names like `c` provide no semantic signal and force the description to carry all disambiguation weight, which degrades the combined name+description clarity Claude uses for selection.
- **D:** `doCalendarStuff` is vague — 'stuff' conveys no information about which calendar action is performed, making the description's job harder.

</details>

---

#### C97

*TS 2.1 · Tool descriptions & boundaries*

**Situation:** A financial assistant has three tools: `get_account_balance` (returns current balance), `get_transaction_history` (returns paginated past transactions), and `get_pending_transactions` (returns transactions that have not yet settled). Despite good individual descriptions, Claude calls `get_transaction_history` for questions about pending charges, which returns no pending items because it only shows settled transactions.

Which combination of description changes most precisely solves this three-way disambiguation problem?

- **A)** Add a routing system prompt that tells Claude to always prefer `get_pending_transactions` when the word 'pending' appears in the user message.
- **B)** Add 'This is the most commonly used tool' to `get_transaction_history` to boost its priority.
- **C)** Add cross-references: in `get_transaction_history` add '— does NOT include pending or unsettled charges; use get_pending_transactions for those'; in `get_pending_transactions` add '— for charges not yet settled or still processing; these will NOT appear in get_transaction_history'; in `get_account_balance` add '— reflects settled balance only, may not reflect pending charges'.
- **D)** Merge `get_transaction_history` and `get_pending_transactions` into one tool with a `status` filter parameter.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

When tools have overlapping semantic domains, cross-referencing between descriptions is the most precise technique. Explicitly naming sister tools and their boundaries within each description creates a web of mutual disambiguation: each tool actively repels queries meant for another by naming the alternative. This is far more robust than keyword routing, merging, or artificial priority signals.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/overview
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use

**Why the others are wrong:**

- **A:** Keyword-matching system prompts are brittle: users rarely use exact keywords like 'pending', and a routing instruction does not fix the underlying description quality; it patches one surface symptom while leaving the root cause unaddressed.
- **B:** Claiming a tool is 'most commonly used' is not a semantic description of its boundaries; it does not help Claude understand when NOT to use it, and it may cause Claude to over-call it.
- **D:** Merging creates a mode-parameter tool that reintroduces disambiguation at the parameter level and loses the ability to write distinct, focused descriptions for settled vs. pending queries.

</details>

---

#### C98

*TS 2.1 · Tool descriptions & boundaries*

**Situation:** A developer exposes a `run_sql_query` tool. The tool description says: 'Runs a SQL query against the database.' During a red-team exercise, the model is observed using this tool to execute `DROP TABLE users` when asked to 'clean up test data'. The developer wants to fix this through tool interface design, not post-hoc output filtering.

Which tool interface change best addresses the safety boundary at the description level?

- **A)** Add a `confirm` boolean parameter that Claude must set to `true` before the query executes.
- **B)** Rename the tool to `run_safe_sql_query` to signal intent.
- **C)** Split into `run_read_query` (SELECT only, description explicitly states it executes read-only SELECT statements and will error on any DDL or DML) and `run_write_query` (INSERT/UPDATE only, description explicitly states it does not accept DROP, TRUNCATE, or ALTER statements), and remove the generic `run_sql_query` tool.
- **D)** Move all SQL execution to a system prompt instruction that lists forbidden SQL keywords.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Splitting by read/write with explicit DDL/DML exclusions baked into the descriptions encodes safety boundaries at the interface level. The read tool's description tells Claude it can only be used for SELECT, preventing it from even attempting destructive DDL. The write tool's description further restricts to INSERT/UPDATE. This defense-in-depth approach makes the safety constraint part of the tool's semantic identity, not a runtime filter.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/overview
- https://docs.anthropic.com/en/docs/build-with-claude/computer-use

**Why the others are wrong:**

- **A:** A `confirm` parameter does not prevent Claude from generating a `DROP TABLE` statement; it only adds an extra field that Claude may set to `true` if it believes the action is requested, providing a false sense of safety.
- **B:** Adding 'safe' to the name is aspirational but provides no actual constraint; the description still does not say what is forbidden, so Claude cannot know that DROP is disallowed.
- **D:** System prompt keyword lists are brittle and can be circumvented by obfuscation or paraphrase; they also do not document the tool's interface contract for developers reading the schema.

</details>

---

#### C99

*TS 2.1 · Tool descriptions & boundaries*

**Situation:** An architect is designing a tool suite for a customer support agent. The suite includes `lookup_customer`, `update_customer_profile`, `create_support_ticket`, `escalate_ticket`, and `send_customer_email`. Usage logs show Claude frequently chains `lookup_customer` → `send_customer_email` without creating a ticket first, even for issues that require follow-up tracking.

Which description strategy best encodes the required workflow order as a soft constraint within the tool interface?

- **A)** Add a `ticket_id` required parameter to `send_customer_email` so the tool call fails at the API level without a ticket ID.
- **B)** Set the tool `input_schema` for `send_customer_email` to `null` to prevent Claude from calling it autonomously.
- **C)** Remove `send_customer_email` from the tool suite and handle email sending outside the agent loop.
- **D)** In the `send_customer_email` description, add: 'Use only after a support ticket has been created for the issue being communicated. If no ticket exists for this interaction, call create_support_ticket first.' In `create_support_ticket`, add: 'Creates a ticket that tracks this customer issue — call this before sending any follow-up email about the issue.'

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Encoding workflow preconditions in the description is the recommended technique for guiding tool call sequencing. By explicitly stating 'call create_support_ticket first' in the email tool's description, and cross-referencing the email tool in the ticket tool's description, Claude receives the ordering constraint as part of its selection signal. This is a soft constraint that guides behavior without breaking the interface for cases where a ticket legitimately already exists.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/overview
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use

**Why the others are wrong:**

- **A:** Adding a required `ticket_id` parameter is a valid hard-enforcement technique, but the question asks about description-level strategy. It would also break legitimate cases where an existing ticket ID must be looked up before passing it, without giving Claude the workflow context it needs.
- **B:** Setting `input_schema` to null is not a valid Anthropic API pattern for restricting tool invocation and would likely cause a schema validation error.
- **C:** Removing a capability to enforce workflow order is a heavy-handed approach that reduces agent functionality; the goal is to guide sequencing, not eliminate the email capability.

</details>

---

#### C100

*TS 2.1 · Tool descriptions & boundaries*

**Situation:** A team has two tools: `search_knowledge_base` and `search_web`. Users frequently ask general questions and Claude is calling `search_web` for answers that are already in the internal knowledge base, increasing cost and latency.

Which description strategy best fixes the tool selection problem?

- **A)** Add explicit disambiguation language to both descriptions: `search_knowledge_base` should state it covers internal, proprietary, or company-specific information and should be tried first; `search_web` should state it is for publicly available information not found in the knowledge base.
- **B)** Remove `search_web` entirely so Claude always uses the internal knowledge base.
- **C)** Merge the two tools into one `search` tool with a `source` parameter set to `internal` or `web`.
- **D)** Prefix both tool names with a priority number, e.g., `1_search_knowledge_base` and `2_search_web`, to communicate ordering.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

The correct fix is to add explicit disambiguation language in both descriptions. By specifying the domain of each tool (internal/proprietary vs. publicly available) and a preference order in the description text, Claude has the signal it needs to make the correct selection without removing tools or changing the interface structure.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/overview
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use

**Why the others are wrong:**

- **B:** Removing `search_web` reduces the agent's capability; the goal is to fix selection logic, not eliminate tools.
- **C:** Merging into a mega-tool with a `source` parameter recreates the same disambiguation problem at the parameter level and violates the focused-tool best practice.
- **D:** Numeric prefixes in names are a brittle hack; Claude does not parse numeric prefixes as priority instructions, and it breaks the clean verb-noun naming convention.

</details>

---

#### C101

*TS 2.1 · Tool descriptions & boundaries*

**Situation:** A developer registers a `send_email` tool with a `recipients` parameter described only as `"List of recipients"`. In production, Claude sometimes passes a flat string like `"alice@example.com, bob@example.com"` instead of an array.

Which parameter description change most directly prevents this error?

- **A)** Add a system prompt instruction telling Claude to always use arrays for all tool parameters.
- **B)** Rename the parameter from `recipients` to `recipientList` to imply it is a list.
- **C)** Change the parameter type to `string` in the schema so Claude always passes a string, then split it server-side.
- **D)** Update the description to explicitly state: `"Array of email address strings, e.g. [\"alice@example.com\", \"bob@example.com\"]. Must be an array even when there is only one recipient. Do not pass a comma-separated string."`

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Parameter descriptions should state the exact expected format, include a concrete example, and explicitly call out common mistakes. Telling Claude what NOT to do (no comma-separated strings) and what TO do (array even for one recipient) in the parameter description gives the model the precision it needs to populate parameters correctly.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/overview

**Why the others are wrong:**

- **A:** A blanket system prompt instruction applies globally and lacks specificity; it may interfere with other tools that legitimately accept strings and does not document the constraint where developers will look — in the tool schema.
- **B:** Renaming the parameter provides a weak naming hint but does not override the ambiguity; without an explicit format description and example, Claude may still pass a string.
- **C:** Changing the schema type to string to work around Claude's behavior shifts the burden to the server and loses type safety; it does not teach Claude the correct interface.

</details>

---

#### C102

*TS 2.1 · Tool descriptions & boundaries*

**Situation:** An agentic assistant has a `get_order_status` tool and a `get_order_history` tool. A product manager notices that Claude calls `get_order_history` (which returns all past orders) when users ask 'where is my current order?', even though `get_order_status` is the correct and cheaper call.

What is the most effective single change to the tool definitions to correct this behavior?

- **A)** Add `"DEPRECATED: use get_order_status instead"` to the `get_order_history` description.
- **B)** Increase the `max_tokens` parameter to give Claude more space to reason about which tool to choose.
- **C)** Swap the order of the two tools in the tools array so `get_order_status` appears first.
- **D)** Update `get_order_status` description to explicitly mention it handles real-time tracking and current delivery status for a single order, and update `get_order_history` to clarify it returns a paginated list of all past orders — not intended for current shipment tracking.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Both descriptions must be updated together for effective disambiguation. The correct tool's description should attract the right queries by mentioning real-time tracking and current status; the incorrect tool's description should repel those queries by clarifying it covers historical data, not live shipment tracking.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/overview

**Why the others are wrong:**

- **A:** Marking a live tool as deprecated is misleading and may cause Claude to avoid it entirely; the tool is valid for its intended purpose and should have an accurate, positive description instead.
- **B:** `max_tokens` controls response length, not tool selection reasoning; tool selection is driven by descriptions, not token budget.
- **C:** Tool array ordering does not influence Claude's selection logic; Claude evaluates all tool descriptions against the request context regardless of order.

</details>

---

#### C103

*TS 2.1 · Tool descriptions & boundaries*

**Situation:** A team is designing a tool that calls an external API with a `date_range` parameter. The API accepts dates in `YYYY-MM-DD` format only and returns an error for any other format. The tool description currently says: `"date_range: The date range for the query."`

Which parameter description includes all the elements needed to prevent format errors?

- **A)** "date_range: See API documentation for format requirements."
- **B)** "date_range: An object with start and end dates."
- **C)** "date_range: Required. Please format correctly."
- **D)** "date_range: Object with keys `start` and `end`, each a date string in ISO 8601 YYYY-MM-DD format (e.g., {start: '2024-01-15', end: '2024-03-31'}). Both fields are required. Dates must not be in the future."

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

A complete parameter description for a constrained input should specify the type/structure, the exact format with the format string (`YYYY-MM-DD`), a concrete example, which fields are required, and any additional constraints (e.g., no future dates). This leaves no ambiguity for Claude to produce an incorrectly formatted value.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/overview

**Why the others are wrong:**

- **A:** Claude cannot access external API documentation at inference time; all constraints must be embedded directly in the parameter description.
- **B:** Stating 'object with start and end dates' omits the critical format string and example; Claude may still generate `01/15/2024` or a Unix timestamp.
- **C:** This is entirely non-informative; it does not describe structure, format, or constraints and will not prevent any formatting errors.

</details>

---

#### C104

*TS 2.1 · Tool descriptions & boundaries*

**Situation:** An architect is designing a document tool suite. They propose a single `document_tool` with an `action` enum parameter accepting `create`, `read`, `update`, `delete`, and `summarize`. A colleague argues this should be five separate tools.

Which argument BEST supports splitting into five separate tools?

- **A)** The Anthropic API has a hard limit of one action per tool and will reject schemas with enum parameters.
- **B)** Five tools will always be faster than one tool because the API processes smaller schemas more quickly.
- **C)** Separate tools allow each tool to have a distinct, focused description that precisely conveys its preconditions, side effects, and return shape — making Claude's selection decision unambiguous and eliminating the need for Claude to also select the correct action value.
- **D)** Having more tools increases the probability that Claude will call at least one of them.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

The core argument for splitting is description quality and selection clarity. Each focused tool can have a tailored description that covers its unique behavior (e.g., `delete_document` can warn about irreversibility; `summarize_document` can describe what the summary contains). A mega-tool forces Claude to make two disambiguation decisions: which tool to call AND which action value to pass, compounding the chance of error.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/overview

**Why the others are wrong:**

- **A:** The Anthropic API supports enum parameters and there is no such restriction; splitting tools is a best practice recommendation, not an API enforcement.
- **B:** Schema size has a negligible effect on API processing time; performance is not the reason to split tools.
- **D:** Having more tools does not increase the likelihood of correct tool calls; it could increase noise if the descriptions are not well written.

</details>

---

#### C105

*TS 2.1 · Tool descriptions & boundaries*

The `get_customer` and `lookup_order` tools both accept an identifier parameter and have nearly identical descriptions: "Retrieves information using the provided identifier." Agents frequently call the wrong tool. What is the most effective first step?

- **A)** Implement a routing classifier that analyzes the user's message and pre-selects the appropriate tool based on keyword detection.
- **B)** Expand each tool's description to include accepted input formats, example queries, expected outputs, boundaries, and when to use it versus the other tool.
- **C)** Consolidate both tools into a single `lookup` tool that internally routes to the correct backend based on the identifier format.
- **D)** Add 8-10 few-shot examples to the system prompt showing correct tool selection for various queries.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Tool descriptions are the primary selection mechanism. Minimal descriptions are the root cause. Fix descriptions first (low effort, high leverage). Consolidation (C) is valid but more effort than a "first step." Few-shot examples (D) add token overhead without fixing the root cause.

**Why the others are wrong:**

- **A:** Routing classifiers are over-engineered.

</details>

---

#### C106

*TS 2.1 · Tool descriptions & boundaries*

Your agent uses an MCP server for payment processing. The `process_refund` tool's description says: "Processes a refund." The agent frequently calls it for balance inquiries because it doesn't understand the tool's scope. What's the fix?

- **A)** Remove the tool and handle refunds manually.
- **B)** Rename the tool to `issue_monetary_refund` so the name is more specific.
- **C)** Enhance the tool description to be specific: "Processes a monetary refund to the customer's original payment method. Requires: order_id, refund_amount, reason. Do NOT use for balance lookups, payment status checks, or transaction history."
- **D)** Add few-shot examples showing when to use each payment tool.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Tool descriptions are the primary selection mechanism. Minimal descriptions are the root cause. Detailed descriptions with explicit scope, required parameters, and anti-use-cases fix selection.

**Why the others are wrong:**

- **B:** Renaming helps but descriptions matter more.
- **D:** Few-shot adds token overhead.

</details>

---

#### C107

*TS 2.1 · Tool descriptions & boundaries*

Inspecting per-turn logs from the customer support agent, you find that `get_customer` is called for order-status questions about 45% of the time, even though `lookup_order` is the intended tool for those queries. The current tool definitions are:


```
get_customer:   description = "Returns customer info"
lookup_order:   description = "Returns order details"
```


Which change is the most effective **root-cause** fix?

- **A)** Merge the two into a single `get_customer_or_order` tool that branches internally based on whether the supplied identifier looks like a customer_id or an order_id.
- **B)** Add a paragraph to the system prompt: "When the user mentions an order, call `lookup_order`, not `get_customer`."
- **C)** Force `tool_choice: {type: "tool", name: "lookup_order"}` for any user message that contains the word "order".
- **D)** Rewrite the two descriptions with concrete trigger phrases — e.g. `get_customer`: "Use for customer profile, account status, contact details, payment methods on file. Requires `customer_id`." and `lookup_order`: "Use for order status, shipping tracking, returns, refunds by order number. Requires `order_id`."

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

With `tool_choice: "auto"` (the default), Claude routes between tools by matching the user's request against each tool's **description**. When two descriptions are minimal and overlapping, the model has nothing to disambiguate on and routes by superficial cues — exactly the failure mode in the logs. The durable fix is to make the descriptions themselves discriminating: name the use cases, the inputs required, and the boundary between the tools.

**Why the others are wrong:**

- **A:** Bundles two distinct responsibilities into one tool, which violates the "one tool, one purpose" guidance — now every caller of `get_customer_or_order` needs to interpret a polymorphic response.
- **B:** Competes against the tool descriptions instead of fixing them; the system prompt and tool descriptions both shape routing, and the descriptions win at scale.
- **C:** Brittle keyword routing — it misses paraphrases ("where's my purchase?", "track my delivery") and over-fires on customer-profile questions that mention orders in passing.

</details>

---

#### C108

*TS 2.2 · Structured error responses*

**Situation:** An agentic loop is executing and the model issues three tool calls in a single response. Two tools succeed immediately, but the third tool call triggers a downstream service that returns an error. The developer must decide how to structure the `tool_result` for the failed call.

What is the correct way to return a tool execution failure to the model so it can reason about the error and decide how to proceed?

- **A)** Return a `tool_result` block for the failed call with `"is_error": true` and include the error details in the `content` field.
- **B)** Raise an exception in the orchestrator and abort the loop; errors in tool execution are unrecoverable.
- **C)** Return a `tool_result` block with `"is_error": false` but include an error message in `content` so the model sees it as a normal result.
- **D)** Omit the failed tool's result from the user message entirely; the model will infer the failure from the absence of a result.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Setting `"is_error": true` on the `tool_result` block is the documented mechanism for communicating tool failures to the model. The model can then reason about the error — retrying with different parameters, choosing an alternative tool, or informing the user — rather than proceeding with incomplete information.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use

**Why the others are wrong:**

- **B:** Many tool errors are recoverable — the model may retry with corrected parameters or take an alternative path. Aborting immediately discards the model's ability to handle errors gracefully.
- **C:** Misrepresenting an error as a success by setting `"is_error": false` deprives the model of the signal it needs to apply error-recovery reasoning, likely causing it to proceed incorrectly on a false assumption.
- **D:** Omitting a result for a tool call that was made violates the protocol: the model expects a result for every `tool_use_id` it issued. Omission can cause undefined behavior or an API validation error.

</details>

---

#### C109

*TS 2.2 · Structured error responses*

**Situation:** An MCP tool handler receives a request to look up a user by ID. The ID provided in the tool call arguments is not a valid integer — it is the string `"abc"`. The tool must signal this failure to the Claude agent.

Which MCP response structure correctly signals a tool-level error back to the Claude agent?

- **A)** Return `{ "isError": false, "content": [{ "type": "text", "text": "Error: invalid ID" }] }`
- **B)** Return an empty `content` array with no `isError` flag set.
- **C)** Raise an unhandled exception and let the MCP transport layer return a JSON-RPC parse error.
- **D)** Return `{ "isError": true, "content": [{ "type": "text", "text": "Validation error: user ID must be an integer, received 'abc'" }] }`

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

The MCP specification requires setting `isError: true` in the tool result to indicate a tool-level failure. The `content` array should carry a human-readable message that explains what went wrong and guides the model toward a corrective action. Option D satisfies both requirements.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use#handling-tool-use-and-tool-results
- https://docs.anthropic.com/en/docs/agents-and-tools/mcp

**Why the others are wrong:**

- **A:** Setting `isError: false` while embedding an error message in the content creates a contradictory signal. The model treats `isError: false` as a successful result and may act on the error text as if it were valid data.
- **B:** An empty content array with no `isError` flag is an ambiguous success response. The agent has no mechanism to detect that a failure occurred and will proceed as if the tool returned nothing.
- **C:** Unhandled exceptions propagate as transport-level JSON-RPC errors, not as tool-level errors. Claude cannot reason about transport errors in the same way it reasons about `isError: true` results — the failure is invisible to the model's planning loop.

</details>

---

#### C110

*TS 2.2 · Structured error responses*

**Situation:** An MCP tool calls a downstream API that returns a `429 Too Many Requests` response. The tool must relay this failure to the Claude agent in a way that tells the model whether it makes sense to try again.

Which of the following error responses best communicates that this error is retryable?

- **A)** `{ "isError": true, "content": [{ "type": "text", "text": "{\"error_type\": \"internal\", \"retryable\": false, \"message\": \"Rate limit exceeded.\"}" }] }`
- **B)** `{ "isError": true, "content": [{ "type": "text", "text": "{\"error_type\": \"rate_limit\", \"retryable\": true, \"message\": \"Rate limit exceeded. Retry after 5 seconds.\"}" }] }`
- **C)** `{ "isError": false, "content": [{ "type": "text", "text": "429 from upstream" }] }`
- **D)** `{ "isError": true, "content": [{ "type": "text", "text": "Error occurred." }] }`

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Option B correctly sets `isError: true`, classifies the error as `rate_limit`, marks it `retryable: true`, and provides actionable guidance (retry after 5 seconds). This structured payload allows the agent to automatically schedule a retry without requiring human intervention.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use#handling-tool-use-and-tool-results
- https://docs.anthropic.com/en/docs/agents-and-tools/mcp

**Why the others are wrong:**

- **A:** Classifying a rate-limit error as `internal` and marking it non-retryable is doubly incorrect. The agent will treat a transient, retryable condition as a permanent failure and will not attempt a retry even though one would succeed.
- **C:** Setting `isError: false` for a failure is a protocol violation. The raw `429` string in the content does not carry machine-readable retry metadata, so the agent cannot determine a correct recovery strategy.
- **D:** While `isError: true` is correct, the message 'Error occurred' gives the agent no information about the error category, retryability, or recommended wait time. The model cannot make an informed decision about how to proceed.

</details>

---

#### C111

*TS 2.2 · Structured error responses*

**Situation:** An MCP tool that queries an internal HR database fails because the requesting agent's service account does not have permission to access salary records. A developer must write the error message that will be returned in the `content` array.

Which error message best follows the MCP guidance of guiding the model's next action without leaking sensitive information?

- **A)** "Access denied to salary records for service account `svc-agent-prod` (ACL rule 14 blocked read on table `hr.salaries`)."
- **B)** "Database error: SELECT on hr.salaries failed with ORA-01031: insufficient privileges for user svc-agent-prod@10.0.0.5."
- **C)** "Permission denied: the agent does not have access to this resource. Contact your administrator to request the `hr:salary:read` permission."
- **D)** "An error occurred."

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Option C conveys the permission category clearly, avoids exposing internal account names, ACL rule IDs, or database details, and gives the model an actionable next step (escalate to an administrator). This matches guidance to write error messages that guide the model without leaking sensitive infrastructure details.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use#handling-tool-use-and-tool-results
- https://docs.anthropic.com/en/docs/agents-and-tools/mcp

**Why the others are wrong:**

- **A:** This message leaks the internal service account name and a specific ACL rule identifier. Exposing these details in an error message is a security risk — they can reveal system internals to any party that can observe the agent's output.
- **B:** Including the raw database error string exposes the database vendor, table schema, username, and internal IP address. This is a significant information-disclosure vulnerability and violates the principle of not leaking sensitive data in error messages.
- **D:** An uninformative message gives the model no signal about what went wrong or what to do next. The agent is likely to retry or hallucinate a solution rather than escalating appropriately.

</details>

---

#### C112

*TS 2.2 · Structured error responses*

**Situation:** Your MCP tool `query_database` can fail in two distinct ways: a transient connection timeout, and a permanent schema validation error caused by a bad user query. The orchestrating Claude agent must handle these differently — retrying on transient failures and escalating on permanent ones.

What is the best practice for structuring MCP tool error responses to enable the agent to distinguish transient from permanent failures?

- **A)** Use two different MCP error codes in the JSON-RPC error object: a retryable code (e.g., -32001) and a permanent code (e.g., -32002), so Claude can branch on the numeric code.
- **B)** Always return `isError: true` with a structured `content` payload that includes a machine-readable `error_type` field (e.g., `"error_type": "transient"` vs `"error_type": "permanent"`) alongside a human-readable message.
- **C)** Return `isError: true` for transient errors only; return a successful result with an `error` field embedded in the JSON body for permanent errors, so Claude uses different code paths.
- **D)** Include both error types in a single error message string and instruct Claude via the system prompt to parse the string for keywords like 'timeout' to decide retry logic.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

The MCP spec's `isError: true` pattern with structured content is the recommended surface for tool errors. Embedding a machine-readable `error_type` field inside the content payload allows the agent to programmatically branch on failure category while preserving the human-readable message for logging and user explanation.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use#handling-tool-use-and-tool-results
- https://docs.anthropic.com/en/docs/agents-and-tools/mcp

**Why the others are wrong:**

- **A:** JSON-RPC error codes in the transport-level error object are reserved for protocol-level failures (e.g., method not found), not application-level tool errors. Using them for business logic violates the MCP specification and may not be surfaced to Claude's reasoning.
- **C:** Mixing `isError: true` for one failure mode and a success envelope with an embedded `error` field for another creates an inconsistent contract. Claude's tool-use reasoning expects a uniform error signal; asymmetric patterns increase the chance of misclassification.
- **D:** Parsing error strings in the system prompt is brittle, locale-sensitive, and extremely difficult to maintain. It conflates orchestration logic with prompt engineering and breaks whenever error message wording changes.

</details>

---

#### C113

*TS 2.2 · Structured error responses*

**Situation:** An MCP server exposes a `run_pipeline` tool used by a long-running Claude agent. The tool can fail with five distinct error types. The team wants a retry policy where the agent retries up to 3 times for retryable errors, immediately escalates `permission` errors to a human, silently skips `not_found` errors, and aborts the entire pipeline on `internal` errors.

Which error response schema design best supports all four routing rules without requiring the agent to parse natural-language error messages?

- **A)** Return `isError: true` and set the HTTP status code of the MCP transport response to 429 for rate limits, 403 for permissions, 404 for not-found, and 500 for internal errors. Instruct the agent to branch on the HTTP code.
- **B)** Use four different tool names (`run_pipeline_safe`, `run_pipeline_retriable`, `run_pipeline_skip`, `run_pipeline_abort`) and route errors by which tool name appeared in the failed call.
- **C)** Return `isError: true` with a structured content payload containing `error_type` (one of: `validation`, `permission`, `not_found`, `rate_limit`, `internal`), `retryable` (boolean), and `action_hint` (one of: `retry`, `escalate`, `skip`, `abort`) fields.
- **D)** Return a single `isError: true` with a `message` field. Encode the routing rule as a prefix in the message string (e.g., `"[RETRY]"` or `"[ESCALATE]"`) and instruct the agent in the system prompt to branch on the prefix.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

A structured content payload with explicit `error_type`, `retryable`, and `action_hint` fields gives the agent three independent machine-readable signals to route decisions without parsing strings. Each field serves a distinct purpose: `error_type` identifies the failure class, `retryable` drives the retry loop, and `action_hint` encodes the specific pipeline action. This schema is maintainable, testable, and requires no system-prompt heuristics.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use#handling-tool-use-and-tool-results
- https://docs.anthropic.com/en/docs/agents-and-tools/mcp
- https://docs.anthropic.com/en/docs/build-with-claude/agents

**Why the others are wrong:**

- **A:** MCP runs over JSON-RPC, not direct HTTP. Transport-level HTTP codes are not propagated into the model's tool result context in standard MCP implementations. Even where HTTP is the transport, using status codes for application-level business logic mixes the transport and application layers.
- **B:** Separate tool names for each error routing path conflate the tool's functional identity with its error-handling behavior. The agent must select a tool based on an error outcome it has not yet observed — a circular design. Tool proliferation also increases schema complexity without benefit.
- **D:** Encoding routing as string prefixes is brittle: prefix formats can diverge across tool implementations, and any prefix change breaks the agent's branching logic without a schema version bump. It also embeds routing logic in the system prompt, making it hard to maintain and test independently.

</details>

---

#### C114

*TS 2.2 · Structured error responses*

**Situation:** An engineering team is building an MCP tool `execute_trade` for a financial services agent. The tool can produce five outcomes: success, a validation error (bad symbol format), a permission error (account not authorized to trade this instrument), a not-found error (symbol does not exist), and an internal error (exchange connectivity failure). A junior developer proposes returning all non-success outcomes as `isError: true` with only a human-readable string message and no structured fields.

What is the most complete and accurate critique of the junior developer's proposal?

- **A)** The proposal is correct for human-facing error display but should add a numeric error code field so the agent can branch programmatically, similar to HTTP status codes.
- **B)** The proposal is flawed only because it does not set `isError: true` consistently; once that is fixed, a human-readable message is sufficient for all five outcomes.
- **C)** The proposal is acceptable for simple agents but should be upgraded to structured fields for production use because structured fields slightly improve performance.
- **D)** The proposal is fundamentally flawed because: (1) relying on Claude to parse natural-language error strings makes error handling brittle and non-deterministic; (2) without a `retryable` field, the agent cannot distinguish the exchange connectivity failure (retryable) from the symbol-not-found error (non-retryable); (3) financial systems require audit-quality structured error logs that natural-language strings cannot reliably support.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

The junior developer's proposal has three distinct, independent flaws. First, language models do not parse error strings with deterministic logic — the same string can produce different agent behaviors across models, temperatures, and context states. Second, without an explicit `retryable` boolean, the agent cannot reliably distinguish a transient exchange connectivity outage from a permanent symbol-not-found condition. Third, in regulated financial contexts, audit trails require structured, machine-parsable error records; free-text strings cannot be reliably indexed, queried, or aggregated for compliance reporting.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use#handling-tool-use-and-tool-results
- https://docs.anthropic.com/en/docs/agents-and-tools/mcp
- https://docs.anthropic.com/en/docs/build-with-claude/agents

**Why the others are wrong:**

- **A:** A numeric code is an improvement over pure strings, but HTTP-style numeric codes are not the recommended MCP pattern. Named `error_type` strings are more readable, self-documenting, and need no external code-to-meaning lookup table. The option also does not address the retryability gap.
- **B:** The proposal already implies `isError: true` is used; the option misidentifies the problem. The actual flaws are the lack of machine-readable categorical fields and retryability metadata, not a missing `isError` flag.
- **C:** Framing the issue as a 'performance' concern understates the severity. The problem is one of correctness, security, and compliance — not performance. In a financial trading context, a misclassified retryable error could cause duplicate trade submissions with real monetary consequences.

</details>

---

#### C115

*TS 2.2 · Structured error responses*

**Situation:** An MCP tool `send_email` fails because the recipient address is malformed. The tool implementation must return an error back to the Claude agent that called it.

What is the correct MCP protocol mechanism for returning a tool-level error so that Claude can reason about the failure and decide on a recovery action?

- **A)** Return an empty result object; Claude will re-invoke the tool with different parameters after detecting an empty response.
- **B)** Return an HTTP 500 status code from the MCP server; Claude will interpret non-200 responses as tool failures.
- **C)** Return a result object with `isError: true` and include a descriptive error message in the `content` array.
- **D)** Throw an unhandled exception inside the tool handler; the MCP SDK will automatically convert it to a structured error.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

The MCP specification defines that tool-level errors should be communicated by returning a result with `isError: true` alongside a human-readable message in `content`. This allows Claude to see the error within the conversation flow and plan a recovery strategy.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use#handling-tool-use-and-tool-results
- https://docs.anthropic.com/en/docs/agents-and-tools/mcp

**Why the others are wrong:**

- **A:** An empty result is not a defined error signal in MCP. Claude has no mechanism to distinguish an intentionally empty result from a silent failure, so it will not automatically retry with different parameters.
- **B:** MCP communicates over JSON-RPC, not raw HTTP response codes. Transport-level errors differ from tool-level errors; returning an HTTP 500 would terminate the request at the transport layer rather than giving Claude actionable error content.
- **D:** While some SDKs may catch exceptions and convert them, relying on unhandled exceptions is not the defined MCP error protocol. It produces inconsistent behavior across implementations and does not guarantee a structured, parsable error message for the model.

</details>

---

#### C116

*TS 2.2 · Structured error responses*

**Situation:** A team is designing a structured error metadata schema for all MCP tools in their platform. They want Claude to be able to programmatically distinguish error categories, determine retryability, and receive a human-readable explanation — all from a single error response.

Which structured error metadata schema best supports the agent's ability to reason about failures and take appropriate recovery actions?

- **A)** `{ "error_type": "validation", "retryable": false, "message": "Field 'email' must be a valid email address.", "details": { "field": "email", "received": "not-an-email" } }`
- **B)** `{ "code": 400, "message": "Bad input" }`
- **C)** `{ "status": "error", "description": "The input was incorrect. Please try again." }`
- **D)** `{ "error": true, "type": 3, "hint": "fix input" }`

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Option A provides all four key components of structured error metadata: a machine-readable `error_type` for categorical routing, a `retryable` boolean so the agent knows whether to retry, a human-readable `message` explaining the failure, and a `details` object with field-level context for precise correction. This enables the agent to take targeted, informed action.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use#handling-tool-use-and-tool-results
- https://docs.anthropic.com/en/docs/agents-and-tools/mcp

**Why the others are wrong:**

- **B:** An HTTP-style numeric code and a vague message do not convey retryability or field-level details. The agent must guess whether to retry and has no structured path to correct the specific invalid input.
- **C:** A plain-language description without machine-readable fields forces the agent to parse natural language to determine category and retryability, which is fragile and inconsistent across different error conditions.
- **D:** Using an opaque numeric type (e.g., `3`) instead of a named string requires the agent to consult external documentation, and a vague hint like 'fix input' gives no actionable guidance on what to actually fix.

</details>

---

#### C117

*TS 2.2 · Structured error responses*

**Situation:** A Claude agent calls an MCP tool `create_order` which internally calls a payment processor. The payment processor returns a `card_declined` error. The developer must decide whether to classify this as retryable or non-retryable and write an appropriate error message.

Which error classification and message is most appropriate for a card-declined failure?

- **A)** Retryable; message: "Payment failed temporarily. Retry the order in a few seconds."
- **B)** Non-retryable; message: "Card declined. Please ask the user to provide a different payment method or check their card details."
- **C)** Retryable; message: "An internal error occurred. The system will automatically retry."
- **D)** Non-retryable; message: "Error code CVD-4042 from Stripe: insufficient_funds for card ending 4242."

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

A card-declined error is a permanent failure for that specific payment attempt — retrying with the same card will not succeed. Classifying it as non-retryable and guiding the agent to request an alternative payment method is the correct action. The message avoids leaking card details or processor-specific error codes.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use#handling-tool-use-and-tool-results
- https://docs.anthropic.com/en/docs/agents-and-tools/mcp

**Why the others are wrong:**

- **A:** Card declines are not transient network errors — retrying the same declined card will produce the same result. Marking this retryable causes the agent to loop uselessly without resolving the underlying problem.
- **C:** Classifying a card decline as a retryable internal error is doubly wrong: the error is permanent (not internal), and automatic retries will not resolve a declined card. This causes unnecessary retries and delays user resolution.
- **D:** Non-retryable is correct, but this message leaks sensitive financial data (the last four digits of the card and the Stripe error code). This violates the guidance to avoid exposing sensitive information in error messages.

</details>

---

#### C118

*TS 2.2 · Structured error responses*

**Situation:** An MCP tool that provisions cloud resources fails because the user's account has reached its quota for virtual machines. The developer must write an error response. The quota limit is a hard organizational policy, not a temporary rate limit.

How should the developer classify and communicate this quota-exceeded failure?

- **A)** Error type `rate_limit`, retryable `true`; message: "VM quota exceeded. Retry after the quota window resets."
- **B)** Error type `permission`, retryable `false`; message: "Your account has reached its VM quota. Contact your cloud administrator to request a quota increase."
- **C)** Error type `internal`, retryable `true`; message: "Resource provisioning failed due to a system constraint. Retrying shortly."
- **D)** Error type `not_found`, retryable `false`; message: "Virtual machine resource not found in this region."

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

A hard organizational quota is an authorization/policy boundary rather than a transient rate limit or missing resource. Classifying it as `permission` with `retryable: false` accurately reflects that the current account is not authorized to exceed its quota. The message directs the agent to escalate to an administrator, which is the only viable next action.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use#handling-tool-use-and-tool-results
- https://docs.anthropic.com/en/docs/agents-and-tools/mcp

**Why the others are wrong:**

- **A:** `rate_limit` implies a time-based throttle that will reset automatically, which is incorrect for a hard quota enforced by organizational policy. Marking it retryable would cause the agent to loop indefinitely waiting for a reset that never comes.
- **C:** Classifying a policy-driven quota error as `internal` and retryable is misleading on both counts. The agent will attempt automatic retries expecting a transient server issue to resolve, when in fact an administrator action is required.
- **D:** `not_found` implies the requested resource class does not exist in the system. Misclassifying a quota error this way causes the agent to search for the resource in alternative locations rather than escalating the quota issue.

</details>

---

#### C119

*TS 2.2 · Structured error responses*

**Situation:** A tool call returns an invalid result. You need to inform the model that the tool failed so it can adjust its approach.

When a tool call fails and you want Claude to retry with corrected parameters, which field in the tool_result block signals that the previous attempt produced an error?

- **A)** isError: true
- **B)** error: true
- **C)** retry: true
- **D)** status: 'failed'

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

The Messages API uses the boolean field `isError: true` inside a tool_result content block to indicate that execution failed. Claude reads this flag and uses the accompanying content as error context when deciding how to adjust.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use#handling-tool-use-and-tool-results

**Why the others are wrong:**

- **B:** `error: true` is not a recognised field in the Anthropic tool_result schema; using it would be silently ignored.
- **C:** `retry: true` is not a valid field; retry decisions are made by the orchestrator, not communicated via this flag.
- **D:** `status: 'failed'` is not part of the tool_result schema and would not be interpreted as an error signal.

</details>

---

#### C120

*TS 2.2 · Structured error responses*

**Situation:** A pipeline processes financial transactions. On retry 2, the model produces a tool call that passes schema validation but the downstream service returns a business-logic error: 'Insufficient funds in source account'. The orchestrator must decide whether to retry, escalate, or abort.

How should the orchestrator handle this business-logic error compared to a schema validation error?

- **A)** Increase the retry limit to 10 because financial errors require more attempts
- **B)** Recognise that this is a state-of-the-world constraint that Claude cannot resolve by changing parameters; escalate immediately to a human or abort the transaction
- **C)** Retry automatically after a 60-second delay to allow the account balance to change
- **D)** Treat it identically to a schema error: return `isError: true` with the message and let Claude retry with corrected parameters

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

A business-logic error like 'Insufficient funds' reflects an external state constraint that Claude cannot fix by adjusting its output. Retrying will not resolve it. The correct response is immediate escalation or abort, preserving the integrity of the financial transaction and following the principle of not persisting with actions that require conditions outside the model's control.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents/build-effective-agents
- https://docs.anthropic.com/en/docs/build-with-claude/agents/agent-security
- https://docs.anthropic.com/en/docs/build-with-claude/agents#human-in-the-loop

**Why the others are wrong:**

- **A:** Increasing the retry limit for financial errors introduces more risk of duplicate transactions and does not address the underlying constraint.
- **C:** Waiting 60 seconds on the assumption that the balance might change is speculative and introduces unacceptable latency and unpredictability in financial workflows.
- **D:** Treating a state-of-the-world constraint as a correctable schema error is a category mistake; Claude cannot generate parameters that create funds that do not exist.

</details>

---

#### C121

*TS 2.2 · Structured error responses*

**Situation:** You are building a pipeline that calls an external API. Sometimes the API returns a transient 503 error unrelated to Claude's parameters. Other times, Claude generates a malformed request body that causes a 400 error.

How should the retry strategy differ between a transient 503 error and a 400 validation error?

- **A)** A 503 should be retried with the same parameters (possibly with backoff); a 400 should include the specific validation error in the tool_result so Claude can correct the request
- **B)** Both should escalate to a human immediately without retrying
- **C)** A 400 should be retried with the same parameters; a 503 should be retried with different parameters
- **D)** Both should be retried immediately with the same parameters

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Transient errors (503) are not caused by bad parameters, so retrying with identical parameters is appropriate. Validation errors (400) indicate bad input from Claude; the error detail must be fed back via `isError: true` so Claude can generate a corrected request.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use#handling-tool-use-and-tool-results
- https://docs.anthropic.com/en/docs/build-with-claude/agents/build-effective-agents

**Why the others are wrong:**

- **B:** Immediate escalation on a transient 503 is wasteful; most transient errors resolve on retry without human intervention.
- **C:** This reverses the correct logic: 503s do not require parameter changes, while 400s do.
- **D:** Retrying a 400 with the same parameters will always fail because the issue is in Claude's output, not the service state.

</details>

---

#### C122

*TS 2.2 · Structured error responses*

The `process_refund` tool fails with a generic error: `{"isError": true, "content": "Operation failed"}`. The agent responds to the customer: "I'm sorry, something went wrong. Please try again later." What improvement to the tool's error response would most help the agent?

- **A)** Return an HTTP status code (e.g., 500, 403) so the agent can look up the standard meaning.
- **B)** Return structured metadata: `errorCategory` (transient/validation/business/permission), `isRetryable`, and a `customer_explanation` field.
- **C)** Include a `suggested_action` field with exactly what the agent should say to the customer verbatim.
- **D)** Include a detailed stack trace so the agent can diagnose the technical issue and explain it to the customer.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Structured error metadata (errorCategory, isRetryable, customer_explanation) lets the agent make appropriate recovery decisions.

**Why the others are wrong:**

- **A:** HTTP codes lack context.
- **C:** Verbatim scripts are too rigid.
- **D:** Stack traces are too technical for customers.

</details>

---

#### C123

*TS 2.2 · Structured error responses*

The `lookup_order` tool sometimes returns empty results (`{"orders": []}`) when the customer provides a valid order number. Investigation reveals this happens during database maintenance windows. The agent currently treats this as "no order found" and tells the customer their order doesn't exist. How should the error handling be improved?

- **A)** Add retry logic with exponential backoff inside the tool, returning results once the database is available.
- **B)** Distinguish between a successful query returning no matches (`isError: false`, empty results = no matching order) and an access failure (`isError: true`, `errorCategory: "transient"`, `isRetryable: true` = database unavailable).
- **C)** Cache order data locally so the agent can serve results even when the database is down.
- **D)** Add a message to the system prompt: "If `lookup_order` returns empty, inform the customer that the system may be temporarily unavailable."

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

The tool must distinguish between "no matches found" (valid result, `isError: false`) and "database unavailable" (transient error, `isError: true`, retryable). Caching (C) and prompt changes (D) don't address the core issue.

**Why the others are wrong:**

- **A:** Retry logic helps but doesn't fix the reporting.

</details>

---

#### C124

*TS 2.2 · Structured error responses*

Your `update_account` tool returns different errors for different failure modes, but all use the same format: `{"success": false, "error": "Update failed"}`. This prevents the agent from distinguishing between a locked account (needs manager unlock), an invalid email format (needs correction), and a system timeout (needs retry). How should the tool be improved?

- **A)** Log errors server-side and have the agent check an error log endpoint.
- **B)** Add error codes (500, 400, 403) matching HTTP conventions.
- **C)** Return structured error metadata: `{"isError": true, "errorCategory": "validation|permission|transient", "isRetryable": true|false, "message": "...", "customer_explanation": "..."}`.
- **D)** Return different error messages for each failure type (but keep the same `success: false` format).

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Structured error metadata with `errorCategory`, `isRetryable`, and `customer_explanation` enables appropriate agent responses.

**Why the others are wrong:**

- **A:** Error logs are async and slow.
- **B:** HTTP codes lack context.
- **D:** Same-format errors don't fix distinguishability.

</details>

---

#### C125

*TS 2.3 · Tool distribution & tool_choice*

**Situation:** An architect is choosing between two designs for a customer support automation system. Design 1: a single large agent with access to all tools (CRM lookup, order management, refund processing, knowledge base search). Design 2: a coordinator with four specialized subagents, each with access to only the tools relevant to its specialty. Both designs are technically feasible.

Which argument most accurately captures the primary advantage of Design 2 (coordinator + specialized subagents) over Design 1 (single large agent) for a production system?

- **A)** Design 2 eliminates the need for error handling because specialized subagents never fail on tasks within their specialty.
- **B)** Design 2 is cheaper because each subagent uses fewer input tokens than the single large agent would.
- **C)** Design 2 is always faster because parallel execution is inherently faster than sequential tool calls in a single agent.
- **D)** Design 2 provides stronger least-privilege isolation: each subagent can only use the tools it needs, limiting the blast radius of a prompt injection or model error to a single specialized subagent rather than exposing all tools to a single error surface.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

The primary production advantage of coordinator + specialized subagents is least-privilege tool isolation. A single agent with all tools represents a large attack surface: a prompt injection that hijacks the agent can trigger any tool, including destructive ones like refund processing. In Design 2, a compromised summarization subagent cannot access the refund tool because it was never granted it. The coordinator enforces tool scope boundaries at spawn time, minimizing blast radius.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems
- https://docs.anthropic.com/en/docs/build-with-claude/agents/tool-use-security

**Why the others are wrong:**

- **A:** Specialization reduces the domain of tasks a subagent handles but does not eliminate failures; transient errors, malformed inputs, and unexpected edge cases still occur within a subagent's specialty. Error handling remains mandatory.
- **B:** Design 2 may actually cost more in tokens overall because each subagent invocation incurs system prompt overhead and context setup; token cost is not the primary argument for specialization.
- **C:** Parallelism is a potential throughput benefit but is not guaranteed to make Design 2 faster than Design 1 in all cases — if customer support requests are naturally sequential (each step depends on the previous), the overhead of spawning subagents can actually make Design 2 slower.

</details>

---

#### C126

*TS 2.3 · Tool distribution & tool_choice*

**Situation:** A multi-agent system processes patient health records. A coordinator spawns subagents to perform diagnosis assistance, billing code generation, and appointment scheduling. A security review finds that the billing subagent, which only needs to write to a billing database, was inadvertently granted access to `file_read`, `database_write`, and `send_email` tools.

An attacker crafts a malicious prompt injection via a patient record that causes the billing subagent to exfiltrate data. Which combination of controls would have been most effective at preventing the exfiltration?

- **A)** Add a second subagent to review the billing subagent's outputs before they are committed.
- **B)** Sanitise all patient records before they reach any subagent, and log all tool calls for post-hoc review.
- **C)** Restrict the billing subagent's `allowed_tools` to `["database_write"]` only, and require human approval before the subagent executes any irreversible action.
- **D)** Encrypt all data passed to subagents so that exfiltrated content is unreadable to the attacker.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

The two most effective controls are: (1) `allowed_tools: ["database_write"]` — this deterministically removes `file_read` and `send_email`, which are the tools a prompt injection would need to read sensitive data and exfiltrate it; and (2) human-in-the-loop approval for irreversible actions, which adds an out-of-band verification step that an automated attack cannot bypass. Together they apply least-privilege and explicit authorisation.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents/agent-security
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems

**Why the others are wrong:**

- **A:** A reviewing subagent adds a check on the content of outputs but does not prevent the billing subagent from calling `send_email` or `file_read` directly — the reviewing agent only sees what the billing agent chooses to surface.
- **B:** Input sanitisation reduces the attack surface but cannot guarantee that all injection vectors are caught, especially in unstructured medical text. Post-hoc logging detects attacks after the fact but does not prevent exfiltration.
- **D:** Encryption protects data in transit between systems but does not prevent the subagent from using its permitted tools to read and then transmit data. If `send_email` is available, the subagent can email plaintext content regardless of how the data was passed to it.

</details>

---

#### C127

*TS 2.3 · Tool distribution & tool_choice*

**Situation:** A senior architect is reviewing a multi-agent pipeline where a coordinator orchestrates subagents for a financial trading system. The coordinator passes trade parameters to subagents that have access to `execute_trade`, `read_portfolio`, and `send_notification` tools. The architect identifies that a compromised subagent could cause significant financial harm.

Which combination of architectural controls best limits the blast radius of a compromised subagent while maintaining system functionality?

- **A)** Scope each subagent's `allowed_tools` to only what its subtask requires, require a human confirmation step before any `execute_trade` call, and validate all subagent outputs against a schema before the coordinator acts on them.
- **B)** Use a single highly trusted subagent with all tools, reducing the attack surface by minimising the number of agents.
- **C)** Run all subagents with the full tool set but in a sandbox environment that records all tool calls for audit.
- **D)** Encrypt all trade parameters passed to subagents so that a compromised subagent cannot read the values it is operating on.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Three controls working in combination provide defence-in-depth: (1) `allowed_tools` scoping ensures each subagent can only invoke tools needed for its specific subtask, deterministically limiting lateral movement; (2) human-in-the-loop confirmation before `execute_trade` ensures no irreversible financial action occurs without out-of-band verification; and (3) output schema validation at the coordinator level catches malformed or anomalous responses before they trigger downstream actions. Together these limit blast radius at the tool, action, and data layers.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents/agent-security
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems
- https://docs.anthropic.com/en/docs/claude-code/sdk

**Why the others are wrong:**

- **B:** Reducing the number of agents does not reduce blast radius — a single highly-trusted subagent with all tools is a single point of total failure. If compromised, an attacker gains access to all tools simultaneously with no lateral-movement barriers.
- **C:** Sandboxing and audit logging are detective controls, not preventive ones. A compromised subagent with the full tool set can still execute trades and send notifications in real time; the audit trail only captures damage after it has occurred.
- **D:** A subagent must be able to read the parameters it is processing in order to function. Encrypting those values would prevent the subagent from operating at all. Encryption protects data in transit, not against a compromised runtime that already has decrypted access.

</details>

---

#### C128

*TS 2.3 · Tool distribution & tool_choice*

**Situation:** A security audit of a multi-agent pipeline reveals that a research subagent has access to `bash`, `file_write`, and `web_search` tools, but its only actual task is to search the web and return summaries. A prompt injection vulnerability was found in one of the websites it visits.

Which remediation most directly addresses the security risk by applying the least-privilege principle?

- **A)** Restrict the subagent's `allowed_tools` to `["web_search"]` only, removing `bash` and `file_write`.
- **B)** Increase the subagent's context window size to detect injection attempts more reliably.
- **C)** Add input sanitisation logic inside the subagent's system prompt.
- **D)** Run the subagent in a separate network zone with no outbound connections.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

The principle of least privilege demands that the subagent only has access to the tools it needs — in this case, only `web_search`. By setting `allowed_tools: ["web_search"]`, a prompt injection attack cannot leverage `bash` (arbitrary command execution) or `file_write` (data exfiltration or persistence) even if the model is manipulated. This is a deterministic, architectural control rather than a heuristic one.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents/agent-security
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems

**Why the others are wrong:**

- **B:** Context window size is irrelevant to detecting or preventing prompt injection; it is a capacity parameter, not a security control.
- **C:** Prompt-based sanitisation is a soft, heuristic control that a sufficiently clever injection can bypass. It does not prevent tool misuse at the enforcement layer.
- **D:** Network isolation is a useful defence-in-depth measure but does not prevent the subagent from misusing `bash` or `file_write` against internal resources or its own filesystem.

</details>

---

#### C129

*TS 2.3 · Tool distribution & tool_choice*

**Situation:** A team is designing a multi-agent pipeline for code review. A coordinator receives a pull request diff and wants to spawn specialist subagents: one for security review, one for performance review, and one for style review. Each subagent should only be able to call tools relevant to its specialty.

Which invocation strategy best implements scoped tool access for each specialist subagent?

- **A)** Use a single subagent with all tools and a combined prompt for all three review types.
- **B)** Invoke each subagent with a distinct `allowed_tools` list scoped to its specialty, passing only the diff relevant to each.
- **C)** Define a global `allowed_tools` list at the coordinator level that all subagents inherit automatically.
- **D)** Give all three subagents access to all tools and rely on their system prompts to instruct them not to use irrelevant ones.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Each specialist subagent should receive an `allowed_tools` list containing only the tools relevant to its function — e.g., the security reviewer may need a CVE database lookup tool, while the style reviewer only needs text analysis tools. This enforces least-privilege per subagent and prevents a compromised agent from misusing tools outside its domain.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems
- https://docs.anthropic.com/en/docs/claude-code/sdk

**Why the others are wrong:**

- **A:** A single subagent handling all review types creates a large, complex context and cannot be granted different tool scopes for different subtasks. It also prevents parallel execution.
- **C:** There is no global `allowed_tools` inheritance in the Agent SDK. Each Task invocation must explicitly specify its own `allowed_tools`; subagents do not automatically inherit coordinator-level settings.
- **D:** Relying on prompt instructions to restrict tool use is a soft control. The model may still call tools not intended for it, especially under adversarial conditions. `allowed_tools` is a deterministic enforcement mechanism that prompt instructions cannot replicate.

</details>

---

#### C130

*TS 2.3 · Tool distribution & tool_choice*

**Situation:** A team builds a research assistant with 20 tools spanning web search, document management, calendar access, email, code execution, and database queries. They notice the overall tool-call accuracy degrades as they add more tools, even when each tool has a well-written individual description.

Which architectural strategy best addresses description overload in a large tool suite?

- **A)** Assign a numeric relevance score to each tool and include it in the tool name, e.g., `95_search_web`.
- **B)** Write longer descriptions for each tool so that more detail helps Claude disambiguate.
- **C)** Increase `max_tokens` to 8192 on every API call to give Claude more space to reason through all 20 tool descriptions.
- **D)** Dynamically inject only the subset of tools relevant to the current task context rather than passing all 20 tools on every call, using a lightweight routing layer that selects the appropriate toolset based on intent classification.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

When a large number of tools are present, even high-quality individual descriptions create a signal-to-noise problem: Claude must consider many tool descriptions simultaneously, and the probability of cross-tool confusion grows. The architectural solution is to dynamically scope the tool context — inject only the tools relevant to the current intent. This is a well-documented pattern in agentic system design: a lightweight classifier or router selects the active toolset per turn, dramatically reducing the disambiguation surface.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/overview
- https://docs.anthropic.com/en/docs/build-with-claude/agents-and-tools/build-effective-agents

**Why the others are wrong:**

- **A:** Numeric relevance scores in tool names are not parsed as priority signals by Claude; tool selection is based on description semantics, and cluttering names with scores reduces readability without providing a selection benefit.
- **B:** Longer descriptions per tool add more tokens to the context, worsening the overload problem rather than solving it; the issue is the number of tools competing for attention, not the per-tool description quality.
- **C:** `max_tokens` controls output length, not the model's reasoning capacity over input; injecting all 20 tool descriptions into the context is a context-length problem, not a generation-length problem, and more output tokens do not help.

</details>

---

#### C131

*TS 2.3 · Tool distribution & tool_choice*

**Situation:** A developer is configuring an agent that must always call at least one tool before returning a response. The agent has four tools available and the developer wants to prevent the model from answering from its parametric knowledge alone.

Which `tool_choice` setting ensures the model always invokes at least one tool?

- **A)** `{"type": "any"}`
- **B)** `{"type": "tool", "name": "search"}`
- **C)** `{"type": "auto"}`
- **D)** Omitting `tool_choice` entirely

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Setting `tool_choice` to `{"type": "any"}` forces the model to call at least one tool from the provided list before responding. The model still selects which tool to use, but it cannot skip tool use entirely. This is the correct setting when you need guaranteed grounding in external data rather than parametric knowledge.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use#controlling-claudes-output
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use#tool-use-best-practices

**Why the others are wrong:**

- **B:** A forced tool name (`{"type": "tool", "name": "search"}`) guarantees that one specific tool is called, not merely that some tool is called. This is a stricter constraint than required and prevents the model from selecting the most appropriate tool.
- **C:** `auto` allows the model to decide whether to call a tool at all. If the model believes it can answer from training data it will bypass all tools, defeating the developer's requirement.
- **D:** Omitting `tool_choice` defaults to `auto` behavior, giving the model full discretion to skip tool use, which violates the stated requirement.

</details>

---

#### C132

*TS 2.3 · Tool distribution & tool_choice*

**Situation:** A developer equips a general-purpose assistant agent with 30 different tools spanning calendar management, email, code execution, database queries, file operations, and web search. After deployment, users report that the agent frequently selects the wrong tool or chains unnecessary tool calls for simple requests.

According to Anthropic's guidance on tool distribution, what is the most likely root cause of the agent's poor tool selection?

- **A)** Too many tools create selection confusion, making it harder for the model to identify the most appropriate tool
- **B)** The agent's system prompt is too short
- **C)** The tools are missing required parameters in their JSON schemas
- **D)** The model's context window is too small for the conversation history

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Anthropic's documentation explicitly warns that providing too many tools degrades the model's ability to select the right one. When the tool list is large and diverse, Claude must reason over many candidates simultaneously, increasing the chance of a suboptimal or incorrect selection. The recommended pattern is to scope tool sets tightly per agent role.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use#best-practices-for-tool-definitions
- https://docs.anthropic.com/en/docs/build-with-claude/agents#designing-effective-agents

**Why the others are wrong:**

- **B:** A short system prompt can reduce context but is not the described failure mode. The symptom—wrong tool selection across many capabilities—points to tool proliferation, not prompt length.
- **C:** Missing required parameters would cause schema validation errors or tool call failures, not the pattern of selecting the wrong tool or making unnecessary chains of calls.
- **D:** Context window exhaustion would manifest as truncated conversations or errors, not as systematic wrong-tool selection on presumably simple requests.

</details>

---

#### C133

*TS 2.3 · Tool distribution & tool_choice*

**Situation:** An architect must design a dynamic tool-provisioning system where a coordinator agent can spawn subagents on demand. Each subagent instance should receive only the tools relevant to the specific subtask it was spawned for, determined at runtime based on the task classification. The coordinator classifies tasks into three categories: 'data-retrieval', 'data-transformation', and 'data-publishing'.

Which design best implements dynamic least-privilege tool provisioning in this multi-agent system?

- **A)** Define one universal subagent with all tools and rely on system prompt instructions to restrict which tools it may call for each task category
- **B)** Pre-define three subagent configurations—one per task category—each with a hard-coded tool list; the coordinator selects the appropriate pre-defined configuration when spawning a subagent instance
- **C)** Give the coordinator the ability to dynamically append tools to a subagent's tool list after spawning, adding only the tools the coordinator deems necessary based on runtime context
- **D)** Give all subagents the same base tool list and use `tool_choice: any` to force them to always call a tool, ensuring they don't skip necessary operations

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Pre-defining three distinct configurations with hard-coded, minimal tool lists per task category is the safest and most architecturally sound approach. The coordinator selects a configuration—it does not construct one dynamically—which means the tool sets are reviewed and approved at design time, not runtime. This prevents a compromised coordinator from granting excessive tools to a subagent and maintains least privilege as a static, auditable property of the system.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents#security-and-trust
- https://docs.anthropic.com/en/docs/build-with-claude/agents#multi-agent-architectures
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use#best-practices-for-tool-definitions

**Why the others are wrong:**

- **A:** A universal subagent with all tools and system-prompt-based restrictions is the weakest design. System prompt instructions are a soft control that can be bypassed by injections or model errors. The subagent always possesses the capability to misuse tools even if instructed not to.
- **C:** Allowing the coordinator to dynamically append tools at runtime is dangerous. A compromised or manipulated coordinator could grant a subagent tools outside its intended scope, and dynamic tool grants are difficult to audit and govern.
- **D:** Giving all subagents the same base tool list eliminates role-based isolation. A data-retrieval subagent should not have publishing tools; uniform tool lists across all categories violates least privilege.

</details>

---

#### C134

*TS 2.3 · Tool distribution & tool_choice*

**Situation:** A developer is building a structured pipeline with three sequential stages: (1) an extraction stage where Claude must always call `parse_document` to extract fields; (2) a validation stage where Claude should call `validate_schema` if the extracted data looks non-trivial, but may skip it for trivially valid data; (3) an output stage where Claude must always call exactly `emit_result` to write the final record. The developer wants to use the minimum necessary constraint at each stage.

Which `tool_choice` sequence—[Stage 1, Stage 2, Stage 3]—applies the minimum necessary constraint at each stage?

- **A)** `[auto, auto, auto]`
- **B)** `[{"type":"tool","name":"parse_document"}, any, {"type":"tool","name":"emit_result"}]`
- **C)** `[any, any, {"type":"tool","name":"emit_result"}]`
- **D)** `[{"type":"tool","name":"parse_document"}, auto, {"type":"tool","name":"emit_result"}]`

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Stage 1 requires a guaranteed call to a specific tool (`parse_document`), so forced tool choice is the minimum necessary constraint. Stage 2 requires conditional tool use—Claude should decide based on context—so `auto` is the minimum constraint (it allows both calling and not calling the tool). Stage 3 requires a guaranteed call to a specific tool (`emit_result`), so forced tool choice again. Option D is the only sequence that satisfies all three requirements with the minimum constraint at each stage.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use#controlling-claudes-output
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use#forcing-tool-use

**Why the others are wrong:**

- **A:** Using `auto` for stages 1 and 3 allows Claude to skip `parse_document` and `emit_result`, breaking the mandatory execution guarantee in both stages. The pipeline would silently produce incomplete results.
- **B:** Using `any` for stage 2 is an over-constraint. It forces Claude to always call some validation tool even for trivially valid data where the developer explicitly wants the model to be able to skip validation. `auto` is the correct minimum constraint for that stage.
- **C:** Using `any` for stage 1 forces tool use but does not guarantee `parse_document` specifically if other tools are in context. Using `any` for stage 2 forces Claude to call some tool even when the data is trivially valid, violating the minimum-constraint principle for that stage.

</details>

---

#### C135

*TS 2.3 · Tool distribution & tool_choice*

**Situation:** A team is migrating a monolithic agent (24 tools, frequent wrong-tool selections) to a hierarchical multi-agent architecture. The new design must preserve all 24 capabilities while fixing tool selection reliability. The team proposes two alternatives: Alternative X groups tools into 4 domain-specific subagents of 6 tools each, with a coordinator that routes via 4 delegation tools. Alternative Y creates 24 single-tool subagents, with a coordinator that routes via 24 delegation tools.

Evaluating both alternatives against Anthropic's tool distribution guidance, which statement is most accurate?

- **A)** Alternative Y is superior because each subagent has exactly one tool, eliminating all within-subagent selection ambiguity
- **B)** Alternative X is superior because it reduces within-subagent selection ambiguity while keeping the coordinator's routing decision space manageable (4 choices vs 24)
- **C)** Both alternatives are equivalent because the total number of tools in the system is the same in both cases
- **D)** Alternative Y is superior because having more agents increases parallelism and therefore throughput

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Alternative X correctly applies Anthropic's guidance at two levels: (1) each subagent has 6 focused domain tools, which is a manageable set that reduces within-subagent selection confusion; (2) the coordinator has only 4 routing choices, which is a small enough set to make high-confidence routing decisions. Alternative Y eliminates within-subagent confusion but pushes all the selection complexity to the coordinator, which now faces 24 options—recreating the original selection confusion problem at the routing layer.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents#designing-effective-agents
- https://docs.anthropic.com/en/docs/build-with-claude/agents#multi-agent-architectures
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use#best-practices-for-tool-definitions

**Why the others are wrong:**

- **A:** Alternative Y trades within-subagent confusion for coordinator confusion. The coordinator now has 24 delegation tools to choose from, which is the same size as the original monolithic tool list. The selection problem is relocated, not solved.
- **C:** The total number of tools in the system is irrelevant; what matters is how many tools any single agent must reason over simultaneously. The two alternatives differ significantly in coordinator selection complexity.
- **D:** Parallelism is a separate concern from tool selection reliability. Alternative Y may enable more parallelism in some scenarios, but it does not improve selection accuracy and introduces coordinator overload. The question asks about tool distribution guidance, not throughput optimization.

</details>

---

#### C136

*TS 2.3 · Tool distribution & tool_choice*

**Situation:** A pipeline developer notices that when their agent is given `tool_choice: auto` and a large tool list, it occasionally calls `web_search` for questions that should be answered by the internal `knowledge_base_query` tool. Switching to `tool_choice: any` did not fix the problem.

What is the most effective architectural change to improve correct tool selection without switching to forced tool choice?

- **A)** Segment the agent into specialized subagents, each with a smaller, focused tool set relevant to its domain
- **B)** Add a post-processing step that detects wrong tool calls and automatically retries
- **C)** Increase the model's temperature so it explores more options before selecting
- **D)** Rename the tools to have longer, more descriptive names

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

When a large heterogeneous tool list causes selection confusion, the architectural remedy is to decompose the single agent into specialized subagents. Each subagent receives only the tools relevant to its domain (e.g., the internal-knowledge subagent gets `knowledge_base_query`; the external-research subagent gets `web_search`). A coordinator routes requests to the appropriate subagent, eliminating the multi-tool ambiguity problem at its root.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents#designing-effective-agents
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use#best-practices-for-tool-definitions

**Why the others are wrong:**

- **B:** A retry loop addresses symptoms rather than the root cause. It also adds latency and cost, and if the model consistently misidentifies the right tool, retries will not reliably converge on the correct answer.
- **C:** Increasing temperature increases randomness in sampling. It does not improve the model's reasoning about which tool is most appropriate and would likely make selection less consistent, not more.
- **D:** Longer tool names can marginally improve disambiguation but do not address the fundamental problem of a large tool list overwhelming the model's selection reasoning. This is a superficial fix.

</details>

---

#### C137

*TS 2.3 · Tool distribution & tool_choice*

**Situation:** A developer is implementing a two-step agentic workflow. In step one, Claude should always call `gather_context` to retrieve relevant background information. In step two, Claude should reason over that information and choose whether to call `finalize_report`, `request_more_data`, or neither if it determines no action is needed.

Which pair of `tool_choice` settings correctly models this two-step workflow?

- **A)** Step 1: `{"type": "any"}`; Step 2: `{"type": "tool", "name": "request_more_data"}`
- **B)** Step 1: `{"type": "any"}`; Step 2: `{"type": "any"}`
- **C)** Step 1: `{"type": "tool", "name": "gather_context"}`; Step 2: `{"type": "auto"}`
- **D)** Step 1: `{"type": "auto"}`; Step 2: `{"type": "tool", "name": "finalize_report"}`

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Step 1 requires a guaranteed call to a specific tool, which maps exactly to the forced `{"type": "tool", "name": "gather_context"}` setting. Step 2 requires flexible reasoning where the model may or may not call a tool, which maps to `auto`. Pairing forced with auto across the two steps correctly captures the described requirements.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use#controlling-claudes-output
- https://docs.anthropic.com/en/docs/build-with-claude/agents#agentic-workflows

**Why the others are wrong:**

- **A:** Using `any` for step 1 forces tool use but does not guarantee `gather_context` specifically—Claude could call a different available tool. Step 2 forcing `request_more_data` prevents Claude from ever finalizing the report.
- **B:** Using `any` for both steps forces Claude to call some tool in step 2, which violates the requirement that Claude may determine no action is needed. `any` prevents a no-tool response.
- **D:** Using `auto` for step 1 allows Claude to skip `gather_context` if it thinks it already has enough context, breaking the mandatory data-gathering guarantee. Step 2 forcing `finalize_report` prevents Claude from choosing `request_more_data` or no action.

</details>

---

#### C138

*TS 2.3 · Tool distribution & tool_choice*

**Situation:** An architect is designing tool access for a coordinator agent and its two subagents. The coordinator routes tasks and synthesizes final answers. Subagent A handles file I/O operations; Subagent B handles API calls to external services. The coordinator does not directly perform file or API operations.

Which tool distribution correctly applies the principle of least privilege in this architecture?

- **A)** Coordinator: no tools; Subagent A: all tools; Subagent B: all tools
- **B)** Coordinator: `delegate_to_subagent_a`, `delegate_to_subagent_b`; Subagent A: `read_file`, `write_file`; Subagent B: `call_api`, `parse_response`
- **C)** Coordinator: `read_file`, `write_file`, `call_api`, `parse_response`, `delegate_to_subagent_a`, `delegate_to_subagent_b`; Subagents: no tools
- **D)** Coordinator: all tools; Subagent A: `read_file`, `write_file`; Subagent B: `call_api`, `parse_response`

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

The coordinator only needs routing capability, so it receives only the delegation tools. Each subagent receives exactly the tools required for its specialized domain and nothing more. This is the textbook application of least privilege: no agent has access to tools it does not need, minimizing the blast radius of any compromise or misbehavior.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents#security-and-trust
- https://docs.anthropic.com/en/docs/build-with-claude/agents#multi-agent-architectures

**Why the others are wrong:**

- **A:** Giving subagents all tools means each subagent can perform operations outside its domain. Subagent A could make API calls and Subagent B could write files, eliminating the isolation benefit of the specialized architecture.
- **C:** Centralizing all tools on the coordinator with no tools on subagents eliminates the subagent specialization entirely and creates a single point of failure and excessive privilege on the coordinator.
- **D:** Giving the coordinator all tools violates least privilege. If the coordinator is compromised or makes an error, it can directly perform file and API operations that should be gated through subagents.

</details>

---

#### C139

*TS 2.3 · Tool distribution & tool_choice*

Your support agent has access to 18 MCP tools covering customer data, orders, refunds, billing, shipping, product catalog, knowledge base, and escalation. Tool selection accuracy has dropped to 60%. What's the most effective architectural change?

- **A)** Implement a tool routing classifier that analyzes each message and pre-selects the 3 most relevant tools.
- **B)** Reduce the agent's tool set to 4-5 core tools relevant to its primary function, routing specialized tasks to dedicated subagents with their own scoped tool sets.
- **C)** Organize tools into categories in the system prompt and instruct the agent to first identify the category, then select the tool.
- **D)** Add detailed descriptions to all 18 tools and add few-shot examples for the most commonly confused pairs.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

18 tools degrades selection reliability. Reduce to 4-5 core tools per agent and route specialized tasks to dedicated subagents with scoped tools.

**Why the others are wrong:**

- **A:** Routing classifiers add complexity.
- **D:** Better descriptions help but don't solve the fundamental cognitive load problem at 18 tools.

</details>

---

#### C140

*TS 2.3 · Tool distribution & tool_choice*

The synthesis subagent needs to verify simple facts (dates, names, statistics) 85% of the time during synthesis. Currently, each verification round-trips through the coordinator to the web search subagent, adding 40% latency. What's the best fix?

- **A)** Have the synthesis subagent batch all verification requests and submit them at the end of its pass.
- **B)** Give the synthesis subagent a scoped `verify_fact` tool for simple lookups, while complex verifications continue routing through the coordinator.
- **C)** Have the web search subagent proactively cache extra context around each source to anticipate verification needs.
- **D)** Give the synthesis subagent access to all web search tools so it can verify anything directly.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Scoped cross-role tool for the 85% common case.

**Why the others are wrong:**

- **A:** Batching creates blocking dependencies.
- **C:** Caching can't predict needs.
- **D:** Full tool access violates separation of concerns.

</details>

---

#### C141

*TS 2.3 · Tool distribution & tool_choice*

Your synthesis subagent has access to 15 tools including web search, document analysis, file operations, and database queries. Despite clear synthesis instructions, it frequently attempts its own web searches instead of synthesizing the provided findings. What's the issue?

- **A)** The system prompt needs stronger instructions to "only synthesize, never search."
- **B)** The subagent has too many tools outside its specialization. Restrict its tool set to synthesis-relevant tools only (e.g., `verify_fact`, `format_report`).
- **C)** Add few-shot examples showing the synthesis subagent using only synthesis tools.
- **D)** The synthesis instructions should be moved from the system prompt to a skill file.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Too many tools outside its specialization leads to misuse. Restrict the tool set.

**Why the others are wrong:**

- **A:** Prompt instructions are probabilistic.
- **C:** Few-shot helps but doesn't address the root cause.
- **D:** Skills don't solve tool distribution.

</details>

---

#### C142

*TS 2.3 · Tool distribution & tool_choice*

The developer productivity agent has 18 tools: 6 built-in (Read, Write, Edit, Bash, Grep, Glob) plus 12 MCP tools for Jira, GitHub, CI/CD, documentation, monitoring, and alerting. Tool selection reliability is poor. How should you restructure?

- **A)** Keep all 18 tools but add detailed descriptions and few-shot examples showing correct tool selection.
- **B)** Create specialized subagents with scoped tool sets: a code exploration agent (Read, Grep, Glob), a code modification agent (Write, Edit, Bash), and a DevOps agent (Jira, GitHub, CI/CD, monitoring tools). The coordinator routes tasks to the appropriate subagent.
- **C)** Remove the least-used MCP tools until you have 10 or fewer total.
- **D)** Prioritize MCP tools over built-in tools in the system prompt to reduce confusion.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

18 tools degrades selection reliability. Create specialized subagents with scoped tool sets (4-5 per agent).

**Why the others are wrong:**

- **A:** Better descriptions help but don't solve the 18-tool cognitive load.
- **C:** Removing tools loses functionality.
- **D:** System prompt priority is unreliable.

</details>

---

#### C143

*TS 2.3 · Tool distribution & tool_choice*

Which `tool_choice` configuration should you use when the agent's first action in every conversation MUST be to call `get_customer` for identity verification?

- **A)** `tool_choice: {"type": "tool", "name": "get_customer"}` — force the model to call `get_customer` specifically.
- **B)** `tool_choice: "required"` — a special mode that requires a specific tool sequence.
- **C)** `tool_choice: "any"` — force the model to call a tool, but let it choose which one.
- **D)** `tool_choice: "auto"` — let the model decide based on the user's message.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Forced tool selection (`{"type": "tool", "name": "get_customer"}`) guarantees a specific tool is called.

**Why the others are wrong:**

- **B:** `"required"` doesn't exist.
- **C:** `"any"` lets the model choose any tool.
- **D:** `"auto"` lets the model skip it.

</details>

---

#### C144

*TS 2.3 · Tool distribution & tool_choice*

Your knowledge agent has 22 tools: 6 built-in plus 16 MCP tools for Confluence, SharePoint, Slack, Jira, HR systems, and various databases. Tool selection is unreliable — the agent often picks wrong tools or makes unnecessary calls. How should you restructure?

- **A)** Remove all MCP tools and rely only on built-in tools for consistency.
- **B)** Create specialized subagents: a documentation agent (Read, Grep, Confluence MCP), a people agent (directory MCP, Slack MCP), and a policy agent (SharePoint MCP, HR MCP). A coordinator routes queries appropriately.
- **C)** Keep all 22 tools but add comprehensive descriptions and 10+ few-shot examples for each tool.
- **D)** Prioritize built-in tools over MCP tools in the system prompt to reduce confusion.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

22 tools degrades selection reliability. Create specialized subagents with scoped tool sets (4-5 per agent).

**Why the others are wrong:**

- **A:** Removing tools loses functionality.
- **C:** Better descriptions help but don't solve cognitive load.
- **D:** Priority prompts are unreliable.

</details>

---

#### C145

*TS 2.3 · Tool distribution & tool_choice*

Your document review agent has 16 tools: 6 built-in plus 10 MCP tools for case law search, precedent lookup, citation checking, clause libraries, and matter management. Tool selection reliability is poor. How should you restructure?

- **A)** Create specialized subagents with scoped tool sets: a research agent (case law, citations), a drafting agent (clause libraries, templates), and a review agent (precedent comparison, risk flags). A coordinator routes tasks appropriately.
- **B)** Remove the least-used MCP tools until you have 10 or fewer total.
- **C)** Keep all 16 tools but add detailed descriptions and few-shot examples showing correct tool selection.
- **D)** Prioritize MCP tools over built-in tools in the system prompt to reduce confusion.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

16 tools degrades selection reliability. Create specialized subagents with scoped tool sets (4-5 per agent).

**Why the others are wrong:**

- **C:** Better descriptions help but don't solve cognitive load.

</details>

---

#### C146

*TS 2.3 · Tool distribution & tool_choice*

The news-research subagent is meant to read web sources only. In practice you discover it occasionally creates a file `notes.md` in the project root with scratch notes, which then leak into git commits. Its current definition is:


```markdown
---
name: news-researcher
description: Investigates recent news on the topic
tools: Read, Write, Edit, WebFetch, WebSearch
---

You research recent news on a given topic and return a summary...
```


What is the simplest durable fix?

- **A)** Append a paragraph to the subagent's system prompt body: "You must never call Write or Edit. Return your notes inline in your reply only."
- **B)** Set `permissionMode: plan` on the subagent so it cannot make filesystem modifications.
- **C)** Add a `PreToolUse` hook scoped to this subagent that intercepts and rejects any `Write` or `Edit` call.
- **D)** Remove `Write, Edit` from the `tools:` frontmatter so the subagent's allowed tool list becomes `Read, WebFetch, WebSearch`.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

The `tools:` frontmatter field is the declarative, purpose-built mechanism for scoping a subagent's capabilities. List only what the subagent needs, and the rest is structurally impossible — no runtime check, no prompt instruction the model might forget.

**Why the others are wrong:**

- **A:** Best-effort; LLMs sometimes violate prohibitions, especially when the task seems to invite the forbidden action.
- **B:** Overshoots. `plan` mode is designed for an interactive plan-then-edit workflow where the human approves the plan before any modification; it's not the right primitive for permanently scoping a worker, and it introduces semantics the subagent's role doesn't need.
- **C:** Works but is runtime enforcement of something the config can express declaratively. Hooks are right when you need behavior beyond the declarative model (logging, dynamic policy); they're overkill for a static allowlist.

</details>

---

#### C147

*TS 2.4 · MCP server integration*

**Situation:** A developer wants to add an MCP server that is only available when working inside a specific project repository. They do not want the server to appear in other projects they work on.

Where should the developer place the MCP server configuration so that it is scoped exclusively to the current project?

- **A)** ~/.claude/mcp-servers.json
- **B)** /etc/claude/mcp.json
- **C)** ~/.claude/settings.json
- **D)** .mcp.json at the project root

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Claude Code uses a `.mcp.json` file at the project root for project-scoped MCP server configuration. This file is loaded only when Claude Code is launched from within that project directory, ensuring the servers defined there are not exposed in other projects.

References:
- https://docs.anthropic.com/en/docs/claude-code/mcp
- https://docs.anthropic.com/en/docs/claude-code/settings

**Why the others are wrong:**

- **A:** ~/.claude/mcp-servers.json is not a real Claude Code configuration path — user-scoped MCP servers live in ~/.claude.json — and user-scoped config applies to all projects for the current user, the opposite of project-scoped behavior.
- **B:** /etc/claude/ is not a recognized Claude Code configuration path and would not be loaded by Claude Code under any normal operating conditions.
- **C:** ~/.claude/settings.json stores user-level Claude Code settings, not MCP server definitions. Placing MCP config here would not scope it to a single project.

</details>

---

#### C148

*TS 2.4 · MCP server integration*

**Situation:** A team wants every Claude Code session on their developer machines to have access to an internal Slack MCP server regardless of which project is currently open. The server configuration should persist across all projects for each individual user.

Which configuration scope and location should the team instruct each developer to use for the Slack MCP server?

- **A)** System-scoped: /usr/local/etc/claude/mcp.json
- **B)** Project-scoped: .mcp.json at the root of each repository
- **C)** Environment-scoped: export MCP_SERVERS=slack in the shell profile
- **D)** User-scoped: the MCP configuration stored in ~/.claude.json

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

User-scoped MCP server configuration is stored in ~/.claude.json and applies to all Claude Code sessions for that user, regardless of which project directory is active. This is the correct scope for servers that should always be available.

References:
- https://docs.anthropic.com/en/docs/claude-code/mcp
- https://docs.anthropic.com/en/docs/claude-code/settings

**Why the others are wrong:**

- **A:** Claude Code does not define a system-scoped /usr/local/etc/claude/mcp.json configuration path. This path would be silently ignored.
- **B:** Project-scoped .mcp.json only activates when Claude Code runs in that specific project directory. Duplicating the config in every repository is error-prone and violates the DRY principle.
- **C:** Claude Code does not support an MCP_SERVERS environment variable for defining server configurations. Environment variables are used for expanding secrets inside .mcp.json, not for defining server lists.

</details>

---

#### C149

*TS 2.4 · MCP server integration*

**Situation:** A developer is writing a .mcp.json file and needs to include a secret API key for an MCP server without hardcoding the key value in the file. The key is already set as an environment variable named ACME_API_KEY on the developer's machine.

Which syntax should the developer use inside .mcp.json to reference the environment variable ACME_API_KEY?

- **A)** "apiKey": "process.env.ACME_API_KEY"
- **B)** "apiKey": "$ACME_API_KEY"
- **C)** "apiKey": "${ACME_API_KEY}"
- **D)** "apiKey": "env:ACME_API_KEY"

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Claude Code supports environment variable expansion inside .mcp.json using the ${ENV_VAR} syntax. At load time, Claude Code substitutes the placeholder with the runtime value of the named environment variable.

References:
- https://docs.anthropic.com/en/docs/claude-code/mcp
- https://docs.anthropic.com/en/docs/claude-code/settings

**Why the others are wrong:**

- **A:** process.env.VARIABLE is Node.js source code syntax, not a JSON configuration value. It has no meaning inside a .mcp.json file.
- **B:** The bare $VARIABLE syntax is a shell convention that Claude Code's JSON parser does not expand. The literal string "$ACME_API_KEY" would be passed to the server unchanged.
- **D:** "env:VARIABLE" is not a recognized expansion syntax in Claude Code's .mcp.json handling. It would be treated as a literal string.

</details>

---

#### C150

*TS 2.4 · MCP server integration*

**Situation:** An architect is explaining MCP resource types to a new team member. The team member asks what makes MCP resources different from MCP tools in terms of how Claude interacts with them.

Which statement correctly distinguishes MCP resources from MCP tools?

- **A)** Resources and tools are functionally identical; the distinction is purely cosmetic naming.
- **B)** Resources are streamed in real time over SSE; tools are only available over stdio transport.
- **C)** Resources execute server-side logic and can modify state; tools expose read-only data.
- **D)** Resources are read-only data that the model can access; tools are callable functions that can execute logic and produce side-effects.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

In the MCP specification, resources represent read-only data (files, database records, API snapshots) that a server exposes for the model to read. Tools are callable functions that can execute arbitrary logic, including writes and side-effects. The distinction is fundamental to how Claude reasons about safety.

References:
- https://docs.anthropic.com/en/docs/agents-and-tools/mcp
- https://modelcontextprotocol.io/docs/concepts/resources

**Why the others are wrong:**

- **A:** The resource/tool distinction is not cosmetic. It carries semantic meaning about mutability and side-effects that the model uses to decide when it is safe to act autonomously.
- **B:** The transport type (stdio vs SSE) is independent of whether a primitive is a resource or a tool. Both resources and tools can be served over either transport.
- **C:** This reverses the definition. Resources are read-only; tools are the construct that can execute logic and cause side-effects.

</details>

---

#### C151

*TS 2.4 · MCP server integration*

**Situation:** An architect is designing a multi-agent pipeline where a Claude Code orchestrator delegates subtasks to sub-agents. Each sub-agent needs access to a secure credentials MCP server whose API key must never appear in any file committed to version control. The pipeline runs in a CI environment where secrets are injected as environment variables at runtime.

Which combination of .mcp.json design decisions correctly satisfies both the security requirement (no secrets in version control) and the runtime availability requirement (secrets accessible to the server process)?

- **A)** Hardcode the API key in .mcp.json and add .mcp.json to .gitignore so it is never committed.
- **B)** Use ${API_KEY} expansion syntax in .mcp.json for the API key value; commit .mcp.json without the secret; inject API_KEY as a CI environment variable at runtime.
- **C)** Store the API key in a separate secrets.json file referenced by .mcp.json via a file:// URI; commit .mcp.json but gitignore secrets.json.
- **D)** Base64-encode the API key and store the encoded value in .mcp.json; decode it inside the server startup script before use.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Using ${API_KEY} in .mcp.json keeps the secret out of the file entirely. The .mcp.json can be safely committed because it contains only a placeholder. At CI runtime, the secret is injected as an environment variable and Claude Code expands it before passing configuration to the server process. This is the canonical pattern for secret management with .mcp.json.

References:
- https://docs.anthropic.com/en/docs/claude-code/mcp
- https://docs.anthropic.com/en/docs/claude-code/settings

**Why the others are wrong:**

- **A:** Adding .mcp.json to .gitignore prevents accidental commits but does not solve the root problem: the secret is still stored in plaintext on disk. It also breaks the CI pipeline because the file would need to be recreated on each runner, negating the benefit of storing it in the repo.
- **C:** Referencing an external secrets.json via a file URI is not supported by .mcp.json's configuration schema. Even if it were, it introduces a separate secret file that must be managed, distributed, and secured independently.
- **D:** Base64 encoding is not encryption. The encoded value is trivially reversible and provides no meaningful security. Committing a base64-encoded secret to version control is equivalent to committing the plaintext secret.

</details>

---

#### C152

*TS 2.4 · MCP server integration*

**Situation:** A team exposes a large internal knowledge base through an MCP server. They have modeled individual articles as MCP resources and implemented a URI template "kb://articles/{articleId}". An agent workflow needs to let Claude retrieve any article by ID and also search for articles matching a query string.

Which design correctly separates the read-only data access pattern from the query/computation pattern according to MCP primitive semantics?

- **A)** Implement both retrieval and search as MCP resources with URI templates: "kb://articles/{articleId}" for retrieval and "kb://search/{query}" for search.
- **B)** Implement both as MCP tools: `get_article(id)` and `search_articles(query)`, since tools are more flexible and can handle both read and compute operations.
- **C)** Implement article retrieval as a resource template "kb://articles/{articleId}" and article search as an MCP tool `search_articles(query: string)` that executes the search logic and returns results.
- **D)** Implement search as an MCP prompt that injects a pre-built query into the conversation, and retrieval as a resource template.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

MCP resources (and resource templates) represent read-only, addressable data — fetching a known article by ID is a pure read with no side-effects, making the resource template "kb://articles/{articleId}" the semantically correct primitive. Search involves executing query logic and computing results, which is a function — the correct primitive is an MCP tool. Mixing these semantics confuses the model's safety reasoning.

References:
- https://docs.anthropic.com/en/docs/agents-and-tools/mcp
- https://modelcontextprotocol.io/docs/concepts/resources
- https://modelcontextprotocol.io/docs/concepts/tools

**Why the others are wrong:**

- **A:** Modeling search as a resource template conflates data retrieval with query execution. A URI template implies that the resource at "kb://search/machine+learning" pre-exists; in reality the result set is computed on demand. Using a tool makes the computational nature explicit.
- **B:** While tools are flexible enough to implement both patterns, using a tool for pure read operations sacrifices the semantic clarity that resources provide. Claude can make better autonomous decisions when read-only access is clearly marked as a resource rather than a potentially side-effecting tool.
- **D:** MCP prompts are pre-defined message templates injected into the conversation context, not a mechanism for executing queries. Using a prompt for search would require the model to then call another primitive to actually execute the search, adding unnecessary indirection.

</details>

---

#### C153

*TS 2.4 · MCP server integration*

**Situation:** A DevOps engineer is configuring a .mcp.json file for a local development environment. The MCP server is a Node.js script that reads a GitHub token from the environment. The engineer wants to launch the server as a child process managed by Claude Code.

Which MCP server type and .mcp.json structure correctly configures a locally-spawned process server that communicates over standard input/output?

- **A)** type: "pipe" with a pipeName field identifying the named pipe the server listens on
- **B)** type: "sse" with a url field pointing to http://localhost:3000/sse
- **C)** type: "http" with a baseUrl field and an Authorization header containing the GitHub token
- **D)** type: "stdio" with a command field (e.g., "node") and an args array pointing to the server script

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

The stdio server type instructs Claude Code to spawn the MCP server as a child process and communicate with it over stdin/stdout. The .mcp.json entry requires a `command` (the executable) and an `args` array (arguments including the script path). This is the standard pattern for locally-run MCP servers.

References:
- https://docs.anthropic.com/en/docs/claude-code/mcp
- https://modelcontextprotocol.io/docs/concepts/transports

**Why the others are wrong:**

- **A:** "pipe" (named pipe) is not a defined MCP server type in Claude Code's configuration. The two supported types are stdio and sse.
- **B:** The sse type is for connecting to an already-running remote server that exposes an SSE endpoint. It does not spawn a local process and is inappropriate when the server should be managed as a child process.
- **C:** "http" is not a defined MCP server type in the Claude Code .mcp.json schema. HTTP-based communication with an external server is handled through the sse type.

</details>

---

#### C154

*TS 2.4 · MCP server integration*

You need to configure an MCP server for your research pipeline that requires a GitHub API token. The project is worked on by a team of 5 researchers. Where should you configure the MCP server and how should you handle the token?

- **A)** In the project's `.mcp.json` with `"env": {"GITHUB_TOKEN": "${GITHUB_TOKEN}"}` — each researcher sets the environment variable locally.
- **B)** In each researcher's `~/.claude.json` with the token hardcoded.
- **C)** In the project's `.mcp.json` with the token committed directly in the configuration file.
- **D)** In a separate `tokens.json` file that is `.gitignore`-ed and referenced by `.mcp.json`.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Project-scoped `.mcp.json` for team sharing + environment variable expansion for secrets.

**Why the others are wrong:**

- **B:** User-scoped isn't shared.
- **C:** Committing tokens is a security risk.
- **D:** Separate tokens file isn't the standard pattern.

</details>

---

#### C155

*TS 2.4 · MCP server integration*

Your agent uses project-scoped `.mcp.json` for MCP server configuration. A developer adds their personal API key directly in `.mcp.json` and commits it. What's the correct configuration pattern?

- **A)** Encrypt the API key in `.mcp.json` using a shared team encryption key.
- **B)** Store API keys in the system prompt where they're only in memory.
- **C)** Use environment variable expansion in `.mcp.json` (e.g., `$API_KEY`) so secrets are resolved at runtime from the developer's environment, not committed to version control.
- **D)** Move all MCP configuration to `~/.claude.json` (user-level) to avoid committing secrets.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Environment variable expansion in `.mcp.json` keeps secrets out of version control while maintaining project-scoped, team-shared configuration.

**Why the others are wrong:**

- **A:** Encryption adds unnecessary complexity.
- **B:** System prompt storage is not a configuration mechanism.
- **D:** User-level config prevents team sharing.

</details>

---

#### C156

*TS 2.4 · MCP server integration*

Your team has an MCP server with a `search_issues` tool that provides semantic search across Jira tickets. However, the agent keeps using the built-in Grep tool to search for issue numbers in local files instead. What's the most likely cause?

- **A)** The MCP server isn't properly connected — check the `.mcp.json` configuration.
- **B)** The MCP tool's description is too minimal. Enhance it to explain capabilities, typical queries, output format, and when to use it versus Grep.
- **C)** Add `"preferred": true` to the MCP tool configuration to prioritize it.
- **D)** Built-in tools always take priority over MCP tools. This can't be changed.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Minimal MCP tool descriptions cause agents to prefer familiar built-in tools. Enhance the description to explain capabilities, outputs, and when to use it vs Grep.

**Why the others are wrong:**

- **C:** `"preferred"` flag doesn't exist.
- **D:** Built-in tools don't inherently take priority.

</details>

---

#### C157

*TS 2.4 · MCP server integration*

You want to expose your internal API documentation hierarchy (100+ endpoints, organized by service) to the agent so it can answer developer questions about APIs without making exploratory tool calls. What MCP concept should you use?

- **A)** Configure 100 individual MCP tools — one per API endpoint.
- **B)** Expose the API documentation as an MCP **resource** — a read-only content catalog that gives the agent visibility into available data.
- **C)** Include all API documentation in the CLAUDE.md file.
- **D)** Create a `list_apis` MCP tool that the agent calls to discover available endpoints.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

MCP resources expose read-only content catalogs (documentation, schemas) without requiring tool calls. 100 individual tools (A) is impractical. CLAUDE.md (C) would be enormous. A discovery tool (D) adds a tool call when passive exposure suffices.

</details>

---

#### C158

*TS 2.4 · MCP server integration*

Your knowledge management agent needs to find all documents mentioning "remote work policy" across the company's Confluence instance. Which approach is correct?

- **A)** Use Grep to search local files for "remote work policy" — this will find all relevant documents.
- **B)** Use the Confluence MCP server's `search_pages` tool with appropriate query parameters — this searches the actual Confluence content repository.
- **C)** Use Glob to find all `.md` files in the local repository that might contain policy information.
- **D)** Ask employees to manually search Confluence and report findings to the agent.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

MCP server tools search the actual content repository.

**Why the others are wrong:**

- **A:** Grep only searches local files.
- **C:** Glob finds file patterns, not content.
- **D:** Manual search defeats the agent's purpose.

</details>

---

#### C159

*TS 2.4 · MCP server integration*

The agent needs to identify subject matter experts for "cloud infrastructure" topics. Your company has an MCP server with an `employee_directory` tool that returns skills and project history. However, the agent keeps suggesting employees based on job titles alone. What's the likely issue?

- **A)** The MCP server isn't properly authenticated — check the `.mcp.json` credentials.
- **B)** The tool description doesn't emphasize using skills/project history over job titles. Enhance the description to guide proper field usage.
- **C)** The agent needs to call `employee_directory` multiple times to get accurate results.
- **D)** Built-in tools always take priority over MCP tools for employee lookups.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Tool descriptions guide agent behavior. Enhance descriptions to emphasize skills/project history over job titles.

**Why the others are wrong:**

- **A:** Authentication isn't the issue.
- **C:** Multiple calls don't fix the selection issue.
- **D:** Built-in tools don't take priority.

</details>

---

#### C160

*TS 2.4 · MCP server integration*

Your team wants to expose the company's API documentation (200+ endpoints across 15 services) to the knowledge agent. Developers should be able to ask "How do I authenticate to the payments API?" without the agent making exploratory tool calls. What MCP concept applies?

- **A)** Create 200 individual MCP tools — one per API endpoint.
- **B)** Expose API documentation as MCP resources — read-only content catalogs that give the agent visibility without requiring tool calls.
- **C)** Include all API documentation in the root CLAUDE.md file.
- **D)** Create a `discover_api` MCP tool that the agent calls before answering each API question.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

MCP resources expose read-only content catalogs without tool calls. 200 individual tools (A) is impractical. CLAUDE.md (C) would be enormous. Discovery tool (D) adds unnecessary calls.

</details>

---

#### C161

*TS 2.4 · MCP server integration*

You want to expose your firm's standard contract clause library (200+ clauses organized by type) to the agent so it can reference standard language without making exploratory tool calls. What MCP concept should you use?

- **A)** Configure 200 individual MCP tools — one per clause.
- **B)** Expose the clause library as an MCP resource — a read-only content catalog that gives the agent visibility into available clauses.
- **C)** Include all clauses in the CLAUDE.md file.
- **D)** Create a `list_clauses` MCP tool that the agent calls to discover available clauses.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

MCP resources expose read-only content catalogs (clause libraries) without requiring tool calls. 200 individual tools (A) is impractical.

</details>

---

#### C162

*TS 2.5 · Built-in tools (Read/Write/Edit/Bash/Grep/Glob)*

**Situation:** An agent needs to locate all TypeScript files in a project before beginning a refactoring task. The project has hundreds of files spread across many nested directories. The agent has not yet opened any files.

Which tool is most appropriate for discovering all TypeScript files by their file name pattern before reading any content?

- **A)** Read on the root directory to list all files recursively
- **B)** Bash with `find . -name '*.ts'` to recursively list TypeScript files
- **C)** Grep with pattern `\.ts$` to search file contents for TypeScript syntax
- **D)** Glob with pattern `**/*.ts` to find all TypeScript files by name

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Glob is the purpose-built tool for finding files by name pattern. The pattern `**/*.ts` matches all TypeScript files in any subdirectory. Glob returns results sorted by modification time, making it ideal as the first step in incremental codebase understanding (Glob → Read → Grep).

References:
- https://docs.anthropic.com/en/docs/claude-code/overview
- https://docs.anthropic.com/en/docs/agents-and-tools/tool-use

**Why the others are wrong:**

- **A:** Read is for reading file contents, not for listing directories. It cannot discover files by name pattern across a project tree.
- **B:** While Bash `find` would work, the guidelines explicitly state to use Glob instead of `find` commands. Glob is the dedicated, optimized tool for this purpose and provides a better experience.
- **C:** Grep searches file CONTENTS by regex, not file names. Using Grep to find TypeScript files would require files to already be known and would search inside them, not discover them by extension.

</details>

---

#### C163

*TS 2.5 · Built-in tools (Read/Write/Edit/Bash/Grep/Glob)*

**Situation:** An agent is asked to fix a bug in an existing function inside `/src/utils/parser.ts`. The file is approximately 300 lines long and only one function needs to change. The agent has already read the file.

Which tool should the agent use to modify only the buggy function without rewriting the entire file?

- **A)** Grep to locate the function, then Write to replace the whole file
- **B)** Write, because it ensures the entire file is replaced with the corrected version
- **C)** Edit, because it sends only the diff and is designed for modifying existing files
- **D)** Bash with a `sed` command to perform an in-place substitution

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Edit is explicitly designed for modifying existing files and sends only the diff rather than the entire file content. This is more efficient and precise for targeted changes. The agent has already read the file, satisfying Edit's prerequisite of reading before editing.

References:
- https://docs.anthropic.com/en/docs/claude-code/overview
- https://docs.anthropic.com/en/docs/agents-and-tools/tool-use

**Why the others are wrong:**

- **A:** Using Grep plus Write is an inefficient two-step anti-pattern. Edit handles targeted modifications directly without requiring a full file rewrite.
- **B:** Write is for creating new files or complete rewrites. Using Write for a small targeted fix is wasteful and risks accidentally overwriting content if the full file content is not perfectly reproduced.
- **D:** The guidelines state to avoid using `sed` or `awk` as Bash commands. Edit is the dedicated tool for file modification and provides a better, reviewable experience.

</details>

---

#### C164

*TS 2.5 · Built-in tools (Read/Write/Edit/Bash/Grep/Glob)*

**Situation:** An agent needs to find all files in a codebase that import a specific module named `AuthService`. The agent wants to see which files reference this module by searching inside file contents.

Which tool should the agent use to find all files containing the string `AuthService` across the codebase?

- **A)** Grep with pattern `AuthService` to search file contents for the import reference
- **B)** Bash with `ls -R` to list all files and then manually inspect each one
- **C)** Read each file individually and check for the string manually
- **D)** Glob with pattern `**/*AuthService*` to find files named after the service

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Grep searches file CONTENTS by regex pattern, making it the correct tool to find all files that contain a specific string like `AuthService`. It returns matching files efficiently without requiring individual file reads.

References:
- https://docs.anthropic.com/en/docs/claude-code/overview
- https://docs.anthropic.com/en/docs/agents-and-tools/tool-use

**Why the others are wrong:**

- **B:** Using `ls -R` via Bash to list files then manually inspect each one is extremely inefficient. The guidelines say to use Grep for content search instead of shell commands.
- **C:** Reading each file individually to search for a string is the most inefficient approach possible and defeats the purpose of having a dedicated content-search tool like Grep.
- **D:** Glob finds files by NAME pattern. The pattern `**/*AuthService*` would only find files whose names contain `AuthService`, not files that import it in their content.

</details>

---

#### C165

*TS 2.5 · Built-in tools (Read/Write/Edit/Bash/Grep/Glob)*

**Situation:** An agent is creating a brand-new configuration file at `/config/database.yaml`. This file does not yet exist in the project. The agent has all the required content ready.

Which tool is correct for creating this new file?

- **A)** Edit, since it can create files if the target path does not exist
- **B)** Bash with `echo` or `cat <<EOF` to write the content to the new file path
- **C)** Grep to verify the file does not exist, then Edit to create it
- **D)** Write, because it is designed to create new files or perform complete rewrites

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Write is the correct tool for creating new files. It is designed specifically for this use case — creating files that do not yet exist — as well as for complete rewrites of existing files.

References:
- https://docs.anthropic.com/en/docs/claude-code/overview
- https://docs.anthropic.com/en/docs/agents-and-tools/tool-use

**Why the others are wrong:**

- **A:** Edit is for modifying existing files. It requires the agent to have already read the file, and it sends a diff. It is not designed to create files from scratch.
- **B:** The guidelines explicitly state to avoid using `echo` or `cat <<EOF` Bash commands for writing files. Write is the dedicated tool for this purpose.
- **C:** Using Grep to verify non-existence is unnecessary overhead. Write handles new file creation directly. Grep searches content; it cannot confirm file existence in the same way as attempting a Read.

</details>

---

#### C166

*TS 2.5 · Built-in tools (Read/Write/Edit/Bash/Grep/Glob)*

**Situation:** An agent needs to edit a function in `/src/services/payment.ts`. When it calls Edit, the tool returns an error: 'old_string is not unique in the file'. The agent had intended to replace the line `return null;` but this exact string appears 12 times across different functions in the 900-line file.

What is the correct strategy to resolve the uniqueness error without using Write to rewrite the entire file?

- **A)** Use Read to re-read the file, then use Write with the full corrected content since Edit cannot handle non-unique strings
- **B)** Use `replace_all: true` on the string `return null;` to change all 12 occurrences simultaneously
- **C)** Use Grep with context flags to locate the exact function, then expand the `old_string` to include enough surrounding lines from that function to make it unique
- **D)** Use Bash with `sed -n` to identify the line number, then use Edit with the line number as a reference

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

When `old_string` is not unique, the correct strategy is to expand it to include more surrounding context — enough lines from the specific function to make it unique in the file. Grep with context flags (-A/-B/-C) helps identify the exact surrounding text in the target function, which is then used as the `old_string` in the Edit call.

References:
- https://docs.anthropic.com/en/docs/claude-code/overview
- https://docs.anthropic.com/en/docs/agents-and-tools/tool-use

**Why the others are wrong:**

- **A:** Using Write to rewrite a 900-line file to avoid a uniqueness error is the least preferred approach. It is error-prone, sends the entire file content, and violates the principle of using Edit for targeted modifications.
- **B:** `replace_all: true` would replace all 12 occurrences of `return null;`, not just the one in the target function. This would incorrectly modify the behavior of 11 other functions in the file.
- **D:** Edit does not support line number references. The `old_string` must be the exact text to replace. Bash `sed` is also discouraged. The correct approach is to use more context in the `old_string`.

</details>

---

#### C167

*TS 2.5 · Built-in tools (Read/Write/Edit/Bash/Grep/Glob)*

**Situation:** An agent is asked to audit an entire monorepo for security vulnerabilities. The monorepo contains 2,000+ files across 15 packages. The agent must find all uses of `eval()`, all `innerHTML` assignments, and all hardcoded strings matching common API key patterns. Performance and thoroughness are equally important.

Which multi-tool strategy is most efficient and thorough for this security audit?

- **A)** Use Bash with three separate `grep -r` commands for each pattern, piping results to a summary file
- **B)** Run three parallel Grep calls — one per pattern — each with `output_mode: 'files_with_matches'` and appropriate glob filters, then Read only the flagged files for deeper analysis
- **C)** Use Read on every file sequentially, building a list of findings as each file is processed
- **D)** Use Glob to list all 2,000+ files, then use Grep once per file for each of the three patterns

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Running three parallel Grep calls (one per pattern) across the entire codebase with `output_mode: 'files_with_matches'` is the most efficient approach. This leverages Grep's content-search capability at scale, avoids reading files that contain no vulnerabilities, and parallelizes the three independent searches. Only flagged files then need deeper Read-based analysis.

References:
- https://docs.anthropic.com/en/docs/claude-code/overview
- https://docs.anthropic.com/en/docs/agents-and-tools/tool-use

**Why the others are wrong:**

- **A:** The guidelines state to use the Grep tool instead of `grep` Bash commands. Three Bash `grep -r` commands also cannot run in parallel within a single Bash call in the same way that three separate Grep tool calls can be parallelized.
- **C:** Reading every one of 2,000+ files sequentially is extremely inefficient. The vast majority will not contain the target patterns. Grep is designed precisely to avoid this by scanning content across many files quickly.
- **D:** Using Glob to list all files then calling Grep once per file is a N×3 operation (2,000 files × 3 patterns = 6,000 Grep calls). Running three broad Grep searches across the entire codebase is far more efficient than scoping each Grep to a single file.

</details>

---

#### C168

*TS 2.5 · Built-in tools (Read/Write/Edit/Bash/Grep/Glob)*

**Situation:** An agent must rename a variable `userToken` to `accessToken` in every occurrence throughout a 600-line file `/src/auth/middleware.ts`. The agent has already read the file and confirmed it contains 47 occurrences of `userToken`.

Which Edit tool parameter is most appropriate for replacing all 47 occurrences in a single operation?

- **A)** Use Bash with `sed -i 's/userToken/accessToken/g'` to perform the global substitution
- **B)** Call Edit 47 times, once per occurrence, to ensure each replacement is handled individually
- **C)** Use Edit with `replace_all: true` to change every instance of `userToken` to `accessToken`
- **D)** Use Write to rewrite the entire 600-line file with all replacements applied

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Edit supports a `replace_all` parameter that changes every instance of the `old_string` in the file. This is the correct tool for renaming a variable across all occurrences — it is described as useful for renaming variables or strings across a file.

References:
- https://docs.anthropic.com/en/docs/claude-code/overview
- https://docs.anthropic.com/en/docs/agents-and-tools/tool-use

**Why the others are wrong:**

- **A:** The guidelines state to avoid `sed` and `awk` Bash commands. Edit with `replace_all: true` is the dedicated, preferred approach and provides a reviewable diff.
- **B:** Calling Edit 47 times is extremely inefficient and error-prone. The `replace_all` parameter exists precisely to handle this scenario in a single operation.
- **D:** Write is reserved for new files or complete rewrites. Using Write for a variable rename risks introducing errors if the full file content is not perfectly reconstructed, and is semantically wrong for this targeted change.

</details>

---

#### C169

*TS 2.5 · Built-in tools (Read/Write/Edit/Bash/Grep/Glob)*

**Situation:** An agent is investigating a TypeScript error and needs to understand which files export a class called `DatabaseConnection`. It knows the class definition begins with `export class DatabaseConnection` but does not know which files contain it.

Which Grep configuration would most accurately find files containing the class declaration?

- **A)** Glob with pattern `**/*DatabaseConnection*.ts` to find files named after the class
- **B)** Grep with `pattern: 'DatabaseConnection'` and `output_mode: 'files_with_matches'`
- **C)** Bash with `grep -r 'export class DatabaseConnection' --include='*.ts'`
- **D)** Grep with `pattern: 'export class DatabaseConnection'`, `type: 'ts'`, and `output_mode: 'files_with_matches'`

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Grep with the specific pattern `export class DatabaseConnection`, filtered to TypeScript files using `type: 'ts'`, and `output_mode: 'files_with_matches'` is the most precise approach. It searches file contents for the exact export declaration and limits scope to TypeScript files, reducing noise.

References:
- https://docs.anthropic.com/en/docs/claude-code/overview
- https://docs.anthropic.com/en/docs/agents-and-tools/tool-use

**Why the others are wrong:**

- **A:** Glob finds files by NAME pattern. Files are not typically named `DatabaseConnection.ts` just because they export that class. This would miss the majority of relevant files.
- **B:** While this would work, searching for just `DatabaseConnection` without the `export class` prefix and without a type filter matches occurrences in imports, comments, and other file types, producing noisier results than option D.
- **C:** The guidelines explicitly state to use the Grep tool instead of running `grep` as a Bash command. The dedicated Grep tool is optimized for correct permissions and provides a better experience.

</details>

---

#### C170

*TS 2.5 · Built-in tools (Read/Write/Edit/Bash/Grep/Glob)*

**Situation:** An agent attempts to use Edit to modify `/src/config/settings.ts`, but the Edit tool returns an error stating it cannot proceed because the file was not read first. The file is 150 lines long.

What is the correct way to resolve this error and proceed with the edit?

- **A)** Switch to Write instead, since Write does not require a prior Read
- **B)** Use Grep to retrieve the specific lines needed, then call Edit without reading the full file
- **C)** Use Read to read the file first, then call Edit with the appropriate `old_string` and `new_string`
- **D)** Use Bash to cat the file contents before calling Edit

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Edit requires the agent to have read the file at least once in the conversation before editing. The correct resolution is to call Read on the file first to satisfy this prerequisite, then call Edit with the exact text to replace (`old_string`) and the replacement (`new_string`).

References:
- https://docs.anthropic.com/en/docs/claude-code/overview
- https://docs.anthropic.com/en/docs/agents-and-tools/tool-use

**Why the others are wrong:**

- **A:** Write is for new files or complete rewrites, not targeted edits. Using Write to avoid reading first would require rewriting all 150 lines perfectly, which is error-prone and semantically incorrect for a targeted change.
- **B:** Grep returns matching lines but does not constitute 'reading the file' in the way Edit requires. Edit needs the full context of the file to ensure `old_string` is unique and accurately matched.
- **D:** The guidelines state to use the Read tool instead of `cat` Bash commands. The Read tool provides the same file content in a structured way that satisfies the Edit prerequisite.

</details>

---

#### C171

*TS 2.5 · Built-in tools (Read/Write/Edit/Bash/Grep/Glob)*

**Situation:** An agent needs to run a database migration script that exists at `/scripts/migrate.sh` and capture its output to verify success. The script takes approximately 10 seconds to run and the agent must confirm the output before proceeding.

Which approach correctly uses Bash for this command while ensuring the output is captured for verification?

- **A)** Use Read to open the script file and parse whether it will succeed without executing it
- **B)** Use Bash with `run_in_background: true` since the task does not need the result immediately
- **C)** Use Bash to execute `bash /scripts/migrate.sh` and capture the output synchronously, since 10 seconds is well within the 2-minute default timeout
- **D)** Use Grep on the script file to verify its contents before deciding whether to run it

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Bash is the correct tool for executing shell commands. A 10-second migration script is well within the 2-minute default timeout (120,000ms). Running it synchronously ensures the output is immediately available for verification of success. Background execution would be appropriate only if the result is not needed immediately.

References:
- https://docs.anthropic.com/en/docs/claude-code/overview
- https://docs.anthropic.com/en/docs/agents-and-tools/tool-use

**Why the others are wrong:**

- **A:** Read retrieves file contents for reading; it cannot execute scripts or predict runtime behavior. Parsing a migration script statically is not a reliable substitute for running it.
- **B:** `run_in_background: true` is appropriate only when the result is not needed immediately. Since the agent needs to verify success from the output, synchronous execution is required.
- **D:** Using Grep to inspect the script's contents before running it may be a useful safety check, but it does not execute the migration. Grep alone cannot accomplish the task of running the script and capturing output.

</details>

---

#### C172

*TS 2.5 · Built-in tools (Read/Write/Edit/Bash/Grep/Glob)*

**Situation:** You are beginning a fresh exploration session in a large monorepo with hundreds of files. You need to understand the overall structure before diving into specific modules.

What is the recommended first step when starting incremental exploration of a large codebase?

- **A)** Use Glob to identify file patterns and structure before reading specific files
- **B)** Read every file in the repository from top to bottom
- **C)** Run Grep searches across all files simultaneously
- **D)** Immediately delegate the entire exploration to a subagent

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

The recommended incremental understanding workflow is Glob → Read → Grep. Glob provides a broad structural overview with minimal context cost, allowing you to identify relevant files before committing to reading their full contents. This prevents context pollution from reading irrelevant files.

References:
- https://docs.anthropic.com/en/docs/claude-code/common-workflows

**Why the others are wrong:**

- **B:** Reading every file consumes enormous context and pollutes the context window with irrelevant content, leading to premature context exhaustion.
- **C:** Running Grep across all files without first understanding the structure can yield overwhelming results and consumes context on noise rather than signal.
- **D:** Delegating before you understand the scope may result in poorly scoped subagent instructions and redundant exploration work.

</details>

---

#### C173

*TS 2.5 · Built-in tools (Read/Write/Edit/Bash/Grep/Glob)*

**Situation:** An agent needs to find all usages of a deprecated `legacyAuth()` function across a TypeScript codebase with 500 files. The agent has already used 60% of its context window on earlier exploration tasks.

What is the most context-efficient approach to locate all usages of `legacyAuth()`?

- **A)** Use Glob to find all `.ts` files, then read each one to check for the function name
- **B)** Ask the user to manually search the codebase and report back
- **C)** Use Grep with the pattern `legacyAuth` scoped to TypeScript files to get only matching file paths and line numbers
- **D)** Read the entire codebase into context and then scan for the function name

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Grep in `files_with_matches` or `content` mode returns only matching lines, consuming a tiny fraction of the context that reading full files would require. With 60% context already used, targeted Grep is the only responsible approach — it finds exactly what is needed without loading irrelevant file content.

References:
- https://docs.anthropic.com/en/docs/claude-code/common-workflows
- https://docs.anthropic.com/en/docs/build-with-claude/context-windows

**Why the others are wrong:**

- **A:** Globbing and then reading each file individually is highly inefficient — it reads entire file contents for most files that won't contain the target function, wasting the remaining 40% of context.
- **B:** Asking the user to search manually defeats the purpose of agent-driven exploration and is not a valid context management strategy.
- **D:** Reading the entire codebase at 60% context usage would immediately cause context overflow, making this approach impossible and contrary to avoiding context pollution.

</details>

---

#### C174

*TS 2.5 · Built-in tools (Read/Write/Edit/Bash/Grep/Glob)*

A developer asks the agent to "find all files that call the `processPayment` function in the codebase." Which built-in tool is appropriate?

- **A)** Grep — it searches file contents for patterns, making it the correct tool to find all files containing `processPayment`.
- **B)** Read — read each source file in the project and search for the function call.
- **C)** Glob — it searches for file patterns like `**/*.ts` to find all TypeScript files, which can then be filtered for the function.
- **D)** Bash — run `find . -name "*.ts"` to locate TypeScript files containing the function.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Grep searches file contents for patterns. Glob searches file names/paths. The question asks to find files calling a function — that's content search = Grep.

**Why the others are wrong:**

- **B:** Read is inefficient at scale.
- **D:** Bash works but isn't the intended built-in tool.

</details>

---

#### C175

*TS 2.5 · Built-in tools (Read/Write/Edit/Bash/Grep/Glob)*

A developer asks the agent to understand a legacy payment processing module. The agent reads all 47 files in the module upfront, consuming most of the context window. Subsequent questions about the module get vague, generic answers. What's the better approach?

- **A)** Ask the developer to identify the specific files that are relevant before research begins.
- **B)** Build understanding incrementally: start with Grep to find entry points, then use Read to follow imports and trace flows — only reading files as needed.
- **C)** Start by reading all files but use `/compact` after each question to free up context.
- **D)** Split the module into smaller submodules and analyze each in a separate session.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Incremental investigation: Grep → find entry points → Read to follow imports → trace flows. Reading all 47 files upfront floods the context.

**Why the others are wrong:**

- **A:** Asking the developer defeats the purpose of the agent.
- **C:** `/compact` loses details.

</details>

---

#### C176

*TS 2.5 · Built-in tools (Read/Write/Edit/Bash/Grep/Glob)*

The agent needs to trace how the `calculateDiscount` function is used across the codebase. It is exported from `src/utils/pricing.ts`, but other modules re-export it through wrapper modules. What's the correct investigation strategy?

- **A)** Read `src/utils/pricing.ts` to understand the function, then Grep for `calculateDiscount` and also identify all exported names from wrapper modules, then Grep for each re-exported name across the codebase.
- **B)** Grep for `calculateDiscount` across the entire codebase — this will find all callers regardless of re-exports.
- **C)** Use Bash to generate a dependency graph and analyze it programmatically.
- **D)** Use Glob to find all files importing from `pricing.ts`, then read each one.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Wrapper modules re-export under different names. The correct strategy: read the source, identify re-exports, then Grep for each re-exported name.

**Why the others are wrong:**

- **B:** Simple Grep misses these.
- **D:** Glob finds importers of the module but not re-exported callers.

</details>

---

#### C177

*TS 2.5 · Built-in tools (Read/Write/Edit/Bash/Grep/Glob)*

Your legal agent needs to find all contracts containing "indemnification" clauses across a repository of 5,000+ legal documents. Which built-in tool is appropriate?

- **A)** Read — read each document in the repository and search for the clause.
- **B)** Bash — run `find . -name "*.pdf"` to locate PDF files containing the clause.
- **C)** Grep — it searches file contents for patterns, making it the correct tool to find all documents containing "indemnification".
- **D)** Glob — it searches for file patterns like `**/*.pdf` to find all PDF documents, which can then be filtered for the clause.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Grep searches file contents for patterns. Glob searches file names/paths. The question asks to find documents containing a clause — that's content search = Grep.

</details>

---

#### C178

*TS 2.5 · Built-in tools (Read/Write/Edit/Bash/Grep/Glob)*

A paralegal asks the agent to understand a complex merger agreement. The agent reads all 85 pages upfront, consuming most of the context window. Subsequent questions about specific clauses get vague, generic answers. What's the better approach?

- **A)** Build understanding incrementally: use Grep to find key sections (definitions, obligations, termination), then Read only those sections as needed.
- **B)** Ask the paralegal to identify the specific pages that are relevant before analysis begins.
- **C)** Split the agreement into smaller sections and analyze each in a separate session.
- **D)** Start by reading all pages but use `/compact` after each question to free up context.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Incremental investigation: Grep → find key sections → Read only those. Reading all 85 pages upfront floods the context.

**Why the others are wrong:**

- **D:** /compact loses details.

</details>

---

#### C179

*TS 2.5 · Built-in tools (Read/Write/Edit/Bash/Grep/Glob)*

Your agent needs to find all products with inventory below reorder point across 10,000+ SKUs. Which approach is correct?

- **A)** Ask warehouse staff to manually check stock levels and report to the agent.
- **B)** Use the `check_inventory` MCP tool with appropriate filter parameters — this queries the actual inventory management system.
- **C)** Use Glob to find all inventory-related files and read them manually.
- **D)** Use Grep to search local inventory files for low-stock items.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

MCP server tools query the actual inventory management system.

**Why the others are wrong:**

- **D:** Grep only searches local files.

</details>

---

<a id="part-c-domain-3"></a>

### Part C · Domain 3 · Claude Code Configuration & Workflows (20%)

#### C180

*TS 3.1 · CLAUDE.md hierarchy & modularity*

**Situation:** A developer is new to Claude Code and wants to set personal preferences — such as preferred coding style and default language — that apply to every project they work on across their machine. They ask where to place these instructions.

Which file path is the correct location for user-level CLAUDE.md instructions that apply globally across all projects?

- **A)** .claude/rules/CLAUDE.md in the project directory
- **B)** ~/.claude/CLAUDE.md
- **C)** ~/CLAUDE.md in the home directory root
- **D)** ./CLAUDE.md in the root of each project

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

The user-level CLAUDE.md lives at ~/.claude/CLAUDE.md. Claude Code reads this file for personal preferences and instructions that should apply universally across all projects on the machine, regardless of which project directory is open.

References:
- https://docs.anthropic.com/en/docs/claude-code/memory

**Why the others are wrong:**

- **A:** .claude/rules/ inside a project directory is for modular rule files scoped to that project, not for machine-wide personal preferences.
- **C:** ~/CLAUDE.md at the home directory root is not a recognized Claude Code configuration path; Claude Code does not scan arbitrary locations in the home directory.
- **D:** ./CLAUDE.md is the project-level location and is scoped to one project; it would need to be duplicated in every project to achieve global reach.

</details>

---

#### C181

*TS 3.1 · CLAUDE.md hierarchy & modularity*

**Situation:** A team is starting a new monorepo and wants to store shared coding conventions — such as commit message format and test requirements — so that every team member's Claude Code session automatically picks them up when working in the project.

Where should the team place the shared project-level CLAUDE.md so all contributors benefit from it when Claude Code is opened in the project root?

- **A)** ~/.claude/CLAUDE.md on each developer's machine
- **B)** ~/.claude/rules/project.md on each developer's machine
- **C)** ./CLAUDE.md in the root of the project repository
- **D)** ./src/CLAUDE.md in the main source directory

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

The project-level CLAUDE.md is placed at the repository root (./CLAUDE.md). Claude Code reads it automatically for every session opened in that project, and it is committed to version control so all team members share the same conventions without any per-machine configuration.

References:
- https://docs.anthropic.com/en/docs/claude-code/memory

**Why the others are wrong:**

- **A:** ~/.claude/CLAUDE.md is the user-level file; placing team conventions there requires each developer to manually copy and maintain it, defeating the purpose of a shared standard.
- **B:** ~/.claude/rules/ is for personal modular rule files, not for project-shared instructions; it is not version-controlled and not shared across the team.
- **D:** ./src/CLAUDE.md would be a directory-level file scoped only to the src/ subtree, not to the entire project; code outside src/ would not inherit these conventions.

</details>

---

#### C182

*TS 3.1 · CLAUDE.md hierarchy & modularity*

**Situation:** A developer has a project CLAUDE.md at the root and wants to add a separate, more targeted set of rules that apply only when Claude Code is working inside the backend/ subdirectory. The root-level rules should still apply everywhere.

How should the developer add rules that are scoped exclusively to the backend/ subdirectory?

- **A)** Add a [backend] section inside the root ./CLAUDE.md using a section header.
- **B)** Create a separate ~/.claude/CLAUDE.md that references the backend/ path.
- **C)** Use the @import syntax in the root CLAUDE.md to import a backend-specific file.
- **D)** Create a CLAUDE.md file inside the backend/ subdirectory at backend/CLAUDE.md.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Directory-level CLAUDE.md files placed inside a subdirectory (e.g., backend/CLAUDE.md) are automatically scoped to that directory. Claude Code applies them only when working within backend/, while the root-level CLAUDE.md continues to apply project-wide.

References:
- https://docs.anthropic.com/en/docs/claude-code/memory

**Why the others are wrong:**

- **A:** CLAUDE.md does not support section headers for path-scoping; the entire content of a CLAUDE.md applies to its directory scope — granular path filtering within a single file requires YAML frontmatter in .claude/rules/ files, not section headers.
- **B:** The user-level ~/.claude/CLAUDE.md applies globally across all projects and cannot be narrowed to a specific subdirectory of one project.
- **C:** @import would pull the backend rules into the root CLAUDE.md, making them apply everywhere rather than exclusively to backend/, which is the opposite of the desired scoping.

</details>

---

#### C183

*TS 3.1 · CLAUDE.md hierarchy & modularity*

**Situation:** A developer has been accumulating a large number of instructions in their project CLAUDE.md, making it difficult to read and maintain. A colleague suggests splitting the instructions into separate files organized by topic.

Which directory and mechanism does Claude Code support for organizing project instructions into multiple focused rule files?

- **A)** Create a CLAUDE.json file at the project root that lists the paths of individual rule files.
- **B)** Add each rule file to a claude.config array in the project's package.json.
- **C)** Store separate rule files under .claude/rules/ in the project and use @import in CLAUDE.md to reference them.
- **D)** Place multiple CLAUDE.md files in different subdirectories and use symbolic links to combine them.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Claude Code supports a .claude/rules/ directory for modular rule files. These files can be imported into a CLAUDE.md using the @import ./path/to/file.md syntax, enabling teams to split instructions by topic (e.g., testing rules, commit conventions, API standards) while keeping each file focused and maintainable.

References:
- https://docs.anthropic.com/en/docs/claude-code/memory

**Why the others are wrong:**

- **A:** There is no CLAUDE.json configuration format in Claude Code; instruction files are Markdown, and their composition is handled via @import in CLAUDE.md files.
- **B:** package.json is a Node.js project manifest; Claude Code does not read claude.config entries from it.
- **D:** Symbolic links are a filesystem-level workaround and are not a Claude Code–supported mechanism; Claude Code does not merge or dereference symlinked CLAUDE.md files.

</details>

---

#### C184

*TS 3.1 · CLAUDE.md hierarchy & modularity*

**Situation:** A developer notices that a rule in their personal ~/.claude/CLAUDE.md ('never add JSDoc comments') conflicts with a rule in the project ./CLAUDE.md ('always add JSDoc comments to all public functions'). They need to determine the effective rule when contributing to this specific project, and separately determine the effective rule in their own personal projects that have no CLAUDE.md.

What are the correct effective rules for each of the two contexts described?

- **A)** In the team project: 'always add JSDoc' (project rule wins). In personal projects: 'never add JSDoc' (user rule applies as sole instruction).
- **B)** In the team project: 'never add JSDoc' (user rule wins because personal preferences override team conventions). In personal projects: 'always add JSDoc' (the last-loaded rule wins).
- **C)** In both contexts: 'always add JSDoc', because once a project-level rule is loaded it persists across all subsequent sessions on the machine.
- **D)** In the team project: both rules apply simultaneously and Claude Code asks the developer to choose at session start. In personal projects: 'never add JSDoc'.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

The priority order user → project → directory means project rules override user rules for that project. In the team project, the project-level 'always add JSDoc' instruction is more specific than the user-level instruction and takes precedence. In personal projects with no CLAUDE.md, there is no project rule to override the user-level file, so 'never add JSDoc' applies unopposed. The two contexts correctly illustrate the hierarchy: more specific scope wins when present; user-level provides the default otherwise.

References:
- https://docs.anthropic.com/en/docs/claude-code/memory

**Why the others are wrong:**

- **B:** User-level rules do not override project-level rules; the priority order is specifically designed so that team conventions (project scope) can supersede personal defaults (user scope) for the duration of work in that project.
- **C:** Project-level rules are not persistent across sessions or other projects; they apply only when Claude Code is operating within that project's directory. They do not bleed into other projects.
- **D:** Claude Code does not interactively prompt the developer to resolve rule conflicts at session start; it resolves conflicts automatically using the defined priority order.

</details>

---

#### C185

*TS 3.1 · CLAUDE.md hierarchy & modularity*

**Situation:** An architect is auditing a Claude Code configuration that has grown organically over 18 months. The project has: a 600-line ./CLAUDE.md with duplicated content from imported files; 14 files in .claude/rules/ some of which are never referenced by any @import; a backend/CLAUDE.md that re-declares rules already present in the root CLAUDE.md; and a ~/.claude/CLAUDE.md that overrides project rules the developer no longer wants to override. They want to rationalize the structure.

Which set of actions most comprehensively resolves the structural problems without losing intended rule coverage?

- **A)** Keep all files as-is but prepend a priority comment to each rule declaring its intended scope; instruct developers to read the comments to understand which rules apply where.
- **B)** Replace the entire CLAUDE.md hierarchy with a single consolidated root CLAUDE.md that contains every rule inline, and remove all .claude/rules/ files to eliminate import complexity.
- **C)** Delete all directory-level CLAUDE.md files and consolidate everything into ./CLAUDE.md, then use YAML path filters on .claude/rules/ files for any path-specific rules.
- **D)** Audit each rule for active @import references and remove unreferenced .claude/rules/ files; deduplicate backend/CLAUDE.md by keeping only rules that differ from the root; trim the root ./CLAUDE.md by extracting repeated content into named .claude/rules/ files and using @import; and remove the user-level overrides from ~/.claude/CLAUDE.md that are no longer desired.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

A comprehensive rationalization addresses each problem individually: removing unreferenced .claude/rules/ files eliminates dead configuration; deduplicating backend/CLAUDE.md removes drift while preserving intentional overrides; extracting repeated content from the root CLAUDE.md into named rule files with @import reduces size and creates a single source of truth; and pruning the user-level overrides restores the intended priority behavior. This preserves all desired coverage while making the structure auditable and maintainable.

References:
- https://docs.anthropic.com/en/docs/claude-code/memory

**Why the others are wrong:**

- **A:** Comments have no semantic effect on Claude Code's rule evaluation; it reads and applies all content in every applicable CLAUDE.md regardless of human-readable annotations. This action leaves all the structural problems intact.
- **B:** Collapsing everything into a single large CLAUDE.md eliminates the modularity that .claude/rules/ and the hierarchy provide. A single 600+-line file is harder to maintain than a well-organized hierarchy, and path-specific rules become impossible to express without re-implementing path filtering.
- **C:** Deleting all directory-level CLAUDE.md files removes the ability to scope rules to subdirectories at that level. YAML path filters in .claude/rules/ can replicate the scoping, but forcing every directory-specific rule through a single indirection layer may add complexity, not reduce it.

</details>

---

#### C186

*TS 3.1 · CLAUDE.md hierarchy & modularity*

**Situation:** An architect wants to keep the project CLAUDE.md short and readable while still providing Claude Code with extensive context about the API design standards, database schema conventions, and test patterns. Each topic currently has its own Markdown file under .claude/rules/.

What is the correct syntax to include the API design rules from .claude/rules/api-standards.md into the project CLAUDE.md?

- **A)** include: .claude/rules/api-standards.md
- **B)** {{.claude/rules/api-standards.md}}
- **C)** @import ./.claude/rules/api-standards.md
- **D)** <!-- import .claude/rules/api-standards.md -->

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Claude Code uses the @import syntax followed by a relative path to inline the content of another Markdown file into the current CLAUDE.md. The correct form is @import ./.claude/rules/api-standards.md, which tells Claude Code to read and include that file as part of the current instruction set.

References:
- https://docs.anthropic.com/en/docs/claude-code/memory

**Why the others are wrong:**

- **A:** include: is not a recognized Claude Code directive; it resembles YAML front matter syntax but is not the import mechanism Claude Code supports.
- **B:** {{...}} is a template placeholder syntax used in some templating engines but is not supported by Claude Code as a file import directive.
- **D:** HTML comments (<!-- -->) are standard Markdown comment syntax but are not interpreted by Claude Code as file imports; this would be treated as a comment and ignored.

</details>

---

#### C187

*TS 3.1 · CLAUDE.md hierarchy & modularity*

**Situation:** A developer wants to understand the practical difference between storing a rule in ~/.claude/CLAUDE.md versus in a project's ./CLAUDE.md. They are deciding where to put a rule that says 'always write commit messages in German.'

If the developer works on both personal and work projects, what is the key behavioral difference between placing this rule in ~/.claude/CLAUDE.md versus ./CLAUDE.md of their personal project?

- **A)** There is no difference; Claude Code reads both files and merges them into a single instruction set regardless of where the rule lives.
- **B)** Placing it in ./CLAUDE.md makes it a higher-priority rule that overrides any contradicting user-level preferences everywhere.
- **C)** Placing it in ~/.claude/CLAUDE.md means Claude Code will only apply it when no project-level CLAUDE.md exists.
- **D)** Placing it in ~/.claude/CLAUDE.md makes it apply to all projects on the machine; placing it in ./CLAUDE.md of the personal project scopes it only to that project and shares it with anyone who clones the repo.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

The user-level ~/.claude/CLAUDE.md is machine-global — every Claude Code session on that machine inherits it, including work projects. The project-level ./CLAUDE.md is scoped to one repository and is committed to version control, meaning all collaborators on that project will also receive the rule. These two placement choices have very different blast radii: one affects all personal sessions, the other affects all contributors to one project.

References:
- https://docs.anthropic.com/en/docs/claude-code/memory

**Why the others are wrong:**

- **A:** While both files are read and their instructions combined in practice, they are not identical in scope: the user-level file applies machine-wide while the project-level file applies only to that project. The distinction is significant for cross-project and team-sharing scenarios.
- **B:** Project-level rules override user-level rules for that project due to the priority order, but this does not mean they override user preferences everywhere — outside that project, the user-level rule still applies unopposed.
- **C:** User-level rules apply regardless of whether a project-level CLAUDE.md exists; they provide a baseline that is active in all sessions unless overridden by a more specific scope.

</details>

---

#### C188

*TS 3.1 · CLAUDE.md hierarchy & modularity*

**Situation:** A security team wants to enforce a rule that prevents Claude Code from suggesting code that uses deprecated cryptographic functions. They want this rule applied across all projects in their organization's monorepo but not to impose it on individual developers' personal projects outside the repo.

What is the most appropriate placement strategy to enforce this rule for all contributors to the monorepo without affecting developers' personal sessions on other projects?

- **A)** Add the rule to a directory-level CLAUDE.md in each service directory inside the monorepo.
- **B)** Add the rule to .claude/rules/security.md with a YAML frontmatter path filter matching every file pattern.
- **C)** Add the rule to ~/.claude/CLAUDE.md on every developer's machine using an onboarding script.
- **D)** Add the rule to ./CLAUDE.md at the monorepo root and commit it to the repository.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Placing the rule in the project-level ./CLAUDE.md at the monorepo root and committing it ensures that every contributor automatically receives the rule when they open the project in Claude Code — no per-machine setup required. It is scoped to the repository and does not leak into developers' personal projects.

References:
- https://docs.anthropic.com/en/docs/claude-code/memory

**Why the others are wrong:**

- **A:** Adding the rule to every service directory's CLAUDE.md would achieve coverage but is brittle: new services added to the repo would lack the rule until manually updated. The root-level project CLAUDE.md is the single authoritative location for repo-wide rules.
- **B:** A .claude/rules/ file with a catch-all path filter is functionally equivalent to placing it in the project CLAUDE.md but adds unnecessary indirection. The simpler and more conventional approach is to put repo-wide rules directly in the root ./CLAUDE.md.
- **C:** Adding rules to each developer's ~/.claude/CLAUDE.md via an onboarding script would apply the rule globally on each machine, affecting personal projects as well — violating the requirement not to impose it outside the monorepo. It also creates a maintenance burden when the rule changes.

</details>

---

#### C189

*TS 3.1 · CLAUDE.md hierarchy & modularity*

A senior developer has configured team coding standards in `~/.claude/CLAUDE.md`. A new hire clones the repository but doesn't get these standards applied. What's the issue?

- **A)** The new hire needs to install the Claude Code extension first.
- **B)** User-level `~/.claude/CLAUDE.md` is personal and not shared via version control. Team standards should be in the project-level `.claude/CLAUDE.md`.
- **C)** The new hire needs to run `/memory` to manually load the standards.
- **D)** The new hire's Claude Code version is outdated and doesn't support the CLAUDE.md hierarchy.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

User-level `~/.claude/CLAUDE.md` is personal and not version-controlled. Team standards belong in project-level `.claude/CLAUDE.md`. Extension installation (A), `/memory` (C), and version (D) are not the issue.

</details>

---

#### C190

*TS 3.1 · CLAUDE.md hierarchy & modularity*

Your monorepo has three packages: a React frontend, a Python API, and a Terraform infrastructure module. Each has different coding conventions and each package has its own maintainer with domain expertise. What's the best way to organize CLAUDE.md?

- **A)** Create separate CLAUDE.md files in each package subdirectory with the full set of conventions duplicated.
- **B)** Create a root CLAUDE.md with shared standards and use `@import` in each package's CLAUDE.md to selectively include relevant standards files based on the maintainer's domain knowledge.
- **C)** Put all conventions in the root CLAUDE.md, organized by headers for each package.
- **D)** Use `.claude/rules/` with one monolithic rules file containing all conventions.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Root CLAUDE.md for shared standards + separate CLAUDE.md in each package directory that references shared standards (via `@import` or by relying on the merge behavior where all levels are loaded).

**Why the others are wrong:**

- **A:** Full duplication creates maintenance burden.
- **C:** Single monolithic file forces irrelevant conventions on every context.
- **D:** One rules file loses package-specific context.

</details>

---

#### C191

*TS 3.2 · Slash commands & skills*

**Situation:** A developer wants to create a custom slash command called `/deploy` that is available only within a specific project, not globally across all their Claude sessions.

Where should the developer place the `deploy.md` file to create a project-scoped slash command?

- **A)** .claude/skills/deploy.md
- **B)** ~/.claude/commands/deploy.md
- **C)** ~/.claude/skills/deploy.md
- **D)** .claude/commands/deploy.md

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Project-scoped commands are placed in `.claude/commands/` within the project directory. This makes the command available only when working in that project. The `~/.claude/commands/` path is for user-level commands available across all sessions.

References:
- https://docs.anthropic.com/en/docs/claude-code/slash-commands

**Why the others are wrong:**

- **A:** `.claude/skills/` is not a valid path for custom slash commands; the correct subdirectory name is `commands/`, not `skills/`.
- **B:** `~/.claude/commands/` is the user-level commands directory, making the command globally available across all projects — not scoped to a single project.
- **C:** `~/.claude/skills/` is neither a valid user-level nor project-level path for slash commands; the correct directory name is `commands/`.

</details>

---

#### C192

*TS 3.2 · Slash commands & skills*

**Situation:** A team lead wants to create a `/review` slash command that should be available to them personally across every project they work on, regardless of which repository they have open.

Which file path should the team lead use to make the `/review` command available in all their Claude sessions?

- **A)** .claude/global/review.md
- **B)** .claude/commands/review.md
- **C)** ~/commands/review.md
- **D)** ~/.claude/commands/review.md

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

User-level commands are stored at `~/.claude/commands/` (in the user's home directory). Commands placed here are available across all Claude Code sessions for that user, regardless of the current project directory.

References:
- https://docs.anthropic.com/en/docs/claude-code/slash-commands

**Why the others are wrong:**

- **A:** `.claude/global/` is not a valid directory structure for Claude Code commands; there is no `global` subdirectory — commands are either in `.claude/commands/` (project) or `~/.claude/commands/` (user).
- **B:** `.claude/commands/review.md` creates a project-scoped command, only available within the specific project directory where it is placed.
- **C:** `~/commands/review.md` is not a recognized path for Claude Code slash commands; the correct user-level path requires the `.claude` subdirectory.

</details>

---

#### C193

*TS 3.2 · Slash commands & skills*

**Situation:** A developer reads about the `context: fork` frontmatter option while setting up a custom slash command and wants to understand its primary purpose.

What does setting `context: fork` in a slash command's frontmatter do?

- **A)** It creates a copy of the command file in the user-level commands directory.
- **B)** It runs the command in a new isolated session that does not share context with the current conversation.
- **C)** It forks the git repository before executing the command.
- **D)** It enables the command to spawn parallel sub-agents.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

`context: fork` causes the slash command to execute in a forked (isolated) session. This means the command starts fresh without inheriting the current conversation history, preventing the command from being influenced by or polluting the ongoing context.

References:
- https://docs.anthropic.com/en/docs/claude-code/slash-commands

**Why the others are wrong:**

- **A:** `context: fork` has no effect on file system operations or command file copying; it is a session isolation mechanism, not a file management directive.
- **C:** The `fork` value refers to session context forking in Claude Code, not git branching or repository operations.
- **D:** Spawning parallel sub-agents is a separate concern handled by the agent orchestration system; `context: fork` solely controls session isolation for the command's execution context.

</details>

---

#### C194

*TS 3.2 · Slash commands & skills*

**Situation:** A DevOps engineer creates the following slash command file at `.claude/commands/deploy-service.md`:


```markdown
---
context: fork
allowed-tools: [Bash]
argument-hint: "<service-name> <environment>"
---
Deploy the service $ARGUMENTS to the target environment.
First validate the deployment config, then run the deployment script.
```


During a production incident, an operator invokes `/deploy-service payments prod` and then immediately tries to ask Claude a follow-up question in the same chat session: "Wait, what was the last deployment version?"

The operator is surprised when Claude has no memory of the deployment that just ran.

Why does Claude have no memory of the deployment that ran via the `/deploy-service` command?

- **A)** The `argument-hint` field suppresses command output from being displayed in the chat.
- **B)** The `$ARGUMENTS` placeholder consumed the entire user message, preventing Claude from recording the interaction.
- **C)** The `allowed-tools: [Bash]` restriction prevents the command output from being written back to the conversation history.
- **D)** The `context: fork` setting caused the command to execute in an isolated session; its execution and output are not visible in the parent conversation context.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

`context: fork` runs the slash command in a completely isolated (forked) session that is separate from the parent conversation. The deployment execution, tool calls, and any output from that forked session are not surfaced back to the original conversation context. This is by design for isolation, but it means the parent session has no awareness of what happened in the fork — hence Claude in the original chat has no memory of the deployment.

References:
- https://docs.anthropic.com/en/docs/claude-code/slash-commands

**Why the others are wrong:**

- **A:** `argument-hint` is a purely cosmetic UI feature that displays a hint string in the input field; it has no effect on output visibility, session scope, or conversation history.
- **B:** `$ARGUMENTS` is a simple string substitution placeholder that resolves to the user-provided text at invocation time; it has no side effects on session recording or conversation history management.
- **C:** `allowed-tools` controls which tools the command can invoke; it has no effect on whether command output is recorded in conversation history. The isolation is caused by `context: fork`, not by the tool restriction.

</details>

---

#### C195

*TS 3.2 · Slash commands & skills*

**Situation:** A security-conscious team wants to create a `/audit` slash command that should only be permitted to read files and run bash commands — it should never be able to edit files or make network requests during its execution.

Which frontmatter configuration correctly restricts the `/audit` command to only the `Read` and `Bash` tools?

- **A)**
  ```yaml
  ---
  restrict-tools: Read, Bash
  ---
  ```
- **B)**
  ```yaml
  ---
  permitted-tools: [Read, Bash]
  ---
  ```
- **C)**
  ```yaml
  ---
  tools: Read, Bash
  ---
  ```
- **D)**
  ```yaml
  ---
  allowed-tools: [Read, Bash]
  ---
  ```

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

The correct frontmatter key for restricting tool access in a slash command is `allowed-tools`. Setting `allowed-tools: [Read, Bash]` ensures the command can only invoke `Read` and `Bash`, blocking all other tools like `Edit`, `Write`, or web fetch tools.

References:
- https://docs.anthropic.com/en/docs/claude-code/slash-commands

**Why the others are wrong:**

- **A:** `restrict-tools` is not a valid frontmatter option; the semantics of `allowed-tools` already define a whitelist, making a separate `restrict-tools` key unnecessary and unsupported.
- **B:** `permitted-tools` is not a recognized frontmatter key in Claude Code slash commands; only `allowed-tools` is supported for tool restriction.
- **C:** `tools` is not a valid frontmatter key for this purpose; the correct key is `allowed-tools` with a list value.

</details>

---

#### C196

*TS 3.2 · Slash commands & skills*

**Situation:** A developer is creating a `/migrate` slash command and wants the Claude Code UI to display a helpful hint to users, showing them what argument format to provide when they type `/migrate` in the chat.

Which frontmatter option should the developer use to provide this argument description hint?

- **A)** argument-hint
- **B)** help-text
- **C)** usage-hint
- **D)** description

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

`argument-hint` is the frontmatter option that provides a short description of the expected argument format. This hint is displayed in the Claude Code UI when users begin typing the command, helping them understand what input the command expects.

References:
- https://docs.anthropic.com/en/docs/claude-code/slash-commands

**Why the others are wrong:**

- **B:** `help-text` is not a valid frontmatter option for Claude Code slash commands; the specification defines `argument-hint` for this purpose.
- **C:** `usage-hint` is not a recognized frontmatter key in Claude Code slash commands; the correct key is `argument-hint`.
- **D:** `description` is a common frontmatter field in other systems but is not the designated key for argument hints in Claude Code slash commands; `argument-hint` is the correct option.

</details>

---

#### C197

*TS 3.2 · Slash commands & skills*

Your team's `/analyze` skill produces verbose codebase exploration output (300+ lines of file listings, import traces, and dependency graphs) that fills the context window. After running the skill, subsequent Claude responses are degraded. What configuration change fixes this?

- **A)** Add `max-output: 50` to the SKILL.md frontmatter to limit output lines.
- **B)** Add `context: fork` to the SKILL.md frontmatter so the skill runs in an isolated sub-agent context, returning only a summary to the main session.
- **C)** Split the skill into smaller sub-skills that each produce less output.
- **D)** Add `allowed-tools: [Read, Grep, Glob]` to the SKILL.md frontmatter to limit the skill's tool access.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

`context: fork` isolates skill output in a sub-agent context. Only the summary returns.

**Why the others are wrong:**

- **A:** `max-output` doesn't exist.
- **C:** Splitting doesn't fix context pollution.
- **D:** `allowed-tools` limits what the skill can do, not its output impact.

</details>

---

#### C198

*TS 3.2 · Slash commands & skills*

Your team lead wants Claude to ask developers for the target environment name whenever the `/deploy` skill is invoked without arguments. Which SKILL.md frontmatter option enables this?

- **A)** `interactive: true`
- **B)** `prompt: "Which environment should I deploy to?"`
- **C)** `required-args: ["environment"]`
- **D)** `argument-hint: "Specify the target environment (staging, production)"`

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

`argument-hint` frontmatter provides a description of expected arguments, prompting the user when the skill is invoked without them. The other options (`required-args`, `prompt`, `interactive`) are not valid SKILL.md frontmatter fields.

</details>

---

#### C199

*TS 3.2 · Slash commands & skills*

A developer on your team created a personal `/quick-test` command for running their preferred test workflow. Another developer wants a different test workflow. How should this be managed?

- **A)** Both developers put their commands in `.claude/commands/` with different names.
- **B)** Each developer creates their personal command in `~/.claude/commands/` — user-scoped commands that don't affect teammates.
- **C)** Create a single configurable command in `.claude/commands/` that accepts a parameter to select the workflow.
- **D)** The second developer forks the repository to maintain their own command set.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Personal commands go in user-scoped `~/.claude/commands/`.

**Why the others are wrong:**

- **A:** Project-scoped `.claude/commands/` would force one developer's preference on all.
- **C:** A configurable command adds unnecessary complexity for personal preference.
- **D:** Forking is extreme.

</details>

---

#### C200

*TS 3.2 · Slash commands & skills*

A partner created a personal `/precedent-search` command for finding precedents using their preferred search criteria. An associate wants different search criteria. How should this be managed?

- **A)** Both attorneys put their commands in `.claude/commands/` with different names.
- **B)** Each attorney creates their personal command in `~/.claude/commands/` — user-scoped commands that don't affect colleagues.
- **C)** Create a single configurable command in `.claude/commands/` that accepts parameters for search criteria.
- **D)** The associate must request the partner to modify the existing command.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Personal commands go in user-scoped ~/.claude/commands/.

**Why the others are wrong:**

- **A:** Project-scoped would force one attorney's preference on all.

</details>

---

#### C201

*TS 3.3 · Path-specific rules*

**Situation:** A team has a monorepo with a backend/ directory containing Python services and a frontend/ directory containing TypeScript code. They want a single rule file at .claude/rules/no-console-logs.md that applies only when Claude Code is working in the frontend/ directory, not in backend/.

Which mechanism in Claude Code allows a rule file under .claude/rules/ to be automatically applied only to a specific directory path?

- **A)** Using the @scope frontend/ directive as the first line of the rule file.
- **B)** Adding a YAML frontmatter block to no-console-logs.md with a path filter that specifies the frontend/ glob pattern.
- **C)** Naming the file frontend-no-console-logs.md; Claude Code infers the scope from the filename prefix.
- **D)** Placing the rule file at frontend/.claude/rules/no-console-logs.md so it is physically located inside the target directory.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Rule files under .claude/rules/ support YAML frontmatter that can include a path filter (e.g., a glob pattern matching frontend/**). Claude Code reads this metadata to determine whether the rule file applies to the current working file's path, enabling a centrally stored rule to be selectively activated only for specific directories or file patterns.

References:
- https://docs.anthropic.com/en/docs/claude-code/memory

**Why the others are wrong:**

- **A:** @scope is not a recognized Claude Code directive; path filtering is achieved through YAML frontmatter, not inline directives.
- **C:** Claude Code does not parse filename prefixes to infer path scope; filenames under .claude/rules/ are arbitrary and the scope is controlled by frontmatter metadata.
- **D:** Rules files placed at frontend/.claude/rules/ would not follow the standard .claude/rules/ convention and rely on physical placement rather than explicit path metadata; the standard mechanism is YAML frontmatter in the centrally located file.

</details>

---

#### C202

*TS 3.3 · Path-specific rules*

**Situation:** A developer is setting up a `.claude/rules/react-conventions.md` file. She wants the rules in this file to activate only when Claude is editing `.tsx` files inside the `src/` directory.

Which YAML frontmatter block correctly restricts this rule file to `src/**/*.tsx` files?

- **A)**
  ```yaml
  ---
  glob: src/**/*.tsx
  ---
  ```
- **B)**
  ```yaml
  ---
  apply_to: src/**/*.tsx
  ---
  ```
- **C)**
  ```yaml
  ---
  paths:
    - src/**/*.tsx
  ---
  ```
- **D)**
  ```yaml
  ---
  scope: src/**/*.tsx
  ---
  ```

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Claude's path-specific rule loading uses the `paths:` key in YAML frontmatter. A list of glob patterns under `paths:` tells Claude to activate the rule file only when the file being edited matches one of those patterns. The other keys (`apply_to`, `glob`, `scope`) are not recognized by Claude's rules system.

References:
- https://docs.anthropic.com/en/docs/claude-code/memory#rules-directory

**Why the others are wrong:**

- **A:** `glob` is not the correct frontmatter field name. The recognized field for path-based activation is `paths:` (plural), which accepts a YAML list of glob patterns.
- **B:** `apply_to` is not a valid frontmatter key in Claude's rules system. Using an unrecognized key means the rule file will either be ignored entirely or applied unconditionally, defeating the conditional loading goal.
- **D:** `scope` is not a recognized frontmatter key in Claude's `.claude/rules/` convention system. Only `paths:` triggers conditional loading based on file globs.

</details>

---

#### C203

*TS 3.3 · Path-specific rules*

**Situation:** A team stores rule files in `.claude/rules/`. One file has no YAML frontmatter at all. Another file has a `paths:` field listing `tests/**/*.test.ts`.

What is the key behavioral difference between a rule file with no frontmatter and one with a `paths:` frontmatter field?

- **A)** A file with no frontmatter applies only to the root directory; a `paths:` file applies recursively.
- **B)** Both files are always active; `paths:` is only used for documentation purposes.
- **C)** A file with no frontmatter is never loaded; a `paths:` file is always loaded.
- **D)** A file with no frontmatter is always active for every file Claude edits; a `paths:` file is only active when the edited file matches a listed glob pattern.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Rule files without frontmatter are unconditionally loaded for every editing context. Rule files with a `paths:` frontmatter field are conditionally loaded — Claude activates them only when the file currently being edited matches at least one of the glob patterns in the list.

References:
- https://docs.anthropic.com/en/docs/claude-code/memory#rules-directory

**Why the others are wrong:**

- **A:** There is no root-directory-only scope for frontmatter-free rule files. They apply globally across all files Claude edits in the project.
- **B:** `paths:` is not decorative; it directly controls whether the rule file is included in Claude's active context for a given edit. Treating it as documentation would cause the rules to apply everywhere.
- **C:** Files without frontmatter are not ignored — they are loaded unconditionally. The absence of a `paths:` field means there is no restriction, not that the file is skipped.

</details>

---

#### C204

*TS 3.3 · Path-specific rules*

**Situation:** A platform team is designing a Claude rules architecture for a large monorepo with this structure:

```
packages/
  web/src/        (React + TypeScript)
  mobile/src/     (React Native + TypeScript)
  shared/src/     (shared TypeScript utilities)
  backend/src/    (Node.js TypeScript)
tests/
  e2e/            (Playwright TypeScript)
  unit/           (Jest TypeScript)
```

The team has these rule requirements:
1. React hooks rules → only `web/` and `mobile/` component files
2. Node.js async patterns → only `backend/`
3. Shared utility standards → only `shared/`
4. General TypeScript rules → all `.ts`/`.tsx` files
5. Test assertion style → all test files
6. E2E page object patterns → only `tests/e2e/`

Which rule file architecture correctly implements all six requirements using path-specific loading?

- **A)** Six `.claude/rules/` files, each with a focused `paths:` frontmatter that covers exactly its stated scope, plus a root `CLAUDE.md` for project-wide conventions.
- **B)** One `.claude/rules/all-rules.md` file with all six rule sets and a `paths:` list containing `**/*.ts` and `**/*.tsx` to catch everything.
- **C)** Six `.claude/rules/` files with no frontmatter, relying on directory-level CLAUDE.md files to override them per package.
- **D)** One rule file per package directory, each without frontmatter, placed inside the package's own CLAUDE.md.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

The correct architecture creates one focused rule file per requirement, each with a precisely scoped `paths:` list:
- `react-hooks.md`: `paths: [packages/web/src/**/*.tsx, packages/mobile/src/**/*.tsx]`
- `node-async.md`: `paths: [packages/backend/src/**/*.ts]`
- `shared-utils.md`: `paths: [packages/shared/src/**/*.ts]`
- `typescript-general.md`: `paths: [**/*.ts, **/*.tsx]`
- `test-assertions.md`: `paths: [tests/**/*.ts]`
- `e2e-patterns.md`: `paths: [tests/e2e/**/*.ts]`

This gives each rule file single responsibility, ensures rules activate only in their correct context, and allows requirements 4 and 5 to combine additively with more specific rules where they overlap.

References:
- https://docs.anthropic.com/en/docs/claude-code/memory#rules-directory
- https://docs.anthropic.com/en/docs/claude-code/memory#directory-structure

**Why the others are wrong:**

- **B:** Consolidating all six rule sets into one file with broad `**/*.ts` and `**/*.tsx` patterns activates every rule for every TypeScript file. React hooks rules would fire for backend Node.js files, and Node.js async patterns for React components — the opposite of the desired isolation.
- **C:** Rule files without frontmatter are globally active and cannot be selectively suppressed by directory-level CLAUDE.md files. Directory CLAUDE.md files add context additively; they do not override or disable `.claude/rules/` files.
- **D:** Rules in directory-level CLAUDE.md files without frontmatter do not enable path-specific loading within a package. Every file edited while Claude is aware of that CLAUDE.md would receive the rules, whether or not it matches the intended scope (e.g., component vs. utility files within `web/src/`).

</details>

---

#### C205

*TS 3.3 · Path-specific rules*

**Situation:** A team has:
- `.claude/rules/strict-null-checks.md` with `paths: [src/**/*.ts]`
- `.claude/rules/migration-helpers.md` with `paths: [src/legacy/**/*.ts, src/migration/**/*.ts]`
- `src/legacy/CLAUDE.md` with notes about avoiding strict null usage during migration

A developer edits `src/legacy/adapters/old-service.ts`.

What is the resulting active rule set, and what architectural tension exists in this configuration?

- **A)** Only `migration-helpers.md` is active because it has the most specific glob. No tension exists.
- **B)** All three sources are active simultaneously. The tension is that `strict-null-checks.md` enforces strict rules that directly contradict the `src/legacy/CLAUDE.md` guidance to avoid strict null usage during migration.
- **C)** `src/legacy/CLAUDE.md` overrides both `.claude/rules/` files because directory-level CLAUDE.md files have higher precedence.
- **D)** `strict-null-checks.md` and `migration-helpers.md` are both active, but `src/legacy/CLAUDE.md` is ignored because `.claude/rules/` files take precedence over directory-level CLAUDE.md files.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

All three sources are additive and simultaneously active when editing `src/legacy/adapters/old-service.ts`: (1) `strict-null-checks.md` matches via `src/**/*.ts`; (2) `migration-helpers.md` matches via `src/legacy/**/*.ts`; (3) `src/legacy/CLAUDE.md` is loaded as an ancestor directory file. The architectural tension is real and significant: `strict-null-checks.md` instructs Claude to enforce strict null handling everywhere in `src/`, while `src/legacy/CLAUDE.md` instructs Claude to relax those rules during migration. Claude receives contradictory instructions. The correct fix is to add a negative exclusion to `strict-null-checks.md` (e.g., exclude `src/legacy/**`) or to remove the broad `src/**/*.ts` pattern and list only non-legacy paths.

References:
- https://docs.anthropic.com/en/docs/claude-code/memory#rules-directory
- https://docs.anthropic.com/en/docs/claude-code/memory#directory-structure

**Why the others are wrong:**

- **A:** Claude does not apply a "most specific glob wins" exclusion. All matching rule files activate simultaneously, so both `.claude/rules/` files are active, creating the overlap. The tension is real and architecturally significant.
- **C:** Directory-level CLAUDE.md files do not have higher precedence that overrides `.claude/rules/` files. The system is additive — all sources contribute to Claude's context, which is exactly why the contradiction is a problem.
- **D:** `.claude/rules/` files do not suppress or override directory-level CLAUDE.md files. Both systems are additive, meaning `src/legacy/CLAUDE.md` is loaded alongside the two rule files, not ignored.

</details>

---

#### C206

*TS 3.3 · Path-specific rules*

**Situation:** A monorepo contains a React frontend in `packages/ui/src/` and a Node.js API in `packages/api/src/`. The team has:
- `.claude/rules/react.md` with `paths: [packages/ui/src/**/*.tsx]`
- `.claude/rules/node-api.md` with `paths: [packages/api/src/**/*.ts]`
- `packages/ui/CLAUDE.md` with React component structure guidelines

An engineer edits `packages/ui/src/components/Button.tsx`.

Which combination of rule sources is active when editing `Button.tsx`?

- **A)** Only `.claude/rules/react.md` is active because it has the most specific glob pattern.
- **B)** `.claude/rules/react.md` and `packages/ui/CLAUDE.md` are both active; `.claude/rules/node-api.md` is not.
- **C)** All three sources are active because Claude loads all available rule files.
- **D)** Only `packages/ui/CLAUDE.md` is active because directory-level CLAUDE.md files take precedence over `.claude/rules/` files.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Claude's memory system combines path-specific rules and directory-level CLAUDE.md files. When `Button.tsx` is edited: (1) `.claude/rules/react.md` is activated because `packages/ui/src/components/Button.tsx` matches the glob `packages/ui/src/**/*.tsx`; (2) `packages/ui/CLAUDE.md` is loaded because it resides in an ancestor directory of the file being edited; (3) `.claude/rules/node-api.md` is not activated because `Button.tsx` does not match `packages/api/src/**/*.ts`.

References:
- https://docs.anthropic.com/en/docs/claude-code/memory#rules-directory
- https://docs.anthropic.com/en/docs/claude-code/memory#directory-structure

**Why the others are wrong:**

- **A:** Claude does not apply a "most specific wins" exclusion rule. All rule files whose `paths:` patterns match the current file are activated simultaneously, alongside any applicable directory-level CLAUDE.md files.
- **C:** `.claude/rules/node-api.md` is not active because `Button.tsx` does not match the pattern `packages/api/src/**/*.ts`. Path-specific rules are only loaded when their glob patterns match the file being edited.
- **D:** Directory-level CLAUDE.md files do not suppress or override `.claude/rules/` files. Both systems are additive — Claude merges all applicable rule sources into its active context.

</details>

---

#### C207

*TS 3.3 · Path-specific rules*

**Situation:** A rule file `.claude/rules/test-conventions.md` contains the following frontmatter:

```yaml
---
paths:
  - tests/**/*.test.ts
  - tests/**/*.spec.ts
  - '**/__tests__/**/*.ts'
---
```

An engineer is editing `src/__tests__/utils/formatter.test.ts`.

Does the rule file activate for this file, and which pattern is responsible?

- **A)** Yes; the pattern `tests/**/*.test.ts` matches because the filename ends in `.test.ts`.
- **B)** Yes; all three patterns match simultaneously and Claude deduplicates the rule content.
- **C)** Yes; the pattern `**/__tests__/**/*.ts` matches because the file path contains `__tests__/`.
- **D)** No; none of the patterns match because the file is in `src/`, not in `tests/`.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Glob matching evaluates each pattern independently against the full file path. `tests/**/*.test.ts` requires the path to start with `tests/`, so it does not match `src/__tests__/...`. `tests/**/*.spec.ts` similarly fails. However, `**/__tests__/**/*.ts` uses a leading `**` that matches any number of path segments, so it matches `src/__tests__/utils/formatter.test.ts`. Therefore the rule file activates, and the responsible pattern is the third one.

References:
- https://docs.anthropic.com/en/docs/claude-code/memory#rules-directory

**Why the others are wrong:**

- **A:** `tests/**/*.test.ts` is anchored to a `tests/` prefix. The file lives under `src/__tests__/`, not `tests/`, so this specific pattern does not match — even though the filename ends in `.test.ts`.
- **B:** Only the third pattern matches; the first two do not. While Claude does load all applicable patterns from a single file, in this case only one pattern is responsible for activation.
- **D:** The third pattern `**/__tests__/**/*.ts` is not anchored and can match any path containing `__tests__` as a directory segment, regardless of the leading `src/`. At least one pattern matches, so the rule file is activated.

</details>

---

#### C208

*TS 3.3 · Path-specific rules*

**Situation:** A project has the following layout:

```
.claude/rules/api-rules.md   (paths: src/api/**/*.ts)
CLAUDE.md                    (project-wide conventions)
src/api/CLAUDE.md            (API-specific conventions)
```

A developer edits `src/api/handlers/user.ts`.

Describe the correct precedence and loading order for these three rule sources.

- **A)** Only the most specific source (`src/api/CLAUDE.md`) is used; the others are suppressed.
- **B)** All three are loaded. Project root `CLAUDE.md` provides a baseline, `src/api/CLAUDE.md` adds directory-scoped context, and `.claude/rules/api-rules.md` adds path-matched rules. All three are merged additively.
- **C)** `.claude/rules/api-rules.md` takes highest precedence and overrides both CLAUDE.md files.
- **D)** Directory-level CLAUDE.md files are loaded first and always override `.claude/rules/` files.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Claude's memory system is additive. When editing `src/api/handlers/user.ts`: (1) The project root `CLAUDE.md` is always loaded as a baseline; (2) `src/api/CLAUDE.md` is loaded because it is in an ancestor directory of the file; (3) `.claude/rules/api-rules.md` is loaded because the file matches `src/api/**/*.ts`. There is no override or suppression between these sources — all contribute to Claude's active context simultaneously.

References:
- https://docs.anthropic.com/en/docs/claude-code/memory#directory-structure
- https://docs.anthropic.com/en/docs/claude-code/memory#rules-directory

**Why the others are wrong:**

- **A:** Claude does not implement a "most specific wins" suppression model. All applicable rule sources — root CLAUDE.md, ancestor directory CLAUDE.md files, and matching `.claude/rules/` files — are merged together.
- **C:** `.claude/rules/` files do not have a higher precedence that overrides CLAUDE.md files. The system is additive; there is no explicit precedence ranking that causes one source to suppress another.
- **D:** Directory-level CLAUDE.md files are not intrinsically more authoritative than `.claude/rules/` files. Both are loaded and contribute to the active context; neither category overrides the other.

</details>

---

#### C209

*TS 3.3 · Path-specific rules*

**Situation:** A team wants different naming conventions for test files depending on whether they are unit tests (`*.test.ts`) or integration tests (`*.integration.ts`). They consider two approaches:

**Option A:** One rule file with two entries under `paths:` and a conditional section inside the markdown.
**Option B:** Two separate rule files — `unit-test-conventions.md` (paths: `**/*.test.ts`) and `integration-test-conventions.md` (paths: `**/*.integration.ts`).

Which approach better follows the principle of high cohesion and low coupling for Claude rule files?

- **A)** Option A, because keeping related test conventions together in one file reduces the number of files to maintain.
- **B)** Option B, because each rule file has a single, focused responsibility and its `paths:` glob precisely targets the file type it governs.
- **C)** Both approaches are equivalent; the choice is purely stylistic.
- **D)** Option A, because Claude's path matching only supports one active rule file per file type at a time.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

High cohesion means each rule file governs exactly one concern. Option B creates two focused files, each with a precise glob that matches only the file type it describes. This avoids conditional logic inside a single file, keeps each rule file small and readable, and prevents unit-test rules from activating when only integration tests are being edited (and vice versa). Option A conflates two distinct concerns into one file, requiring readers to mentally parse which section applies.

References:
- https://docs.anthropic.com/en/docs/claude-code/memory#rules-directory

**Why the others are wrong:**

- **A:** Fewer files is not always better. A single file that uses internal conditionals to cover multiple distinct concerns has lower cohesion. When unit-test conventions change, a developer editing the combined file must also parse the integration-test section, increasing cognitive load.
- **C:** The approaches are not equivalent in terms of maintainability and precision. Option B allows each rule to activate independently and be updated independently, which is a meaningful structural difference, not just a stylistic one.
- **D:** Claude supports multiple rule files activating simultaneously for the same file. There is no one-active-file-per-type constraint, so this is not a valid reason to prefer Option A.

</details>

---

#### C210

*TS 3.3 · Path-specific rules*

**Situation:** An architect reviews a `.claude/rules/` directory and finds a rule file with this frontmatter:

```yaml
---
paths:
  - '**/*.ts'
  - '**/*.tsx'
  - '**/*.js'
  - '**/*.jsx'
  - '**/*.json'
  - '**/*.css'
  - '**/*.md'
---
```

The file contains general code style guidelines.

What is the primary architectural problem with this configuration?

- **A)** The `paths:` field must use absolute file paths, not glob patterns.
- **B)** Using `**` wildcards in `paths:` patterns causes performance degradation in large codebases.
- **C)** YAML frontmatter does not support more than three entries in a `paths:` list.
- **D)** The rule file effectively activates for nearly every file in the project, which is the same behavior as a rule file with no frontmatter — making the conditional loading mechanism pointless.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

The purpose of `paths:` frontmatter is to restrict a rule file to a specific context. When the patterns collectively cover almost every file type in the project, the rule file activates for virtually every edit — the same outcome as having no frontmatter at all. This defeats the goal of conditional convention loading. If the rules are truly universal, the correct approach is to remove the frontmatter entirely. If some rules are specific to certain file types, those rules should be split into focused, narrowly-scoped rule files.

References:
- https://docs.anthropic.com/en/docs/claude-code/memory#rules-directory

**Why the others are wrong:**

- **A:** `paths:` patterns are glob patterns, not absolute paths. Using glob syntax like `**/*.ts` is the correct and documented usage.
- **B:** There is no documented performance concern with `**` wildcard patterns in `.claude/rules/` frontmatter. The glob matching is a simple file-path check, not a filesystem traversal.
- **C:** There is no documented limit on the number of entries in a `paths:` list. YAML supports arbitrarily long sequences, and Claude processes all listed patterns.

</details>

---

#### C211

*TS 3.3 · Path-specific rules*

Test files in your project are co-located with source code (`Button.test.tsx` alongside `Button.tsx`, `api.test.py` alongside `api.py`) spread across 40+ directories. All test files should follow the same testing conventions. What's the most maintainable configuration?

- **A)** Put testing conventions in the root CLAUDE.md under a "Testing" header.
- **B)** Create a `.claude/rules/testing.md` file with YAML frontmatter: `paths: ["**/*.test.tsx", "**/*.test.ts", "**/*.test.py"]`.
- **C)** Create a CLAUDE.md file in every directory that contains test files.
- **D)** Create a testing skill in `.claude/skills/testing/SKILL.md`.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Glob patterns in `.claude/rules/` match scattered files by type regardless of directory.

**Why the others are wrong:**

- **A:** Root CLAUDE.md loads always, even when not editing tests.
- **C:** Per-directory CLAUDE.md is impractical at 40+ directories.
- **D:** Skills require manual invocation.

</details>

---

#### C212

*TS 3.3 · Path-specific rules*

Your CI code review is configured with a `CLAUDE.md` that includes review criteria. But a specific subpackage `packages/legacy-api/` has different standards (it uses callbacks instead of async/await, and that's intentional). How should you configure this?

- **A)** Add an exception note to the root CLAUDE.md: "In packages/legacy-api/, callbacks are acceptable."
- **B)** Create `.claude/rules/legacy-api.md` with `paths: ["packages/legacy-api/**/*"]` containing the specific conventions for this package, or create a `packages/legacy-api/CLAUDE.md` with the local standards.
- **C)** Instruct reviewers to ignore findings from the legacy-api package.
- **D)** Exclude the `packages/legacy-api/` directory from CI reviews entirely.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Path-specific rules in `.claude/rules/` or a directory-level CLAUDE.md for the legacy package. Ignoring findings (C) and excluding directories (D) lose review value.

**Why the others are wrong:**

- **A:** Root CLAUDE.md exceptions become messy at scale.

</details>

---

#### C213

*TS 3.3 · Path-specific rules*

Your shopping agent uses a `CLAUDE.md` with general product recommendation guidelines. But a specific category `electronics/premium-audio/` has different rules (requires age verification for certain products, special shipping restrictions). How should you configure this?

- **A)** Instruct the agent to always ask about age verification for electronics.
- **B)** Create `.claude/rules/premium-audio.md` with `paths: ["electronics/premium-audio/**/*"]` containing category-specific rules, or create `electronics/premium-audio/CLAUDE.md` with local standards.
- **C)** Add an exception note to the root CLAUDE.md: "In electronics/premium-audio/, age verification is required."
- **D)** Exclude the `electronics/premium-audio/` category from AI-assisted shopping entirely.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Path-specific rules in `.claude/rules/` or directory-level CLAUDE.md for category-specific requirements.

**Why the others are wrong:**

- **A:** Always asking creates friction.
- **C:** Root exceptions become messy.
- **D:** Excluding categories loses business value.

</details>

---

#### C214

*TS 3.3 · Path-specific rules*

Your supply chain agent uses a `CLAUDE.md` with general inventory guidelines. But a specific warehouse `warehouses/cold-storage/` has different rules (temperature-sensitive items, shorter shelf lives). How should you configure this?

- **A)** Instruct the agent to always ask about temperature requirements for cold storage items.
- **B)** Add an exception note to the root CLAUDE.md: "In cold-storage/, shelf life rules differ."
- **C)** Exclude the `warehouses/cold-storage/` warehouse from AI-assisted management entirely.
- **D)** Create `.claude/rules/cold-storage.md` with `paths: ["warehouses/cold-storage/**/*"]` containing warehouse-specific rules, or create `warehouses/cold-storage/CLAUDE.md` with local standards.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Path-specific rules in .claude/rules/ or directory-level CLAUDE.md for warehouse-specific requirements.

</details>

---

#### C215

*TS 3.4 · Plan mode vs direct execution*

**Situation:** A developer asks Claude to fix a single typo in a variable name inside one file. The variable appears in three places and needs to be renamed from `usr` to `user`.

Should Claude use plan mode or direct execution for this task?

- **A)** Plan mode, because any code change could have unintended side effects that warrant upfront analysis.
- **B)** Direct execution, because this is a simple, well-defined single-file rename with a clear and bounded scope.
- **C)** Plan mode, because the architect should always document intent before touching source code.
- **D)** Direct execution only after running the Explore subagent to map all usages across the entire codebase.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Direct execution is appropriate for simple, well-defined, single-file edits where the scope is clear and bounded. A three-occurrence variable rename in one file carries minimal risk and does not benefit from the overhead of a planning phase. Planning would add latency with no protective value.

References:
- https://docs.anthropic.com/en/docs/claude-code/plan-mode
- https://docs.anthropic.com/en/docs/claude-code/overview

**Why the others are wrong:**

- **A:** While side effects are always a concern, the planning overhead is not justified for a trivially scoped rename limited to one file; applying plan mode universally reduces its signal value.
- **C:** Documenting intent is valuable for complex architectural work, but imposing that overhead on a one-line rename is wasteful and slows delivery.
- **D:** Running the Explore subagent before a clearly scoped single-file rename adds unnecessary overhead; Explore is most valuable when the codebase impact is unknown, not when the change is fully specified.

</details>

---

#### C216

*TS 3.4 · Plan mode vs direct execution*

**Situation:** A team lead asks Claude to redesign the authentication module to support OAuth 2.0, replacing the current session-based system. This will affect the middleware stack, database schema, frontend login flow, and at least six API endpoints.

Which execution strategy is most appropriate for this request?

- **A)** Direct execution with a rollback script prepared in advance.
- **B)** Plan mode, because the multi-system scope and irreversibility risk make upfront planning essential to prevent costly mistakes.
- **C)** Direct execution, because experienced architects know OAuth patterns well and can implement without planning.
- **D)** Direct execution, starting with the database schema and working outward.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

This is exactly the type of task that warrants plan mode: it spans multiple systems, involves an irreversible architectural shift, and a mistake in one layer (e.g., the database schema) could cascade into breaking changes across middleware, frontend, and API endpoints. Planning surfaces dependencies and risks before any code is touched.

References:
- https://docs.anthropic.com/en/docs/claude-code/plan-mode
- https://docs.anthropic.com/en/docs/claude-code/common-workflows

**Why the others are wrong:**

- **A:** A rollback script is a recovery mechanism, not a substitute for planning; it does not prevent mistakes during implementation, it only enables undoing them after the fact.
- **C:** Domain familiarity with OAuth does not eliminate the need to plan a cross-system migration; the risk comes from the breadth of change, not the technology.
- **D:** Starting with the database schema without a plan may produce a schema that conflicts with decisions made later in the middleware or API layers, requiring costly rework.

</details>

---

#### C217

*TS 3.4 · Plan mode vs direct execution*

**Situation:** A developer tells Claude: 'Add a `created_at` timestamp field to the `Order` model and update the one migration file that creates the orders table.'

What is the primary reason this task does NOT warrant plan mode?

- **A)** Plan mode is only for architectural decisions, and adding a field is a feature task.
- **B)** Plan mode requires a minimum of five affected files to be justified.
- **C)** The task is simple and well-defined: one model field and one migration file with a clear, bounded scope that does not justify planning overhead.
- **D)** The task is read-only and therefore carries no risk.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Plan mode overhead is not justified when the task is simple and well-defined with a clear scope. The developer has fully specified what needs to change (one field, one file), leaving no ambiguity that planning would resolve. Applying plan mode here would add latency without adding protective value.

References:
- https://docs.anthropic.com/en/docs/claude-code/plan-mode
- https://docs.anthropic.com/en/docs/claude-code/overview

**Why the others are wrong:**

- **A:** The plan vs. direct distinction is based on complexity and risk, not on a classification of 'architectural' vs. 'feature' work; simple feature tasks and simple architectural tasks both skip planning.
- **B:** There is no file-count threshold for triggering plan mode; the criterion is task complexity, ambiguity, and risk of costly mistakes.
- **D:** The task is not read-only; it modifies source code and a migration file. The reason to skip planning is scope clarity, not read-only status.

</details>

---

#### C218

*TS 3.4 · Plan mode vs direct execution*

**Situation:** A developer asks Claude to extract a utility function `formatCurrency` that currently appears duplicated in three service files into a shared `utils/currency.ts` file, and update the three import sites.

Should Claude use plan mode or direct execution, and why?

- **A)** Direct execution only if the developer confirms no other files import the function.
- **B)** Plan mode, because any refactor touching multiple files requires a documented plan.
- **C)** Direct execution, because the task is a clearly scoped, well-defined refactor with a known set of four files and no architectural ambiguity.
- **D)** Plan mode, because creating a new file introduces architectural decisions about module structure.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

This refactor is well-defined: one new utility file and three known import-site updates. The scope is fully bounded by the developer's description. Direct execution is appropriate because there is no ambiguity and no risk of cascading mistakes that planning would prevent. The task is clear enough that planning would add overhead without value.

References:
- https://docs.anthropic.com/en/docs/claude-code/plan-mode
- https://docs.anthropic.com/en/docs/claude-code/common-workflows

**Why the others are wrong:**

- **A:** The developer has specified the three import sites; if there were other sites, the developer would need to clarify, but that clarification is a question to ask, not a reason to enter plan mode.
- **B:** Touching multiple files does not automatically justify plan mode; the criterion is complexity and ambiguity. A bounded four-file refactor with a fully specified plan from the developer does not benefit from Claude generating a redundant planning layer.
- **D:** Creating a `utils/` file is a routine structural decision, not an architectural decision that requires upfront planning; it does not affect system boundaries, data flow, or cross-team contracts.

</details>

---

#### C219

*TS 3.4 · Plan mode vs direct execution*

**Situation:** An architect is evaluating whether to use plan mode for a task described as: 'Migrate our CI/CD pipeline from Jenkins to GitHub Actions, update all Dockerfiles to multi-stage builds, and refactor the Kubernetes deployment manifests to use Helm charts.' The team has never used Helm before and the codebase contains 12 microservices each with separate CI configurations.

Which analysis most precisely justifies the correct execution strategy for this task?

- **A)** Plan mode is warranted, but only after direct execution of one microservice as a proof-of-concept to inform the plan.
- **B)** Direct execution is appropriate because CI/CD, Docker, and Kubernetes are well-documented technologies with abundant examples.
- **C)** The Explore subagent alone is sufficient; after it maps the 12 services, Claude can execute directly without a formal plan.
- **D)** Plan mode is warranted because the task is multi-dimensional (three distinct migration tracks), introduces technology the team has never used (Helm), spans 12 services, and carries a high risk of service-disrupting mistakes if sequencing is wrong — all criteria that make planning prevent costly errors.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

This task satisfies multiple plan-mode triggers simultaneously: multi-dimensional scope (three distinct migration tracks), unfamiliar technology (first Helm usage introducing a learning-curve risk), breadth (12 services), and high consequences if sequencing or configuration decisions are wrong (production CI disruption). Plan mode prevents costly mistakes by surfacing these dependencies and risks before any service is touched.

References:
- https://docs.anthropic.com/en/docs/claude-code/plan-mode
- https://docs.anthropic.com/en/docs/claude-code/sub-agents
- https://docs.anthropic.com/en/docs/claude-code/common-workflows

**Why the others are wrong:**

- **A:** A proof-of-concept execution before planning inverts the recommended workflow; the POC will surface decisions that should be captured in the plan first, not used to retroactively inform it. Mistakes in the POC may also be costly to unwind.
- **B:** Technology familiarity reduces individual task complexity but does not eliminate the risk posed by three concurrent migration tracks across 12 services with an unfamiliar tool; the compounding scope is the primary risk driver.
- **C:** Explore maps what exists; it does not resolve the sequencing, Helm design, and cross-service dependency decisions that plan mode addresses. Explore feeds the plan; it does not replace it.

</details>

---

#### C220

*TS 3.4 · Plan mode vs direct execution*

**Situation:** A developer asks Claude to 'clean up the codebase.' No further specification is provided. The codebase has 80,000 lines across 200 files in a production e-commerce system.

What is the correct sequence of actions Claude should take, and why?

- **A)** Enter plan mode immediately and generate a comprehensive cleanup plan based on common code quality heuristics.
- **B)** Begin direct execution by removing unused imports and dead code across all 200 files.
- **C)** Refuse the task because 'clean up the codebase' is too vague to act on safely.
- **D)** Use the Explore subagent to understand codebase structure and identify cleanup opportunities, then enter plan mode to define scope, prioritize changes, and get explicit approval before touching any production code.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

An underspecified request affecting 200 files in a production system requires the full sequence: Explore first to ground the analysis in actual code quality issues, then plan mode to define scope and sequence, then explicit approval before execution. 'Clean up' is ambiguous (dead code removal? formatting? refactoring? dependency pruning?) and each interpretation carries different risk profiles in a production system. The Explore→Plan→Approve→Execute sequence prevents both wasted effort and production-breaking changes.

References:
- https://docs.anthropic.com/en/docs/claude-code/plan-mode
- https://docs.anthropic.com/en/docs/claude-code/sub-agents
- https://docs.anthropic.com/en/docs/claude-code/common-workflows

**Why the others are wrong:**

- **A:** Planning with only heuristics and no codebase investigation will produce a plan disconnected from actual code quality issues; it may target clean code and miss real problems, or propose changes that break working patterns specific to this codebase.
- **B:** Direct execution of a vague 'clean up' request across 200 production files is extremely high risk; without scope definition and approval, Claude may remove code that appears unused but is invoked dynamically, breaking production.
- **C:** Refusing the task is unhelpful; Claude should use Explore to gather information and surface a concrete, scoped proposal for developer approval, rather than declining to engage.

</details>

---

#### C221

*TS 3.4 · Plan mode vs direct execution*

**Situation:** A senior engineer argues that plan mode should always be used because 'planning never hurts and always improves outcomes.' A junior engineer counters that plan mode should never be used because 'it just adds latency.' An architect must adjudicate the debate.

Which response most precisely articulates the correct criterion for choosing between plan mode and direct execution?

- **A)** The senior engineer is correct: the overhead of planning is negligible and the benefit always outweighs the cost.
- **B)** The junior engineer is correct: direct execution with rapid iteration is always faster than planning, even for complex tasks.
- **C)** Both are wrong: the decision should be based on a complexity assessment — plan mode is warranted when tasks are multi-step, cross-cutting, architecturally significant, or carry a high risk of cascading mistakes; direct execution is appropriate when tasks are simple, well-defined, bounded, and the cost of a wrong first step is low.
- **D)** Plan mode should be used based on file count: tasks touching more than five files always warrant planning.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

The correct framework is a complexity assessment that weighs task scope, ambiguity, reversibility, and cascading risk. Plan mode adds genuine value when mistakes are costly, scope is unclear, or changes are irreversible; its overhead is unjustified when tasks are simple and well-defined. Neither always-plan nor never-plan is correct — the criterion is contextual risk and complexity.

References:
- https://docs.anthropic.com/en/docs/claude-code/plan-mode
- https://docs.anthropic.com/en/docs/claude-code/overview
- https://docs.anthropic.com/en/docs/claude-code/common-workflows

**Why the others are wrong:**

- **A:** Planning has real costs: latency, token consumption, and the risk of over-engineering simple tasks. The value of planning scales with task complexity; for a simple bug fix, those costs outweigh benefits.
- **B:** Rapid iteration on a complex, irreversible refactor without a plan accumulates technical debt and mistakes faster than planning would have cost; the 'just iterate' approach breaks down when each wrong step is expensive to reverse.
- **D:** File count is a weak proxy for complexity; a five-file change could be a simple rename or a critical interface migration. Architectural significance, reversibility, and cascading risk are more accurate criteria than a file threshold.

</details>

---

#### C222

*TS 3.4 · Plan mode vs direct execution*

**Situation:** Claude has been asked to implement a new feature: real-time collaborative editing for a document platform. The request spans WebSocket infrastructure, operational transformation (OT) conflict resolution, persistence of document deltas, frontend state synchronization, and presence indicators. The team has an existing WebSocket server but no OT library in use.

Which execution strategy and rationale is most architecturally sound?

- **A)** Plan mode immediately, generating a plan from the feature description without codebase investigation.
- **B)** Direct execution using an off-the-shelf OT library, since library selection removes the main complexity.
- **C)** Direct execution, starting with the WebSocket layer since it already exists, and discovering OT requirements iteratively.
- **D)** Explore subagent to map the existing WebSocket infrastructure and persistence layer, followed by plan mode to design the OT strategy, delta schema, and synchronization protocol, with explicit checkpoints before implementing each subsystem.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Real-time collaborative editing is a canonical complex multi-step task: it combines infrastructure, algorithm selection (OT), schema design, and frontend protocol changes that are tightly interdependent. The correct approach is Explore first (to understand existing WebSocket and persistence infrastructure), then plan mode (to design the OT strategy and integration points before any code is written), with subsystem checkpoints to catch design errors early. Mistakes in OT algorithm selection or delta schema are extremely costly to reverse.

References:
- https://docs.anthropic.com/en/docs/claude-code/plan-mode
- https://docs.anthropic.com/en/docs/claude-code/sub-agents
- https://docs.anthropic.com/en/docs/claude-code/common-workflows

**Why the others are wrong:**

- **A:** Planning without Explore produces a plan that ignores the actual shape of the existing WebSocket server and persistence layer; the plan may specify integration points that conflict with existing constraints, requiring full revision after Explore would have revealed them upfront.
- **B:** Library selection resolves one decision (OT algorithm) but does not eliminate the need to plan how it integrates with the existing infrastructure, how deltas are persisted, or how the frontend synchronizes state; the cross-system integration complexity remains.
- **C:** Starting with WebSocket and discovering OT requirements iteratively means OT algorithm and delta schema decisions will be made under implementation pressure with incomplete context, leading to design choices that are expensive to reverse once the WebSocket layer is built around them.

</details>

---

#### C223

*TS 3.4 · Plan mode vs direct execution*

**Situation:** An architect asks Claude to migrate a monolithic Node.js application to a microservices architecture. Before Claude begins writing any code, it needs to understand which modules exist, how they share data, and which inter-module dependencies are tightly coupled.

Which Claude Code feature should Claude use FIRST before entering plan mode?

- **A)** Request that the developer provide a full dependency diagram before Claude does anything.
- **B)** Plan mode immediately, using only the information provided in the prompt.
- **C)** The Explore subagent, to investigate the codebase structure and surface dependency relationships before planning.
- **D)** Direct execution — start splitting files immediately to discover dependencies empirically.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

The Explore subagent is designed for codebase investigation before planning. It reads and analyzes existing code to surface module structure, coupling patterns, and hidden dependencies that would be invisible from the prompt alone. Running Explore first ensures that the subsequent plan is grounded in accurate codebase reality rather than assumptions, reducing the risk of a plan that omits critical constraints.

References:
- https://docs.anthropic.com/en/docs/claude-code/plan-mode
- https://docs.anthropic.com/en/docs/claude-code/sub-agents

**Why the others are wrong:**

- **A:** Waiting for the developer to produce a full dependency diagram adds human overhead that Claude can handle autonomously with the Explore subagent; this is a case where Claude should investigate rather than delegate.
- **B:** Entering plan mode with only the prompt information for a codebase-wide architectural refactor will produce a plan built on assumptions; Explore is specifically designed to fill this gap before planning.
- **D:** Starting execution before understanding the dependency graph in a monolith-to-microservices migration is high risk; tightly coupled modules require careful sequencing that cannot be discovered cheaply by trial and error.

</details>

---

#### C224

*TS 3.4 · Plan mode vs direct execution*

**Situation:** A developer reports: 'The `calculateTax` function in `billing/tax.ts` returns the wrong value for EU VAT rates. The formula uses the wrong divisor.' They provide the correct formula.

Why is direct execution appropriate here, and what overhead does plan mode add without value?

- **A)** Neither mode applies; the developer should fix this themselves since they already know the correct formula.
- **B)** Direct execution is appropriate because the bug is precisely located, the fix is fully specified, and the scope is a single function. Plan mode would add latency without surfacing new information or preventing mistakes.
- **C)** Plan mode is actually warranted here because tax calculations affect financial data, which is high-stakes.
- **D)** Direct execution is appropriate because tax logic is always simple. Plan mode would add value for complex tax logic.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

The key criteria for skipping plan mode are: the fix is precisely located (one function, one file), the correct behavior is fully specified (developer provided the formula), and the scope is bounded. Plan mode adds value when there is ambiguity or cascading risk — neither is present here. The planning overhead would only delay a well-understood fix.

References:
- https://docs.anthropic.com/en/docs/claude-code/plan-mode
- https://docs.anthropic.com/en/docs/claude-code/overview

**Why the others are wrong:**

- **A:** Claude's role is to assist with well-defined fixes; the developer sharing the correct formula is providing specification, not delegating back the problem.
- **C:** High-stakes domains are not sufficient on their own to justify plan mode; the criterion is task complexity and ambiguity. A clearly specified single-function fix in a high-stakes domain is still a direct-execution candidate.
- **D:** The reason to skip plan mode is specificity of the fix, not the inherent simplicity of tax logic; a complex multi-file tax refactor with unclear requirements would warrant planning even if the domain is the same.

</details>

---

#### C225

*TS 3.4 · Plan mode vs direct execution*

**Situation:** A developer asks Claude to refactor the entire data-access layer of a 50,000-line codebase to use the Repository pattern. The current layer mixes raw SQL queries directly into service classes across 40+ files.

Which combination of steps represents the correct approach for this large refactor?

- **A)** Plan mode immediately, generating a generic repository pattern template without first investigating the codebase.
- **B)** Run the Explore subagent to map all SQL usage sites, then enter plan mode to define the repository interface, migration order, and rollback strategy before touching any files.
- **C)** Direct execution starting with the largest service class, then proceeding file by file.
- **D)** Ask the developer to produce a file list of all 40+ files before Claude begins.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

A large-scale refactor across 40+ files is a clear plan-mode candidate. However, for planning to be useful rather than generic, the Explore subagent should first map the actual SQL usage patterns, identify hotspots, and surface any non-standard patterns that would complicate a generic repository template. The resulting plan will then address the real codebase, not a hypothetical one.

References:
- https://docs.anthropic.com/en/docs/claude-code/plan-mode
- https://docs.anthropic.com/en/docs/claude-code/sub-agents
- https://docs.anthropic.com/en/docs/claude-code/common-workflows

**Why the others are wrong:**

- **A:** A generic plan built without Explore will miss codebase-specific patterns (e.g., raw transaction management, dynamic query building) that require tailored repository designs; the plan will need revision mid-execution, defeating its purpose.
- **C:** Starting direct execution on a 40-file refactor without a plan risks inconsistent interfaces, missed SQL sites, and breaking changes that compound as work progresses; this is exactly the scenario where planning prevents costly mistakes.
- **D:** Generating a file list is a task Claude can do autonomously with Explore; delegating this research to the developer adds unnecessary human overhead.

</details>

---

#### C226

*TS 3.4 · Plan mode vs direct execution*

**Situation:** An architect asks Claude to evaluate whether to use plan mode for the following two tasks: (1) Add a `description` field to the `Product` database model and the corresponding API response. (2) Introduce event sourcing to replace the current state-mutation persistence model across the entire domain layer.

Which assessment correctly applies the plan-vs-direct criteria to both tasks?

- **A)** Task 1 warrants plan mode because API changes are public contracts; Task 2 is direct execution because event sourcing is a well-understood pattern.
- **B)** Both warrant plan mode because both touch the database.
- **C)** Task 1 is direct execution (simple, bounded, well-defined two-file change); Task 2 warrants plan mode (architectural shift, broad scope, high risk of cascading mistakes).
- **D)** Neither warrants plan mode because both have known solutions.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

The plan-vs-direct decision is driven by complexity, scope, and the risk of cascading mistakes. Task 1 is a bounded two-file addition with no ambiguity — direct execution is appropriate. Task 2 is a domain-wide architectural shift from state-mutation to event sourcing: it affects persistence, query models, transaction handling, and replay logic across the entire domain layer, exactly the profile that justifies plan mode to prevent costly mistakes.

References:
- https://docs.anthropic.com/en/docs/claude-code/plan-mode
- https://docs.anthropic.com/en/docs/claude-code/overview

**Why the others are wrong:**

- **A:** API changes are public contracts, but adding a single field to one endpoint response is well-defined and bounded, not complex; conversely, dismissing event sourcing as 'well-understood' ignores the scope and irreversibility of migrating an entire domain layer.
- **B:** Database changes do not uniformly warrant plan mode; the criterion is scope and risk, not the layer touched. A simple field addition is categorically different from a persistence model overhaul.
- **D:** 'Known solutions' are not the relevant criterion; breadth of impact and risk of cascading mistakes are. Event sourcing is well-understood in theory but high-risk in practice when applied to an existing domain layer.

</details>

---

#### C227

*TS 3.4 · Plan mode vs direct execution*

You need to restructure a monolithic application into microservices. The change will affect 45+ files and requires decisions about service boundaries, API contracts, and database partitioning. Which approach is correct?

- **A)** Start with direct execution and only switch to plan mode if unexpected complexity arises.
- **B)** Use direct execution with a detailed upfront specification of exactly how each service should be structured.
- **C)** Use direct execution to start making changes and let the architecture emerge from the implementation.
- **D)** Enter plan mode to explore the codebase, understand dependencies, and design the approach before making changes.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Plan mode for complex, multi-file architectural decisions.

**Why the others are wrong:**

- **A:** Starting with direct execution ignores the stated complexity.
- **B:** Direct execution risks costly rework.
- **C:** Direct execution risks costly rework.

</details>

---

#### C228

*TS 3.4 · Plan mode vs direct execution*

During a complex investigation of a legacy codebase, the Explore subagent reads 200+ files and generates verbose output about class hierarchies, database schemas, and API endpoints. How should you handle this information?

- **A)** Keep all Explore output in the main conversation for completeness.
- **B)** The Explore subagent isolates verbose discovery output and returns a summary to the main agent, preserving the main session's context window.
- **C)** Manually copy the relevant findings and start a new session.
- **D)** Use `/compact` immediately after the Explore subagent finishes.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

The Explore subagent isolates verbose discovery output and returns a summary. Manual copying (C) and `/compact` (D) are reactive workarounds, not architectural solutions.

**Why the others are wrong:**

- **A:** Keeping everything exhausts context.

</details>

---

#### C229

*TS 3.5 · Iterative refinement*

**Situation:** A developer prompts Claude to write a sorting algorithm. The first attempt returns a working but unoptimized bubble sort. The developer wants a more efficient solution.

Which iterative refinement technique is MOST appropriate as an immediate next step?

- **A)** Provide specific feedback on the output, referencing the bottleneck (O(n²) complexity) and requesting a more efficient algorithm such as quicksort or mergesort.
- **B)** Increase the temperature parameter so Claude explores a wider variety of responses.
- **C)** Discard the conversation entirely and restart with a new system prompt.
- **D)** Add more examples of unrelated sorting problems to the prompt to give Claude more context.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Iterative refinement works by giving Claude targeted, specific feedback on what is wrong with the current output and what improvement is desired. Naming the exact issue (quadratic complexity) and the direction of improvement (efficient algorithm) gives Claude a precise correction signal, enabling progressive improvement without discarding prior context.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/chain-prompts

**Why the others are wrong:**

- **B:** Temperature affects output diversity, not capability or quality. Raising temperature risks introducing bugs rather than producing a more efficient algorithm.
- **C:** Restarting from scratch throws away the working baseline and conversational context. Iterative refinement is the appropriate pattern when a partial result already exists and only targeted improvements are needed.
- **D:** Adding unrelated examples introduces noise without addressing the specific deficiency. Feedback loops should be surgical: reference what was produced and state exactly what must change.

</details>

---

#### C230

*TS 3.5 · Iterative refinement*

**Situation:** A team wants Claude to generate product descriptions in a specific brand voice. They have several approved examples of copy written by their marketing department.

What is the MOST effective way to demonstrate the desired output format and tone to Claude?

- **A)** Provide two or three approved product descriptions as input/output examples in the prompt, showing the exact format and voice expected.
- **B)** Ask Claude which tone it recommends and accept that suggestion.
- **C)** Describe the tone in abstract terms such as 'professional yet friendly'.
- **D)** Append a long list of adjectives that describe the brand personality.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Input/output examples (few-shot prompting) are one of the most reliable ways to demonstrate desired behavior. Concrete approved examples show Claude the exact tone, structure, and vocabulary expected far more precisely than abstract adjectives or descriptions, enabling consistent replication of the brand voice.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-examples
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview

**Why the others are wrong:**

- **B:** Claude's default tone preferences may not align with the brand's established voice. The refinement objective is to transfer a specific known standard, not to discover a new one.
- **C:** Abstract descriptors like 'professional yet friendly' are ambiguous and interpreted inconsistently across runs. Concrete examples eliminate ambiguity and anchor Claude to a specific observable standard.
- **D:** A list of adjectives describes attributes of the tone but does not demonstrate how those attributes manifest in actual prose. Examples convey more information per token than adjective lists.

</details>

---

#### C231

*TS 3.5 · Iterative refinement*

**Situation:** A developer is building a complex data pipeline and asks Claude in a single prompt to design the architecture, write all the code, handle edge cases, and document everything.

What does progressive refinement suggest about this approach?

- **A)** Claude performs better when given the full specification upfront, so this approach is optimal.
- **B)** Combining all tasks into one prompt maximizes efficiency and should be the standard approach.
- **C)** The developer should start with a simple version — for example, the architecture alone — verify it meets requirements, then incrementally add complexity in subsequent turns.
- **D)** The developer should ask Claude to estimate a complexity score before deciding how to proceed.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Progressive refinement prescribes starting simple and adding complexity incrementally. Attempting to specify every detail in one prompt makes it difficult to identify which part of a failure is the root cause. A phased approach — architecture first, then code, then edge cases, then documentation — allows verification at each step and keeps the feedback loop tight.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/chain-prompts
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview

**Why the others are wrong:**

- **A:** While complete specifications can help Claude understand scope, they do not substitute for iterative verification. Claude may produce an internally consistent but incorrect solution that went unchecked at any intermediate stage.
- **B:** Combining everything into one prompt reduces the ability to course-correct at intermediate stages. When the single large output is wrong, it is unclear whether the architecture, the code, or the documentation is responsible.
- **D:** Complexity scoring is not a standard technique and does not determine how prompts should be structured. The decision to iterate is driven by task decomposability, not by an abstract score.

</details>

---

#### C232

*TS 3.5 · Iterative refinement*

**Situation:** A user asks Claude to draft an email, and Claude produces a draft that is too formal. The user wants a more conversational tone.

Which feedback approach will MOST reliably produce the desired correction in the next iteration?

- **A)** Point out specifically that the tone is too formal, provide one or two example phrases from the draft that should be changed, and explain the desired register (e.g., 'conversational, as if writing to a colleague').
- **B)** Clear the conversation and rewrite the original request from scratch with no examples.
- **C)** Simply say 'Try again' without additional context.
- **D)** Tell Claude the email is 'not quite right' and ask it to redo it.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Effective feedback in a refinement loop is specific, actionable, and references the actual output. Identifying the dimension that needs change (tone), quoting concrete phrases, and describing the target state gives Claude a clear correction signal. Vague feedback forces Claude to guess what was wrong, producing inconsistent improvements.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-examples

**Why the others are wrong:**

- **B:** Restarting discards the draft that is otherwise close to correct. Iterative refinement is the right technique when the output only needs targeted adjustment, not wholesale replacement.
- **C:** 'Try again' provides no signal about what was wrong. Without knowing the failure mode, Claude may generate a different but equally formal draft or change something that was already correct.
- **D:** 'Not quite right' is only marginally more informative than 'try again'. It confirms something is wrong but gives no guidance on dimension, severity, or direction of change.

</details>

---

#### C233

*TS 3.5 · Iterative refinement*

**Situation:** An engineer is using the interview pattern to have Claude gather requirements before generating a database schema. The engineer sends: 'Design a database schema for our app.' Claude immediately outputs a complete schema without asking any questions.

How should the engineer's prompt be modified to correctly invoke the interview pattern?

- **A)** Explicitly instruct Claude in the system prompt to ask a series of clarifying questions about the domain, scale, and access patterns before producing any schema, and to wait for answers before proceeding.
- **B)** Add 'Be thorough' to the prompt so Claude asks more questions.
- **C)** Use a higher temperature so Claude is more likely to output questions rather than answers.
- **D)** Include a sample schema in the prompt so Claude can model what level of detail is expected.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

The interview pattern requires an explicit instruction that changes Claude's default behavior from 'answer immediately' to 'ask questions first, then answer.' Without this instruction, Claude follows its default of providing a helpful answer directly. The system prompt is the right place to configure this behavior persistently, and the instruction must specify both the action (ask clarifying questions) and the condition (before producing output).

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/chain-prompts

**Why the others are wrong:**

- **B:** 'Be thorough' signals that the output should be detailed, not that Claude should gather requirements first. It does not alter the sequencing of question-then-answer.
- **C:** Temperature controls output diversity, not the structure of the interaction. The interview pattern is a behavioral instruction, not a sampling parameter.
- **D:** A sample schema demonstrates format, not process. It would make Claude produce a similarly shaped schema, not prompt it to ask questions about the specific domain first.

</details>

---

#### C234

*TS 3.5 · Iterative refinement*

**Situation:** A team is using test-driven iteration to build a Claude-powered text classifier. They write a test suite of labeled examples and check the classifier's predictions against them. After the first implementation, 15 of 50 test cases fail. They provide Claude the failing cases and ask it to revise the classification logic.

Which statement BEST describes test-driven iteration as a refinement technique?

- **A)** The team should increase the number of test cases to 200 before attempting any refinement, to ensure statistical significance.
- **B)** Because 70% of tests pass, the implementation is acceptable and no further refinement is needed.
- **C)** The team should only show Claude the passing cases so it knows what correct behavior looks like.
- **D)** The failing test cases serve as structured, objective feedback that pinpoints specific gaps in the implementation, allowing Claude to make targeted corrections rather than guessing what is wrong.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Test-driven iteration uses failing tests as a precise, objective feedback signal. Showing Claude the specific inputs where its output was wrong — along with the expected output — gives it exactly the information needed to correct the logic. This is more efficient than describing the problem in the abstract because the test cases are executable specifications of desired behavior.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview
- https://docs.anthropic.com/en/docs/build-with-claude/evaluate-and-improve/define-success

**Why the others are wrong:**

- **A:** Adding more test cases broadens coverage but does not help Claude fix the already-identified failures. Refinement should begin on known failures; expanding the test suite is a separate quality activity.
- **B:** A 70% pass rate means 30% of the real-world use cases are handled incorrectly. Whether that is acceptable depends on requirements, but declaring it sufficient without examining the failure patterns skips the refinement step entirely.
- **C:** Showing only passing cases gives Claude positive reinforcement but no signal about what to fix. The failing cases are the most informative data points for improvement.

</details>

---

#### C235

*TS 3.5 · Iterative refinement*

**Situation:** After six rounds of iterative refinement on a code generation prompt, Claude's output still has the same structural flaw: it always wraps results in a deeply nested object rather than a flat one. Each iteration only moves the nesting one level shallower.

Which approach is MOST appropriate given this pattern?

- **A)** Switch to a different Claude model to see whether the structural preference changes.
- **B)** Accept the nested structure since it is functionally equivalent and document the behavior for downstream consumers.
- **C)** Evaluate whether the fundamental issue is in the prompt's framing or examples rather than the incremental feedback, and consider rewriting the prompt with an explicit flat-structure example and a negative example showing what to avoid.
- **D)** Continue iterating — six rounds is insufficient; ten rounds is a common threshold before considering a restart.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

When iterative refinement produces diminishing returns or the same error persists across many cycles, that is a signal to pause and diagnose the root cause rather than iterate further. Recurring structural errors often indicate a missing or contradictory constraint in the original prompt. Adding a concrete positive example (flat structure) and a negative example (nested structure to avoid) directly addresses the misalignment, which incremental feedback alone may be unable to correct.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-examples

**Why the others are wrong:**

- **A:** Switching models without diagnosing the prompt issue is likely to reproduce the same problem. Structural errors in output are almost always addressable through prompt changes before model substitution is warranted.
- **B:** Accepting a wrong structural pattern and documenting it as behavior shifts the cost downstream. If the requirement is a flat structure, the prompt should be fixed rather than propagating a workaround.
- **D:** There is no universal iteration threshold. Continuing to iterate when the same error repeats suggests the feedback mechanism is insufficient, not that more cycles will eventually fix it.

</details>

---

#### C236

*TS 3.5 · Iterative refinement*

**Situation:** A product manager wants to use Claude to generate a user research plan. They have never done this before and are unsure what information Claude needs to produce a useful plan.

Why is the interview pattern particularly well-suited to this situation?

- **A)** The interview pattern instructs Claude to produce multiple alternative plans so the product manager can choose.
- **B)** The interview pattern allows Claude to surface the unknowns the product manager has not yet considered — such as target user segment, research goals, and timeline — before committing to a plan that may need to be discarded.
- **C)** The interview pattern is required by the Anthropic API when the user is not an expert in the domain.
- **D)** The interview pattern forces Claude to produce output faster by skipping the planning phase.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

The interview pattern is most valuable when the requester does not know what they do not know. By asking structured clarifying questions before generating output, Claude helps surface hidden assumptions and missing requirements. This prevents the common failure mode where an elaborate output is delivered but turns out to be wrong because a critical parameter (e.g., target users, budget, methodology preference) was never specified.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/chain-prompts

**Why the others are wrong:**

- **A:** Producing multiple alternatives is a different technique (comparative generation). The interview pattern is specifically about gathering requirements before committing to any output.
- **C:** The interview pattern is a prompt engineering technique, not an API feature. It is implemented entirely through prompt instructions and can be applied in any domain regardless of user expertise.
- **D:** The interview pattern actually adds a question-and-answer step before output, which takes more turns. Its value is in output quality and relevance, not speed.

</details>

---

#### C237

*TS 3.6 · Claude Code in CI/CD*

**Situation:** A DevOps engineer wants to run Claude Code inside a GitHub Actions workflow to automatically summarize what changed in each pull request. The workflow must complete without any human interaction.

Which Claude Code flag is required to prevent Claude Code from waiting for user input during the CI workflow run?

- **A)** --headless to disable the interactive UI layer
- **B)** -p "<prompt>" to run Claude Code in non-interactive mode
- **C)** --no-tty to suppress terminal allocation
- **D)** --batch to enable bulk processing of multiple prompts

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

The `-p` (or `--print`) flag accepts a prompt string and runs Claude Code in non-interactive mode, returning the response and exiting immediately. This is the correct way to drive Claude Code from a CI script where no human is present to respond to prompts.

References:
- https://docs.anthropic.com/en/docs/claude-code/cli-reference
- https://docs.anthropic.com/en/docs/claude-code/github-actions

**Why the others are wrong:**

- **A:** `--headless` is not a Claude Code flag. Claude Code's interactive vs. non-interactive behavior is controlled by the presence of the `-p` flag.
- **C:** `--no-tty` is not a Claude Code flag. TTY allocation is a shell/SSH concern, not a Claude Code concern. The `-p` flag is the proper mechanism for non-interactive execution.
- **D:** `--batch` is not a valid Claude Code flag. Non-interactive single-invocation mode is controlled by `-p`.

</details>

---

#### C238

*TS 3.6 · Claude Code in CI/CD*

**Situation:** A CI pipeline script needs to parse Claude Code's response programmatically. The script will extract the list of issues found in a code review and create GitHub issue tickets for each one.

Which Claude Code flag should be added to ensure the response is machine-parseable by the CI script?

- **A)** --output-format json
- **B)** --serialize output
- **C)** --format machine-readable
- **D)** --response-type structured

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

`--output-format json` instructs Claude Code to emit its response as a JSON object rather than plain text. This makes the output directly parseable by downstream CI steps using tools like `jq` or any JSON library.

References:
- https://docs.anthropic.com/en/docs/claude-code/cli-reference
- https://docs.anthropic.com/en/docs/claude-code/github-actions

**Why the others are wrong:**

- **B:** `--serialize` is not a valid Claude Code flag. JSON output is enabled with `--output-format json`.
- **C:** `--format machine-readable` is not a valid Claude Code flag. The correct flag is `--output-format json`.
- **D:** `--response-type` is not a valid Claude Code flag. Structured output is requested via `--output-format json`.

</details>

---

#### C239

*TS 3.6 · Claude Code in CI/CD*

**Situation:** A CI pipeline runs Claude Code twice in the same job: first to review the code, then to generate a test suite. The engineer wants to ensure that context from the first invocation does not influence the second.

What is the default session behavior of Claude Code when invoked via `-p` in a CI pipeline, regarding context isolation?

- **A)** Context is shared within a job but isolated between workflow runs
- **B)** Each `-p` invocation automatically shares a session with previous invocations in the same job
- **C)** Each `-p` invocation starts a fresh, isolated session with no shared state from prior runs
- **D)** Sessions persist across workflow runs until explicitly cleared with `--clear-session`

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

When Claude Code is invoked with `-p` in CI, each invocation is stateless and starts a fresh session. There is no shared conversation history or context between separate `-p` invocations. This isolation is the correct behavior for CI pipelines where each step should be deterministic and independent.

References:
- https://docs.anthropic.com/en/docs/claude-code/cli-reference
- https://docs.anthropic.com/en/docs/claude-code/github-actions

**Why the others are wrong:**

- **A:** There is no job-level session sharing. Each `-p` invocation is fully isolated regardless of whether it runs in the same job or a different workflow run.
- **B:** Separate `-p` invocations do not share session state. Each call is independently stateless, which is exactly what CI pipelines require for reproducibility.
- **D:** Sessions do not persist across workflow runs. Claude Code does not maintain cross-invocation state when called non-interactively via `-p`.

</details>

---

#### C240

*TS 3.6 · Claude Code in CI/CD*

**Situation:** A platform team is designing a CI system where Claude Code performs automated PR reviews across 50 microservice repositories. Each repository has a different coding standard. The team wants a single reusable GitHub Actions workflow that adapts the review prompt per repository. Repository-specific review criteria are stored in a `.claude-review-config.json` file at each repo root.

What is the MOST robust architecture for injecting per-repository review criteria into the Claude Code prompt while enforcing consistent JSON output structure across all repositories?

- **A)** Read `.claude-review-config.json` in the workflow, inject its content into the `-p` prompt string, and use a shared `--json-schema` file to enforce output structure
- **B)** Store a separate GitHub Actions workflow file per repository, each with a hardcoded prompt containing that repository's criteria
- **C)** Use a GitHub Actions matrix strategy with the prompt as a matrix variable, referencing `--json-schema` from a centralized schema registry URL
- **D)** Pass criteria via a `CLAUDE_SYSTEM_PROMPT` environment variable and rely on `--output-format json` without schema enforcement for flexibility

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Reading `.claude-review-config.json` dynamically in the workflow and interpolating it into the `-p` prompt allows a single reusable workflow to adapt per repository without duplication. Pairing this with a shared `--json-schema` file (stored in the centralized workflow repo) enforces consistent output structure. This separates concerns: content customization in config, structure enforcement in schema.

References:
- https://docs.anthropic.com/en/docs/claude-code/cli-reference
- https://docs.anthropic.com/en/docs/claude-code/github-actions

**Why the others are wrong:**

- **B:** Maintaining 50 separate workflow files with hardcoded prompts creates massive duplication, makes updates expensive, and is error-prone. A single reusable workflow with dynamic config injection is the correct pattern.
- **C:** A matrix strategy is appropriate for running the same job against multiple values (e.g., Node versions), not for per-repository config injection. Reading the config file directly in a single job is cleaner and avoids matrix sprawl.
- **D:** `CLAUDE_SYSTEM_PROMPT` is not a documented Claude Code environment variable for injecting system prompts via CLI. Omitting `--json-schema` trades enforcement for flexibility but breaks the downstream parsers that depend on consistent structure.

</details>

---

#### C241

*TS 3.6 · Claude Code in CI/CD*

**Situation:** A team wants Claude Code to perform automated PR code review and output a structured report that always contains exactly three fields: `summary` (string), `issues` (array of strings), and `severity` (one of "low", "medium", "high"). The CI script will fail if unexpected fields are present.

Which combination of Claude Code flags enforces this exact JSON structure in the output?

- **A)** `--output-format json` only, with the schema described in the prompt text
- **B)** `--structured-output` with an inline schema passed via `--schema '{...}'`
- **C)** `--output-format json` together with `--json-schema <schema-file>` to enforce the exact structure
- **D)** `--output-format json` together with `--validate-schema` pointing to a JSON Schema file

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

`--output-format json` requests JSON output and `--json-schema <schema-file>` provides a JSON Schema definition that Claude Code uses to constrain its response to exactly the specified structure. Together they guarantee schema-conformant output that downstream CI steps can reliably parse.

References:
- https://docs.anthropic.com/en/docs/claude-code/cli-reference
- https://docs.anthropic.com/en/docs/claude-code/github-actions

**Why the others are wrong:**

- **A:** Describing the schema only in the prompt text is a soft instruction with no enforcement. Claude Code may still produce extra fields or deviate from the structure. `--json-schema` provides hard structural enforcement.
- **B:** `--structured-output` and `--schema` are not valid Claude Code flags. The correct flags are `--output-format json` and `--json-schema`.
- **D:** `--validate-schema` is not a valid Claude Code flag. Schema enforcement is done with `--json-schema` at invocation time, not as a post-processing validation step.

</details>

---

#### C242

*TS 3.6 · Claude Code in CI/CD*

**Situation:** An engineer configures a CI workflow where two sequential steps both invoke Claude Code with `-p`. The first step asks Claude to analyze the architecture. The second step asks Claude to suggest refactoring based on that analysis. The second step's output is unexpectedly generic and ignores the first step's findings.

Why does the second Claude Code invocation not leverage the first invocation's analysis?

- **A)** The CI environment clears session files between steps; setting `CLAUDE_SESSION_DIR` would fix this
- **B)** Claude Code only shares context within a single job step, not across multiple steps
- **C)** Each `-p` invocation is stateless; the first step's output must be explicitly passed into the second step's prompt
- **D)** The second invocation needs the `--inherit-context` flag to access the prior session

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Each `-p` invocation is completely stateless with no shared session. To chain analysis results, the pipeline must capture the first step's output (e.g., via `$(claude -p "..." --output-format json)`) and explicitly include it in the second step's prompt. This is the standard CI pattern for multi-step AI pipelines.

References:
- https://docs.anthropic.com/en/docs/claude-code/cli-reference
- https://docs.anthropic.com/en/docs/claude-code/github-actions

**Why the others are wrong:**

- **A:** `CLAUDE_SESSION_DIR` is not a valid environment variable for Claude Code session management. Sessions are stateless per invocation by design, not due to a directory being cleared.
- **B:** Claude Code does not maintain any step-level or job-level session state in CI. Each invocation is isolated regardless of where it runs in the workflow.
- **D:** `--inherit-context` is not a valid Claude Code flag. Context from prior invocations is never automatically inherited; it must be explicitly passed in the prompt.

</details>

---

#### C243

*TS 3.6 · Claude Code in CI/CD*

Your CI pipeline runs:

```bash
claude "Review this PR for security vulnerabilities"
```

The job hangs indefinitely in the pipeline. What's wrong?

- **A)** The command needs `CLAUDE_HEADLESS=true` set as an environment variable.
- **B)** The command is missing the `-p` flag. It should be: `claude -p "Review this PR for security vulnerabilities"`.
- **C)** stdin needs to be redirected: `claude "..." < /dev/null`.
- **D)** The command needs the `--batch` flag for CI environments.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

`-p` (or `--print`) is the documented way to run Claude Code non-interactively. `CLAUDE_HEADLESS` (A), stdin redirect (C), and `--batch` (D) are not valid Claude Code features.

</details>

---

#### C244

*TS 3.6 · Claude Code in CI/CD*

Your CI pipeline generates PR review comments using Claude Code. After a developer pushes fixes based on the first review, the second review run duplicates all previous findings plus new ones, flooding the PR with redundant comments. What's the fix?

- **A)** Clear the review history between runs so each review starts fresh.
- **B)** Include the prior review findings in context and instruct Claude to report only new or still-unaddressed issues.
- **C)** Use a separate Claude session for each review run with explicit instructions to only check the new commits.
- **D)** Add `--incremental` flag to only analyze changed files.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Include prior findings in context and ask for only new/unresolved issues.

**Why the others are wrong:**

- **A:** Clearing history loses context.
- **C:** Separate sessions don't have prior review context.
- **D:** `--incremental` doesn't exist.

</details>

---

#### C245

*TS 3.6 · Claude Code in CI/CD*

You want Claude Code to produce machine-parseable structured output for your CI pipeline so review findings can be automatically posted as inline PR comments. What CLI flags should you use?

- **A)** `claude -p "..." --format json`
- **B)** `claude -p "..." --json`
- **C)** `claude -p "..." --structured-output`
- **D)** `claude -p "..." --output-format json --json-schema '{"type":"object",...}'`

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

`--output-format json` and `--json-schema` are the correct CLI flags.

**Why the others are wrong:**

- **A:** This flag combination doesn't exist.
- **B:** This flag combination doesn't exist.
- **C:** This flag combination doesn't exist.

</details>

---

#### C246

*TS 3.6 · Claude Code in CI/CD*

Your CI pipeline generates test cases for new code. It suggests tests that already exist in the test suite, and suggests testing patterns that conflict with your team's testing standards. How do you fix both issues?

- **A)** Limit test generation to only uncovered functions identified by a coverage tool.
- **B)** Post-filter the generated tests to remove duplicates.
- **C)** Provide existing test files in context to avoid duplicate suggestions, and document testing standards/conventions/available fixtures in CLAUDE.md.
- **D)** Run the generated tests and discard any that fail due to conflicts.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Provide existing tests in context to avoid duplicates. Document standards, conventions, and fixtures in CLAUDE.md for quality.

**Why the others are wrong:**

- **A:** Coverage tools don't address convention compliance.
- **B:** Post-filtering is reactive.
- **D:** Running and discarding is wasteful.

</details>

---

#### C247

*TS 3.6 · Claude Code in CI/CD*

Your CI pipeline posts Claude Code review comments on every PR. After a developer pushes fixes in response to the first review, the re-review posts the same findings again alongside new ones. Developers complain about duplicate comment spam. What's the fix?

- **A)** Use PR labels to track which findings have been addressed.
- **B)** Only review the diff between the latest commit and the previous review.
- **C)** Delete all previous review comments before each new review run.
- **D)** Include the prior review findings in context when re-running, and instruct Claude to report only new or still-unaddressed issues.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Include prior findings in context, ask for only new/unresolved issues.

**Why the others are wrong:**

- **A:** PR labels require manual tracking.
- **B:** Diff-only review misses issues from earlier code.
- **C:** Deleting comments loses history.

</details>

---

#### C248

*TS 3.6 · Claude Code in CI/CD*

You need your CI review to output findings as structured JSON that can be programmatically posted as inline PR comments at specific line numbers. What's the correct CLI invocation?

- **A)** `claude -p "Review this PR" --output-format json --json-schema '{"type":"object","properties":{"findings":{"type":"array","items":{"type":"object","properties":{"file":{"type":"string"},"line":{"type":"integer"},"severity":{"type":"string"},"message":{"type":"string"}}}}}}'`
- **B)** `claude -p "Review this PR" --json`
- **C)** `claude -p "Review this PR" | jq '.findings'`
- **D)** `claude "Review this PR" --output-format json`

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

`claude -p "..." --output-format json --json-schema '{...}'` is the correct CLI invocation. Note: `-p` is required for CI (non-interactive).

**Why the others are wrong:**

- **B:** This flag combination is incorrect.
- **C:** This flag combination is incorrect.
- **D:** This flag combination is incorrect.

</details>

---

#### C249

*TS 3.6 · Claude Code in CI/CD*

Your team uses Claude Code to generate unit tests during CI. The generated tests are syntactically correct but test trivial cases (e.g., `expect(true).toBe(true)`) instead of meaningful behavior. What improves quality?

- **A)** Add "Write meaningful tests" to the prompt.
- **B)** Increase the model's temperature for more creative test generation.
- **C)** Document testing standards, what constitutes a valuable test, and available test fixtures in CLAUDE.md. Provide concrete examples of good vs bad tests.
- **D)** Use a separate model to review and filter generated tests.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Document testing standards, valuable test criteria, and available fixtures in CLAUDE.md with concrete good/bad examples.

**Why the others are wrong:**

- **A:** "Write meaningful tests" is too vague.
- **B:** Temperature doesn't improve test quality.
- **D:** A review model adds latency.

</details>

---

#### C250

*TS 3.6 · Claude Code in CI/CD*

Your CI pipeline needs to run Claude Code non-interactively but occasionally the `-p` flag conflicts with CI-specific requirements. Which of the following statements about `-p` is correct?

- **A)** `-p` formats the output for printing to a physical printer.
- **B)** `-p` reads input from a file instead of stdin.
- **C)** `-p` (or `--print`) runs Claude Code in non-interactive mode: it processes the prompt, outputs the result to stdout, and exits without waiting for user input.
- **D)** `-p` enables parallel processing of multiple prompts.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

`-p` / `--print` = non-interactive mode. Processes prompt, outputs to stdout, exits.

**Why the others are wrong:**

- **A:** This description is incorrect.
- **B:** This description is incorrect.
- **D:** This description is incorrect.

</details>

---

#### C251

*TS 3.6 · Claude Code in CI/CD*

You need Claude Code to both generate tests AND review code on each PR. A colleague suggests using a single Claude session for both tasks to "share context." What's the issue?

- **A)** A single session can't run two different prompts.
- **B)** The combined context might exceed the model's token limit.
- **C)** There's no issue — shared context improves both tasks.
- **D)** Using the same session for generation and review introduces self-review bias. The generator's reasoning context makes the reviewer less likely to find issues in its own generated tests. Use separate, independent sessions.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Self-review bias affects both generation and review in the same session. The generator's reasoning context makes the reviewer less critical. Use separate, independent sessions.

**Why the others are wrong:**

- **C:** Context sharing is the problem, not a benefit.

</details>

---

#### C252

*TS 3.6 · Claude Code in CI/CD*

You need your shopping agent to output product recommendations as structured JSON that can be rendered as rich product cards with images, prices, and "Add to Cart" buttons. What's the correct CLI invocation?

- **A)** `claude -p "Recommend products" --output-format json --json-schema '{"type":"object","properties":{"recommendations":{"type":"array","items":{"type":"object","properties":{"product_id":{"type":"string"},"name":{"type":"string"},"price":{"type":"number"},"image_url":{"type":"string"}}}}}}'`
- **B)** `claude -p "Recommend products" --json`
- **C)** `claude -p "Recommend products" | jq '.recommendations'`
- **D)** `claude "Recommend products" --output-format json`

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

`claude -p "..." --output-format json --json-schema '{...}'` is the correct CLI invocation.

**Why the others are wrong:**

- **B:** This flag combination is incorrect.
- **C:** This flag combination is incorrect.
- **D:** This flag combination is incorrect.

</details>

---

#### C253

*TS 3.6 · Claude Code in CI/CD*

You need your agent to output inventory alerts as structured JSON that can be programmatically sent to the warehouse management system. What's the correct CLI invocation?

- **A)** `claude -p "Check inventory" | jq '.alerts'`
- **B)** `claude "Check inventory" --output-format json`
- **C)** `claude -p "Check inventory" --output-format json --json-schema '{"type":"object","properties":{"alerts":{"type":"array","items":{"type":"object","properties":{"sku":{"type":"string"},"current_level":{"type":"number"},"reorder_point":{"type":"number"}}}}}}'`
- **D)** `claude -p "Check inventory" --json`

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

claude -p "..." --output-format json --json-schema '{...}' is the correct CLI invocation.

</details>

---

#### C254

*TS 3.6 · Claude Code in CI/CD*

Your agent generates purchase orders. After an operator approves and modifies a PO, the re-generation posts the same items alongside the modifications. Operators complain about duplicate entries. What's the fix?

- **A)** Use PO labels to track which items have been approved.
- **B)** Only regenerate POs for items that have changed since the last version.
- **C)** Delete all previous PO drafts before each new generation run.
- **D)** Include the prior PO state in context when re-running, and instruct the agent to report only changes or new items.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Include prior PO state in context, ask for only changes or new items.

**Why the others are wrong:**

- **C:** Deleting comments loses history.

</details>

---

<a id="part-c-domain-4"></a>

### Part C · Domain 4 · Prompt Engineering & Structured Output (20%)

#### C255

*TS 4.1 · Explicit criteria & false positives*

**Situation:** A content moderation system uses the prompt: 'Remove content that seems harmful.' After deployment, reviewers notice that Claude removes some borderline satire and ignores some explicit threats.

What is the PRIMARY cause of these inconsistent moderation decisions?

- **A)** Claude cannot perform content moderation without a fine-tuned model.
- **B)** The system prompt is too short and needs to be at least 500 words.
- **C)** Claude's temperature setting is too high.
- **D)** The prompt lacks explicit criteria defining what constitutes 'harmful,' so Claude interprets the term differently depending on context.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

The word 'harmful' is a vague, context-dependent qualifier. Without explicit criteria — such as specific categories of prohibited content, example cases, or decision rules — Claude applies its own judgment, which varies by context and results in inconsistent outcomes. Replacing vague qualifiers with explicit rubrics (e.g., 'remove content that contains direct threats of physical violence or instructions for illegal activity') produces consistent behavior.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/be-clear-and-direct
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/chain-of-thought

**Why the others are wrong:**

- **A:** Claude is capable of content moderation tasks via prompting; a fine-tuned model is not required to achieve consistent results when criteria are explicit.
- **B:** Prompt length is not the issue; a 500-word prompt with equally vague language would produce the same problem. Precision matters more than length.
- **C:** Temperature affects creativity and randomness in generation, not the interpretation of evaluation criteria. Setting temperature to 0 would reduce variance but would not resolve the ambiguity of 'harmful.'.

</details>

---

#### C256

*TS 4.1 · Explicit criteria & false positives*

**Situation:** You are designing a prompt to classify customer feedback as 'Positive,' 'Neutral,' or 'Negative.' You want consistent results.

Which classification instruction is most likely to produce consistent results across many different feedback samples?

- **A)** Classify the feedback based on your overall sense of the customer's mood.
- **B)** Classify as Positive if overall sentiment is good, Negative if bad, otherwise Neutral.
- **C)** Classify as Positive if the feedback contains praise or thanks with no complaints; Negative if the feedback contains at least one explicit complaint or request for a refund; otherwise Neutral.
- **D)** Use your best judgment to decide whether the feedback is positive, neutral, or negative.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Option C provides explicit, observable criteria for each class: the presence of praise/thanks without complaints for Positive, and the presence of a complaint or refund request for Negative. These criteria can be applied mechanically, reducing reliance on subjective interpretation. Options A, B, and D rely on vague qualifiers ('overall sense,' 'good,' 'best judgment') that introduce inconsistency.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/be-clear-and-direct
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-examples

**Why the others are wrong:**

- **A:** 'Overall sense of mood' is entirely subjective and will vary across inputs with similar wording, leading to high variance in borderline cases.
- **B:** 'Good' and 'bad' are qualitative descriptors that do not define observable signals, making this criterion nearly as vague as saying 'classify correctly.'.
- **D:** 'Best judgment' defers entirely to Claude's internal heuristics, which may shift with context, token position, or prior conversation, making results unreproducible.

</details>

---

#### C257

*TS 4.1 · Explicit criteria & false positives*

**Situation:** A prompt engineer defines the following rubric for grading a short essay on a scale of 1–5: '5 = excellent, 4 = good, 3 = average, 2 = poor, 1 = very poor.'

What is the main weakness of this rubric for automated grading?

- **A)** Five-point scales are too granular; a binary pass/fail would be more reliable.
- **B)** The labels (excellent, good, average, poor, very poor) are qualitative and do not specify observable criteria, making grading inconsistent.
- **C)** Claude cannot assign numeric scores and will always return text labels instead.
- **D)** The scale should use letters (A–F) instead of numbers.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Words like 'excellent' and 'poor' are relative and undefined. Two essays that different evaluators would grade differently may both receive a 4 depending on Claude's contextual interpretation. A strong rubric ties each score to observable, measurable criteria — for example, '5 = thesis is clearly stated in the first paragraph, all three required arguments are supported with evidence, no grammatical errors.'

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/be-clear-and-direct

**Why the others are wrong:**

- **A:** Scale granularity is a design choice based on requirements; a five-point scale is not inherently unreliable — the problem is the absence of criteria, not the number of levels.
- **C:** Claude can output numeric scores when instructed to do so; this is not a capability limitation.
- **D:** Letter vs. number scale is a cosmetic difference that does not resolve the underlying problem of undefined criteria.

</details>

---

#### C258

*TS 4.1 · Explicit criteria & false positives*

**Situation:** An architect designs a multi-step pipeline: (Step 1) Claude extracts claims from a news article. (Step 2) A second Claude call classifies each claim as 'Verified,' 'Unverified,' or 'Disputed.' (Step 3) A third Claude call generates a bias-flagging report. The team finds that Step 2 has high false positive rates for 'Disputed,' and Step 3 generates inconsistently structured reports. They have a budget for one prompt redesign sprint.

Which redesign strategy delivers the highest ROI by addressing the root cause of both problems?

- **A)** Add a human review gate between Steps 2 and 3 to catch Step 2 false positives before they pollute Step 3.
- **B)** Redesign Step 2's prompt with explicit classification criteria (e.g., 'Disputed = at least two authoritative sources make contradictory claims with citations') and add a structured output schema to Step 3.
- **C)** Redesign Step 3's prompt to include explicit report structure requirements; Step 2 false positives are acceptable in a fact-checking pipeline.
- **D)** Merge Steps 2 and 3 into a single prompt to reduce error propagation between steps.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Step 2's false positives in 'Disputed' propagate into Step 3's inputs, causing both the incorrect classification and the structural inconsistency in reports (because Step 3 must handle more 'Disputed' items than expected, altering its output shape). By fixing Step 2's criteria (explicit definition of 'Disputed' grounded in source evidence), false positives decrease and Step 3 receives cleaner, more consistent inputs. Adding an explicit schema to Step 3 simultaneously resolves report structure inconsistency. This two-part fix addresses both root causes within one sprint. Merging steps increases prompt complexity without resolving the definitional gap. Human review adds latency without fixing the upstream criteria.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/be-clear-and-direct
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/chain-of-thought
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-examples

**Why the others are wrong:**

- **A:** Human review is a control measure, not a design solution; it adds cost and latency but leaves the false positive root cause unaddressed, requiring indefinite human intervention.
- **C:** Fixing only Step 3's structure while accepting Step 2 false positives leaves the pipeline producing well-formatted reports with incorrect content — which may be worse than inconsistently structured correct reports.
- **D:** Merging Steps 2 and 3 into one prompt increases context length and task complexity; it may reduce some error propagation but does not fix the missing explicit criteria for 'Disputed,' which is the root cause.

</details>

---

#### C259

*TS 4.1 · Explicit criteria & false positives*

**Situation:** A startup uses Claude to automatically generate acceptance criteria for user stories. The initial prompt: 'Write acceptance criteria for this user story.' After 200 stories, QA finds that 35% of generated criteria are either too vague to test ('the system should be fast'), duplicate existing criteria, or miss error-path scenarios entirely.

A senior architect proposes a rubric-based prompt that instructs Claude to: (i) generate exactly 4–6 criteria per story, (ii) follow the 'Given/When/Then' format, (iii) include at least one error-path scenario, and (iv) flag any criterion that cannot be expressed as a pass/fail test with confidence < 0.8. Which of the following BEST describes why criterion (iv) is architecturally valuable compared to simply saying 'make criteria testable'?

- **A)** Criterion (iv) is redundant because following Given/When/Then format automatically guarantees testability.
- **B)** Criterion (iv) is harmful because it teaches Claude to express self-doubt, which will reduce confidence in all criteria including clearly testable ones.
- **C)** Criterion (iv) instructs Claude to refuse to generate criteria it is unsure about, reducing overall output volume.
- **D)** Criterion (iv) creates a machine-readable signal (the flag) that downstream tooling, CI pipelines, or human reviewers can act on without re-reading the criterion, while 'make criteria testable' produces no actionable output artifact.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

The key architectural insight is that criterion (iv) externalizes Claude's uncertainty as a structured, actionable signal (a flag) rather than leaving it as an implicit quality of the text. A criterion like 'the system should respond quickly' may pass syntactic Given/When/Then formatting while still being untestable. The flag creates an explicit artifact that CI tooling can count, QA teams can filter on, and dashboards can track — enabling automated quality gates without re-parsing the criterion text. 'Make criteria testable' is an internal instruction that changes Claude's generation behavior but produces no machine-readable output. The flag transforms a qualitative intent into a quantitative, actionable interface.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/be-clear-and-direct
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/chain-of-thought

**Why the others are wrong:**

- **A:** Given/When/Then format ensures structural consistency but does not guarantee testability; a vague 'Given the system is fast / When a request is made / Then it should respond quickly' satisfies the format while remaining unmeasurable.
- **B:** Expressing calibrated uncertainty about specific criteria does not infect confidence in clearly testable ones; confidence flags are per-criterion and do not propagate across the full output.
- **C:** The instruction is to flag criteria with confidence < 0.8, not to refuse generation; Claude still produces the criterion but attaches a machine-readable signal, preserving output completeness while exposing quality issues.

</details>

---

#### C260

*TS 4.1 · Explicit criteria & false positives*

**Situation:** A compliance team evaluates whether vendor contracts meet company policy. They have three prompt variants under consideration: (Variant A) 'Assess whether this contract complies with our vendor policy.' (Variant B) 'Assess compliance. Flag any clause that violates policy. Output: compliant=true/false, violations=list of clause numbers.' (Variant C) 'For each of the following 8 policy requirements (listed below with pass/fail criteria for each), determine whether the contract satisfies the requirement. Output a table: Requirement | Status (Pass/Fail) | Confidence (0–1) | Evidence (verbatim quote). Flag the overall contract as non-compliant if any requirement scores Fail or has confidence < 0.75.'

A new compliance auditor asks why Variant C is used instead of Variant A or B despite being much longer. What is the MOST architecturally complete explanation?

- **A)** Variant C is longer, so it gives Claude more context, which always improves accuracy.
- **B)** Variant C decomposes a complex judgment into a structured checklist with per-requirement criteria, explicit confidence thresholds, evidence anchoring, and a deterministic aggregation rule — together these ensure reproducibility, auditability, and calibrated false positive management that Variants A and B cannot provide.
- **C)** Variant C avoids false positives entirely by requiring verbatim evidence quotes.
- **D)** Variants A and B would cause Claude to hallucinate policy requirements, whereas Variant C lists them explicitly.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Variant C's superiority rests on four interlocking design decisions: (1) Decomposition — each of the 8 requirements is assessed independently, preventing one complex clause from dominating the overall assessment; (2) Explicit per-requirement criteria — pass/fail is defined for each requirement, not inferred from vague policy language; (3) Evidence anchoring — verbatim quotes tether each finding to specific contract text, enabling auditor verification; (4) Confidence thresholds with a deterministic aggregation rule — the < 0.75 flag transforms calibrated uncertainty into a machine-readable signal and defines exactly when the overall contract is flagged. Variants A and B lack decomposition, evidence, and confidence management. Variant C is not merely longer — its length encodes a complete, auditable methodology.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/be-clear-and-direct
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/chain-of-thought
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-examples

**Why the others are wrong:**

- **A:** Length alone does not improve accuracy; more context is valuable only when it encodes relevant constraints. A longer prompt with vague criteria would still produce inconsistent results.
- **C:** Evidence quotes reduce hallucination risk and improve auditability but do not eliminate false positives; confidence thresholds and explicit per-requirement criteria are equally important components of false positive management.
- **D:** Claude does not hallucinate policy requirements when given a vague prompt — it applies its own interpretation of what constitutes a violation, which is a consistency problem, not a hallucination problem per se. Variant C solves the consistency problem by explicitly defining each requirement.

</details>

---

#### C261

*TS 4.1 · Explicit criteria & false positives*

**Situation:** A financial services company uses Claude to extract named entities from regulatory filings. The current prompt says: 'Extract all important entities from the filing.' Reviewers find that Claude sometimes extracts company names but misses jurisdiction codes and monetary amounts, and vice versa.

Which prompt revision will MOST reliably improve extraction completeness and consistency?

- **A)** Add 'Be thorough' to the existing instruction.
- **B)** Ask Claude to read the filing twice before extracting.
- **C)** Switch to a larger model — the extraction misses are a model capacity issue, not a prompting issue.
- **D)** Replace the vague instruction with an explicit enumeration: 'Extract the following entity types: (1) company names, (2) jurisdiction codes (e.g., US-NY, EU-DE), (3) monetary amounts with currency symbols.'

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Explicit enumeration of entity types removes ambiguity about what 'important' means. By listing each category with examples, the prompt defines the extraction task precisely, so Claude applies the same criteria to every filing. 'Be thorough' adds no new information about which entities matter. Re-reading increases token cost without resolving the definitional gap. Model size is not the cause — the prompt is underspecified.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/be-clear-and-direct
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-examples

**Why the others are wrong:**

- **A:** 'Be thorough' is a qualitative intensifier that does not define what should be extracted; it may improve recall marginally but will not fix systematic category omissions.
- **B:** Re-reading the filing may catch some missed tokens but does not teach Claude which entity types to look for; the same categories will be inconsistently applied.
- **C:** This is a prompt design failure, not a model capacity limitation. A smaller model with explicit criteria will outperform a larger model with vague criteria for well-defined extraction tasks.

</details>

---

#### C262

*TS 4.1 · Explicit criteria & false positives*

**Situation:** A developer wants Claude to generate product descriptions. The prompt says: 'Write a good product description for the following item.' After reviewing 50 outputs, marketing finds wide variation in length (50–400 words), tone (casual to formal), and feature emphasis.

Which set of explicit criteria would BEST address all three sources of variation?

- **A)** Add: 'Write a very good product description.'
- **B)** Add: 'Use a professional tone.'
- **C)** Add: 'Write exactly 120 words. Use a friendly, conversational tone. Lead with the primary benefit, then list three key features as bullet points, and close with a call to action.'
- **D)** Add: 'Write a description that customers will enjoy reading.'

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Option C addresses all three observed variations with explicit, measurable constraints: a word count (120) eliminates length variation, a defined tone descriptor ('friendly, conversational') reduces tone drift, and a structural template (benefit → bullets → CTA) standardizes feature emphasis. Options A and D add subjective qualifiers. Option B addresses only tone, leaving length and structure uncontrolled.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/be-clear-and-direct
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/chain-of-thought

**Why the others are wrong:**

- **A:** Intensifying the adjective ('very good') provides no structural guidance and will not reduce variance in length, tone, or feature selection.
- **B:** Tone guidance alone does not constrain length or feature emphasis; the other two sources of variation remain unaddressed.
- **D:** 'Customers will enjoy reading' is a vague outcome statement that gives Claude no actionable criteria for length, tone, or structure.

</details>

---

#### C263

*TS 4.1 · Explicit criteria & false positives*

**Situation:** An analyst team uses a prompt: 'Summarize the key risks in this earnings call transcript.' They find the summaries are helpful but hard to compare across calls because risk descriptions vary in specificity, category, and order.

Which prompt design change would BEST enable structured, comparable risk summaries?

- **A)** Use a lower temperature to make responses more consistent.
- **B)** Ask Claude to sort risks alphabetically.
- **C)** Ask Claude to 'be more structured.'
- **D)** Specify a rubric: 'Identify risks in exactly these four categories in order: (1) Market Risk, (2) Operational Risk, (3) Regulatory Risk, (4) Financial Risk. For each category, state whether a risk was mentioned (Yes/No) and provide a one-sentence summary if Yes.'

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

A fixed schema with defined categories, a binary presence indicator, and a length constraint (one sentence) ensures every summary has the same structure and level of specificity. This makes outputs directly comparable across calls. 'Be more structured' is vague. Temperature affects randomness, not schema adherence. Alphabetical sorting addresses order but not category coverage or specificity.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/be-clear-and-direct
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-examples

**Why the others are wrong:**

- **A:** Reducing temperature to 0 minimizes token-level randomness but does not impose a schema; two calls with different risk profiles will still produce differently shaped summaries.
- **B:** Alphabetical sorting ensures a consistent order within a response but does not standardize categories, presence indicators, or sentence-level specificity across calls.
- **C:** 'Be more structured' is a qualitative instruction that does not define a structure; Claude will invent a structure that varies across calls.

</details>

---

#### C264

*TS 4.1 · Explicit criteria & false positives*

**Situation:** A developer asks Claude to flag potential bugs in code reviews. The original prompt says: 'Flag any code that looks buggy.' After one month, engineers complain that Claude flags obvious style issues as bugs and misses null-pointer risks.

Which explicit-criteria revision would BEST reduce false positives and false negatives for this task?

- **A)** Change the prompt to: 'Flag code that looks very buggy.'
- **B)** Ask Claude to flag bugs and add 'ignore style issues' at the end of the prompt.
- **C)** Replace Claude with a static analysis linter.
- **D)** Change the prompt to: 'Flag lines that contain: (a) unhandled null/undefined dereferences, (b) off-by-one errors in loop bounds, or (c) uncaught exceptions from external calls. Do NOT flag style issues, naming conventions, or formatting.'

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Option D does two things simultaneously: it defines a positive list of bug categories (reducing false negatives by naming specific risk patterns) and an explicit exclusion list (reducing false positives by prohibiting style flags). This is the classic rubric pattern — tell the model exactly what to look for and exactly what to ignore. Option B is an improvement but does not define which bug categories to include. Option A adds no new information.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/be-clear-and-direct
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/chain-of-thought

**Why the others are wrong:**

- **A:** Adding 'very' intensifies the qualifier but does not define what 'buggy' means, so false positives and false negatives remain driven by Claude's implicit interpretation.
- **B:** Excluding style is an improvement, but without a positive list of bug types, Claude still has discretion in what it flags, leaving false negatives unaddressed.
- **C:** Replacing Claude with a linter may be appropriate in some contexts but is not a prompt design solution; the question asks how to improve the prompt.

</details>

---

#### C265

*TS 4.1 · Explicit criteria & false positives*

Your code review prompts produce detailed instructions: "Review for bugs, security issues, code quality, performance, accessibility, documentation completeness, test coverage, naming conventions, and style consistency." The output is inconsistent — detailed for some categories, superficial for others. What's the fix?

- **A)** Add "be thorough in every category" to the prompt.
- **B)** Define explicit criteria specifying which issues to report (bugs, security) versus skip (minor style), with concrete code examples for each severity level.
- **C)** Increase the model's `max_tokens` to allow more complete output.
- **D)** Run the review multiple times and combine the results.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Explicit criteria with concrete examples for each severity level achieve consistency. More tokens (C) and repeated runs (D) don't address the root cause.

**Why the others are wrong:**

- **A:** Vague instructions don't improve precision.

</details>

---

#### C266

*TS 4.1 · Explicit criteria & false positives*

Your CI review prompt says: "Review this code and report any issues you find. Be thorough." The output varies wildly between runs — sometimes a detailed 20-finding report, sometimes 3 superficial comments. What's the root cause?

- **A)** The model's `max_tokens` varies between runs, limiting output length.
- **B)** The model needs a higher temperature for consistent creative output.
- **C)** Add `--deterministic` flag to the Claude Code CLI for consistent output.
- **D)** The instructions are too vague. "Be thorough" doesn't define what to look for. Replace with explicit criteria specifying which issue types to report (bugs, security) versus skip (style, naming).

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Vague instructions ("be thorough") produce inconsistent output. Explicit criteria defining what to report vs skip is the fix.

**Why the others are wrong:**

- **A:** `max_tokens` doesn't vary.
- **B:** Temperature isn't the issue.
- **C:** `--deterministic` doesn't exist.

</details>

---

#### C267

*TS 4.1 · Explicit criteria & false positives*

Your automated code review flags "unused import" in 70% of Python files, but developers dismiss most of these because the imports are used in type annotations behind `if TYPE_CHECKING:` blocks. This has eroded trust in all review findings. What's the fix?

- **A)** Lower the confidence threshold to only report high-confidence findings.
- **B)** Temporarily disable the "unused import" category to restore developer trust while refining the prompt to handle `TYPE_CHECKING` patterns correctly.
- **C)** Add "be more conservative" to the review prompt.
- **D)** Require developers to add `# noqa` comments to all type-checking imports.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

High false positives in one category damage trust in ALL categories. Disable the noisy category, fix its prompt separately, re-enable when accuracy is acceptable. Confidence thresholds (A) and "be conservative" (C) are too vague.

**Why the others are wrong:**

- **D:** `# noqa` burdens developers.

</details>

---

#### C268

*TS 4.1 · Explicit criteria & false positives*

You want to define explicit severity levels for your CI review. Currently, the agent inconsistently classifies findings as "high" or "low." How do you achieve consistent classification?

- **A)** Add `severity_threshold: high` to the output schema.
- **B)** Add "Only report high-severity issues" to the prompt.
- **C)** Use a separate classifier model to re-classify the agent's findings.
- **D)** Define explicit severity criteria with **concrete code examples** for each level: Critical (SQL injection, RCE), High (null dereference, data loss), Medium (missing error handling), Low (unused variable).

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Explicit severity criteria with concrete code examples for each level.

**Why the others are wrong:**

- **A:** Schema thresholds don't improve classification.
- **B:** "Only report high-severity" is vague.
- **C:** A separate classifier adds complexity without fixing the root cause.

</details>

---

#### C269

*TS 4.1 · Explicit criteria & false positives*

Your CI review prompt includes: "When unsure about a finding, report it with a note about your uncertainty." This results in high false positive rates as the agent reports many uncertain findings that are incorrect. What's the better approach?

- **A)** Add "Only report findings you're at least 90% confident about."
- **B)** Replace uncertainty-based reporting with explicit categorical criteria: define exactly what constitutes a reportable issue (bugs, security, data loss) versus what to skip (style, naming, uncertain patterns).
- **C)** Add a confidence score field and filter findings below 0.8 in post-processing.
- **D)** Have the agent run a second pass to verify its uncertain findings.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Explicit categorical criteria over confidence-based filtering. Post-processing filters (C) and second passes (D) add complexity without fixing the root cause.

**Why the others are wrong:**

- **A:** "90% confident" is uncalibrated and vague.

</details>

---

#### C270

*TS 4.1 · Explicit criteria & false positives*

Your recommendation prompt says: "Help customers find products they'll love. Be helpful and thorough." The output varies wildly — sometimes detailed comparisons, sometimes single product suggestions. What's the root cause?

- **A)** The model needs a higher temperature for consistent creative output.
- **B)** The model's `max_tokens` varies between runs, limiting output length.
- **C)** Add `--deterministic` flag to the Claude Code CLI for consistent output.
- **D)** The instructions are too vague. Replace with explicit criteria: what information to gather (budget, use case, preferences), what comparisons to provide, what to recommend vs. skip.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Vague instructions produce inconsistent output. Explicit criteria defining what to gather and how to recommend is the fix.

**Why the others are wrong:**

- **A:** Temperature isn't the issue.
- **B:** `max_tokens` doesn't vary.
- **C:** `--deterministic` doesn't exist.

</details>

---

#### C271

*TS 4.1 · Explicit criteria & false positives*

Your contract review prompt says: "Review this contract and flag any issues. Be thorough." The output varies wildly — sometimes 30 findings, sometimes 5. What's the root cause?

- **A)** The instructions are too vague. Replace with explicit criteria specifying which issue types to report (liability caps, indemnity scope, termination rights) versus skip (minor formatting).
- **B)** Add `--deterministic` flag to the Claude Code CLI for consistent output.
- **C)** The model needs a higher temperature for consistent creative output.
- **D)** The model's `max_tokens` varies between runs, limiting output length.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Vague instructions produce inconsistent output. Explicit criteria defining what to report vs skip is the fix.

</details>

---

#### C272

*TS 4.1 · Explicit criteria & false positives*

Your demand prediction prompt says: "Predict which products will run out of stock. Be helpful." The output varies wildly between runs. What's the fix?

- **A)** The model's `max_tokens` varies between runs, limiting output length.
- **B)** Add `--deterministic` flag to the Claude Code CLI for consistent output.
- **C)** The model needs a higher temperature for consistent creative output.
- **D)** The instructions are too vague. Define explicit criteria: what data to consider (sales velocity, seasonality, lead times), what confidence threshold to use, what to report vs. skip.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Vague instructions produce inconsistent output. Explicit criteria defining what data to consider is the fix.

</details>

---

#### C273

*TS 4.2 · Few-shot prompting*

**Situation:** A senior prompt engineer is building a code review assistant. They follow a test-driven iteration workflow: write 40 labeled code samples (20 'needs review', 20 'approved'), run the assistant, and measure precision and recall. After 8 iterations, precision is 0.95 but recall is 0.60 — the assistant approves many samples that should be flagged.

What does this pattern indicate about the prompt, and what is the MOST targeted refinement action?

- **A)** High precision with low recall indicates the assistant is over-cautious. The prompt should be made more restrictive.
- **B)** The engineer should add more 'approved' training examples to balance the dataset before iterating further.
- **C)** The precision/recall tradeoff is a model limitation and cannot be addressed through prompt engineering.
- **D)** High precision with low recall indicates the assistant is under-cautious: it approves too many samples. The prompt should be refined by adding examples of the false negatives (samples incorrectly approved) with explicit annotation of why they need review.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

High precision and low recall means the assistant is correct when it flags something (few false positives) but misses many samples that should be flagged (many false negatives). This is an under-flagging bias. The targeted remedy is to show Claude the specific false negatives from the test suite — the approved samples that were actually problematic — with explanatory annotations showing why each should have been flagged. This directly addresses the gap in Claude's recognition of the failure patterns it is missing.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/evaluate-and-improve/define-success
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-examples

**Why the others are wrong:**

- **A:** High precision/low recall is a sign of under-flagging, not over-caution. An over-cautious assistant would flag everything (high recall, low precision). Making the prompt more restrictive would increase false positives and reduce the already-high precision.
- **B:** Adding more 'approved' examples to the test dataset changes the evaluation balance but does not change the prompt. The underperformance is in the prompt instructions and examples, not in test set composition.
- **C:** Precision/recall tradeoffs are heavily influenced by the examples and thresholds specified in the prompt. Adding targeted negative examples is a well-established technique for improving recall without sacrificing precision.

</details>

---

#### C274

*TS 4.2 · Few-shot prompting*

**Situation:** A legal team wants Claude to assess whether contract clauses are 'acceptable,' 'requires review,' or 'unacceptable.' The first version of the prompt provides only these three labels with no definitions. After testing, they find borderline clauses land in different categories across runs.

What technique would MOST reduce inter-run inconsistency for borderline clauses?

- **A)** Provide a few-shot rubric: define each label with explicit criteria and include two to three example clauses per category showing how the criteria are applied.
- **B)** Use a longer system prompt with more background information about contract law.
- **C)** Instruct Claude to always default to 'requires review' when uncertain.
- **D)** Run the same clause through Claude five times and take the majority vote.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

A few-shot rubric combines two powerful techniques: explicit criteria (telling Claude what each label means in observable terms) and worked examples (anchoring Claude's interpretation with ground-truth classifications). Together, they constrain the decision boundary around borderline cases. Majority voting reduces variance but does not improve accuracy and multiplies cost. Defaulting to 'requires review' inflates false positives. More legal background does not define the decision rule.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-examples
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/be-clear-and-direct

**Why the others are wrong:**

- **B:** Background information about contract law increases Claude's domain knowledge but does not define which observable clause features map to each label, leaving the decision rule ambiguous.
- **C:** Defaulting borderline cases to 'requires review' reduces false negatives but does not improve accuracy — acceptable clauses will be unnecessarily escalated, increasing legal team workload.
- **D:** Majority voting is a statistical approach to variance reduction; it does not fix the underlying ambiguity and costs five times as much per assessment.

</details>

---

#### C275

*TS 4.2 · Few-shot prompting*

**Situation:** A developer is building a customer-support bot and notices that Claude's responses vary wildly in tone and length when handling refund requests — sometimes terse, sometimes multi-paragraph. They have not yet used any examples in their prompt.

What is the PRIMARY purpose of adding few-shot examples to this prompt?

- **A)** To prevent Claude from generating any response that was not already shown in the examples.
- **B)** To demonstrate the desired tone, length, and format through concrete input-output pairs, making Claude's outputs more consistent.
- **C)** To replace the system prompt entirely, since examples convey all necessary instructions.
- **D)** To increase Claude's token budget so it can produce longer responses.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Few-shot examples are input-output demonstrations that calibrate Claude's behavior by showing — rather than just telling — the desired format, tone, and length. They act as a formatting contract that anchors Claude's style across varied inputs, directly addressing the consistency problem described.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-examples
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview

**Why the others are wrong:**

- **A:** Claude is not restricted to only the patterns seen in examples — it generalizes from them. Examples guide style and format, not constrain the output space to exact replicas.
- **C:** Examples complement system prompts but should not replace them; system prompts carry instructions that would be awkward or impossible to convey purely through examples.
- **D:** Few-shot examples do not increase Claude's token budget; they actually consume tokens from the context window.

</details>

---

#### C276

*TS 4.2 · Few-shot prompting*

**Situation:** An engineer wants to use few-shot prompting to teach Claude to classify customer emails as 'Billing', 'Technical', or 'General'. She has 200 labeled emails available.

Which selection strategy will produce the MOST effective few-shot set?

- **A)** Use only emails from the most common class so Claude learns the dominant pattern first.
- **B)** Pick the 200 emails at random to maximize diversity.
- **C)** Select representative examples from each class plus boundary cases where the correct label is non-obvious.
- **D)** Pick the shortest emails to minimize token usage regardless of content.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Effective few-shot sets combine representative examples (one or more clear instances of each class) with boundary cases (emails that are ambiguous or sit near a decision boundary). This teaches Claude both the prototypical pattern of each class and how to handle edge cases, yielding the best generalization.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-examples
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview

**Why the others are wrong:**

- **A:** Biasing examples toward the most common class will cause Claude to over-predict that class and under-represent rare but important categories.
- **B:** Using all 200 examples would consume most of the context window and still include many near-duplicates; quality and diversity matter more than sheer quantity.
- **D:** Selecting by length ignores coverage across classes and edge cases; short emails may all be trivially easy cases that do not teach disambiguation.

</details>

---

#### C277

*TS 4.2 · Few-shot prompting*

**Situation:** A developer needs Claude to output structured JSON for extracted invoice fields. Without examples, Claude sometimes wraps the JSON in markdown code fences, sometimes includes extra commentary, and sometimes omits optional fields.

How do few-shot examples MOST directly solve this problem?

- **A)** They force Claude to download the correct JSON schema from the internet.
- **B)** They increase Claude's internal confidence score, reducing hallucination.
- **C)** They add runtime validation that rejects non-conforming responses automatically.
- **D)** They demonstrate the exact output format — including field presence, quoting style, and absence of prose — so Claude mirrors the demonstrated structure.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Few-shot examples act as format demonstrations. When every example shows bare JSON with specific field names and no surrounding text, Claude learns to match that exact structure. This is far more reliable than prose instructions alone because Claude can directly observe the desired output shape.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-examples
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/overview

**Why the others are wrong:**

- **A:** Claude does not fetch external schemas at inference time; examples are self-contained within the prompt.
- **B:** Few-shot examples influence output style, not an internal confidence metric; there is no such score exposed to the prompt.
- **C:** Examples in the prompt are input signals, not runtime validators; output validation requires external code, not in-prompt demonstrations.

</details>

---

#### C278

*TS 4.2 · Few-shot prompting*

**Situation:** A team is debating whether to add ten few-shot examples to a simple prompt that asks Claude to translate a single word from English to Spanish. The task has no format ambiguity and the zero-shot performance is already near-perfect.

What is the BEST decision regarding adding ten few-shot examples here?

- **A)** Add examples only if the translations will be reviewed by a human, not in automated pipelines.
- **B)** Add ten examples because more examples always improve reliability.
- **C)** Always add at least five examples to any prompt as a best practice regardless of performance.
- **D)** Skip the examples; for a simple, unambiguous task where zero-shot already performs well, examples waste tokens without improving output.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Few-shot examples cost tokens from the context window and add latency. For a simple, well-defined task where zero-shot performance is already near-perfect, adding examples provides negligible benefit while consuming context and increasing cost. Examples are most valuable when the task has format ambiguity, stylistic requirements, or edge cases that are hard to describe in prose.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-examples
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview

**Why the others are wrong:**

- **A:** Whether responses are reviewed by a human is unrelated to whether few-shot examples help; the relevant factor is task complexity and format ambiguity.
- **B:** More examples do not monotonically improve reliability — beyond a certain point they consume context window space that could be used for the actual user input, and can introduce noise.
- **C:** There is no universal rule requiring a minimum number of examples; the decision should be driven by task complexity and observed performance, not by convention.

</details>

---

#### C279

*TS 4.2 · Few-shot prompting*

**Situation:** A team builds a multi-step reasoning prompt for legal contract analysis. They add five few-shot examples that each show the correct final answer but omit the intermediate reasoning steps. During evaluation, Claude produces correct answers on simple contracts but fails on complex multi-clause contracts with conflicting obligations.

What is the MOST precise diagnosis of the failure mode, and what is the correct fix?

- **A)** Legal contract analysis requires tool calls; few-shot text examples cannot teach reasoning over structured documents.
- **B)** The examples are too long and are causing context window overflow; they should be shortened to single-sentence answers.
- **C)** The examples demonstrate conclusions without demonstrating the reasoning process; for complex tasks, few-shot examples should show complete chain-of-thought — including intermediate steps — so Claude learns to apply structured reasoning rather than pattern-matching on surface features.
- **D)** Five examples is the wrong number; exactly seven examples are needed for multi-step legal reasoning tasks.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

When few-shot examples show only final answers, Claude learns to predict the output distribution but does not learn the intermediate reasoning structure needed for complex inputs. For multi-step or compositional reasoning tasks, examples must include chain-of-thought steps: 'Given clause 3.1 states X and clause 7.2 states Y, there is a conflict... therefore the governing rule is Z.' This teaches Claude the analytical process, not just the answer shape, enabling it to generalize to complex cases that require actual reasoning rather than shallow pattern matching.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-examples
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/chain-of-thought

**Why the others are wrong:**

- **A:** Few-shot text examples are entirely capable of teaching reasoning patterns over documents; tool calls are appropriate for retrieval or external actions, not for demonstrating analytical reasoning.
- **B:** Shortening examples to single-sentence answers would exacerbate the problem by removing any remaining reasoning signal; longer, reasoning-complete examples are what is needed.
- **D:** There is no magic number of examples for any task type; the issue is the quality and structure of the examples (missing reasoning steps), not the count.

</details>

---

#### C280

*TS 4.2 · Few-shot prompting*

**Situation:** An engineer is using few-shot prompting to build a content moderation system. She crafts eight examples: six showing acceptable content with the label 'ALLOW' and two showing clearly harmful content with the label 'BLOCK'. During A/B testing, the system has high recall for obvious violations but a 40% false-negative rate on subtle policy violations — content that is technically borderline but should be blocked per company policy.

What combination of changes to the few-shot set would MOST effectively reduce the false-negative rate on subtle violations?

- **A)** Replace the two 'BLOCK' examples with ten 'ALLOW' examples to teach the model what is acceptable more thoroughly.
- **B)** Add more 'BLOCK' examples that specifically cover subtle and boundary-case violations, include negative examples with brief rationale explaining why borderline content fails policy, and rebalance the class ratio so 'BLOCK' examples represent a higher fraction of the set.
- **C)** Keep the current examples and instead lower the API temperature to 0.0 to make the model more conservative.
- **D)** Switch from few-shot prompting to zero-shot prompting with a detailed policy description, since few-shot examples cannot teach nuanced distinctions.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

The false-negative rate on subtle violations is caused by three compounding factors: (1) class imbalance (six 'ALLOW' vs. two 'BLOCK' examples creates a prior toward ALLOW), (2) absence of boundary-case 'BLOCK' examples (only obvious violations were shown), and (3) lack of rationale that would teach Claude which policy dimension is being violated. The fix requires adding 'BLOCK' examples at the decision boundary, providing rationale in each example so Claude learns the policy logic (not just the label), and rebalancing toward more 'BLOCK' representation relative to the current 3:1 imbalance.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-examples
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview
- https://docs.anthropic.com/en/docs/about-claude/models/overview

**Why the others are wrong:**

- **A:** Adding more 'ALLOW' examples would increase the prior toward ALLOW further, directly worsening the false-negative rate on violations rather than improving it.
- **C:** Temperature affects output randomness/diversity; setting it to 0.0 does not make the model more conservative about policy. A deterministic model that has learned to predict 'ALLOW' for subtle cases will do so even more consistently at temperature 0.
- **D:** Few-shot examples are well-suited to teaching nuanced distinctions — in fact, they are often superior to purely descriptive policies for borderline cases. The issue is example quality and class balance, not a fundamental limitation of the technique.

</details>

---

#### C281

*TS 4.2 · Few-shot prompting*

**Situation:** A senior architect is evaluating two prompt designs for a structured data extraction task. Design A uses a detailed 800-word system prompt describing every extraction rule in prose, with zero examples. Design B uses a concise 200-word system prompt and six carefully chosen few-shot examples (three typical cases, two edge cases, one negative example showing what incorrect extraction looks like). Both designs are evaluated on a held-out test set of 500 documents including a mix of typical and atypical formats.

On which category of test inputs is Design B MOST likely to outperform Design A, and what architectural principle does this illustrate?

- **A)** Design B outperforms on typical cases because six examples are always better than prose rules; the principle is that examples replace instructions.
- **B)** Design B outperforms on edge cases and atypical formats because examples directly demonstrate how to apply rules to hard cases, while prose rules require the model to bridge the gap from abstract description to concrete novel input; this illustrates the principle that showing beats telling for ambiguous situations.
- **C)** Design A outperforms on all inputs because a more detailed system prompt always dominates few-shot examples.
- **D)** Both designs perform identically because the same rules are encoded in both prompts, just in different representations.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Prose rules describe what to do but require the model to infer how to apply them to novel inputs. Few-shot examples directly demonstrate application, especially for edge and atypical cases where the mapping from abstract rule to concrete situation is non-trivial. Design B's edge-case and negative examples act as anchors for the decision boundary, while Design A's prose must be interpreted by Claude at inference time. The architectural principle is 'showing beats telling' for tasks with ambiguous or context-sensitive application: examples ground abstract rules in concrete behavior. For purely typical inputs, both designs may perform similarly — the delta emerges at the boundary.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-examples
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/chain-of-thought

**Why the others are wrong:**

- **A:** Examples do not replace instructions — they complement them. Design B works well precisely because it combines a concise instruction base with targeted examples. The principle is complementarity, not replacement.
- **C:** A longer system prompt does not automatically outperform a shorter one with examples; verbosity can introduce ambiguity or inconsistency. The empirical question is which encoding generalizes better to hard cases, and examples typically win on edge cases.
- **D:** The representations are not equivalent — prose rules and concrete examples encode information differently and have different generalization properties. Even if the 'same rules' are intended, the model's ability to apply them varies significantly based on encoding format.

</details>

---

#### C282

*TS 4.2 · Few-shot prompting*

**Situation:** An architect is designing a prompt to extract sentiment (Positive / Negative / Neutral) from product reviews. She has assembled five examples, all of which express clearly positive or clearly negative sentiment. There are no ambiguous or mixed-sentiment examples.

What is the MOST significant risk of this few-shot set design?

- **A)** Five examples is too few to consume enough tokens to anchor Claude's behavior.
- **B)** Claude will always output 'Neutral' because no neutral examples were provided.
- **C)** Claude may perform poorly on mixed-sentiment or hedged reviews because the examples do not teach it how to handle the decision boundary between classes.
- **D)** Claude will memorize the exact wording of the five examples and only respond correctly when the input matches one of them verbatim.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Few-shot sets without boundary or edge-case examples fail to teach the model how to handle ambiguous inputs. In sentiment classification, mixed reviews ('The product is amazing but the delivery was terrible') sit on the decision boundary. Without at least one example showing how to resolve such cases, Claude will extrapolate inconsistently from the clear-case examples, reducing real-world accuracy.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-examples
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview

**Why the others are wrong:**

- **A:** Five examples is a reasonable few-shot set size; the problem here is content coverage (missing edge cases), not quantity.
- **B:** Claude does not default to under-represented classes; it generalizes from patterns. Without neutral examples, it is more likely to force ambiguous inputs into Positive or Negative, not to default to Neutral.
- **D:** Claude generalizes from few-shot examples rather than doing exact-match lookup; it applies the demonstrated patterns to novel inputs.

</details>

---

#### C283

*TS 4.2 · Few-shot prompting*

**Situation:** A prompt engineer is building a code-review assistant. She writes four few-shot examples where all reviewed code snippets are correct and the expected output is always 'LGTM — no issues found.' She then deploys the assistant and discovers it almost never flags real bugs.

Which principle of few-shot design does this scenario MOST directly violate?

- **A)** Few-shot examples should be randomly sampled from production data rather than hand-crafted.
- **B)** Examples should always be written in the same programming language as the target code.
- **C)** Negative examples — demonstrating how to respond to inputs that should trigger a non-default action — must be included when the task requires the model to identify failures or deviations.
- **D)** At least half of all examples must use tool calls rather than direct text responses.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

When all few-shot examples show the same output class (here: 'no issues'), Claude learns a strong prior toward that output. For tasks where identifying deviations (bugs, violations, anomalies) is the primary value, the few-shot set must include negative examples — cases where the correct action is to flag a problem — so Claude learns both the positive (LGTM) and negative (bug found) response patterns.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-examples
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview

**Why the others are wrong:**

- **A:** Hand-crafted examples are a legitimate and often superior approach to curating a high-quality few-shot set; random sampling can introduce noise and may not cover edge cases.
- **B:** The programming language of examples is irrelevant to the core problem; the issue is that zero buggy examples were shown, not that the language was wrong.
- **D:** There is no requirement that a certain fraction of few-shot examples use tool calls; the appropriate output format depends on the task design.

</details>

---

#### C284

*TS 4.2 · Few-shot prompting*

**Situation:** A developer is building a prompt that must handle both very short inputs (one word) and very long inputs (multi-paragraph documents). His current few-shot set contains only medium-length examples of around 50 words each.

What specific risk does this example set introduce for production inputs, and what is the fix?

- **A)** Claude will refuse very short inputs because they are shorter than any example it was shown.
- **B)** The format or depth of Claude's output may not scale appropriately for inputs that differ greatly in length from the examples; adding at least one short-input and one long-input example teaches Claude how to calibrate response length and detail.
- **C)** Using only 50-word examples will cause Claude to always produce exactly 50-word outputs regardless of input length.
- **D)** Length variation in examples is irrelevant; only the output label matters for few-shot learning.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Few-shot examples shape Claude's intuition about how to handle inputs of various characteristics. If all examples are medium-length, Claude has no demonstrated guidance for whether a one-word input should get a one-sentence reply or a full paragraph, or how to handle a multi-paragraph document. Including at least one short-input/short-output example and one long-input/long-output example teaches the expected calibration at both ends of the input spectrum.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-examples
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview

**Why the others are wrong:**

- **A:** Claude does not refuse inputs that are shorter or longer than demonstrated examples; it generalizes. The risk is miscalibrated output length or depth, not refusal.
- **C:** Claude does not lock onto a fixed token count from examples; it generalizes the style and structure, not the exact length. However, if all examples produce similar-length outputs, Claude may exhibit a mild bias toward that length.
- **D:** Input characteristics — including length — do influence how Claude calibrates its output. The input side of few-shot examples is as important as the output side for teaching appropriate behavior.

</details>

---

#### C285

*TS 4.2 · Few-shot prompting*

Your extraction system handles invoices well but fails on handwritten receipts — dates are missed, amounts are extracted incorrectly, and merchant names are garbled. Adding 5 handwritten receipt examples to the extraction prompt significantly improves accuracy for similar receipts. What prompting technique is this?

- **A)** Few-shot prompting — targeted examples for a problematic document type that demonstrate correct extraction from varied formats
- **B)** Self-consistency prompting
- **C)** Zero-shot prompting with enhanced instructions
- **D)** Chain-of-thought prompting

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Few-shot prompting: targeted examples for problematic document types that demonstrate correct extraction from varied formats. This enables generalization to novel similar documents.

</details>

---

#### C286

*TS 4.2 · Few-shot prompting*

Extraction of required fields from invoices sometimes returns empty strings for `vendor_name` — but only for invoices with non-standard layouts (header at the bottom, vendor info in a sidebar). Adding few-shot examples of these non-standard layouts fixes the problem. Why?

- **A)** The few-shot examples increase the model's confidence in non-standard layouts.
- **B)** Few-shot examples demonstrate correct handling of varied document structures, enabling the model to generalize extraction patterns to novel layouts rather than only matching the standard format.
- **C)** More examples always improve extraction accuracy regardless of the type of error.
- **D)** The examples provide the model with the specific vendor names to extract.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Few-shot examples demonstrate varied document structures, enabling generalization. It's not about confidence (A), quantity of examples (C), or specific values (D).

</details>

---

#### C287

*TS 4.2 · Few-shot prompting*

Your CI review prompt includes 12 few-shot examples showing various bug types. Adding 3 more examples demonstrating that certain patterns are **acceptable** (not bugs) reduced false positives by 40% without missing real bugs. Why were these "non-bug" examples so effective?

- **A)** More examples always improve accuracy by giving the model more training data.
- **B)** Few-shot examples showing acceptable patterns (non-bugs) help the model distinguish genuine issues from acceptable patterns, enabling generalization to novel cases rather than matching only pre-specified bug patterns.
- **C)** The non-bug examples increased the model's context window, giving it more room for analysis.
- **D)** The examples reduced the model's temperature, making it more conservative.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Few-shot examples showing acceptable patterns help the model distinguish genuine issues from non-issues, enabling generalization to novel cases. It's not about data volume (A), context size (C), or temperature (D).

</details>

---

#### C288

*TS 4.2 · Few-shot prompting*

Your agent recommends products based on customer preferences, but the recommendations feel generic ("popular items") rather than personalized. What improves quality?

- **A)** Increase the model's temperature for more creative recommendations.
- **B)** Add "Be more personalized" to the prompt.
- **C)** Document personalization criteria in CLAUDE.md: how to weigh purchase history, browsing behavior, stated preferences, and demographic signals. Provide examples of generic vs. personalized recommendations.
- **D)** Use a separate recommendation engine model and have the agent just format the output.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Document personalization criteria in CLAUDE.md with examples of generic vs. personalized recommendations.

**Why the others are wrong:**

- **A:** Temperature doesn't improve quality.
- **B:** Vague instructions don't help.
- **D:** External engine removes agent capability.

</details>

---

#### C289

*TS 4.2 · Few-shot prompting*

Your recommendation prompt includes 10 few-shot examples showing good product matches. Adding 5 more examples showing "near-miss" products (why they don't match customer criteria) reduced inappropriate recommendations by 35%. Why were these "non-match" examples so effective?

- **A)** More examples always improve accuracy by giving the model more training data.
- **B)** Few-shot examples showing non-matches help the model distinguish suitable from unsuitable products, enabling generalization to new products rather than matching only pre-specified patterns.
- **C)** The non-match examples increased the model's context window, giving it more room for analysis.
- **D)** The examples reduced the model's temperature, making it more conservative.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Few-shot examples showing non-matches help the model distinguish suitable from unsuitable products, enabling generalization. It's not about data volume (A), context size (C), or temperature (D).

</details>

---

#### C290

*TS 4.2 · Few-shot prompting*

Your content generation prompt produces clean output for standard lessons but struggles with interdisciplinary units that reference standards from multiple subjects. For example, a STEM unit references both science and math standards, but the extraction only captures math. What prompting technique fixes this?

- **A)** Create a separate "science standards" pass.
- **B)** Add "Check all subjects" to the system prompt.
- **C)** Pre-process the request to separate into single-subject units.
- **D)** Add 2-3 few-shot examples showing correct generation from interdisciplinary units, demonstrating that the agent should search for and incorporate standards from all referenced subjects.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Few-shot examples showing extraction from interdisciplinary units. Demonstrates that critical data can come from multiple subjects.

</details>

---

#### C291

*TS 4.3 · Structured output via tool use & schemas*

**Situation:** A developer wants Claude to always respond with a JSON object containing exactly the fields `name` (string) and `score` (integer). They define a tool called `submit_result` with an `input_schema` matching that structure and call the API.

Which `tool_choice` configuration guarantees Claude will call `submit_result` and never return a plain-text answer?

- **A)** `{"type": "tool", "name": "submit_result"}` — Claude must call exactly this tool
- **B)** `{"type": "auto"}` — Claude decides whether to call a tool
- **C)** `{"type": "any"}` — Claude must call at least one of the available tools
- **D)** Omit `tool_choice` entirely and rely on the system prompt to ask for JSON

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Setting `tool_choice` to `{"type": "tool", "name": "submit_result"}` forces Claude to invoke that specific tool on every turn, guaranteeing the response arrives as a structured `tool_use` block whose `input` conforms to the declared `input_schema`. No other configuration provides this hard guarantee.

References:
- https://docs.anthropic.com/en/docs/tool-use/overview
- https://docs.anthropic.com/en/docs/tool-use/forcing-tool-use

**Why the others are wrong:**

- **B:** `auto` lets Claude decide; it may return a text answer instead of calling the tool, breaking downstream parsers that expect structured output.
- **C:** `any` forces a tool call but does not pin which tool. If multiple tools are registered, Claude may call a different one, and no specific schema is guaranteed.
- **D:** Relying on the system prompt alone is a soft instruction. Claude may still produce free-form text, especially when it judges that a direct answer is more helpful than a tool call.

</details>

---

#### C292

*TS 4.3 · Structured output via tool use & schemas*

**Situation:** An architect is designing a tool schema for extracting product data. Some products have a `discount_percent` field and some do not. The team debates whether to mark the field `required` or to allow it to be absent.

What is the recommended approach for optional fields in a tool `input_schema` to prevent Claude from hallucinating values?

- **A)** Add the field to `required` and tell Claude in the system prompt to skip it when unknown
- **B)** Omit the field from the schema entirely so Claude never knows it exists
- **C)** Include the field in `required` and set a default value of `0` in the description
- **D)** List the field in `properties` but exclude it from the `required` array, and allow `null` via a type union or `nullable` flag

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Optional fields should appear in `properties` so Claude knows they exist, but must be absent from `required` so Claude is not forced to produce a value. Combining this with a nullable type (e.g., `["number", "null"]`) lets Claude emit `null` when the data is missing rather than fabricating a number, which prevents hallucination.

References:
- https://docs.anthropic.com/en/docs/tool-use/overview
- https://docs.anthropic.com/en/docs/tool-use/tool-use-best-practices

**Why the others are wrong:**

- **A:** System-prompt instructions conflict with the schema's `required` constraint. The schema enforcement wins at the API level, so Claude will still be compelled to produce a value despite the natural-language instruction.
- **B:** Omitting the field means Claude cannot return it even when it is genuinely present in the source data, causing silent data loss on products that do have a discount.
- **C:** Marking the field `required` forces Claude to always supply it. When the actual value is unknown, Claude will invent a plausible number rather than leave it absent, introducing hallucinated data.

</details>

---

#### C293

*TS 4.3 · Structured output via tool use & schemas*

**Situation:** A team needs Claude to classify support tickets into one of three categories: `billing`, `technical`, or `other`. They want the API response to contain only one of those exact strings, never a free-form label.

Which schema pattern best enforces that the `category` field can only contain `billing`, `technical`, or `other`?

- **A)** Type `string` with a description listing the three values as examples
- **B)** Type `string` with an `enum` array: `["billing", "technical", "other"]`
- **C)** Type `object` with a boolean field for each category
- **D)** Type `string` with a `pattern` regex: `^(billing|technical|other)$`

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

The `enum` keyword in JSON Schema restricts the field to an explicit set of allowed values. Claude treats the schema as a hard contract; it will only emit one of the listed strings. This is the canonical way to express constrained categorical values in tool schemas.

References:
- https://docs.anthropic.com/en/docs/tool-use/overview
- https://docs.anthropic.com/en/docs/tool-use/tool-use-best-practices

**Why the others are wrong:**

- **A:** A plain `string` type with examples in the description is a soft hint. Claude may still produce variations like `"Billing"` or `"account issue"` because the schema does not enforce the constraint at a structural level.
- **C:** Using a boolean field per category changes the data model entirely, requires the consumer to iterate fields to find the active category, and does not prevent multiple `true` values simultaneously.
- **D:** `pattern` with a regex also works in standard JSON Schema, but `enum` is more explicit, more readable, and is the idiomatic Anthropic-recommended approach for a closed set of known values.

</details>

---

#### C294

*TS 4.3 · Structured output via tool use & schemas*

**Situation:** An architect designs a pipeline where Claude extracts structured financial events from earnings call transcripts. The tool `record_event` has this partial schema:


```json
{
  "type": "object",
  "properties": {
    "event_type": { "type": "string", "enum": ["revenue", "guidance", "acquisition", "risk"] },
    "amount_usd": { "type": "number" },
    "currency_override": { "type": ["string", "null"] },
    "metadata": {
      "type": "object",
      "properties": {
        "fiscal_quarter": { "type": "string" },
        "confidence": { "type": "number", "minimum": 0, "maximum": 1 }
      },
      "required": ["fiscal_quarter"]
    }
  },
  "required": ["event_type", "metadata"]
}
```


In production, `amount_usd` is absent in roughly 40% of calls (for qualitative events), but Claude sometimes emits `0` or `null` for it.

Which combination of schema adjustments best resolves the fabricated `amount_usd` values while preserving all other constraints?

- **A)** Add `"amount_usd"` to `required` and change its type to `["number", "null"]` so Claude must always emit either a number or an explicit null
- **B)** Remove `amount_usd` from `properties` entirely so Claude never attempts to populate it
- **C)** Keep `amount_usd` absent from `required` (already the case) and change its type to `["number", "null"]`; add a description clarifying it should be omitted when no monetary figure is present
- **D)** Move `amount_usd` inside the `metadata` sub-object and add it to `metadata.required` alongside `fiscal_quarter`

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

The field is already optional (not in `required`), so the structural fix is to ensure Claude has a null escape hatch via `["number", "null"]` and clear semantic guidance in the `description`. When a field is optional AND nullable, Claude understands it may be omitted or null; the description reinforces that omission is correct for qualitative events. Making it required (A) would force fabrication on qualitative rows. Removing it (B) silently drops monetary data on quantitative events. Moving it to a sub-object (D) changes the data model and still does not address the fabrication root cause.

References:
- https://docs.anthropic.com/en/docs/tool-use/overview
- https://docs.anthropic.com/en/docs/tool-use/tool-use-best-practices

**Why the others are wrong:**

- **A:** Adding `amount_usd` to `required` reintroduces the fabrication pressure that the question is trying to eliminate. Claude would be forced to supply a value for every event, producing `0` or a hallucinated figure for qualitative events.
- **B:** Removing the field from `properties` prevents Claude from returning monetary amounts even when they are clearly stated in the transcript, causing silent data loss on the 60% of events where the amount is present.
- **D:** Relocating the field to a sub-object is a structural change with no bearing on the nullable/required semantics. Unless `amount_usd` is also removed from `metadata.required`, it would become mandatory for every event, recreating the same hallucination problem.

</details>

---

#### C295

*TS 4.3 · Structured output via tool use & schemas*

**Situation:** A data platform ingests diverse documents and routes them through three Claude tools: `extract_invoice`, `extract_contract`, and `extract_correspondence`. Each tool has a different schema. The system currently uses `tool_choice: {"type": "any"}` and a routing instruction in the system prompt. In production, Claude occasionally calls the wrong tool for a document type, producing a response that is structurally valid for the wrong schema.

What architectural change best guarantees that the correct schema is enforced for each document type?

- **A)** Merge all three schemas into a single `extract_document` tool with a `document_type` discriminator field and use `tool_choice: {"type": "tool", "name": "extract_document"}`
- **B)** Add stronger routing instructions to the system prompt and increase `temperature` to 0 so Claude reliably selects the correct tool
- **C)** Switch to `tool_choice: {"type": "auto"}` so Claude has full flexibility to choose the right tool based on the document content
- **D)** Classify the document type in a first API call (using a lightweight classification prompt), then issue a second API call with only the matching tool registered and `tool_choice: {"type": "tool", "name": "<matched_tool>"}` forced

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Separating classification from extraction via a two-call pipeline eliminates the ambiguity at the root of the problem. The first lightweight call determines document type; the second call registers only the matching tool and forces it with `tool_choice`. Claude has no choice but to use the correct schema because it is the only tool available. This is the standard pattern for deterministic multi-schema routing.

References:
- https://docs.anthropic.com/en/docs/tool-use/forcing-tool-use
- https://docs.anthropic.com/en/docs/tool-use/overview
- https://docs.anthropic.com/en/docs/tool-use/tool-use-best-practices

**Why the others are wrong:**

- **A:** A single tool with a discriminator field adds schema complexity: the merged schema must cover all three document types, so many fields become optional for any given type. This weakens validation and hands the routing decision back to Claude's judgment inside the tool call rather than removing it.
- **B:** Setting `temperature: 0` improves determinism for a given prompt but does not change the fundamental ambiguity of having three tools available under `any` routing. A deterministically wrong answer is still wrong, and prompt instructions remain soft constraints.
- **C:** `auto` gives Claude maximum flexibility, which is exactly what causes wrong-tool selections. Increasing flexibility worsens the problem rather than solving it.

</details>

---

#### C296

*TS 4.3 · Structured output via tool use & schemas*

**Situation:** An engineer is migrating a structured-output pipeline from JSON mode (system prompt instruction + text parsing) to the tool-use approach. The original system prompt says: 'You must respond with a JSON object matching this schema: {…}'. The engineer replaces it with a tool definition and `tool_choice: {"type": "tool", "name": "output_result"}`. During migration testing, they notice that Claude's reasoning explanation (previously in a `reasoning` field of the JSON) no longer appears.

What is the most accurate explanation, and what is the correct way to recover the reasoning field?

- **A)** JSON mode produces a `reasoning` field automatically; tool use does not. The migration must keep the JSON mode approach to preserve reasoning output
- **B)** Claude never produces reasoning inside tool calls. The reasoning must be captured by examining the `stop_reason` field of the API response
- **C)** Tool use suppresses all reasoning. Use extended thinking (`"thinking": {"type": "enabled"}`) instead of a `reasoning` field in the schema
- **D)** The `reasoning` field was only produced because the old system prompt explicitly named it. In the tool schema, add `reasoning` to `properties` (as a string) and to `required`; Claude will populate it as part of the structured output

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

The `reasoning` field did not disappear because of an inherent tool-use limitation — it disappeared because the new schema does not declare it. Claude faithfully fills fields that are declared in the `input_schema`. Adding `reasoning` as a required string field instructs Claude to provide its rationale as part of the structured output, exactly as before. This is a schema completeness issue, not a fundamental capability difference between JSON mode and tool use.

References:
- https://docs.anthropic.com/en/docs/tool-use/overview
- https://docs.anthropic.com/en/docs/tool-use/tool-use-best-practices
- https://docs.anthropic.com/en/docs/build-with-claude/extended-thinking

**Why the others are wrong:**

- **A:** There is no automatic `reasoning` field in JSON mode. In the original pipeline, the field appeared because the system prompt schema explicitly included it. The same mechanism works in tool use — the field must be declared in the schema.
- **B:** `stop_reason` indicates why Claude stopped generating (e.g., `end_turn`, `tool_use`) and contains no reasoning content. Reasoning content must be an explicit field in the response body, not metadata.
- **C:** Extended thinking is a separate feature (streaming thinking tokens) and is not required to obtain a reasoning field. A simple string field in the schema achieves the same goal with less complexity and no additional API parameters.

</details>

---

#### C297

*TS 4.3 · Structured output via tool use & schemas*

**Situation:** A pipeline uses `tool_choice: {"type": "tool", "name": "extract_entity"}` to force structured extraction. The tool schema has `entity_type` (enum: person, org, location) and `confidence` (number 0–1) both in `required`. During testing, Claude occasionally sets `confidence` to `1.0` for entities it clearly cannot verify.

What schema change best reduces fabricated confidence values without losing the field entirely?

- **A)** Move `confidence` out of `required` and add `"null"` to its type so Claude can emit `null` when certainty is unavailable
- **B)** Remove `confidence` from the schema entirely and compute it post-hoc with a separate heuristic
- **C)** Add a `minimum: 0` and `maximum: 1` constraint to prevent out-of-range values
- **D)** Change `confidence` from `number` to `string` and use an enum: `["low", "medium", "high"]`

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

When a field is `required`, Claude must always supply a value, which creates hallucination pressure for uncertain cases. Making `confidence` optional (removed from `required`) with a nullable type gives Claude a semantically honest escape hatch: it emits the number when genuinely available and `null` when it cannot estimate reliably. This preserves useful signal while eliminating forced fabrication.

References:
- https://docs.anthropic.com/en/docs/tool-use/tool-use-best-practices
- https://docs.anthropic.com/en/docs/tool-use/overview

**Why the others are wrong:**

- **B:** Removing the field loses the model's signal entirely. When Claude does have genuine confidence information, that signal is valuable and should be captured, not discarded.
- **C:** `minimum`/`maximum` constraints prevent out-of-range numbers but do nothing to prevent Claude from defaulting to an extreme value like `1.0` within the valid range when it feels compelled to supply something.
- **D:** Switching to an ordinal enum does constrain the value space and reduces false precision, but an enum is still `required` under the original schema. Claude would still be compelled to pick a bucket even when it has no basis for the choice.

</details>

---

#### C298

*TS 4.3 · Structured output via tool use & schemas*

**Situation:** An engineer evaluates two approaches for generating structured JSON from Claude: (A) using a tool with `tool_choice` forced, and (B) using a system prompt that says 'Respond only in valid JSON' and then parsing `content[0].text`. The application runs in production where downstream services parse every response.

Which statement most accurately compares the reliability of these two approaches?

- **A)** Both approaches are equally reliable because Claude follows system prompt instructions as strictly as it follows schemas
- **B)** Approach B is more reliable because it avoids the overhead of tool definitions and keeps the response in a single text block
- **C)** Approach A provides stronger guarantees because the schema is enforced at the API level, while Approach B relies on natural-language compliance that can fail under edge cases
- **D)** Approach B is more reliable for nested JSON because tool schemas cannot express nested object structures

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Tool use with a forced `tool_choice` and a well-defined `input_schema` is enforced structurally: the API rejects responses that do not conform. Natural-language instructions like 'respond only in valid JSON' are soft constraints — Claude generally respects them but may deviate when it encounters ambiguous inputs, very long contexts, or situations where it judges a direct answer more helpful. For production pipelines where every parse must succeed, the tool-use approach is the correct choice.

References:
- https://docs.anthropic.com/en/docs/tool-use/overview
- https://docs.anthropic.com/en/docs/tool-use/forcing-tool-use

**Why the others are wrong:**

- **A:** Schema constraints and natural-language instructions are not equivalent. Schemas are validated mechanically; instructions depend on the model's judgment and can be overridden by competing pressures in the prompt.
- **B:** The overhead of a tool definition is negligible compared to the cost of a single downstream parse failure in production. Reliability, not overhead, is the deciding factor.
- **D:** Tool schemas fully support nested objects by using `type: object` within `properties`. Deeply nested structures are expressible and commonly used in production tool schemas.

</details>

---

#### C299

*TS 4.3 · Structured output via tool use & schemas*

Your extraction prompt instructs: "Extract the payment terms from each invoice." For invoices that don't contain payment terms, Claude fabricates plausible values like "Net 30" to fill the required field. What schema change fixes this?

- **A)** Add a validation rule that checks extracted payment terms against a known list.
- **B)** Add "Do not fabricate information" to the extraction prompt.
- **C)** Change the `payment_terms` field from `required` with type `"string"` to optional with type `["string", "null"]`, allowing the model to return `null` when the information is absent.
- **D)** Add a default value of "N/A" to the `payment_terms` field.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Nullable optional fields prevent hallucination. When information is absent, Claude returns `null` instead of fabricating.

**Why the others are wrong:**

- **A:** Validation catches but doesn't prevent.
- **B:** Prompt instructions are probabilistic.
- **D:** Default values mask missing data.

</details>

---

#### C300

*TS 4.3 · Structured output via tool use & schemas*

You need Claude to classify each invoice into a category. Most invoices fall into "Office Supplies," "Software," "Professional Services," or "Travel." Occasionally, unusual categories appear (e.g., "Laboratory Equipment"). How should you design the schema field?

- **A)** `"category": {"type": "string", "enum": ["Office Supplies", "Software", "Professional Services", "Travel", "other"]}` with a sibling `"category_detail": {"type": ["string", "null"]}` — enum with "other" + detail.
- **B)** `"category": {"type": "string", "enum": ["Office Supplies", "Software", "Professional Services", "Travel"]}` — strict enum.
- **C)** `"categories": {"type": "array", "items": {"type": "string"}}` — allow multiple categories.
- **D)** `"category": {"type": "string"}` — free-form text.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

`enum` with `"other"` + detail string balances structured categories with extensibility.

**Why the others are wrong:**

- **B:** Strict enum fails on unusual categories.
- **C:** Multiple categories changes the semantics.
- **D:** Free-form loses structure.

</details>

---

#### C301

*TS 4.3 · Structured output via tool use & schemas*

Your JSON schema enforces correct syntax via `tool_use`, but extracted invoices sometimes have line items that don't sum to the stated total ($1,250 stated, but line items sum to $1,275). What type of error is this, and can `tool_use` prevent it?

- **A)** Add arithmetic validation rules directly into the JSON schema definition.
- **B)** This is a semantic validation error. `tool_use` eliminates syntax errors but NOT semantic errors. Implement a separate validation step that checks arithmetic consistency.
- **C)** This is a schema syntax error. Using strict JSON schemas with `tool_use` will prevent it.
- **D)** This is a data quality error in the source document. No extraction system can fix source data issues.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Semantic errors (incorrect arithmetic) are NOT prevented by `tool_use`. `tool_use` eliminates syntax errors only. A separate validation step is needed.

**Why the others are wrong:**

- **A:** Schema-level arithmetic rules aren't supported in JSON Schema.

</details>

---

#### C302

*TS 4.3 · Structured output via tool use & schemas*

Your self-correction pipeline extracts `stated_total` and independently calculates `calculated_total` from line items. When they differ, it flags `conflict_detected: true`. Which additional schema design further improves reliability?

- **A)** Add `extraction_method: "auto"` to track how each field was extracted.
- **B)** Add `verification_needed: true/false` as a top-level field.
- **C)** Add `confidence_score` as a float field for the overall extraction.
- **D)** Add `"unclear"` as an enum option for ambiguous fields and require `ambiguity_reason` when used.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

`"unclear"` enum with `ambiguity_reason` handles genuinely ambiguous cases without forcing the model to guess.

**Why the others are wrong:**

- **A:** Extraction method tracking doesn't improve accuracy.
- **B:** Boolean flags lack granularity.
- **C:** Confidence scores are poorly calibrated.

</details>

---

#### C303

*TS 4.3 · Structured output via tool use & schemas*

Your extraction schema marks `effective_date` as `required` with type `"string"`. For contracts that are undated (surprisingly common in draft contracts), Claude fabricates a plausible date. You also notice Claude invents a `governing_law` value when the contract is silent on jurisdiction. What single schema design principle fixes both issues?

- **A)** Implement a post-extraction verification step that checks all dates against the document.
- **B)** Add validation rules that check extracted dates against a reasonable range.
- **C)** Make fields that may be absent from source documents optional with nullable types (`["string", "null"]`) and remove them from `required`. This prevents the model from fabricating values to satisfy schema constraints.
- **D)** Add "Do not hallucinate" to the extraction prompt.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Nullable optional fields prevent fabrication for both `effective_date` and `governing_law`. When information is absent, Claude returns `null`.

**Why the others are wrong:**

- **A:** Post-verification is reactive.
- **B:** Validation catches but doesn't prevent.
- **D:** Prompt instructions are probabilistic.

</details>

---

#### C304

*TS 4.3 · Structured output via tool use & schemas*

Your financial statement extraction tool uses `tool_choice: "auto"`. The model processes 80% of documents correctly but returns conversational text for 20%: "This appears to be a financial statement from..." without calling the extraction tool. Changing to `tool_choice: "any"` resolves this. However, you now have a new problem: you have two extraction tools (`extract_income_statement` and `extract_balance_sheet`) and the model sometimes picks the wrong one. What's the best overall configuration?

- **A)** Keep `tool_choice: "any"` and improve both tool descriptions to make them clearly distinguishable.
- **B)** Use `tool_choice: "auto"` with stronger prompt instructions.
- **C)** Create a single combined `extract_financial_data` tool that handles both document types.
- **D)** Use forced tool selection to always call `extract_income_statement` first.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

`tool_choice: "any"` guarantees a tool call. The wrong-tool problem is solved by improving descriptions. Better descriptions are the primary mechanism for tool selection.

**Why the others are wrong:**

- **C:** Combined tool loses type-specific extraction logic.
- **D:** Forced selection doesn't handle varying document types.

</details>

---

#### C305

*TS 4.3 · Structured output via tool use & schemas*

Your extraction pipeline processes medical records. For the field `blood_pressure`, some documents contain "120/80 mmHg," some contain "BP normal," and some don't mention blood pressure at all. How should you design the schema?

- **A)** `"blood_pressure": {"type": "string"}` — accept any text and normalize later.
- **B)** `"blood_pressure": {"type": ["string", "null"]}` (nullable) with format normalization rules in the prompt: "Normalize to 'systolic/diastolic mmHg' format. If described as 'normal' without values, use 'normal'. If not mentioned, return null."
- **C)** `"blood_pressure_systolic": {"type": "integer"}, "blood_pressure_diastolic": {"type": "integer"}` — separate numeric fields.
- **D)** `"blood_pressure": {"type": "string", "enum": ["normal", "elevated", "high", "critical"]}` — categorize instead of extracting raw values.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Nullable field + format normalization rules handles all three cases: structured values, descriptive values ("normal"), and absence (null). Separate integers (C) can't handle "normal." Enum (D) loses actual values.

**Why the others are wrong:**

- **A:** Plain string pushes normalization downstream.

</details>

---

#### C306

*TS 4.3 · Structured output via tool use & schemas*

Your self-correction pipeline extracts data, validates it, and retries on failure. For a lease agreement, the extraction includes `monthly_rent: 2500` but the lease actually states "twenty-five hundred per month for the first year, increasing to three thousand in year two." What additional schema field would capture this complexity?

- **A)** Add `"rent_range": {"type": "object", "properties": {"min": {"type": "number"}, "max": {"type": "number"}}}`.
- **B)** `"rent_notes": {"type": "string"}` for free-text context.
- **C)** Add `"rent_is_variable": {"type": "boolean"}`.
- **D)** Design a structured rent schedule: `"rent_schedule": {"type": "array", "items": {"type": "object", "properties": {"period": {"type": "string"}, "monthly_amount": {"type": "number"}, "start_date": {"type": ["string", "null"]}, "end_date": {"type": ["string", "null"]}}}}`.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

A structured rent schedule array captures multi-period variable amounts. Single field (existing) loses the year-two increase.

**Why the others are wrong:**

- **A:** Range loses temporal information.
- **B:** Free-text notes aren't structured.
- **C:** Boolean flag indicates variability but doesn't capture amounts.

</details>

---

#### C307

*TS 4.3 · Structured output via tool use & schemas*

Your extraction pipeline processes transaction records. For the field `transaction_type`, some documents contain "wire transfer," some contain "WT," and some don't specify the type at all. How should you design the schema?

- **A)** `"transaction_type": {"type": "string", "enum": ["wire_transfer", "ach", "check", "other"]}` — categorize instead of extracting raw values.
- **B)** `"transaction_type": {"type": ["string", "null"]}` (nullable) with format normalization rules in the prompt: "Normalize to standard codes (WT=wire transfer, ACH=automated clearing house). If not specified, return null."
- **C)** `"transaction_type_code": {"type": "string"}, "transaction_type_description": {"type": "string"}` — separate fields for code and description.
- **D)** `"transaction_type": {"type": "string"}` — accept any text and normalize later.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Nullable field + format normalization rules handles all three cases: structured values, abbreviations, and absence (null).

**Why the others are wrong:**

- **A:** Enum can't handle unspecified types.
- **C:** Separate fields add complexity.
- **D:** Plain string pushes normalization downstream.

</details>

---

#### C308

*TS 4.3 · Structured output via tool use & schemas*

Your extraction schema marks `termination_date` as `required` with type `"string"`. For contracts without specified termination dates (evergreen agreements), Claude fabricates a plausible date. What schema design principle fixes this?

- **A)** Add validation rules that check extracted dates against a reasonable range.
- **B)** Implement a post-extraction verification step that checks all dates against the document.
- **C)** Make fields that may be absent optional with nullable types (`["string", "null"]`) and remove from `required`. This prevents fabrication to satisfy schema constraints.
- **D)** Add "Do not hallucinate dates" to the extraction prompt.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Nullable optional fields prevent fabrication. When information is absent, Claude returns null.

**Why the others are wrong:**

- **A:** Validation catches but doesn't prevent.

</details>

---

#### C309

*TS 4.3 · Structured output via tool use & schemas*

`process_refund` expects `{order_id: string, amount: number, reason_code: "DEFECT"|"NOT_AS_DESCRIBED"|"OTHER"}`. In production, about 8% of calls fail schema validation in your tool executor: Claude sometimes sends `amount: "$50"` (string with currency symbol), `reason_code: "defect"` (wrong case), or omits `reason_code` entirely. Which fix most directly eliminates this class of error?

- **A)** Run every tool call through a separate Claude API call first that cleans the arguments before forwarding to `process_refund`.
- **B)** Catch the validation error in the tool executor and return a structured error message describing the schema violation, so Claude can correct on the next turn.
- **C)** Add three few-shot examples of correctly-formatted `process_refund` calls to the system prompt.
- **D)** Add `strict: true` to the `process_refund` tool definition.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

`strict: true` constrains Claude's tool call output to conform to the JSON schema at the API layer. With it set, the model cannot emit `"$50"` for a `number`-typed field, cannot lowercase an enum value, and cannot omit a required property. This is the purpose-built primitive for exactly this problem.

**Why the others are wrong:**

- **A:** Doubles inference cost and latency to do what a single config flag does for free.
- **B:** A useful belt-and-braces fallback, but it pays for a wasted turn (and risks an unbounded correction loop) instead of preventing the error.
- **C:** Helps in expectation but is best-effort; in-context examples don't *guarantee* conformance.

</details>

---

#### C310

*TS 4.4 · Validation, retry & feedback loops*

**Situation:** A CI pipeline generates tests with Claude Code and immediately runs them. In production, the pipeline started failing intermittently: sometimes Claude Code returns syntactically valid JSON but the `testCode` field contains Python syntax errors that cause the test runner to crash. The team wants to make the pipeline self-healing without adding manual review gates.

Which pipeline design BEST addresses intermittent invalid test code generation while maintaining full automation?

- **A)** Increase the `--max-tokens` limit to give Claude Code more space to produce correct code
- **B)** Add a `--temperature 0` flag to Claude Code to make output deterministic and eliminate syntax errors
- **C)** Switch from `--output-format json` to plain text output, then use regex to extract the code block, which is more reliable than JSON parsing
- **D)** After Claude Code generates tests, parse the `testCode` field, run a syntax check (e.g., `python -m py_compile`), and if it fails, re-invoke Claude Code with the error appended to the prompt for self-correction

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

The correct approach is a validate-and-retry loop: extract the generated test code, run a lightweight syntax check, and if validation fails, re-invoke Claude Code with the original prompt plus the syntax error as additional context. This self-correction pattern is idiomatic for LLM pipelines and leverages Claude's ability to fix its own output when given explicit error feedback.

References:
- https://docs.anthropic.com/en/docs/claude-code/cli-reference
- https://docs.anthropic.com/en/docs/claude-code/github-actions

**Why the others are wrong:**

- **A:** `--max-tokens` is not a Claude Code CLI flag. Syntax errors are not caused by token limits; they are model output quality issues that require a validation-and-retry strategy.
- **B:** `--temperature` is not a Claude Code CLI flag; temperature is set at the API level. Even at temperature 0, LLMs can still produce syntactically invalid code. Determinism does not guarantee correctness.
- **C:** Switching to plain text and using regex is strictly worse than JSON: it is more fragile, harder to maintain, and loses the structural guarantees that `--output-format json` with `--json-schema` provides.

</details>

---

#### C311

*TS 4.4 · Validation, retry & feedback loops*

**Situation:** A junior engineer is crafting the error message returned to Claude after a failed JSON validation.

Which error message provides the most useful feedback to Claude for self-correction?

- **A)** "Error occurred in the pipeline."
- **B)** "Bad input."
- **C)** "Validation failed. Try again."
- **D)** "Field 'email' must be a valid RFC 5321 address; received 'user@@example'."

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Specific feedback naming the exact field, the constraint violated, and the invalid value received gives Claude the precise context it needs to correct only the failing parameter. Vague messages force Claude to guess what changed.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use#handling-tool-use-and-tool-results
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/be-clear-and-direct

**Why the others are wrong:**

- **A:** "Error occurred in the pipeline" is too abstract and could describe dozens of different root causes.
- **B:** "Bad input" conveys no structure; Claude has no basis for targeted correction and may change the wrong parameter.
- **C:** "Validation failed. Try again." gives no actionable information; Claude cannot determine which field to fix or why.

</details>

---

#### C312

*TS 4.4 · Validation, retry & feedback loops*

**Situation:** You are reviewing an existing agentic system. On tool failure the orchestrator currently returns the raw exception stack trace as the `content` of the `isError: true` tool_result block. Claude then retries. The team reports that retries rarely succeed and the model sometimes leaks internal path information in its responses.

What are the TWO most significant problems with returning a raw stack trace as the error feedback content, and what is the correct alternative?

- **A)** Stack traces expose internal implementation details (security risk) and provide no actionable parameter-level guidance (ineffective feedback); the correct alternative is a structured error message naming the specific field or constraint that failed with a corrective hint
- **B)** Stack traces are formatted incorrectly for JSON and break the API; the alternative is to base64-encode them
- **C)** Stack traces are too long and cause context window overflow; the alternative is to truncate the trace to 50 characters
- **D)** Stack traces cause Claude to believe it has a bug in its own code and stop responding; the alternative is to prefix the trace with 'ignore this'

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Raw stack traces create two distinct problems: (1) they expose internal file paths, library versions, and implementation details that could be surfaced to end users or logged insecurely; (2) they contain no field-level guidance that helps Claude correct its parameters — the model cannot infer from 'NullPointerException at line 47' which argument was wrong. The solution is to intercept the exception, map it to a user-facing error message that names the offending field and the constraint violated, and return that as the `isError: true` content.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use#handling-tool-use-and-tool-results
- https://docs.anthropic.com/en/docs/build-with-claude/agents/agent-security
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/be-clear-and-direct

**Why the others are wrong:**

- **B:** Stack traces are valid strings and do not break the JSON API; base64-encoding them makes them even less readable to Claude and does not address either core problem.
- **C:** Truncating the trace to 50 characters preserves the security leak, removes even the minimal context the trace contained, and still provides no actionable feedback.
- **D:** Claude does not interpret stack traces as evidence of its own code bugs; the described behaviour is not accurate, and prefixing with 'ignore this' would simply cause Claude to discard the only error context it received.

</details>

---

#### C313

*TS 4.4 · Validation, retry & feedback loops*

**Situation:** Your pipeline validates tool output before passing it to the next stage. A schema check reveals a missing required field. The model has already used two of its three allowed retries.

On the third (final) retry attempt, the tool output still fails validation. What is the correct next action?

- **A)** Restart the entire pipeline from the beginning
- **B)** Silently drop the output and proceed with downstream steps using partial data
- **C)** Increase the retry limit to allow more attempts
- **D)** Escalate to a human operator or trigger a defined fallback workflow

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

After exhausting the retry budget (2–3 attempts), the pipeline must not continue with invalid data or loop further. Escalating to a human or invoking a fallback preserves data integrity, limits blast radius, and follows Anthropic's minimal-footprint principle.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents#human-in-the-loop
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use#handling-tool-use-and-tool-results

**Why the others are wrong:**

- **A:** Restarting the full pipeline wastes work completed before the failure point and does not address the root cause.
- **B:** Proceeding with partial data violates the validation contract and may corrupt downstream steps or produce incorrect outputs.
- **C:** Dynamically expanding the retry limit defeats the purpose of the cap and risks runaway token consumption.

</details>

---

#### C314

*TS 4.4 · Validation, retry & feedback loops*

**Situation:** Claude calls a create_record tool and receives `isError: true` with the message 'duplicate key on field user_id'. On the next attempt, Claude sends the same user_id again.

What is the most likely root cause of Claude repeating the same erroneous value, and how should the error message be improved?

- **A)** The error message lacked actionable guidance; it should state the conflicting value and suggest a corrective action such as checking for an existing record
- **B)** The tool schema is missing a description field; add one
- **C)** The model's context window is too small; increase max_tokens
- **D)** The retry logic is firing too quickly; add a delay

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Claude adjusts behaviour based on the content of the error message. If the message only names the constraint without indicating what the model should do differently (e.g., look up the existing record or generate a new ID), Claude has insufficient signal to change its approach.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use#handling-tool-use-and-tool-results
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/be-clear-and-direct

**Why the others are wrong:**

- **B:** Adding a description to the tool schema helps with initial parameter selection but does not address how Claude interprets runtime errors.
- **C:** Context window size does not explain why Claude repeats the same value; the error message is already in context.
- **D:** A timing delay does not change the information available to Claude and would not prevent repeating the same value.

</details>

---

#### C315

*TS 4.4 · Validation, retry & feedback loops*

Pydantic validation catches an error: the extracted `invoice_date` is "March 5" (no year). You retry by sending Claude all of the following: the original document, the failed extraction, and the specific error message ("invoice_date must be ISO 8601 format with year"). Claude corrects it to "2025-03-05". In which of these situations would this retry approach be **ineffective**?

- **A)** The extracted date is in the wrong format (DD/MM/YYYY instead of ISO 8601).
- **B)** The invoice total is missing a decimal point ($1250 instead of $12.50).
- **C)** The payment terms are listed in a separate attachment document that was not provided to Claude.
- **D)** A line item description is truncated due to a long product name.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Retries fail when information is absent from the provided source. If payment terms are in a separate attachment not given to Claude, no amount of retrying will find them. Format errors (A), decimal errors (B), and truncation (D) can all be fixed via retry.

</details>

---

#### C316

*TS 4.4 · Validation, retry & feedback loops*

After extraction, Pydantic validation catches: `"contract_value"` is `"approximately $2.5 million"` but the schema expects a numeric field. You retry, providing the original document, the failed extraction, and the specific error. Claude corrects to `2500000.00`. After adding format normalization rules to the prompt ("Convert currency descriptions to numeric: '$2.5 million' → 2500000"), the same error stops appearing in new documents. What does this demonstrate?

- **A)** Format normalization rules alone are sufficient; retries are unnecessary.
- **B)** Pydantic validation should be replaced with Claude-based validation.
- **C)** Retries always fix format errors eventually.
- **D)** The most effective approach combines validation-retry for immediate correction AND prompt refinement to prevent the error category from recurring, reducing iterative resubmission costs.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Validation-retry for immediate correction + prompt refinement to prevent the error category from recurring. Both mechanisms together optimize quality and cost.

**Why the others are wrong:**

- **A:** Rules alone don't handle edge cases.
- **C:** Retries alone fix individual cases but not the pattern.

</details>

---

#### C317

*TS 4.4 · Validation, retry & feedback loops*

Your extraction of `total_contract_value` works by extracting both the stated total and independently summing all line items. In 5% of documents, the stated total and calculated sum differ. Your system currently discards these documents. What's a better approach?

- **A)** Average the two values.
- **B)** Always use the calculated sum — it's mathematically correct.
- **C)** Always use the stated total — it's what the original document says.
- **D)** Design the schema to extract both `stated_total` and `calculated_total`, add `conflict_detected: boolean`, and route conflicts to human review with both values and the discrepancy amount.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Extract both values, flag conflict, route to human review. Preserving both with a conflict flag enables informed human decisions.

**Why the others are wrong:**

- **A:** Averaging is nonsensical for financial data.
- **B:** Always using one value risks errors.
- **C:** Always using one value risks errors.

</details>

---

#### C318

*TS 4.5 · Batch processing*

**Situation:** A developer is submitting a batch job using the Message Batches API and wants to track which response corresponds to which input request.

What field should be included in each request within a batch to correlate responses back to their originating inputs?

- **A)** correlation_token
- **B)** batch_index
- **C)** request_id
- **D)** custom_id

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

The Message Batches API uses a `custom_id` field on each individual request. When results are returned, each result carries the same `custom_id`, allowing you to map responses back to the exact input that produced them, regardless of ordering or partial failures.

References:
- https://docs.anthropic.com/en/api/creating-message-batches

**Why the others are wrong:**

- **A:** `correlation_token` is not part of the API contract; it is an invented field name that would have no effect.
- **B:** `batch_index` is not a real field; the API does not expose positional indexing for correlation.
- **C:** `request_id` is not a field defined by the Message Batches API; using it would cause the request to be rejected or ignored.

</details>

---

#### C319

*TS 4.5 · Batch processing*

**Situation:** A startup is evaluating the cost impact of switching a high-volume document summarization pipeline from synchronous API calls to batch processing.

What cost benefit does the Message Batches API provide compared to real-time synchronous API calls?

- **A)** 10% discount on input tokens only
- **B)** Free processing for batches under 1,000 requests
- **C)** 25% discount on all tokens
- **D)** 50% discount on all tokens

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

The Message Batches API offers a 50% cost discount compared to standard synchronous API pricing. This discount applies to the overall token cost, making batch processing significantly more economical for workloads that can tolerate asynchronous, non-real-time processing.

References:
- https://docs.anthropic.com/en/api/creating-message-batches
- https://www.anthropic.com/pricing

**Why the others are wrong:**

- **A:** A 10% discount applies only to input tokens is not the correct model; the batch discount is 50% across all tokens.
- **B:** There is no free-processing tier for small batches; the 50% discount applies uniformly regardless of batch size.
- **C:** 25% is not the documented discount rate; the actual discount is 50%.

</details>

---

#### C320

*TS 4.5 · Batch processing*

**Situation:** An engineer needs to decide between the real-time Messages API and the Message Batches API for a live customer support chatbot that must respond within two seconds.

Why is the Message Batches API unsuitable for a live customer support chatbot requiring sub-second responses?

- **A)** The Batches API has a 24-hour processing window and is asynchronous, not real-time
- **B)** The Batches API is limited to text-only inputs and cannot handle customer queries
- **C)** The Batches API charges higher per-token rates for interactive sessions
- **D)** The Batches API does not support multi-turn conversations

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

The Message Batches API is designed for asynchronous, latency-tolerant workloads. Batches are processed within a 24-hour window, meaning results are not returned immediately. A live chatbot requiring responses within two seconds must use the synchronous Messages API instead.

References:
- https://docs.anthropic.com/en/api/creating-message-batches

**Why the others are wrong:**

- **B:** The Batches API supports the same input modalities as the Messages API; text-only is not a restriction.
- **C:** The Batches API is cheaper, not more expensive; this is the opposite of reality.
- **D:** Multi-turn conversation support is not the limiting factor; the Batches API can technically handle conversation-shaped messages, but the async nature makes it unfit for real-time chat.

</details>

---

#### C321

*TS 4.5 · Batch processing*

**Situation:** A machine learning team uses batch processing to generate training labels for a dataset of 500,000 text samples. After retrieving batch results, they find that 12,000 samples (2.4%) received error responses due to content policy triggers on certain edge-case inputs. The team must decide how to handle these failures without corrupting the training dataset.

What is the most architecturally sound approach for handling the 12,000 errored samples while maintaining dataset integrity?

- **A)** Use `custom_id` to retrieve the original inputs for errored samples, apply input sanitization or prompt restructuring, and resubmit as a new batch; track which samples were reprocessed with modified prompts in the audit log
- **B)** Assign a default neutral label to all 12,000 errored samples to preserve dataset size
- **C)** Resubmit the entire 500,000-sample batch to ensure all items are processed with identical prompts
- **D)** Exclude all 12,000 errored samples from the training dataset permanently to avoid noise

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

The correct approach uses `custom_id` to identify the 12,000 failed inputs precisely, then investigates the failure cause (content policy triggers). Prompt restructuring can often resolve policy rejections without altering the semantic content of the labeling task. Resubmitting only the failed subset in a new batch minimizes cost and processing time. Tracking reprocessed samples maintains dataset provenance integrity.

References:
- https://docs.anthropic.com/en/api/creating-message-batches
- https://docs.anthropic.com/en/api/retrieving-message-batch-results

**Why the others are wrong:**

- **B:** Assigning arbitrary default labels to policy-rejected samples introduces label noise and misrepresents the actual content of those samples, which degrades model quality.
- **C:** Resubmitting all 500,000 samples wastes significant cost and time re-processing 488,000 already-successful completions; batch results are per-request and the successful ones need not be rerun.
- **D:** Permanently excluding 12,000 samples creates a systematic bias in the training dataset by removing edge cases that triggered policy filters; these may be the most informative samples for robust model training.

</details>

---

#### C322

*TS 4.5 · Batch processing*

**Situation:** An architect is designing a hybrid inference pipeline for an e-commerce platform. Product page generation requires real-time generation (sub-500ms) for live user browsing. Nightly catalog enrichment (SEO descriptions, keyword extraction, attribute tagging) for 200,000 products requires completion by 6 AM. The platform also runs real-time recommendation explanations triggered by user clicks. The team wants to minimize total API spend.

Which routing strategy correctly assigns workloads to the appropriate API tier to minimize cost while meeting all latency requirements?

- **A)** Route all three workloads through the real-time API and use aggressive caching to offset the higher per-token cost
- **B)** Route nightly catalog enrichment and recommendation explanations to the Batches API; route only product page generation to the real-time API
- **C)** Route all three workloads through the Message Batches API to maximize the 50% discount
- **D)** Route product page generation and recommendation explanations to the real-time API; route nightly catalog enrichment to the Message Batches API split into 20 batches of 10,000

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Product page generation (sub-500ms, live user) and recommendation explanations (triggered by user clicks, inherently real-time) both require synchronous API calls. Nightly catalog enrichment (200,000 products, 6 AM deadline from a presumably late-night start) has high latency tolerance and maps perfectly to the Batches API. 200,000 items require 20 batches of 10,000. The 50% discount on the highest-volume workload (catalog enrichment) maximizes cost savings without compromising user-facing latency.

References:
- https://docs.anthropic.com/en/api/creating-message-batches
- https://www.anthropic.com/pricing

**Why the others are wrong:**

- **A:** Using the real-time API for 200,000 nightly catalog enrichment operations eliminates the 50% batch discount on the highest-volume workload, significantly increasing costs without any latency benefit since users are not waiting for those results.
- **B:** Recommendation explanations are triggered by individual user clicks and require immediate responses; routing them to the batch API would mean users receive no explanation during their browsing session, breaking the user experience.
- **C:** Routing live user browsing and real-time recommendation explanations through the Batches API would result in responses arriving hours later; both workloads have strict sub-second latency requirements that batch processing cannot meet.

</details>

---

#### C323

*TS 4.5 · Batch processing*

**Situation:** A healthcare analytics company processes de-identified patient discharge summaries to extract structured clinical codes. They have 30,000 summaries ready each morning and need final coded results by 5 PM the same day. Their current synchronous pipeline fails intermittently under load, and the team wants to redesign it using the Message Batches API. A strict requirement is that every summary must be traceable to its coded result for HIPAA compliance purposes.

Which design decision is most critical for ensuring HIPAA-compliant traceability while correctly leveraging the Message Batches API?

- **A)** Use sequential integers as `custom_id` values and rely on submission order to reconstruct the mapping after results are returned
- **B)** Submit all 30,000 summaries in 3 batches of 10,000 with no `custom_id` and match results by response content comparison
- **C)** Store the full patient summary content inside the `custom_id` field for direct traceability
- **D)** Assign each summary a pseudonymous internal document ID as the `custom_id`; maintain a separate secure mapping table linking document IDs to patient records outside the API payload

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

HIPAA compliance requires minimizing PHI exposure in external systems. Using a pseudonymous document ID as `custom_id` keeps PHI out of the API payload while still enabling precise traceability through a secure internal mapping table. This satisfies both the technical requirement (correlating results to inputs via `custom_id`) and the compliance requirement (no PHI transmitted as metadata to the API).

References:
- https://docs.anthropic.com/en/api/creating-message-batches
- https://docs.anthropic.com/en/api/retrieving-message-batch-results

**Why the others are wrong:**

- **A:** Relying on submission order is fragile; individual requests within a batch can fail independently, and the API does not guarantee result ordering matches submission ordering. This approach would break traceability whenever any request fails.
- **B:** Matching results by content comparison is computationally expensive, error-prone for similar summaries, and does not provide a reliable audit trail; it also fails entirely when errors occur since error responses contain no summary content.
- **C:** Embedding patient summary content or direct patient identifiers in `custom_id` would transmit PHI as metadata in API requests and responses, creating a HIPAA compliance violation and expanding the PHI footprint unnecessarily.

</details>

---

#### C324

*TS 4.5 · Batch processing*

**Situation:** A data engineering team submits a batch of 8,000 document classification requests. After the batch completes, they discover that 200 requests have a status of `errored` while 7,800 succeeded.

What is the correct strategy for handling this partial-failure result from the Message Batches API?

- **A)** Use the `custom_id` on errored results to identify failed inputs, then resubmit only those 200 in a new batch
- **B)** Contact Anthropic support to trigger automatic retry of failed requests within the original batch
- **C)** Discard all 8,000 results and resubmit the entire batch to guarantee consistency
- **D)** Treat all 8,000 results as failed because a batch is atomic and partial success is invalid

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Individual requests in a batch can fail independently; the batch is not atomic. Each result carries the original `custom_id`, so you can filter for errored results, retrieve the original inputs by matching `custom_id`, and resubmit only the failed subset in a new batch. This avoids re-running successful work and keeps costs low.

References:
- https://docs.anthropic.com/en/api/creating-message-batches
- https://docs.anthropic.com/en/api/retrieving-message-batch-results

**Why the others are wrong:**

- **B:** There is no automatic retry mechanism within an existing batch; you must initiate a new batch for failed requests.
- **C:** Discarding all results wastes the 7,800 successful completions and incurs unnecessary re-processing cost; batch results are per-request, not all-or-nothing.
- **D:** Batches are explicitly not atomic; the API design intentionally allows partial success, and treating partial success as full failure is incorrect.

</details>

---

#### C325

*TS 4.5 · Batch processing*

**Situation:** A compliance team generates audit reports from large policy documents every weekend. The reports are not needed until Monday morning. The team is currently using the synchronous Messages API with a rate-limited retry loop.

Which architectural change would best optimize this workload's cost and operational complexity?

- **A)** Deploy a dedicated on-premises LLM to eliminate API costs entirely
- **B)** Increase the synchronous API concurrency to saturate rate limits faster
- **C)** Switch to the Message Batches API to leverage the 50% cost discount and remove the need for a retry loop
- **D)** Pre-cache all policy documents in a vector store and use retrieval-augmented generation to reduce token usage

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Weekend batch report generation has high latency tolerance (results needed Monday morning), making it an ideal fit for the Message Batches API. Switching eliminates the complex rate-limited retry loop, provides a 50% cost discount, and fits comfortably within the 24-hour processing window.

References:
- https://docs.anthropic.com/en/api/creating-message-batches
- https://www.anthropic.com/pricing

**Why the others are wrong:**

- **A:** On-premises LLM deployment involves substantial capital expense, operational overhead, and model capability trade-offs that far outweigh the cost savings from batch pricing.
- **B:** Increasing synchronous concurrency increases cost and operational complexity; it does not address the fundamental inefficiency of using a real-time API for a latency-tolerant workload.
- **D:** RAG can reduce token usage but does not address the core architectural mismatch of using real-time APIs for a batch workload; it also adds significant implementation complexity.

</details>

---

#### C326

*TS 4.5 · Batch processing*

**Situation:** A developer polls a batch status endpoint every 30 seconds to check whether their 10,000-request batch has finished. Batch processing takes around 4 hours on average.

What is the most efficient approach to determine when a batch job has completed, given the 24-hour processing window?

- **A)** Poll every 30 seconds throughout the entire processing window
- **B)** Always wait the full 24 hours before attempting to retrieve results
- **C)** Submit the same batch twice so whichever finishes first can be used
- **D)** Poll with exponential backoff starting at a few minutes, or use a webhook/callback mechanism if supported

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Aggressive polling every 30 seconds wastes API quota and compute for a workload that typically takes hours. The recommended approach is exponential backoff (e.g., check after 5 minutes, then 10, then 20) to reduce unnecessary requests while still detecting completion promptly. If the platform supports event-driven callbacks, those are even more efficient.

References:
- https://docs.anthropic.com/en/api/retrieving-message-batches
- https://docs.anthropic.com/en/api/creating-message-batches

**Why the others are wrong:**

- **A:** Polling every 30 seconds over a 4-hour window generates hundreds of unnecessary status checks, consuming rate limit quota and adding operational noise without benefit.
- **B:** Waiting the full 24 hours regardless of actual completion introduces unnecessary delay; most batches finish well before the maximum window.
- **C:** Submitting duplicate batches doubles cost and produces redundant results; it does not improve detection latency and creates reconciliation complexity.

</details>

---

#### C327

*TS 4.5 · Batch processing*

**Situation:** An architect is designing a system that classifies customer feedback into 15 sentiment categories. The classification pipeline runs hourly on roughly 3,000 new feedback items. The business requires results within 45 minutes of collection.

Should this workload use the Message Batches API or the real-time Messages API, and why?

- **A)** Real-time API, because 45 minutes is insufficient for the 24-hour batch window
- **B)** Real-time API for the first 1,000 items and Batches API for the remaining 2,000
- **C)** Batches API with priority processing enabled to guarantee 45-minute SLA
- **D)** Batches API, because 3,000 items fits within the 10,000-request limit

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

The Message Batches API has a 24-hour processing window, meaning results could arrive anywhere within that window. A business SLA of 45 minutes cannot be reliably met with batch processing. The real-time Messages API must be used here, even though it costs more, because the latency requirement disqualifies asynchronous batch processing.

References:
- https://docs.anthropic.com/en/api/creating-message-batches

**Why the others are wrong:**

- **B:** Splitting requests between real-time and batch APIs adds architectural complexity and still leaves 2,000 items potentially delayed beyond 45 minutes; it does not solve the SLA problem.
- **C:** There is no priority processing tier in the Message Batches API that guarantees a specific sub-24-hour SLA; the 24-hour window is the only commitment.
- **D:** Fitting within the 10,000-request limit is necessary but not sufficient; latency requirements must also be met, and 45 minutes cannot be guaranteed by the batch API.

</details>

---

#### C328

*TS 4.5 · Batch processing*

Your manager wants to batch-process 10,000 contracts using the Message Batches API. What's the correct approach before submitting the full batch?

- **A)** Submit all 10,000 immediately to take advantage of the 50% cost savings.
- **B)** Test the extraction prompt on a sample of 50 documents first, analyze error patterns, refine the prompt, then submit the full batch.
- **C)** Split into 100 batches of 100 documents each, submitting them sequentially.
- **D)** Run the first 100 synchronously, then switch to batch for the remaining 9,900.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Test on a sample first, refine the prompt, then submit the full batch. Sequential batches (C) and hybrid approaches (D) don't address prompt quality.

**Why the others are wrong:**

- **A:** Submitting all immediately risks costly failures.

</details>

---

#### C329

*TS 4.5 · Batch processing*

A batch processing job for 500 legal documents completes. 480 succeed, but 20 fail — all with `context_limit_exceeded` errors. Each failed document is 150+ pages. What's the recovery strategy?

- **A)** Resubmit the 20 failed documents with a higher `max_tokens` parameter.
- **B)** Identify the 20 failures by `custom_id`, chunk each oversized document into smaller sections, and resubmit only the failed documents.
- **C)** Resubmit the entire batch of 500 documents with a more concise extraction prompt.
- **D)** Switch the 20 failed documents to synchronous processing, which has a larger context window.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Identify failures by `custom_id`, chunk oversized documents, resubmit only failures.

**Why the others are wrong:**

- **A:** `max_tokens` isn't the cause of context limit errors.
- **C:** Don't resubmit all 500.
- **D:** Synchronous API doesn't have a larger context window.

</details>

---

#### C330

*TS 4.5 · Batch processing*

Your team's overnight compliance report is run using the synchronous Claude API. Each month, it processes 5,000 documents at significant cost. No one reads the results until the next morning. How could you optimize this?

- **A)** Switch to the Message Batches API for 50% cost savings. The 24-hour processing window is acceptable since no one reads results until morning.
- **B)** Process documents in parallel streams using multiple synchronous API calls.
- **C)** Reduce the number of documents processed by sampling.
- **D)** Keep using synchronous API — consistency is more important than cost savings.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Non-blocking overnight workload with no latency requirement = ideal for Batch API (50% savings).

**Why the others are wrong:**

- **B:** Parallel streams cost the same.
- **C:** Sampling reduces accuracy.
- **D:** Synchronous wastes money.

</details>

---

#### C331

*TS 4.5 · Batch processing*

Your batch job processes 1,000 contracts overnight using the Batches API. 950 succeed. Of the 50 failures, 30 are `context_limit_exceeded` (very long contracts), 15 are extraction validation failures, and 5 are transient API errors. What's the correct recovery strategy?

- **A)** Resubmit only the 50 failures with the same prompt and hope for different results.
- **B)** Handle each failure category separately: chunk the 30 oversized contracts, refine the prompt for the 15 validation failures (test on a sample first), and simply resubmit the 5 transient errors. Identify all by `custom_id`.
- **C)** Switch all 50 failures to synchronous processing.
- **D)** Resubmit all 1,000 contracts with a more concise prompt.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Handle each failure category differently: chunk oversized, refine prompt for validation errors (test on sample), resubmit transient errors. Use `custom_id` to identify each.

**Why the others are wrong:**

- **A:** Resubmitting all is wasteful.
- **C:** Synchronous doesn't fix the underlying issues.
- **D:** Resubmitting all is wasteful.

</details>

---

#### C332

*TS 4.5 · Batch processing*

Your team processes 5,000 documents weekly. A pre-merge CI check blocks developers until compliance documents are extracted and validated. A weekly compliance audit extracts data from historical documents for regulatory review. Your manager proposes using the Batch API for both workflows. What's correct?

- **A)** Use synchronous API for the pre-merge check (developers are blocking) and Batch API for the weekly audit (latency-tolerant, cost-sensitive).
- **B)** Use Batch API for both — the 50% savings applies to both workflows.
- **C)** Use Batch API for the pre-merge check with aggressive polling for fast results.
- **D)** Use synchronous API for both to guarantee consistent processing.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Blocking pre-merge = synchronous (developers wait). Weekly audit = batch (latency-tolerant, cost-sensitive).

**Why the others are wrong:**

- **B:** Batch for blocking risks 24h delays for developers.
- **C:** Batch for blocking risks 24h delays for developers.
- **D:** Synchronous for everything wastes 50% on the audit.

</details>

---

#### C333

*TS 4.6 · Multi-instance & multi-pass review*

**Situation:** A developer asks why a single Claude instance sometimes fails to catch errors in code it just wrote.

What is the primary reason a model reviewing its own output tends to miss errors?

- **A)** The model cannot read code it has already written in the same session
- **B)** The model carries the same assumptions and reasoning biases from generation into review, making it likely to agree with its own output
- **C)** The model runs out of context window space during review
- **D)** Single-instance review always produces lower quality than human review

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Self-review suffers from confirmation bias: the model that generated the output used certain assumptions and reasoning paths. When it reviews the same output, it tends to evaluate it through the same lens, causing it to overlook flaws it introduced. This is the core self-review limitation that motivates multi-instance architectures.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems

**Why the others are wrong:**

- **A:** Models can read their own prior output within a session; the limitation is cognitive bias, not memory access.
- **C:** Context window exhaustion is a separate concern and is not the core reason self-review misses errors in typical scenarios.
- **D:** Single-instance review can sometimes match human review quality; the issue is specifically the bias introduced by reviewing one's own output.

</details>

---

#### C334

*TS 4.6 · Multi-instance & multi-pass review*

**Situation:** An architect is explaining independent review instances to a junior engineer.

What is the defining characteristic of an independent review instance in a multi-instance review architecture?

- **A)** It always uses a different model family than the generator
- **B)** It starts with a fresh context and has no access to the reasoning or intermediate steps of the generating instance
- **C)** It uses a smaller, faster model to reduce latency
- **D)** It runs on a different physical server than the generation instance

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Independence in review architectures means the reviewing instance begins with a fresh context — it sees only the artifact to be reviewed, not the chain of thought or intermediate decisions that produced it. This prevents the reviewer from inheriting the generator's biases and assumptions.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems

**Why the others are wrong:**

- **A:** Using the same model family is acceptable as long as the reviewing instance has a fresh context; model diversity can add value but is not the defining criterion.
- **C:** Model size is a cost/latency trade-off decision, not what defines independence; a small model reviewing with shared context is not independent.
- **D:** Physical server separation is irrelevant; independence is a property of context and reasoning state, not infrastructure placement.

</details>

---

#### C335

*TS 4.6 · Multi-instance & multi-pass review*

**Situation:** A team is designing a code review pipeline for a large monorepo with hundreds of files changed per pull request.

In a two-phase review architecture, what is the primary purpose of the Phase 1 per-file review pass?

- **A)** To evaluate each file in isolation for correctness, style, and local logic errors
- **B)** To generate documentation for each file before the cross-file pass
- **C)** To assign severity scores that Phase 2 uses to decide which files to skip
- **D)** To check that all files are syntactically valid before running tests

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Phase 1 (per-file pass) focuses on individual file quality — logic correctness, naming, error handling, and style — without needing knowledge of the full system. This localised focus keeps context manageable and allows parallel execution across files.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems

**Why the others are wrong:**

- **B:** Documentation generation is a separate concern; Phase 1 is an analytical review pass, not a doc-generation step.
- **C:** Phase 1 findings inform Phase 2 but do not determine which files to skip; Phase 2 checks cross-cutting concerns regardless of Phase 1 severity scores.
- **D:** Syntax validation is typically handled by linters or compilers before the review pipeline; Phase 1 addresses deeper code quality concerns.

</details>

---

#### C336

*TS 4.6 · Multi-instance & multi-pass review*

**Situation:** An architect is designing a review system for auto-generated infrastructure-as-code (IaC) templates. The templates are generated in 50-file batches. The team must choose between: Option X — one review instance that reads all 50 files in a single context, and Option Y — 50 parallel per-file instances followed by a cross-file consistency instance. Budget and time are constrained.

Which option should the architect choose, and what is the most accurate reasoning?

- **A)** Option Y, because parallel per-file instances are always faster and cheaper than a single large-context review
- **B)** Option Y, because per-file instances each operate at optimal context depth for local issues, and the cross-file instance receives structured summaries rather than 50 raw files, keeping both passes within effective context utilisation — while also parallelising Phase 1 to reduce wall-clock time
- **C)** Option X, because a single large context allows the reviewer to see all interdependencies simultaneously without coordination overhead
- **D)** Option X, because multi-instance architectures always produce more false positives than single-instance review

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Option Y is superior for this workload. A single instance reviewing 50 IaC files faces context dilution — attention quality degrades as context grows, and the model may miss local issues buried in a large prompt. Option Y keeps Phase 1 reviewers focused on one file at optimum context depth. Phase 2 receives compact structured summaries (not raw files), allowing it to reason about cross-file consistency without context overload. Phase 1 runs in parallel, reducing wall-clock time despite the additional coordination. The quality and latency benefits justify the modest coordination overhead.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems

**Why the others are wrong:**

- **A:** Option Y is not always cheaper in total tokens — it may use more tokens overall — but it is better quality; cost and speed are secondary arguments; the primary justification is context quality per review pass.
- **C:** While a single large context can see all files, attention quality degrades with context length, making it less reliable for catching local issues across 50 files; interdependency detection does not require all files in one prompt if Phase 2 receives structured cross-file summaries.
- **D:** Multi-instance architectures do not inherently produce more false positives; false positive rates depend on reviewer calibration and prompt design, not instance count.

</details>

---

#### C337

*TS 4.6 · Multi-instance & multi-pass review*

**Situation:** An architect is designing a review pipeline for a microservices codebase. Phase 1 reviewers examine individual services; Phase 2 receives the Phase 1 findings and the full set of changed files.

What class of defect is Phase 2 (cross-file pass) specifically designed to detect that Phase 1 cannot reliably find?

- **A)** Missing null checks inside a single service
- **B)** Off-by-one errors within a single function
- **C)** Inconsistent API contracts, data model mismatches, and violated invariants that span multiple files or services
- **D)** Unused imports and dead code within individual files

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Cross-file (Phase 2) review is specifically designed to catch systemic issues that only become visible when multiple files are considered together: inconsistent interface definitions, shared data model drift, violated cross-cutting invariants, and integration mismatches. These are invisible to a per-file reviewer with localised context.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems

**Why the others are wrong:**

- **A:** Missing null checks are a local code quality concern addressable in Phase 1.
- **B:** Off-by-one errors are local to a single function and are well-suited to Phase 1 per-file review.
- **D:** Unused imports and dead code are per-file concerns handled by Phase 1 or static analysis tools, not the cross-file pass.

</details>

---

#### C338

*TS 4.6 · Multi-instance & multi-pass review*

**Situation:** A financial services company is evaluating whether to use single-instance or multi-instance review for contract clause analysis. Contracts can be 200 pages long and errors carry significant legal liability.

What combination of factors most strongly justifies choosing multi-instance review in this scenario?

- **A)** The contracts are long and the team wants to reduce API latency
- **B)** The company wants to reduce total token costs by distributing work across instances
- **C)** High stakes (legal liability), complex artifacts (200 pages), and the need for cross-validation to reduce the probability of a missed clause
- **D)** Multi-instance review is always required when documents exceed 50 pages

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

The decision to use multi-instance review is driven by the combination of high consequence of error (legal liability) and artifact complexity (200-page contracts with many interdependent clauses). These factors make the additional cost of multiple review passes worthwhile because a single missed clause could have severe downstream consequences.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems

**Why the others are wrong:**

- **A:** Multi-instance review increases total compute and typically increases latency, not reduces it; latency is not a justification for multi-instance review.
- **B:** Multi-instance review increases token costs overall; it is justified by quality and risk reduction, not cost savings.
- **D:** There is no universal page-count threshold; the decision is based on stakes, complexity, and error cost, not document length alone.

</details>

---

#### C339

*TS 4.6 · Multi-instance & multi-pass review*

The same Claude Code session that generated a feature's code is asked to review its own changes. The review finds no issues despite obvious problems that an independent reviewer would catch. Why?

- **A)** The model's temperature is too low for review tasks, making it overly agreeable.
- **B)** The model retains reasoning context from generation, making it biased toward confirming its own decisions. Use an independent review instance without prior reasoning context.
- **C)** The system prompt doesn't explicitly instruct the model to be critical of its own code.
- **D)** Code generation and code review require different model configurations (temperature, max_tokens).

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Self-review bias: the model retains reasoning context from generation. Use an independent instance. Temperature (A), explicit instructions (C), and different configs (D) don't eliminate the bias from retained context.

</details>

---

#### C340

*TS 4.6 · Multi-instance & multi-pass review*

A pull request modifies the authentication module across 8 files. Your review prompt uses few-shot examples that demonstrate reviewing a single-file utility change. The review misses cross-file security implications (a middleware bypassing auth checks due to incorrect import). What architectural change addresses this?

- **A)** Split into per-file local analysis passes plus a separate cross-file integration pass that specifically examines data flow, import chains, and authentication pathways between files.
- **B)** Use a larger context window model.
- **C)** Route all authentication-related PRs to human reviewers instead.
- **D)** Add more few-shot examples covering multi-file reviews.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Multi-pass review: per-file local analysis + cross-file integration pass.

**Why the others are wrong:**

- **B:** Larger context doesn't fix attention dilution.
- **C:** Human-only review doesn't scale.
- **D:** Single-file examples don't demonstrate cross-file analysis.

</details>

---

#### C341

*TS 4.6 · Multi-instance & multi-pass review*

Your CI pipeline reviews a PR and produces findings. The same session is then asked to review its own findings for accuracy. It marks all findings as accurate. An independent human review reveals 3 of 12 findings are false positives. Why?

- **A)** The model needs explicit instructions to be self-critical.
- **B)** The model retains reasoning context from the initial review, biasing it toward confirming its own decisions. Use an independent Claude instance (separate session) for review validation.
- **C)** The model's temperature should be higher for the self-review pass.
- **D)** The original review prompt is too lenient, so the self-review inherits the same leniency.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Self-review bias — the model retains reasoning context. Use an independent instance. Temperature (C) and prompt leniency (D) aren't the root cause.

**Why the others are wrong:**

- **A:** Instructions to be self-critical don't overcome retained context bias.

</details>

---

#### C342

*TS 4.6 · Multi-instance & multi-pass review*

Your agent recommends products and then reviews its own recommendations for quality. It marks all recommendations as appropriate. An independent review reveals 4 of 15 recommendations don't match stated customer preferences. Why?

- **A)** The model needs explicit instructions to be self-critical.
- **B)** The model retains reasoning context from the initial recommendation, biasing it toward confirming its own decisions. Use an independent Claude instance (separate session) for recommendation validation.
- **C)** The model's temperature should be higher for the self-review pass.
- **D)** The original recommendation prompt is too lenient, so the self-review inherits the same leniency.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Self-review bias — the model retains reasoning context. Use an independent instance. Temperature (C) and prompt leniency (D) aren't the root cause.

**Why the others are wrong:**

- **A:** Instructions don't overcome retained context bias.

</details>

---

#### C343

*TS 4.6 · Multi-instance & multi-pass review*

You need the agent to both recommend products AND validate that recommendations comply with inventory and pricing rules. A colleague suggests using a single session for both to "share context." What's the issue?

- **A)** There's no issue — shared context improves both tasks.
- **B)** The combined context might exceed the model's token limit.
- **C)** Using the same session for recommendation and validation introduces self-review bias. Use separate, independent sessions for generation and validation.
- **D)** A single session can't run two different prompts.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Self-review bias affects both recommendation and validation in the same session. Use separate, independent sessions.

**Why the others are wrong:**

- **A:** Context sharing is the problem, not a benefit.
- **B:** Token limits aren't the primary concern.

</details>

---

<a id="part-c-domain-5"></a>

### Part C · Domain 5 · Context Management & Reliability (15%)

#### C344

*TS 5.1 · Context preservation*

**Situation:** An engineer inherits an agentic loop codebase. The loop works but intermittently produces responses where the model seems to 'forget' its earlier instructions or repeat tool calls it already made. Reviewing the code, the engineer discovers the message history is built by keeping only the last 10 messages to avoid context limits, which sometimes truncates the initial system instructions and early tool interactions.

What is the most architecturally sound strategy for managing message history to prevent both context overflow and loss of critical early context?

- **A)** Replace the entire message history with a single summarization prompt after every 5 turns, discarding all prior messages.
- **B)** Keep the full message history always; if it exceeds the context limit, increase `max_tokens` to compensate.
- **C)** Preserve the system prompt and the first 1–2 user/assistant turns verbatim, use a compact structured summary to represent completed intermediate work, and keep the most recent K turns in full to preserve immediate context.
- **D)** Keep the system prompt and the most recent N user/assistant turns; discard older turns from the middle of the conversation.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

The architecture that best balances context constraints with fidelity preserves three zones: (1) the immutable founding context — system prompt and initial instructions, which define the agent's identity and constraints; (2) a compact summary of completed work, replacing the verbose middle of the conversation with a high-signal digest; and (3) the recent verbatim turns, which provide the immediate context the model needs for coherent continuation. This avoids both the truncation problem (losing the system prompt) and the context-overflow problem.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/implement-tool-use
- https://docs.anthropic.com/en/docs/build-with-claude/agents/build-effective-agents

**Why the others are wrong:**

- **A:** Replacing the entire history with a summarization prompt every 5 turns is too aggressive — each summarization loses detail, and chained summarizations compound the information loss, causing the model to drift from its original instructions over long sessions.
- **B:** `max_tokens` controls the output budget, not the input context window. The context window limit is fixed per model — increasing `max_tokens` does not expand how much history the model can attend to.
- **D:** Keeping only the last N turns while discarding only the middle still risks losing the system prompt if the conversation is long and N is small. More critically, this approach discards intermediate decisions and tool results that may be relevant to the current task.

</details>

---

#### C345

*TS 5.1 · Context preservation*

**Situation:** A developer is building a Claude-powered assistant and notices that important user preferences mentioned at the very beginning of a long conversation are being ignored later in the session.

What is the PRIMARY architectural explanation for why Claude loses track of information placed in the middle of a long context window?

- **A)** Claude's tokenizer strips repeated content after the first occurrence to save space.
- **B)** Claude automatically deletes content older than 10 turns to stay within the context window.
- **C)** Claude applies equal attention to every token in the context, so nothing is ever lost.
- **D)** The 'lost in the middle' phenomenon means models attend less reliably to content far from the start or end of the context.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Research into large language models has consistently shown that content positioned in the middle of a long context receives less reliable attention than content near the beginning or end. This 'lost in the middle' effect means that critical information should be anchored at the start or end of the context rather than buried in the middle of a long conversation.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/context-windows
- https://docs.anthropic.com/en/docs/build-with-claude/conversation-management

**Why the others are wrong:**

- **A:** The tokenizer does not deduplicate content; every token is encoded and passed to the model regardless of repetition.
- **B:** Claude does not have an automatic deletion mechanism; all tokens within the active context window are present, but attention distribution is non-uniform.
- **C:** Attention is not uniform across positions; empirical studies show middle-of-context recall degrades significantly as context length grows.

</details>

---

#### C346

*TS 5.1 · Context preservation*

**Situation:** An architect is designing a system prompt for a long-running coding assistant. The system prompt includes security constraints, the user's preferred programming language, and a lengthy history of past decisions.

Where should the most safety-critical security constraints be placed within the context to ensure Claude reliably adheres to them?

- **A)** At the very start or very end of the system prompt, not buried in the middle.
- **B)** Constraints do not need special placement because Claude attends equally to all positions.
- **C)** Embedded within a large block of past decisions in the middle of the system prompt.
- **D)** In the first user turn rather than the system prompt.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Due to the 'lost in the middle' phenomenon, content placed in the middle of long contexts receives less reliable attention. Safety-critical constraints should be anchored at the start or end of the system prompt so that they remain in the high-attention zones regardless of how long the conversation grows.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/context-windows
- https://docs.anthropic.com/en/docs/build-with-claude/system-prompts

**Why the others are wrong:**

- **B:** This contradicts established research; attention is not uniformly distributed across token positions, especially as context length increases.
- **C:** Embedding constraints inside a large block of historical decisions in the middle is precisely the anti-pattern the 'lost in the middle' effect warns against; those constraints are most likely to be under-attended.
- **D:** System prompt is the correct location for persistent constraints; placing them in the first user turn exposes them to the same middle-of-context risk as the conversation grows and more turns are prepended or appended.

</details>

---

#### C347

*TS 5.1 · Context preservation*

**Situation:** A tool used in an agentic pipeline returns a 50,000-token JSON dump of a database query result. The agent only needs three specific fields from that output.

What is the recommended approach for handling this large tool output before passing it back into Claude's context?

- **A)** Pass the full 50,000-token output unchanged so Claude can decide what is relevant.
- **B)** Trim or filter the tool output server-side to include only the fields the agent actually needs before inserting it into the context.
- **C)** Increase the model's context window by switching to a larger model tier.
- **D)** Store the output in a vector database and never include it in the context.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Large tool outputs should be trimmed before being inserted into the context. When only three fields are needed, including the full 50,000-token dump wastes context space, increases latency and cost, and pushes important earlier content further into the 'lost in the middle' zone. Server-side filtering to the relevant fields is the correct practice.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/overview
- https://docs.anthropic.com/en/docs/build-with-claude/context-windows

**Why the others are wrong:**

- **A:** Passing the full output shifts the filtering burden onto Claude at inference time, consumes significant context budget unnecessarily, and may cause critical earlier content to fall into low-attention middle positions.
- **C:** Switching to a larger context model treats a symptom rather than the cause; the output still contains 49,997 tokens of irrelevant data that should be removed regardless of model capacity.
- **D:** Vector retrieval is a valid long-term memory strategy, but categorically never including tool output in the context prevents the model from reasoning over retrieved data in the current turn.

</details>

---

#### C348

*TS 5.1 · Context preservation*

**Situation:** An architect is designing a multi-session agent that performs ongoing scientific literature review. Each session ends with a progressive summary passed to the next session. After six sessions, the team discovers that an important methodological caveat noted in session one (that one key study used a non-standard sample population) has disappeared from the summary and is now absent from the agent's reasoning.

Which combination of techniques would BEST prevent critical caveats from being lost across progressive summarization rounds?

- **A)** Run /compact after every turn within each session to minimize the content that needs to be summarized at session end.
- **B)** Switch to a model with a larger context window so that progressive summarization is never needed.
- **C)** Maintain a separate structured 'critical facts' register outside the progressive summary that is explicitly preserved verbatim across sessions, and instruct the summarization step to never omit entries in that register.
- **D)** Use a higher temperature when generating session summaries so the model explores more diverse retention strategies.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

The fundamental weakness of pure progressive summarization is that each compression round makes a salience judgment, and low-frequency but high-importance caveats are systematically under-prioritized. The architectural solution is to maintain a separate verbatim 'critical facts' register — a structured artifact that is never compressed and is explicitly carried forward each session. The summarization step for the narrative body is instructed to check and preserve all entries in this register. This hybrid approach protects rare, critical information from progressive information loss.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/conversation-management
- https://docs.anthropic.com/en/docs/build-with-claude/context-windows

**Why the others are wrong:**

- **A:** Running /compact within each session reduces intra-session context size but does not address the inter-session summarization problem; the session-end summary still compresses the full session narrative and will still drop low-salience caveats.
- **B:** A larger context window reduces how often summarization is needed but does not eliminate the need across six or more long sessions; the architectural weakness of pure progressive summarization remains, and the caveat loss would simply be deferred rather than prevented.
- **D:** Higher temperature increases output variance but does not encode a systematic preference for retaining rare caveats; it is equally likely to introduce hallucinations as it is to retain the caveat, and is not a reliable preservation strategy.

</details>

---

#### C349

*TS 5.1 · Context preservation*

**Situation:** An operator is evaluating three context management designs for a 500-turn enterprise workflow agent: Design X uses progressive summarization every 50 turns with no pinned register. Design Y uses a pinned structured register at the start of the context and summarization only for the narrative body. Design Z starts a fresh conversation every 50 turns with a manually curated handoff document.

For a workflow where task continuity is critical, decisions accumulate over time, and rare edge-case constraints must never be lost, which design is architecturally superior and why?

- **A)** Design Z, because fresh conversations eliminate all attention distribution problems and the curated handoff ensures nothing is lost.
- **B)** All three designs are equivalent; context management strategy has no measurable effect on task performance over 500 turns.
- **C)** Design Y, because it separates compressible narrative context from non-compressible critical decisions, preserves rare constraints verbatim in the pinned register, and avoids the cumulative loss of progressive-only summarization.
- **D)** Design X, because frequent summarization keeps the context small and prevents any 'lost in the middle' effect.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Design Y is architecturally superior for this specific scenario. It solves three distinct problems simultaneously: (1) the pinned register at the start of the context ensures rare edge-case constraints are never lost to compression; (2) summarization of only the narrative body limits compression to content that can tolerate it; (3) the pinned register also keeps critical data in a high-attention zone, mitigating the 'lost in the middle' problem. Design X loses edge cases through progressive compression. Design Z depends on the quality of a manual curation step, introduces discontinuities, and risks human error in the handoff document.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/conversation-management
- https://docs.anthropic.com/en/docs/build-with-claude/context-windows
- https://docs.anthropic.com/en/docs/build-with-claude/system-prompts

**Why the others are wrong:**

- **A:** Design Z introduces manual curation as a dependency; if the operator misses a constraint in the handoff document, it is permanently lost. Fresh conversations also lose the implicit context of accumulated decisions that inform later reasoning, introducing subtle discontinuities.
- **B:** This is demonstrably false; the 'lost in the middle' phenomenon and progressive summarization information loss are empirically documented problems with measurable effects on task performance in long-horizon workflows.
- **D:** Design X's frequent summarization does keep the context small, but each compression round is a lossy operation for rare constraints. By turn 500 (ten compression cycles), edge-case constraints are highly likely to have been silently dropped.

</details>

---

#### C350

*TS 5.1 · Context preservation*

**Situation:** A senior architect must choose between three tool output handling strategies for an agent that runs 20 tool calls per session, each returning between 500 and 80,000 tokens: Strategy P — inject all tool outputs verbatim. Strategy Q — trim all outputs to 500 tokens regardless of content. Strategy R — apply content-aware trimming: extract only the fields the next reasoning step requires, preserve structure, and set a soft cap proportional to the tool's relevance score for the current task.

Which strategy best balances context efficiency, reasoning quality, and preservation of critical information, and what is the key architectural principle it embodies?

- **A)** Strategy R, because content-aware trimming applies the principle of minimum necessary context: inject precisely what the next reasoning step needs, no more, preserving signal density while preventing context bloat.
- **B)** Strategy Q, because a fixed 500-token cap guarantees predictable context usage and prevents any single tool from dominating the window.
- **C)** Strategy Q and Strategy P are equivalent in reasoning quality; only latency differs between them.
- **D)** Strategy P, because verbatim injection ensures no information is lost and Claude can filter internally.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Strategy R embodies the principle of minimum necessary context: provide exactly the information the next reasoning step requires and nothing more. This principle maximizes signal-to-noise ratio in the context, prevents high-volume tool outputs from pushing critical earlier content into the low-attention middle zone, and avoids the arbitrary truncation of Strategy Q which may cut relevant content. The relevance-score soft cap further adapts to the dynamic importance of each tool's output within the current task, rather than applying a context-agnostic fixed limit.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/overview
- https://docs.anthropic.com/en/docs/build-with-claude/context-windows
- https://docs.anthropic.com/en/docs/build-with-claude/conversation-management

**Why the others are wrong:**

- **B:** Strategy Q's fixed 500-token cap is context-agnostic and will arbitrarily truncate outputs that contain critical information beyond token 500. A tool returning a 600-token JSON where the key field is at token 550 will silently lose that field, creating hard-to-diagnose reasoning errors.
- **C:** Strategy Q (fixed 500-token truncation) and Strategy P (verbatim injection) differ significantly in reasoning quality: Q silently drops content beyond 500 tokens regardless of relevance, while P floods the context with noise. Neither is equivalent in quality to content-aware trimming.
- **D:** Strategy P assumes Claude can reliably filter internally, but verbatim injection of up to 80,000 tokens per call (20 calls = up to 1.6 million tokens) rapidly exhausts the context window and guarantees 'lost in the middle' degradation for earlier critical content. Upstream trimming is more reliable.

</details>

---

#### C351

*TS 5.1 · Context preservation*

**Situation:** A team runs a multi-session research assistant. After each session ends, the assistant produces a progressive summary of findings. In the third session, the team notices the assistant has forgotten a key constraint from session one that was only mentioned briefly in session one's summary.

Which limitation of progressive summarization most likely caused this information loss?

- **A)** Claude's summarization always prioritizes the most recent turn over all previous turns.
- **B)** Session boundaries automatically reset all prior constraints regardless of the summary strategy used.
- **C)** The summarization prompt used a temperature of 0, which prevents creative retention of edge-case details.
- **D)** Progressive summaries compress content each round, and details not reinforced in later sessions gradually drop out of the summary.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Progressive summarization involves compressing an already-compressed summary each session. Each compression round introduces information loss, and details that appear only briefly — especially constraints or edge cases — are disproportionately likely to be dropped when the model decides what is most salient. Critical constraints must be explicitly pinned or re-injected rather than relying on progressive compression to preserve them.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/conversation-management
- https://docs.anthropic.com/en/docs/build-with-claude/context-windows

**Why the others are wrong:**

- **A:** Claude does not have a hard-coded recency bias rule that always deprioritizes earlier content; the issue is the cumulative information loss inherent in repeated summarization.
- **B:** Session boundaries are an application-layer concept; Claude itself has no mechanism that automatically resets constraints. The information loss is due to summarization compression, not a session reset.
- **C:** Temperature controls output randomness for generation tasks; it does not affect what semantic content a summarization model decides to retain or discard.

</details>

---

#### C352

*TS 5.1 · Context preservation*

**Situation:** A developer building a long-running coding assistant observes that after 30 turns, Claude starts violating a coding style rule (always use snake_case for variable names) that was stated in the system prompt. The system prompt is 8,000 tokens long with the style rule buried 4,000 tokens in.

Which TWO actions, applied together, would most effectively prevent this regression?

- **A)** Move the style rule to the beginning of the system prompt AND periodically re-state it at the end of the most recent assistant turn.
- **B)** Increase the max_tokens parameter AND switch to a model with a larger context window.
- **C)** Move the style rule to the end of every user turn AND disable the system prompt entirely.
- **D)** Use /compact after each turn to keep the context short AND never include style rules in the system prompt.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

The 'lost in the middle' problem causes rules buried 4,000 tokens into an 8,000-token system prompt to receive less attention as the conversation grows. Moving the rule to the start of the system prompt places it in a high-attention zone. Periodically restating it at the end of the most recent turn keeps it in the recent high-attention zone as well. These two actions together reinforce the rule from both ends of the attention spectrum.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/system-prompts
- https://docs.anthropic.com/en/docs/build-with-claude/context-windows

**Why the others are wrong:**

- **B:** Increasing max_tokens and context window size does not address the attention distribution problem; a larger window means the middle zone is even larger and the buried rule is even less attended.
- **C:** Disabling the system prompt removes all persistent instruction infrastructure; style rules belong in the system prompt, and appending them to every user turn is a workaround that clutters user messages and is not architecturally sound.
- **D:** Running /compact after each turn is wasteful and destroys useful context prematurely; it does not solve the fundamental placement issue, and removing style rules from the system prompt is an anti-pattern.

</details>

---

#### C353

*TS 5.1 · Context preservation*

**Situation:** A team is building a customer service bot that handles hour-long sessions. They want to preserve the customer's issue description, any commitments the agent has made, and the list of troubleshooting steps already tried — without letting the context window fill up.

Which context management strategy best balances preservation of critical information with efficient use of the context window?

- **A)** Keep only the last five turns in the context and discard everything older.
- **B)** Let the conversation grow unrestricted until the model raises a context length error, then start fresh.
- **C)** Summarize the entire conversation including critical fields every five turns, discarding the original.
- **D)** Store critical fields (issue, commitments, tried steps) in a structured section at the start of the system prompt and refresh it after each turn; use /compact or summarization only for the conversational body.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Structured pinning separates critical, non-compressible data (issue, commitments, tried steps) from the conversational body that can tolerate summarization. Storing these fields in a structured block at the start of the system prompt and refreshing it ensures they remain in the high-attention zone and are never lost to progressive compression. The conversational body can then be summarized as needed without risking loss of the most critical information.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/conversation-management
- https://docs.anthropic.com/en/docs/build-with-claude/system-prompts

**Why the others are wrong:**

- **A:** Discarding all but the last five turns loses the issue description and the full list of tried steps, which are precisely the fields the team identified as critical.
- **B:** Waiting for a context length error is a reactive failure mode; it provides no graceful degradation and results in complete loss of session history.
- **C:** Summarizing critical fields like commitments every five turns introduces progressive information loss; these fields should be pinned verbatim, not compressed.

</details>

---

#### C354

*TS 5.1 · Context preservation*

**Situation:** An agentic pipeline processes a sequence of tool calls. One tool returns a full web page HTML (120,000 characters). The next step only requires the page title and the first paragraph of body text.

Beyond saving context space, what is an ADDITIONAL architectural benefit of trimming the tool output before injecting it into the context?

- **A)** Trimming increases the model's output token limit for the subsequent response.
- **B)** Trimmed outputs are automatically cached by Claude's prompt cache, while full outputs are not eligible for caching.
- **C)** Trimming forces the model to attend to the relevant signal rather than potentially fixating on irrelevant parts of a large HTML document, improving reasoning accuracy.
- **D)** Trimmed outputs bypass the safety classifier, reducing latency.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Beyond context space savings, trimming large outputs improves reasoning quality. A 120,000-character HTML document contains navigation bars, scripts, ads, metadata, and other noise. Including this noise exposes the model to irrelevant tokens that can distract attention and degrade the quality of the downstream reasoning step. Providing only the title and first paragraph gives the model a focused, high-signal input.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use/overview
- https://docs.anthropic.com/en/docs/build-with-claude/context-windows

**Why the others are wrong:**

- **A:** The output token limit for a response is controlled by the max_tokens parameter and model capability; it is not affected by the size of tool output injected into the context.
- **B:** Prompt cache eligibility is determined by whether content is in the system prompt prefix or stable conversation prefix, not by whether tool output was trimmed; trimming does not automatically confer cache eligibility.
- **D:** Safety classifiers process all content regardless of output size; trimming does not bypass or alter safety evaluation pipelines.

</details>

---

#### C355

*TS 5.1 · Context preservation*

During a long support session with multiple order lookups, the agent starts confusing details from earlier orders with the current one, citing incorrect amounts and dates. What is the root cause and fix?

- **A)** The model's temperature is too high, causing hallucinated details. Lower the temperature parameter.
- **B)** Verbose tool results are accumulating in context. Implement a `PostToolUse` hook to trim each `lookup_order` result to only the relevant fields.
- **C)** The model needs a larger context window. Upgrade to a higher-tier model.
- **D)** The agent should clear its conversation history after each resolved issue.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Verbose tool outputs accumulate and consume context. A PostToolUse hook trims each result to relevant fields, preventing confusion. Temperature (A), model size (C), and history clearing (D) don't address the root cause.

</details>

---

#### C356

*TS 5.1 · Context preservation*

Your research reports have grown too long, and stakeholders complain they can't find key findings. The coordinator currently concatenates raw subagent outputs into one document. What structural improvement addresses this?

- **A)** Have each subagent rank its findings by importance and only include the top 5.
- **B)** Place key findings summaries at the beginning of the aggregated output, with detailed results in clearly-labeled sections, to mitigate the lost-in-the-middle effect.
- **C)** Set a maximum word count for each subagent's output.
- **D)** Use `/compact` to condense the report before delivering it.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Lost-in-the-middle: place key findings at the beginning, use clear section headers.

**Why the others are wrong:**

- **A:** Top-5 only is arbitrary.
- **C:** Word limits lose information.
- **D:** `/compact` is for conversation history, not reports.

</details>

---

#### C357

*TS 5.1 · Context preservation*

Your CI review analyzes one large file (2,000 lines) and flags a function on line 450 as problematic. The same function pattern appears on line 1,100, but the review doesn't flag it. What explains this inconsistency?

- **A)** The model's random seed varies between inference calls.
- **B)** The model's attention is non-uniform across long inputs — the "lost-in-the-middle" effect. Content near the middle of a long document gets less attention. Structure the review to ensure coverage across all file sections.
- **C)** The model's `max_tokens` ended before it could analyze the second instance.
- **D)** The model only reviews the first half of files exceeding a certain length.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Lost-in-the-middle effect: content near the middle of long inputs gets less attention. Structure the review to ensure coverage across all sections (e.g., chunking or multi-pass). Random seed (A), max_tokens (C), and length limits (D) aren't the mechanism.

</details>

---

#### C358

*TS 5.1 · Context preservation*

A customer writes: "Order #5582 hasn't arrived. It's been two weeks." The agent calls `lookup_order` and gets:


```json
{"order_id": "5582", "status": 3, "carrier": "USPS", 
 "tracking": "9400111...", "ship_date": 1709251200,
 "estimated_delivery": 1709856000, "items": [...],
 "internal_notes": "Flagged for weight discrepancy",
 "warehouse_id": "WH-47", "picking_station": "A-14",
 "audit_log": [... 40 entries ...]}
```


This 40+ field response accumulates across multiple order lookups, filling the context. What's the fix?

- **A)** Implement a `PostToolUse` hook that trims `lookup_order` results to only customer-relevant fields: `order_id`, `status`, `carrier`, `tracking`, `ship_date`, `estimated_delivery`, `items`.
- **B)** Instruct the model to ignore irrelevant fields in the prompt.
- **C)** Store full results in a database and give the model only a reference ID.
- **D)** Increase the model's max context window.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

PostToolUse hook trims results to relevant fields. 40+ fields per lookup (including internal audit logs) rapidly fills context.

**Why the others are wrong:**

- **B:** Prompt instructions are probabilistic.
- **C:** Database references add complexity.
- **D:** Larger context doesn't fix the waste.

</details>

---

#### C359

*TS 5.1 · Context preservation*

The agent handles a complex return and mentions: "Your refund of $347.50 for order #8842 has been processed." But looking at the actual order, the amount was $374.50. The agent transposed the digits from a previous order lookup earlier in the session. What pattern prevents this?

- **A)** Double-check all amounts using a calculator tool before stating them.
- **B)** Extract transactional facts (amounts, order numbers, dates) into a persistent "case facts" block that is included in every prompt, separate from the summarizable conversation history.
- **C)** Use lower temperature to reduce hallucination.
- **D)** Always re-call `lookup_order` immediately before stating any amounts.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Persistent case facts block prevents detail transposition. Exact amounts, IDs, and dates in a structured block that's included in every prompt, surviving any compression. Calculator tools (A) and re-lookups (D) are reactive, not preventive.

</details>

---

#### C360

*TS 5.1 · Context preservation*

Your agent analyzes a large product catalog file (3,000 lines) and recommends items from the first 500 lines and last 500 lines, but rarely from the middle section. What explains this pattern?

- **A)** The model's random seed varies between inference calls.
- **B)** The model's attention is non-uniform across long inputs — the "lost-in-the-middle" effect. Content near the middle gets less attention. Structure searches to ensure coverage across all catalog sections.
- **C)** The model's `max_tokens` ended before it could analyze the middle section.
- **D)** The model only processes the first and last portions of files exceeding a certain length.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Lost-in-the-middle effect: content near the middle of long inputs gets less attention. Structure searches to ensure coverage across all catalog sections. Random seed (A), max_tokens (C), and length limits (D) aren't the mechanism.

</details>

---

#### C361

*TS 5.1 · Context preservation*

A patient provides symptom information. The `collect_symptoms` tool returns 50+ fields including internal coding, provider notes, and audit trails. This accumulates across multiple symptom collections, filling the context. What's the fix?

- **A)** Store full results in a database and give the model only a reference ID.
- **B)** Instruct the model to ignore irrelevant fields in the prompt.
- **C)** Implement a `PostToolUse` hook that trims `collect_symptoms` results to only patient-relevant fields: symptom description, severity, duration, affecting body systems.
- **D)** Increase the model's max context window.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

`PostToolUse` hook trims results to relevant fields. 50+ fields per collection rapidly fills context.

**Why the others are wrong:**

- **A:** Database references add complexity.
- **B:** Prompt instructions are probabilistic.
- **D:** Larger context doesn't fix the waste.

</details>

---

#### C362

*TS 5.1 · Context preservation*

The agent tells a patient: "Your appointment is scheduled for 2:30 PM on March 20th." But the actual booking was for 3:30 PM. The agent transposed the time from a previous appointment lookup earlier in the session. What pattern prevents this?

- **A)** Double-check all times using a calendar tool before stating them.
- **B)** Extract appointment facts (times, dates, provider names, locations) into a persistent "visit details" block included in every prompt, separate from summarizable conversation history.
- **C)** Use lower temperature to reduce hallucination.
- **D)** Always re-call `book_appointment` immediately before stating any appointment details.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Persistent visit details block prevents detail transposition. Exact times, dates, and provider info in a structured block included in every prompt, surviving any compression. Calendar tools (A) and re-bookings (D) are reactive.

</details>

---

#### C363

*TS 5.1 · Context preservation*

After two weeks of production runs you notice the synthesis subagent's briefings regularly omit findings from the academic-research subagent. Inspecting the transcripts: each research subagent dumps 8–15 paper abstracts (often 500+ tokens each) into its tool-call results as part of returning its findings to the coordinator. By the time the synthesis subagent runs, its context window contains ~40 abstracts plus the original instructions, and it appears to lose track of earlier material. What is the most appropriate **architectural** fix?

- **A)** Increase `max_tokens` on the synthesis subagent so its briefing has room to mention every finding.
- **B)** Have each research subagent return only a short structured summary (e.g. `{key_findings: string[], top_sources: {title, url, one_line_takeaway}[]}`) and persist the full abstracts to a file on disk that the synthesis subagent can fetch on demand with `Read`.
- **C)** Add a `Stop` hook that automatically compacts the conversation whenever the running token count exceeds 100,000.
- **D)** Switch the synthesis subagent to Claude Opus 4.7, which handles long contexts better than Sonnet.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

The whole point of the subagent pattern is **context isolation**: a subagent does work in its own context window and returns only a summary to the parent. Letting subagents dump raw abstracts back into the coordinator's context defeats the architecture — you've moved the work but not the bloat. The fix is to make the summary genuinely small and let the synthesis agent pull detail on demand.

**Why the others are wrong:**

- **A:** Treats the symptom from the wrong side. A longer output doesn't help a model that has lost track of the input.
- **C:** A lossy band-aid. Compaction loses information; redesigning the data flow does not.
- **D:** A model swap, not an architectural fix. Even with a longer context window, you've designed the system to grow context without bound.

</details>

---

#### C364

*TS 5.2 · Escalation & ambiguity resolution*

**Situation:** After a failed tool call, the orchestrator must decide whether to retry automatically or pause and ask a human for guidance.

In which situation is it most appropriate to escalate to a human rather than retry automatically?

- **A)** The tool schema description is missing optional metadata
- **B)** The tool returns a well-structured validation error that clearly names the offending field
- **C)** The retry limit has been reached and the action involves an irreversible side effect such as sending a payment
- **D)** A transient network timeout occurs during the first attempt

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Irreversible or high-stakes actions that have exhausted retries require human judgement. Anthropic's guidance emphasises that agentic systems should prefer reversible actions and escalate when irreversibility combines with persistent failure.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents#human-in-the-loop
- https://docs.anthropic.com/en/docs/build-with-claude/agents/agent-security

**Why the others are wrong:**

- **A:** Missing optional metadata degrades Claude's initial parameter quality but does not by itself warrant human escalation.
- **B:** A clear, field-level validation error is exactly the case where automated retry with corrected parameters is appropriate.
- **D:** A transient network timeout on the first attempt is a strong candidate for automated retry with exponential backoff.

</details>

---

#### C365

*TS 5.2 · Escalation & ambiguity resolution*

**Situation:** An AI assistant is helping a customer with a refund request. The customer's message is: 'I want my money back for the thing I bought last week.' The agent has no order history loaded in context.

What is the most appropriate initial action for the agent?

- **A)** Deny the refund and ask the customer to call back with more information.
- **B)** Ask the customer to clarify which product or order they are referring to before proceeding.
- **C)** Escalate immediately to a human agent because the request is ambiguous.
- **D)** Assume the customer means their most recent order and proceed with the refund.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

When input is ambiguous and the missing information is easily obtainable from the user, the agent should ask a targeted clarifying question rather than assuming or escalating. Escalation is reserved for situations where the agent cannot resolve the ambiguity itself or where stakes are high enough to warrant human involvement.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents#human-in-the-loop

**Why the others are wrong:**

- **A:** Denying a valid request due to missing context is unhelpful and damages trust; the correct response is to gather context, not refuse.
- **C:** Immediate escalation for a simple clarifiable ambiguity wastes human agent capacity and degrades user experience when the agent could resolve it directly.
- **D:** Assuming the most recent order without verification risks processing the wrong refund, causing downstream errors and customer harm.

</details>

---

#### C366

*TS 5.2 · Escalation & ambiguity resolution*

**Situation:** A designer is building an escalation system and proposes using sentiment analysis to automatically route angry customers to human agents, arguing that negative sentiment is a reliable escalation signal.

What is the primary problem with this approach?

- **A)** Human agents should only handle positive-sentiment interactions.
- **B)** Sentiment analysis is unreliable as an escalation signal; explicit signals should be used instead.
- **C)** Sentiment analysis is too expensive to run in real time.
- **D)** Negative sentiment always means the customer wants to cancel, not escalate.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Sentiment analysis is considered unreliable for driving escalation decisions because it frequently misclassifies tone and does not capture the customer's actual intent or need. Explicit signals — such as the customer directly requesting a human, a defined keyword, or a rule-based trigger — are far more reliable escalation indicators.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents#human-in-the-loop

**Why the others are wrong:**

- **A:** This is incorrect; human agents handle a wide range of sentiment types, including frustrated, complex, and high-stakes interactions regardless of tone.
- **C:** Cost is not the primary concern; the fundamental issue is reliability, not latency or expense.
- **D:** Negative sentiment has many interpretations; assuming it always signals cancellation intent is another form of the same unreliability problem.

</details>

---

#### C367

*TS 5.2 · Escalation & ambiguity resolution*

**Situation:** A customer has previously indicated in past sessions that they prefer self-service resolution and do not want to be transferred to a human agent unless absolutely necessary. Today they ask a complex billing question that falls just below the agent's confidence threshold.

How should the customer preference tracking data influence the escalation decision?

- **A)** The agent should immediately escalate because billing questions always require human review.
- **B)** The agent should attempt a more thorough self-service resolution before triggering escalation, honoring the customer's documented preference.
- **C)** The agent should ask the customer to re-state their preference before deciding.
- **D)** Customer preferences should be ignored; confidence threshold rules always take precedence.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Customer preference tracking is specifically intended to personalize escalation behavior. When a customer has documented a preference for self-service, the agent should honor that by attempting additional resolution steps — such as retrieving supplementary information or rephrasing the answer — before triggering escalation. The threshold is a guide, not an absolute override of documented customer preferences.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents#human-in-the-loop

**Why the others are wrong:**

- **A:** Billing questions are not categorically escalation-required; complexity and confidence thresholds, combined with preferences, determine the appropriate path.
- **C:** Re-asking for a preference that is already documented creates unnecessary friction and undermines the value of preference tracking.
- **D:** This ignores the purpose of preference tracking entirely; personalization is a first-class design concern in escalation systems.

</details>

---

#### C368

*TS 5.2 · Escalation & ambiguity resolution*

**Situation:** An agent retrieves two internal documents about a return policy. Document A states returns are accepted within 30 days; Document B (newer) states returns are accepted within 14 days. The customer is asking whether their 20-day-old purchase is returnable.

Which escalation trigger is active, and what is the correct handling approach?

- **A)** Conflicting information trigger; flag the conflict, do not commit to either policy, and route to human review or a supervisor agent.
- **B)** Conflicting information trigger; surface both policies to the customer and let them choose.
- **C)** Ambiguous input trigger; ask the customer to restate their question more clearly.
- **D)** Confidence below threshold trigger; answer with the more restrictive policy (14 days) to minimize business risk.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Contradictory internal documents activate the conflicting information trigger. The agent should not arbitrarily choose one policy, as this could misrepresent the company's actual current policy and harm either the customer or the business. The correct action is to flag the conflict, withhold a definitive answer, and escalate to human review or a supervisor agent that can determine which document takes precedence.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents#human-in-the-loop

**Why the others are wrong:**

- **B:** Surfacing conflicting internal policy documents to customers is inappropriate; it exposes internal inconsistencies and forces a policy decision onto the customer.
- **C:** The input is not ambiguous — the customer's question is clear. The problem is conflicting data sources, not the user's message.
- **D:** Silently choosing the more restrictive policy is a unilateral policy decision by the agent, which it is not authorized to make; this bypasses proper escalation.

</details>

---

#### C369

*TS 5.2 · Escalation & ambiguity resolution*

**Situation:** A user asks an agent: 'Can you help me deal with my neighbor?' The agent must decide whether to ask for clarification or make an assumption and proceed.

What principle should guide this decision?

- **A)** Escalate to a human agent because neighbor disputes may involve safety concerns.
- **B)** Always assume the most benign interpretation and proceed to maximize conversational flow.
- **C)** Weigh the cost of a wrong assumption against the cost of asking; ask for clarification when the risk of assuming incorrectly is high relative to the friction of the question.
- **D)** Always ask for clarification when any ambiguity exists, regardless of stakes.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

The decision to clarify vs. assume should be a risk-weighted judgment. 'Help me deal with my neighbor' could mean a noise complaint, a legal boundary dispute, or interpersonal conflict advice — the consequences of assuming incorrectly differ significantly. The agent should ask a single focused clarifying question when the space of possible interpretations is wide and the cost of a wrong assumption is non-trivial.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents#human-in-the-loop

**Why the others are wrong:**

- **A:** Immediate escalation over a vague statement is premature; there is no explicit safety signal yet — a clarifying question is the appropriate first step.
- **B:** Always assuming the most benign interpretation is a heuristic, not a principle; it ignores cases where benign assumptions lead to wrong or unhelpful responses.
- **D:** Asking for clarification on every ambiguity creates excessive friction; minor ambiguities with low consequence should often be assumed and confirmed inline.

</details>

---

#### C370

*TS 5.2 · Escalation & ambiguity resolution*

Production data shows the agent escalates to a human about 5% of the time, well below the ~20% your product team expects for billing disputes specifically. The system prompt today says: *"Escalate to a human when you cannot resolve the issue confidently."* Which change is most likely to bring escalation behavior in line with product intent **reliably**?

- **A)** Switch the primary model from Claude Sonnet 4.6 to Claude Opus 4.7 — Opus's stronger judgment will handle ambiguous escalation decisions better.
- **B)** Add a `confidence_score: number` parameter to `escalate_to_human` and instruct the model: "Call `escalate_to_human` whenever your confidence in resolving the issue is below 0.7."
- **C)** Add a `PostToolUse` hook that, after every tool call, runs a separate Claude API call asking "should this be escalated?" and forces escalation if the answer is yes.
- **D)** Replace the vague rule with concrete enumerable triggers: "Always escalate when (1) refund amount exceeds $200, (2) the customer mentions legal action or a regulator, (3) the account is locked or flagged for fraud, (4) any tool call errors twice in the same turn, or (5) the customer explicitly asks for a human."

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Concrete, enumerable conditions are the most reliable way to drive agent decisions. Each trigger is independently verifiable from the agent's view of the conversation, so behavior is predictable and reviewable.

**Why the others are wrong:**

- **A:** May marginally improve judgment but doesn't fix the root cause: the rule itself is ambiguous. A model upgrade is the wrong lever when the spec is underdetermined.
- **B:** Asks the model to invent a calibrated confidence score, which language models are notoriously poor at. The threshold (`0.7`) is grounded in nothing.
- **C:** Doubles inference cost and latency on every tool call to do work a well-written rule already does. The judge call has the same vague rule and the same calibration problem.

</details>

---

#### C371

*TS 5.2 · Escalation & ambiguity resolution*

A customer says: "I want to speak to a manager. This is ridiculous." The agent's current behavior is to first investigate the issue and attempt resolution before offering escalation. What should the agent do?

- **A)** Investigate the issue quickly and present the resolution. If the customer reiterates their request, then escalate.
- **B)** Escalate immediately. When a customer explicitly requests a human agent, honor the request without attempting investigation first.
- **C)** Perform sentiment analysis and determine that the customer's frustration level warrants escalation.
- **D)** Offer to resolve the issue and explain that wait times for human agents are currently long, to encourage the customer to continue with the AI.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

When a customer explicitly requests a human, honor immediately. Don't investigate first (A), use sentiment (C), or try to retain (D). This is a tested escalation pattern.

</details>

---

#### C372

*TS 5.2 · Escalation & ambiguity resolution*

A `get_customer` call for "Sarah Johnson" returns three matching records in your system. How should the agent proceed?

- **A)** Select the record with the most recent activity, as it is most likely the active customer.
- **B)** Ask the customer for an additional identifier (email, phone number, or account number) to disambiguate.
- **C)** Present all three records and ask the customer which one is theirs.
- **D)** Select the record in the same geographic region as the customer's IP address.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Multiple matches require disambiguation. Ask for additional identifiers (email, phone). Don't guess based on recency (A), IP address (D), or expose all records (C — privacy risk).

</details>

---

#### C373

*TS 5.2 · Escalation & ambiguity resolution*

A customer escalation policy states: "Escalate when the customer's request falls outside documented policies." A customer asks for a competitor price match. Your price adjustment policy covers only matching your own website's lower prices. What should the agent do?

- **A)** Deny the request because competitor price matching isn't in the policy.
- **B)** Ask the customer to provide a link to the competitor's price for verification before deciding.
- **C)** Escalate to a human agent because the policy is silent on competitor price matching — this is a policy gap.
- **D)** Attempt the price match since the customer's request is reasonable and similar to an existing policy.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

When policy is silent on the customer's request, escalate — this is a policy gap. Don't deny (A — may lose a valid customer), don't attempt (B — no policy authority), don't proceed (D — still no policy).

</details>

---

#### C374

*TS 5.2 · Escalation & ambiguity resolution*

A customer says: "I'm frustrated with your company, but I think you can fix this. My order arrived late." The agent should:

- **A)** Acknowledge the frustration, then investigate and attempt to resolve the late delivery issue — the customer indicated they want the AI to help.
- **B)** Transfer to a human agent because any negative emotion suggests the AI can't handle the situation.
- **C)** Escalate immediately because the customer expressed frustration.
- **D)** Use sentiment analysis to determine whether the frustration level warrants escalation.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

The customer expressed frustration BUT indicated willingness to let the agent help ("I think you can fix this"). Acknowledge frustration, investigate.

**Why the others are wrong:**

- **B:** Sentiment escalation is unreliable — the customer's words override sentiment signals.
- **C:** Sentiment escalation is unreliable — the customer's words override sentiment signals.
- **D:** Sentiment escalation is unreliable — the customer's words override sentiment signals.

</details>

---

#### C375

*TS 5.2 · Escalation & ambiguity resolution*

After deploying the agent, metrics show it escalates 45% of cases — far above the 20% target. Analysis reveals it escalates all billing disputes regardless of complexity, even straightforward ones like duplicate charges with clear evidence. What's the most effective fix?

- **A)** Have the agent self-report a confidence score and only escalate below a threshold.
- **B)** Remove the `escalate_to_human` tool to force the agent to handle everything.
- **C)** Implement a separate routing model that pre-classifies cases as "agent-handleable" or "needs human."
- **D)** Add explicit escalation criteria with few-shot examples to the system prompt, showing when to escalate (policy gaps, customer insistence) versus resolve autonomously (clear-cut duplicate charges, standard returns).

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Explicit escalation criteria with few-shot examples fix over-escalation.

**Why the others are wrong:**

- **A:** Self-confidence is poorly calibrated.
- **B:** Removing the tool removes a safety valve.
- **C:** Routing models add complexity.

</details>

---

#### C376

*TS 5.2 · Escalation & ambiguity resolution*

A customer provides their name as "Alex Chen" and says their phone number is "555-0142." The `get_customer` tool returns:


```json
{"matches": [
  {"id": "C-1001", "name": "Alex Chen", "phone": "555-0142", "email": "alex@work.com"},
  {"id": "C-1002", "name": "Alex Chen", "phone": "555-0142", "email": "alexc@home.com"}
]}
```


Both records match on name AND phone. How should the agent proceed?

- **A)** Select C-1001 because it appears first in the results.
- **B)** Select the one with the most recent activity.
- **C)** Ask the customer for an additional identifier — in this case, their email address — to disambiguate between the two accounts.
- **D)** Merge the two records and proceed with the combined information.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Both name and phone match two accounts — need additional identifiers (email). First-result (A) and recency (B) are heuristics that may be wrong.

**Why the others are wrong:**

- **D:** Merging records is a data integrity violation.

</details>

---

#### C377

*TS 5.2 · Escalation & ambiguity resolution*

Your knowledge agent successfully answers 85% of employee queries autonomously. For the remaining 15%, it should escalate to human experts. Currently, it escalates 40% of queries — many are straightforward questions it could answer. What's the fix?

- **A)** Remove the escalation tool to force the agent to answer everything.
- **B)** Have the agent self-report confidence scores and only escalate below a threshold.
- **C)** Implement a separate routing model that pre-classifies queries as "agent-handleable" or "needs human."
- **D)** Add explicit escalation criteria with few-shot examples to the system prompt: when to escalate (ambiguous queries, conflicting sources) vs. resolve autonomously (clear policy questions, documented procedures).

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Explicit escalation criteria with few-shot examples fix over-escalation.

**Why the others are wrong:**

- **A:** Removing the tool removes a safety valve.
- **B:** Self-confidence is poorly calibrated.
- **C:** Routing models add complexity.

</details>

---

#### C378

*TS 5.2 · Escalation & ambiguity resolution*

Your agent generates product comparison tables for customers. It sometimes includes products from different categories (comparing laptops to tablets) because the customer's query was ambiguous. How do you fix this?

- **A)** Limit comparisons to a maximum of 3 products to reduce errors.
- **B)** When queries are ambiguous, have the agent ask clarifying questions before generating comparisons. Document this behavior in CLAUDE.md with examples of good clarification questions.
- **C)** Post-filter comparison tables to remove mismatched categories.
- **D)** Only allow comparisons within the same top-level category regardless of customer intent.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Ask clarifying questions when queries are ambiguous. Document this in CLAUDE.md with examples.

**Why the others are wrong:**

- **A:** Product limits don't address the root cause.
- **C:** Post-filtering is reactive.
- **D:** Category restrictions limit helpfulness.

</details>

---

#### C379

*TS 5.2 · Escalation & ambiguity resolution*

Your agent prompt includes: "When unsure about product suitability, mention your uncertainty to the customer." This results in customers losing confidence as the agent frequently expresses doubt. What's the better approach?

- **A)** Add "Never express uncertainty" to the prompt.
- **B)** Replace uncertainty-based communication with confident recommendations based on explicit matching criteria. If confidence is genuinely low (insufficient customer info), ask clarifying questions instead of expressing doubt.
- **C)** Add a confidence score field and only show recommendations above 0.8 confidence.
- **D)** Have the agent run a second pass to verify its recommendations before showing them.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Replace uncertainty communication with confident recommendations based on explicit criteria. If confidence is low, ask clarifying questions instead of expressing doubt. Confidence scores (C) and second passes (D) add complexity.

**Why the others are wrong:**

- **A:** "Never express uncertainty" is too absolute.

</details>

---

#### C380

*TS 5.2 · Escalation & ambiguity resolution*

A patient says: "I'm worried this might be serious, but I trust you can help me figure out what to do." The agent should:

- **A)** Escalate immediately because the patient expressed worry about severity.
- **B)** Acknowledge the concern, collect appropriate symptom information per triage protocol, and route to appropriate care level — the patient indicated they want the AI to help with initial assessment.
- **C)** Use sentiment analysis to determine whether the worry level warrants immediate escalation.
- **D)** Transfer to a human agent because any expression of worry suggests the AI can't handle the situation.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

The patient expressed worry BUT indicated willingness to let the agent help. Acknowledge concern, collect symptom information per triage protocol, route appropriately.

**Why the others are wrong:**

- **A:** Sentiment escalation is unreliable — the patient's words override sentiment signals.
- **C:** Sentiment escalation is unreliable — the patient's words override sentiment signals.
- **D:** Sentiment escalation is unreliable — the patient's words override sentiment signals.

</details>

---

#### C381

*TS 5.2 · Escalation & ambiguity resolution*

After deploying the agent, metrics show it escalates 55% of cases — far above the 25% target. Analysis reveals it escalates all symptom triage cases regardless of severity, even routine matters like prescription refills. What's the most effective fix?

- **A)** Have the agent self-report a confidence score and only escalate below a threshold.
- **B)** Remove the `escalate_to_clinician` tool to force the agent to handle everything.
- **C)** Add explicit escalation criteria with few-shot examples to the system prompt: when to escalate (red flag symptoms, patient insistence, policy gaps) vs. handle autonomously (routine refills, appointment scheduling, insurance questions).
- **D)** Implement a separate routing model that pre-classifies cases as "agent-handleable" or "needs clinician."

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Explicit escalation criteria with few-shot examples fix over-escalation.

**Why the others are wrong:**

- **A:** Self-confidence is poorly calibrated.
- **B:** Removing the tool removes a safety valve.
- **D:** Routing models add complexity.

</details>

---

#### C382

*TS 5.2 · Escalation & ambiguity resolution*

A patient provides their name as "Jordan Smith" and date of birth as "1985-06-15." The `lookup_patient_record` tool returns two matching records with identical name and DOB. How should the agent proceed?

- **A)** Select the first record because it appears first in the results.
- **B)** Select the one with the most recent visit history.
- **C)** Ask the patient for an additional identifier — in this case, their phone number or medical record number — to disambiguate between the two accounts.
- **D)** Merge the two records and proceed with the combined information.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Both name and DOB match two accounts — need additional identifiers (phone or MRN). First-result (A) and recency (B) are heuristics that may be wrong.

**Why the others are wrong:**

- **D:** Merging records is a data integrity violation.

</details>

---

#### C383

*TS 5.2 · Escalation & ambiguity resolution*

A patient asks: "Can you prescribe medication for my migraine during this intake call?" Your policy states: "Prescriptions require a licensed clinician consultation. Intake agents can schedule these consultations but cannot prescribe." What should the agent do?

- **A)** Deny the request, citing the policy: "Prescriptions require a clinician consultation. I can schedule that for you."
- **B)** Attempt to process the prescription since the patient's request seems reasonable and urgent.
- **C)** Escalate to a clinician because the policy is ambiguous about prescription authority.
- **D)** Ask the patient for their medication history to evaluate whether a prescription is appropriate.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

The policy explicitly states prescriptions require clinician consultation. This is NOT a policy gap — the policy directly addresses the request. Denial with offer to schedule is correct.

**Why the others are wrong:**

- **C:** Escalation is for gaps, not clearly defined policies.

</details>

---

#### C384

*TS 5.2 · Escalation & ambiguity resolution*

After deploying the agent, metrics show it creates unnecessary purchase orders for 35% of recommendations — far above the 10% target. Analysis reveals it creates POs for all low-stock items regardless of pending orders or upcoming deliveries. What's the most effective fix?

- **A)** Implement a separate approval model that pre-classifies POs as "necessary" or "unnecessary."
- **B)** Add explicit PO creation criteria with few-shot examples: when to create (no pending orders, lead time exceeded) vs. wait (pending orders exist, delivery scheduled).
- **C)** Have the agent self-report confidence scores and only create POs above a threshold.
- **D)** Remove the `create_purchase_order` tool to force manual review.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Explicit PO creation criteria with few-shot examples fix over-creation.

**Why the others are wrong:**

- **D:** Removing the tool removes capability.

</details>

---

#### C385

*TS 5.2 · Escalation & ambiguity resolution*

A teacher says: "I'm worried this content might be too advanced for my students, but I trust you can help me adjust it." The agent should:

- **A)** Escalate immediately because the teacher expressed worry about difficulty level.
- **B)** Transfer to a human curriculum specialist because any expression of worry suggests the AI can't handle the situation.
- **C)** Acknowledge the concern, collect appropriate information about student levels, and adapt content per differentiation protocol — the teacher indicated they want the AI to help with adjustment.
- **D)** Use sentiment analysis to determine whether the worry level warrants immediate escalation.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

The teacher expressed worry BUT indicated willingness to let the agent help. Acknowledge concern, collect information, adapt content.

</details>

---

#### C386

*TS 5.2 · Escalation & ambiguity resolution*

After deploying the agent, metrics show it escalates 50% of content requests — far above the 20% target. Analysis reveals it escalates all adaptation requests regardless of complexity, even straightforward simplifications. What's the most effective fix?

- **A)** Have the agent self-report a confidence score and only escalate below a threshold.
- **B)** Remove the escalation tool to force the agent to handle everything.
- **C)** Implement a separate routing model that pre-classifies requests as "agent-handleable" or "needs specialist."
- **D)** Add explicit escalation criteria with few-shot examples to the system prompt: when to escalate (novel subject matter, conflicting standards) vs. handle autonomously (reading level adjustment, format conversion).

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Explicit escalation criteria with few-shot examples fix over-escalation.

**Why the others are wrong:**

- **B:** Removing the tool removes a safety valve.

</details>

---

#### C387

*TS 5.2 · Escalation & ambiguity resolution*

A teacher provides their school ID as "SCH-4421" and grade level as "7th Grade Math." The `search_standards` tool returns two matching standard sets with identical IDs. How should the agent proceed?

- **A)** Ask the teacher for an additional identifier — in this case, their state or district — to disambiguate between the two standard sets.
- **B)** Merge the two standard sets and proceed with the combined information.
- **C)** Select the first set because it appears first in the results.
- **D)** Select the one with the most recent update date.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Both ID and grade match two standard sets — need additional identifiers (state/district).

**Why the others are wrong:**

- **C:** First-result is a heuristic that may be wrong.

</details>

---

#### C388

*TS 5.2 · Escalation & ambiguity resolution*

A teacher asks: "Can you certify that this lesson meets state requirements for graduation?" Your policy states: "AI-generated content must be reviewed by certified educators before official use. The agent can suggest alignment but cannot provide official certification." What should the agent do?

- **A)** Ask the teacher for their state's graduation requirements to evaluate whether certification is appropriate.
- **B)** Deny the certification request, citing the policy: "Official certification requires educator review. I can suggest alignment for your review."
- **C)** Escalate to a curriculum specialist because the policy is ambiguous about certification authority.
- **D)** Attempt to provide certification since the teacher's request seems reasonable.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

The policy explicitly states AI cannot provide official certification. Denial with offer to suggest alignment is correct.

</details>

---

#### C389

*TS 5.3 · Error propagation in multi-agent systems*

**Situation:** A coordinator agent dispatches five subagents in parallel. After 30 seconds, three have returned results but two are still running. One of the two still-running subagents then returns an error indicating it encountered a transient API timeout.

What is the correct coordinator behavior when a subagent reports a transient failure during parallel execution?

- **A)** Hand the failing subagent's task to one of the three already-completed subagents and ask it to run both tasks simultaneously.
- **B)** Log the error, retain the three successful results, and retry the failed subagent — applying a backoff strategy — before proceeding to aggregation.
- **C)** Ignore the failure, proceed with the three successful results, and silently omit the failed subagent's contribution from the final output.
- **D)** Immediately cancel all remaining subagents and return an error to the user, since partial results cannot be used.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

The coordinator is responsible for error recovery and retries. A transient timeout warrants a retry with backoff rather than immediate abort. Preserving the already-completed results avoids repeating successful work. Silently omitting a result would corrupt the aggregated output without the user's knowledge.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems

**Why the others are wrong:**

- **A:** Asking a completed subagent to re-run a different task conflates the subagent's original specialized context with a new assignment; retrying the original subagent with the same task is the correct pattern.
- **C:** Silently dropping a failed subagent's contribution violates the error-handling principle that errors must never be swallowed; the user or downstream logic needs to know the output is incomplete.
- **D:** Aborting all work on the first transient error is overly aggressive and wastes the three successful results; transient errors are specifically those that are likely to succeed on retry.

</details>

---

#### C390

*TS 5.3 · Error propagation in multi-agent systems*

**Situation:** A coordinator agent orchestrates 10 subagents in parallel, each analysing a different section of a large legal document. Each subagent returns a structured JSON summary. The coordinator must then synthesise a final report. A developer notices that one subagent consistently times out, causing the entire synthesis step to block indefinitely.

Which architectural change best resolves the blocking problem while preserving the parallel execution model?

- **A)** Increase the maximum context window of the timing-out subagent so it can process its section faster.
- **B)** Implement a timeout and fallback at the coordinator level: if a subagent's Task call exceeds a threshold, proceed with the partial results and mark that section as unavailable.
- **C)** Move the synthesis logic into the timing-out subagent to eliminate the coordinator's dependency on its result.
- **D)** Switch all subagents to sequential execution so a timeout on one does not affect others.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

The correct pattern is a coordinator-level timeout with graceful degradation. The coordinator should set a deadline on each Task invocation and, if a subagent does not respond in time, continue synthesis with partial results — marking the missing section explicitly. This preserves parallelism, prevents indefinite blocking, and produces a degraded-but-complete output rather than a hard failure.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems
- https://docs.anthropic.com/en/docs/build-with-claude/agents/agent-security

**Why the others are wrong:**

- **A:** Context window size affects how much text the model can process in one turn, not execution speed or I/O latency. A timeout caused by slow processing or a hanging external tool call cannot be fixed by increasing context.
- **C:** Consolidating synthesis into one subagent creates a monolithic design, reintroduces single-point-of-failure concerns, and does not address the underlying cause of the timeout.
- **D:** Sequential execution eliminates the parallelism that makes the architecture efficient. One slow subagent would still block all subsequent ones in sequence, and total latency would be the sum of all subagent latencies.

</details>

---

#### C391

*TS 5.3 · Error propagation in multi-agent systems*

**Situation:** An agent in a multi-agent pipeline attempts to fetch customer records from a database. The database returns an HTTP 403 Forbidden response. A downstream summarization agent is waiting for the results.

How should the fetching agent classify and communicate this outcome in its error payload?

- **A)** Return an empty array so the downstream agent can handle missing data uniformly
- **B)** Return a structured error payload with type 'EMPTY_RESULT', retryable: true, since no data was returned
- **C)** Retry the request three times before returning any error to the downstream agent
- **D)** Return a structured error payload with type 'ACCESS_DENIED', retryable: false, and an empty partialResults field

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

A 403 Forbidden is an access failure, not an empty result. It must be clearly classified as 'ACCESS_DENIED' with retryable: false — retrying will not resolve a permission denial. Returning an empty array would mask the failure, causing the downstream agent to incorrectly treat a permission error as a valid query with no matches. A structured error payload with the correct type, retryable flag, and empty partial results preserves error context across the agent boundary.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use

**Why the others are wrong:**

- **A:** An empty array conflates two fundamentally different outcomes: a valid query that found no matches, and an access failure that prevented any query from completing. Downstream agents cannot distinguish these cases and may make incorrect decisions based on the apparent 'empty' result.
- **B:** Classifying a 403 as 'EMPTY_RESULT' with retryable: true is a category error. EMPTY_RESULT indicates a valid query returned no matches. A 403 is an access failure that must be distinguished from empty results, and it is not retryable.
- **C:** Retrying after a 403 Forbidden is wasteful and incorrect. Access failures are non-retryable by nature — the permission will not change between retries. Delaying the error propagation adds latency without any chance of recovery.

</details>

---

#### C392

*TS 5.3 · Error propagation in multi-agent systems*

**Situation:** A search agent queries three data sources in parallel. Two sources return results successfully. The third source times out after the configured deadline. The orchestrator needs a response immediately.

What is the correct behavior for the search agent when one of three parallel data sources times out?

- **A)** Return the results from the two successful sources along with metadata indicating which source timed out and that its results are missing
- **B)** Discard all results and return a single TIMEOUT error to signal complete failure
- **C)** Return only the results from the two successful sources without any indication that a third source was queried
- **D)** Wait indefinitely for the timed-out source before returning anything to the orchestrator

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Partial results with metadata is the correct pattern. The agent has valid, useful data from two sources and should return it immediately rather than blocking or discarding it. The metadata must include which source timed out and that its contribution is absent, so the orchestrator and downstream agents can make informed decisions about the completeness of the data they received.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use

**Why the others are wrong:**

- **B:** Discarding all results because one source timed out is disproportionate and wasteful. Two-thirds of the data was retrieved successfully. The partial results pattern exists precisely to prevent this unnecessary total loss.
- **C:** Returning partial results silently — without metadata about the missing source — is a form of silent failure. Downstream agents would treat the partial data as complete, potentially making incorrect decisions based on an incomplete picture.
- **D:** Waiting indefinitely for a timed-out source defeats the purpose of timeout handling. It blocks the entire pipeline and prevents any results from flowing downstream, turning a partial failure into a total stall.

</details>

---

#### C393

*TS 5.3 · Error propagation in multi-agent systems*

**Situation:** A validation agent in a pipeline receives a structured error payload from an upstream data-fetching agent. The payload includes: type: 'NETWORK_ERROR', message: 'Connection refused on port 5432', retryable: true, partialResults: [].

What should the validation agent do with the retryable: true flag in this error payload?

- **A)** Replace retryable: true with retryable: false before passing the error downstream to prevent retry loops
- **B)** Ignore the flag and immediately propagate the error downstream without retrying
- **C)** Use the flag to decide whether to trigger a retry of the upstream operation or propagate the error, based on its retry budget
- **D)** Treat retryable: true as confirmation that the operation will eventually succeed and wait indefinitely

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

The retryable flag is informational — it signals that the error condition could resolve on a subsequent attempt (e.g., a transient network issue). The receiving agent should consult its retry budget (e.g., maximum attempt count, backoff policy) to decide whether to retry the upstream call or propagate the error further. Retryable: true does not guarantee success, only that a retry is worth attempting.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use

**Why the others are wrong:**

- **A:** Altering the retryable flag before passing an error downstream corrupts the error context. Downstream agents and operators rely on accurate error metadata to make decisions; silently changing it introduces hidden inconsistencies.
- **B:** Ignoring the retryable flag discards valuable error context. A transient network error that resolves on retry is a common and recoverable situation; immediately propagating it as a permanent failure is unnecessarily pessimistic.
- **D:** Retryable: true means a retry is worth attempting, not that success is guaranteed or that waiting indefinitely is appropriate. Waiting without a bound transforms a recoverable error into a pipeline stall.

</details>

---

#### C394

*TS 5.3 · Error propagation in multi-agent systems*

**Situation:** An analyst agent queries a database for all transactions in a date range. The query executes successfully, but the database contains zero records matching the criteria. The agent must return a result to an orchestrating agent.

How should the analyst agent correctly represent this outcome?

- **A)** Return a successful response with an empty results array and a message indicating no records matched the criteria
- **B)** Return a successful response with null to indicate no data was available
- **C)** Return an error payload indicating the query failed to retrieve any results
- **D)** Return a structured error payload with type: 'NO_DATA_FOUND' and retryable: false

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

An empty result set from a successful query is not an error — it is a valid outcome. The correct representation is a success response with an empty results array and a clear message that no records matched the criteria. This preserves the critical distinction between access failures (which prevent query execution) and valid queries that return no data.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use

**Why the others are wrong:**

- **B:** Returning null rather than an empty array is ambiguous. It does not clearly convey whether the query succeeded, whether data access failed, or whether null means 'no results'. An explicit empty array with a descriptive message is unambiguous.
- **C:** The query did not fail — it succeeded. Wrapping an empty result in an error payload is semantically incorrect and would cause downstream agents to invoke error-handling logic for a normal, expected outcome.
- **D:** Returning an error for a successful query with no matches conflates two different concepts. A query that executed successfully but found no records is not a failure; treating it as one forces downstream agents to handle a non-error case through error-handling paths.

</details>

---

#### C395

*TS 5.3 · Error propagation in multi-agent systems*

**Situation:** An orchestrator manages a fan-out to 10 worker agents. Workers 1–7 complete successfully. Worker 8 fails with a transient NETWORK_ERROR (retryable: true). Worker 9 fails with a permanent ACCESS_DENIED (retryable: false). Worker 10 is still running when the pipeline timeout fires. The orchestrator must respond to its caller immediately with whatever it has.

Which response structure correctly encodes all three failure modalities while preserving the partial results?

- **A)** { results: [7 results], metadata: { completed: 7, failed: [{ worker: 8, error: { type: 'NETWORK_ERROR', retryable: true } }, { worker: 9, error: { type: 'ACCESS_DENIED', retryable: false } }], timed_out: [10], total: 10 } }
- **B)** Return only the 7 successful results; errors and timeouts are internal concerns not relevant to the caller
- **C)** Return a single error payload with the first error encountered (NETWORK_ERROR), since only a single error can be communicated at a time
- **D)** { results: [7 results], metadata: { completed: 7, errors: 2, timed_out: 1 } } — with error counts only, since detailed error types are a security concern

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

A multi-failure scenario requires a response that carries all three failure modalities distinctly: the two failed workers with their individual structured error payloads (preserving type and retryable flags so the caller can decide to retry Worker 8 but not Worker 9), and the timed-out worker as a separate category. The caller needs this granularity to make informed recovery decisions — retrying only the retryable failure, escalating the access denial, and deciding whether to re-request Worker 10's output. Aggregating failures into a single error or counts discards the information needed for precise recovery.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use

**Why the others are wrong:**

- **B:** Hiding error and timeout information from the caller makes intelligent recovery impossible. The caller cannot retry Worker 8's retryable error, cannot escalate Worker 9's access denial, and cannot decide whether to re-request Worker 10's output if it has no knowledge these failures occurred.
- **C:** Returning only the first error encountered discards information about Workers 9 and 10 entirely. In a multi-agent fan-out, multiple independent failures can occur simultaneously; a single error payload cannot represent this state accurately and prevents differentiated recovery strategies.
- **D:** Error counts alone, without error types or retryability flags, do not support recovery: from '2 errors' the caller cannot tell a transient network error (worth retrying) from a permanent access denial. Calling error types a security concern is no reason to strip actionable metadata.

</details>

---

#### C396

*TS 5.3 · Error propagation in multi-agent systems*

**Situation:** A six-stage pipeline: Ingest → Parse → Enrich → Score → Decide → Notify. Enrich calls three external APIs in parallel with individual 5-second timeouts. Two APIs respond; one times out. Enrich propagates a partial result to Score. Score applies its logic on the available data and produces a score with a confidence interval. Decide receives the scored result.

What must the Decide agent verify before acting on the scored result to prevent decisions based on silently degraded data?

- **A)** Accept the score at face value; individual agent quality control is the responsibility of each agent, not of downstream consumers
- **B)** Verify that the score value is within the expected numeric range, since out-of-range values indicate incomplete enrichment
- **C)** Check the metadata chain: confirm Score's response includes a completeness flag or partial-results indicator inherited from Enrich's timeout, and apply a decision policy appropriate for degraded-confidence scores
- **D)** Re-run the Enrich stage from scratch to guarantee a fully enriched input before scoring

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Error context must be preserved across agent boundaries. When Enrich produces a partial result due to a timeout, that incompleteness marker must travel with the data through Score and be visible to Decide. The Decide agent must inspect the metadata chain — confirming that Score flagged a degraded-confidence result — and apply an appropriate decision policy (e.g., route to manual review, widen confidence thresholds, or decline to automate). Acting on a degraded score as if it were a complete score is a cascading failure pattern where a data quality problem silently drives incorrect downstream decisions.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use

**Why the others are wrong:**

- **A:** Assuming each upstream agent guarantees complete quality and accepting results at face value is the root cause of cascading failures. In a system that supports partial results by design, downstream agents must inspect completeness metadata — otherwise degraded data silently propagates to decisions.
- **B:** A score produced from two-thirds of the enrichment data may still fall within the expected numeric range — partial data does not necessarily produce an out-of-range value. Range validation cannot detect incompleteness; only explicit partial-result metadata in the payload can reliably signal it.
- **D:** Re-running the entire Enrich stage from Decide puts the responsibility boundary in the wrong place — Decide should not orchestrate upstream data collection. If re-enrichment is the desired recovery, the orchestrator or Score should trigger it. This also violates forward-only data flow in a pipeline.

</details>

---

#### C397

*TS 5.3 · Error propagation in multi-agent systems*

**Situation:** An architect is designing a new multi-agent system for financial trade processing. The system must distinguish between: (1) a trade lookup that returns no trades for a valid account (normal empty day), (2) a permission failure preventing any trades from being read, (3) a network timeout that may have only partially read the trade log, and (4) a corrupted response from the trade database. All four outcomes must be handled differently by downstream risk and compliance agents.

Which error taxonomy and payload design correctly supports all four distinct outcomes?

- **A)** Use a two-field payload: { ok: boolean, data: any } — ok: true with empty data for empty results, ok: false with an error string for all failure cases
- **B)** Use HTTP status codes alone: 200 for empty results, 403 for permission, 408 for timeout, 500 for corruption — downstream agents inspect the status code
- **C)** Use a single error type 'DATA_UNAVAILABLE' with a free-text message field for all four cases, letting downstream agents parse the message to determine the actual condition
- **D)** Use a structured payload with: { status: 'SUCCESS'|'PARTIAL'|'ERROR', data: [...], metadata: { completeness: 'FULL'|'PARTIAL'|'NONE', reason?: string }, error?: { type: 'ACCESS_DENIED'|'NETWORK_TIMEOUT'|'DATA_CORRUPTION'|null, retryable: boolean, partialResults: [...] } } — each outcome maps to a unique combination of fields

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

A structured payload with orthogonal fields for status, completeness metadata, and a typed error object is the only design that can unambiguously represent all four outcomes: (1) SUCCESS + FULL + no error for a valid empty day; (2) ERROR + NONE + ACCESS_DENIED (retryable: false) for a permission failure; (3) PARTIAL + PARTIAL + NETWORK_TIMEOUT (retryable: true) + whatever partial data was received for a mid-read timeout; (4) ERROR + NONE + DATA_CORRUPTION (retryable: false) for a corrupted response. Each outcome is machine-parseable with no ambiguity, allowing downstream risk and compliance agents to take differentiated, automated actions.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use

**Why the others are wrong:**

- **A:** A two-field ok/data payload collapses all three failure modes (access denied, timeout, corruption) into a single 'ok: false' state with only a string message. Downstream agents cannot take differentiated automated actions (retry timeouts, escalate corruptions, alert on access denials) from that.
- **B:** HTTP status codes conflate transport semantics with business logic. A 403 might mean the agent lacks credentials or that the account itself is restricted — these need different downstream actions. HTTP codes also cannot represent partial reads or separate data corruption from other 500-class errors.
- **C:** A single error type with a free-text message for all four cases forces downstream agents into brittle string parsing to find the actual error condition. Any change to the message format breaks downstream logic, making automated recovery unreliable and defeating structured, machine-readable payloads.

</details>

---

#### C398

*TS 5.3 · Error propagation in multi-agent systems*

**Situation:** A five-agent pipeline processes loan applications: ingestion → validation → credit-check → decision → notification. The credit-check agent fails with a transient timeout connecting to an external bureau. The decision agent is waiting for the credit score.

Which error propagation strategy best prevents a cascading failure across the remaining pipeline stages?

- **A)** The credit-check agent silently returns a neutral credit score of 0 to allow the pipeline to continue processing
- **B)** The credit-check agent propagates a generic error message to the decision agent, which propagates it to the notification agent, which alerts the user
- **C)** The credit-check agent returns a structured error payload with type: 'TIMEOUT', retryable: true, and partialResults containing any cached credit data; the orchestrator decides whether to retry or route to a fallback path
- **D)** The credit-check agent retries immediately in a tight loop until the bureau responds, blocking the decision agent

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Returning a structured error payload with complete metadata — type, retryable flag, and any cached partial results — gives the orchestrator the information it needs to make an intelligent recovery decision (retry with backoff, route to a fallback, or halt). This prevents cascading failure by stopping error propagation at the point of failure and centralizing recovery decisions rather than forcing every downstream agent to handle the error independently.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use

**Why the others are wrong:**

- **A:** Silently substituting a neutral score is a form of data fabrication. It allows the pipeline to continue but produces decisions based on incorrect information. This is one of the most dangerous patterns in multi-agent systems — it hides failures while corrupting outputs.
- **B:** Propagating a generic error without structured metadata through each agent strips out context at every boundary. By the time the notification agent receives it, the original error type, retryability, and any partial data are lost, making automated recovery impossible.
- **D:** Retrying in a tight loop without backoff or a retry budget can exhaust resources and increase load on an already-struggling external service. Blocking the decision agent while retrying indefinitely converts a transient fault into a pipeline stall.

</details>

---

#### C399

*TS 5.3 · Error propagation in multi-agent systems*

**Situation:** An orchestrator agent delegates subtasks to three worker agents: A (data fetch), B (enrichment), and C (formatting). Agent A completes successfully. Agent B fails with a non-retryable permission error. Agent C has not yet started.

How should the orchestrator handle Agent B's non-retryable permission error to avoid unnecessary cascading work?

- **A)** Allow Agent C to start and run to completion, then combine its output with Agent A's output, omitting Agent B's contribution
- **B)** Cancel Agent C before it starts, return Agent A's partial results to the caller with metadata indicating Agent B failed with a permission error and Agent C was cancelled
- **C)** Restart all three agents from the beginning to ensure a consistent result
- **D)** Return only Agent A's results without any indication that Agents B and C did not complete

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

When a non-retryable failure occurs in a dependency, the orchestrator should cancel downstream work that depends on that failure — Agent C requires Agent B's enrichment output to be meaningful. Starting Agent C anyway wastes resources and produces an incomplete result without the caller knowing why. The correct pattern is to cancel C, preserve A's partial results, and return a structured response that describes exactly what succeeded, what failed (with error type and non-retryable flag), and what was cancelled.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use

**Why the others are wrong:**

- **A:** Running Agent C when it depends on Agent B's output produces either an error in C or a result that is missing enrichment data. This wastes compute and delivers output of unknown quality. The caller receives no signal that the result is incomplete.
- **C:** Restarting all agents when only Agent B has a non-retryable permission error is wasteful. Agent A succeeded and need not repeat its work. A permission error will not resolve on retry, so restarting Agent B is pointless.
- **D:** Returning Agent A's results silently without metadata about the downstream failures is a form of silent partial failure. The caller cannot distinguish between a complete result and a partial one, leading to decisions made on incomplete data.

</details>

---

#### C400

*TS 5.3 · Error propagation in multi-agent systems*

**Situation:** An agent crosses three agent boundaries in a chain: Fetcher → Transformer → Aggregator. The Fetcher encounters a network error. It logs the error locally and then re-throws a new generic RuntimeException to the Transformer, which does the same to the Aggregator.

What error propagation anti-pattern is being demonstrated, and what is the correct alternative?

- **A)** Error context loss — the correct alternative is to propagate the original structured error payload intact across boundaries, or to wrap it in a new payload that explicitly includes the original as a cause
- **B)** Over-logging — the correct alternative is to log the error only once at the final boundary
- **C)** Exception tunneling — the correct alternative is to re-throw the original exception unchanged at each boundary
- **D)** Error swallowing — the correct alternative is to catch and discard errors at each boundary to prevent the chain from breaking

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Catching a specific structured error and re-throwing a generic RuntimeException destroys the error context — the original error type, retryable flag, and partial results are all lost. Each boundary strips away more information. The correct pattern is to propagate the original structured error payload unchanged, or to wrap it in an envelope that explicitly carries the original payload as the 'cause', so that any agent in the chain (and the final caller) has full diagnostic information.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use

**Why the others are wrong:**

- **B:** Logging is a separate concern from error propagation. The problem here is not the logging strategy but the conversion of structured error information into a generic exception that discards all context. Logging at a single boundary would not solve the loss of error type and retryability metadata.
- **C:** Exception tunneling means passing errors through layers that don't understand them — not converting structured errors to generic ones. Re-throwing the original exception unchanged would actually be better than the scenario; the problem is that a new generic exception is created at each step.
- **D:** Error swallowing means catching errors and not propagating them at all. The scenario explicitly states errors are being re-thrown, so they are not being swallowed. Swallowing errors would be a different (and also incorrect) anti-pattern.

</details>

---

#### C401

*TS 5.3 · Error propagation in multi-agent systems*

**Situation:** A report generation pipeline has an orchestrator that calls five specialist agents sequentially. The orchestrator enforces a total pipeline timeout of 30 seconds. Each agent has a 10-second individual timeout. After 25 seconds, two agents have completed and the third is still running.

When the third agent exceeds the remaining 5-second pipeline budget, what should the orchestrator do?

- **A)** Cancel all five agents and return a total failure, since the pipeline did not complete within budget
- **B)** Wait for the third agent to finish regardless of the pipeline timeout, since its result may be needed
- **C)** Extend the pipeline timeout to 60 seconds to allow all agents to complete
- **D)** Cancel the third agent, skip the remaining two agents, and return the partial results from the two completed agents with metadata indicating which agents timed out and were skipped

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

When the pipeline-level timeout budget is exhausted, the orchestrator must cancel the running agent and skip unstarted work. The correct response is to return the partial results already accumulated with metadata that identifies: which agents completed, which agent timed out mid-execution, and which agents were never started. This preserves the value of completed work, respects the caller's time contract, and gives the caller enough information to decide on recovery.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use

**Why the others are wrong:**

- **A:** Discarding the two successfully completed agents' results when two-thirds of the work succeeded is unnecessarily destructive. Partial results with clear metadata are more useful than a total failure that forces the entire pipeline to restart from scratch.
- **B:** Waiting for the third agent after the pipeline deadline has passed violates the caller's timeout contract. The orchestrator made a commitment to respond within 30 seconds; exceeding that unilaterally undermines the reliability contract of the pipeline.
- **C:** Dynamically extending the timeout at the point of expiration defeats the purpose of having a timeout. Timeouts exist to provide callers with a reliable upper bound on latency. An orchestrator that extends its own timeout dynamically offers no meaningful latency guarantee.

</details>

---

#### C402

*TS 5.3 · Error propagation in multi-agent systems*

**Situation:** An agent system processes medical records. A patient-lookup agent successfully retrieves 847 out of 850 records before hitting a memory limit. The remaining 3 records could not be fetched. The agent must respond to the requesting aggregation agent.

Which response structure correctly implements the partial results pattern for this scenario?

- **A)** { success: true, results: [847 records], metadata: { total_requested: 850, total_returned: 847, missing_ids: [id1, id2, id3], reason: 'MEMORY_LIMIT' } }
- **B)** { success: true, results: [850 records] } — fabricate placeholder records for the 3 missing entries to maintain count consistency
- **C)** { success: true, results: [847 records] } — return the records silently, since 847/850 is nearly complete
- **D)** { success: false, error: { type: 'MEMORY_LIMIT', message: '...', retryable: false } } — return an error since the operation did not fully complete

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

The partial results pattern requires returning both the available data and metadata that precisely describes what is missing and why. The response should be marked as successful (the agent did return valid data) but include: total requested, total returned, the specific IDs that are missing, and the reason for incompleteness. This gives the aggregation agent the information it needs to decide whether to proceed with partial data, request the missing records separately, or escalate.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use

**Why the others are wrong:**

- **B:** Fabricating placeholder records to maintain count consistency is data falsification. In a medical context this is particularly dangerous — downstream agents would process artificial records as if they were real patient data, producing incorrect analyses or decisions.
- **C:** Returning 847 records without any indication that 3 are missing is a silent partial failure. The aggregation agent would treat the result as complete, potentially producing analyses based on an incomplete patient population — a critical error in a medical context.
- **D:** Marking the response as a pure failure discards 847 valid records. The agent did successfully retrieve the vast majority of data; calling it a total failure loses that value and forces a complete retry. The partial results pattern exists to avoid this waste.

</details>

---

#### C403

*TS 5.3 · Error propagation in multi-agent systems*

The web search subagent encounters a timeout while searching for academic papers on AI ethics. It has found 3 of the expected 8 sources before the timeout. How should it report this to the coordinator?

- **A)** Return a generic error: `{"status": "error", "message": "Search timed out"}`
- **B)** Throw an exception that terminates the entire research workflow.
- **C)** Return an empty result set marked as successful to avoid disrupting the workflow.
- **D)** Return structured error context: `{"status": "partial_failure", "failure_type": "timeout", "partial_results": [...3 sources...], "attempted_query": "...", "alternatives": ["try academic database", "narrow search scope"]}`

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Structured error context enables intelligent coordinator recovery.

**Why the others are wrong:**

- **A:** Generic errors hide context.
- **B:** Terminating wastes partial results.
- **C:** Silent suppression prevents recovery.

</details>

---

#### C404

*TS 5.3 · Error propagation in multi-agent systems*

Your coordinator agent handles complex cases by delegating billing inquiries to a billing subagent and shipping inquiries to a shipping subagent. The billing subagent encounters a transient database error when looking up invoice records. It retries 3 times, then returns to the coordinator:


```json
{"status": "error", "message": "Service unavailable"}
```


The coordinator tells the customer: "I can't help with billing right now." What should the subagent return instead?

- **A)** The subagent should keep retrying indefinitely until the database becomes available.
- **B)** Structured error context: `{"status": "partial_failure", "errorCategory": "transient", "attempted": "lookup invoice #4521", "retries": 3, "partial_results": ["invoice found in cache: $284.50 due 2025-04-01"], "alternatives": ["retry in 5 minutes", "check cached records"]}`.
- **C)** A generic message: `{"status": "error", "isRetryable": true}`.
- **D)** The raw database error log with stack trace for debugging.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Structured error context with category, partial results, retries attempted, and alternatives.

**Why the others are wrong:**

- **A:** Infinite retries cause hangs.
- **C:** Generic errors prevent intelligent recovery.
- **D:** Stack traces aren't useful for the coordinator.

</details>

---

#### C405

*TS 5.3 · Error propagation in multi-agent systems*

Your coordinator agent handles complex cases by delegating insurance questions to an insurance subagent and scheduling to a scheduling subagent. The insurance subagent encounters a transient payer system error. It retries 3 times, then returns: `{"status": "error", "message": "Service unavailable"}`. The coordinator tells the patient: "I can't verify your insurance." What should the subagent return instead?

- **A)** Structured error context: `{"status": "partial_failure", "errorCategory": "transient", "attempted": "verify insurance for patient #12345", "retries": 3, "partial_results": ["insurance carrier identified: BlueCross"], "alternatives": ["retry in 10 minutes", "collect insurance card photo for manual verification"]}`.
- **B)** The raw error log with stack trace for debugging.
- **C)** A generic message: `{"status": "error", "isRetryable": true}`.
- **D)** The subagent should keep retrying indefinitely until the payer system becomes available.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Structured error context with category, partial results, retries attempted, and alternatives.

**Why the others are wrong:**

- **B:** Stack traces aren't useful for the coordinator.
- **C:** Generic errors prevent intelligent recovery.
- **D:** Infinite retries cause hangs.

</details>

---

#### C406

*TS 5.3 · Error propagation in multi-agent systems*

Your coordinator agent handles complex requests by delegating standards alignment to an alignment subagent and content generation to a content subagent. The alignment subagent encounters a transient standards database error. It retries 3 times, then returns: `{"status": "error", "message": "Service unavailable"}`. The coordinator tells the educator: "I can't align to standards right now." What should the subagent return instead?

- **A)** The subagent should keep retrying indefinitely until the standards database becomes available.
- **B)** Structured error context: `{"status": "partial_failure", "errorCategory": "transient", "attempted": "align lesson to standards for Grade 5 Math", "retries": 3, "partial_results": ["standards domain identified: Number Operations"], "alternatives": ["retry in 10 minutes", "use cached standards mapping"]}`.
- **C)** The raw error log with stack trace for debugging.
- **D)** A generic message: `{"status": "error", "isRetryable": true}`.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Structured error context with category, partial results, retries attempted, and alternatives.

**Why the others are wrong:**

- **D:** Generic errors prevent intelligent recovery.

</details>

---

#### C407

*TS 5.4 · Large-codebase context management*

**Situation:** An architect is designing a long-running agentic workflow that spans multiple model API calls. The workflow must be resumable after failures: if the process crashes after step 3 of 8, it must be able to resume from step 4 without re-executing completed steps. Each step mutates external systems (database writes, API calls).

Which workflow state management design best satisfies the resumability requirement?

- **A)** Persist a durable workflow state object after each step completes, recording which steps have executed, their outputs, and any external identifiers needed for idempotent continuation. On restart, load this state and skip completed steps before resuming.
- **B)** Use a message queue where each step publishes its output; on restart, replay all messages from the beginning to reconstruct state.
- **C)** Add a system prompt instruction telling the model to track its own progress and resume from the last completed step if interrupted.
- **D)** Store the full conversation history in memory and replay it from the beginning on restart, relying on tool call caching to skip already-executed calls.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Durable workflow state that is persisted after each step is the standard pattern for resumable workflows with side-effecting steps. The state records completion status, step outputs, and external identifiers so that on restart the orchestrator can deterministically skip completed steps and resume at the correct point. This prevents double-execution of side-effecting operations. The state must be persisted durably (e.g., to a database or file) before continuing to the next step.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/agents/multi-agent-systems

**Why the others are wrong:**

- **B:** Replaying messages from the beginning to reconstruct state would re-trigger the side effects of every completed step unless idempotency is perfectly implemented for all operations. This is fragile and expensive compared to storing explicit completion state.
- **C:** Asking the model to track its own progress is probabilistic and unreliable. The model has no persistent memory across process restarts and cannot reliably determine which steps completed without reading explicit state. This approach also cannot handle crashes that occur mid-step.
- **D:** Replaying the conversation from the beginning and relying on tool call caching does not prevent re-execution of side-effecting steps. Tool call caching in the Anthropic API applies to prompt tokens, not to external tool executions. Database writes and API calls would be repeated.

</details>

---

#### C408

*TS 5.4 · Large-codebase context management*

**Situation:** An architect is explaining session management to a junior engineer. The junior asks what the `/compact` command does inside the Claude CLI.

What does the `/compact` command primarily do?

- **A)** Switches the active model to a smaller, cheaper variant
- **B)** Exports the conversation to a JSON file for offline storage
- **C)** Summarizes the conversation history into a condensed form and frees context window space
- **D)** Deletes the current session and starts a brand-new conversation

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

`/compact` triggers an in-place summarization of the conversation: Claude produces a structured summary of what has been discussed, replaces the verbose history with that summary, and thereby frees a large portion of the context window for continued work without losing the key facts.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/sessions
- https://docs.anthropic.com/en/docs/claude-code/cli-reference

**Why the others are wrong:**

- **A:** Model switching is a separate configuration concern unrelated to context management commands.
- **B:** Exporting to JSON is not what `/compact` does; it operates on the live context window in memory.
- **D:** Deleting the session and starting fresh is a separate action; `/compact` preserves continuity by summarizing rather than discarding.

</details>

---

#### C409

*TS 5.4 · Large-codebase context management*

**Situation:** A large enterprise uses Claude for multi-week software architecture engagements. Sessions regularly reach 150k tokens. When `/compact` is run naively at that point, engineers report that the model loses track of unresolved trade-offs and previously rejected options, leading to repeated work.

What root cause explains this failure, and what is the correct mitigation?

- **A)** Session forks should be created before every `/compact` operation so the original verbose history is always recoverable
- **B)** Naive `/compact` without a structured pre-summary prompt loses nuanced state; the fix is to prompt Claude for an explicit structured summary — covering decisions, rationale, rejected options, and open questions — before compacting, so the summary preserves the semantics needed for continuity
- **C)** The model hallucinates during compaction; the fix is to disable `/compact` and instead manually delete older messages
- **D)** The context window is too small; the fix is to upgrade to a model with a larger context window so `/compact` is never needed

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Naive compaction produces a generic recap that omits the nuanced engineering state: which options were rejected and why, what trade-offs are still open, and what constraints were established. The fix is to explicitly prompt for a structured summary with named sections before invoking `/compact`. This summary then becomes the durable record of the session's semantic state, preventing repeated discussions of already-closed questions.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/sessions
- https://docs.anthropic.com/en/docs/build-with-claude/context-windows

**Why the others are wrong:**

- **A:** Forking before every compaction creates an ever-growing library of stale branches and does not fix the underlying loss of nuanced state during compaction; the resumed branch still loses the details if the summary was unstructured.
- **C:** Manual deletion is itself a form of naive truncation and suffers the same information-loss problem as unstructured `/compact`; it also removes tool-call structure, which can corrupt conversation integrity.
- **D:** A larger context window delays but does not eliminate the compaction problem; sessions will always eventually grow beyond practical limits, and the structural issue of unstructured compaction remains regardless of window size.

</details>

---

#### C410

*TS 5.4 · Large-codebase context management*

**Situation:** A team is nearing the context limit of an ongoing Claude session that spans a week of architectural design work. They must continue in the same session and cannot afford to lose critical decisions already made.

Which approach best preserves important context while freeing space to continue working?

- **A)** Start a fresh session and paste the last 10 assistant messages as the new system prompt
- **B)** Ask Claude to produce a structured summary of decisions, open questions, and current state, then use `/compact` or replace history with that summary
- **C)** Lower `max_tokens` on future requests so the model generates shorter responses
- **D)** Delete all tool-result messages to reduce token count before the next request

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Requesting a structured summary before hitting the context limit is the recommended pattern. The summary captures decisions, rationale, open questions, and current state in a compact form. Using `/compact` or replacing raw history with this summary frees the majority of the context window while preserving semantic continuity — far better than naive truncation.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/sessions
- https://docs.anthropic.com/en/docs/build-with-claude/context-windows

**Why the others are wrong:**

- **A:** Pasting the last 10 assistant messages captures only recent output with no structure; it omits earlier decisions and cannot replace a purposefully authored summary.
- **C:** Lowering `max_tokens` reduces future response length but does nothing to free existing context; the history continues to grow and the limit will still be reached.
- **D:** Deleting tool-result messages is a form of naive truncation that can corrupt the conversation structure; the model may lose track of what actions were taken and their outcomes.

</details>

---

#### C411

*TS 5.4 · Large-codebase context management*

**Situation:** A support agent built on Claude has been running for 40 turns. The operator wants to free up context space without losing the key issue the user is trying to resolve.

Which built-in Claude Code mechanism is specifically designed to summarize the current conversation and free context space in a single step?

- **A)** /clear
- **B)** /flush
- **C)** /compact
- **D)** /summarize

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

The /compact command in Claude Code generates a concise summary of the current conversation, replaces the full conversation history with that summary, and thereby frees a significant portion of the context window while preserving the essential narrative thread.

References:
- https://docs.anthropic.com/en/docs/claude-code/cli-reference
- https://docs.anthropic.com/en/docs/build-with-claude/conversation-management

**Why the others are wrong:**

- **A:** /clear erases the conversation entirely without producing a summary, resulting in complete loss of context rather than a condensed preservation of it.
- **B:** /flush is not an available Claude Code command; it is not part of the Claude Code CLI command set.
- **D:** /summarize is not an available Claude Code command; the correct command is /compact.

</details>

---

#### C412

*TS 5.4 · Large-codebase context management*

**Situation:** A Claude agent has been exploring a large codebase for 45 minutes and has accumulated extensive context about authentication flows, database schemas, and API routes. The user now asks about a completely unrelated billing module.

What is the primary purpose of writing findings to scratchpad files during a long exploration session?

- **A)** To satisfy documentation requirements
- **B)** To preserve discovered knowledge so it survives context resets or compaction
- **C)** To share findings with other team members
- **D)** To create a permanent audit log of all exploration activity

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Scratchpad files serve as persistent external memory. When context grows large and must be compacted or reset, findings written to files remain intact and can be re-read in a future session. This is the core strategy for surviving context degradation over long exploration sessions.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/conversation-management

**Why the others are wrong:**

- **A:** Documentation requirements are a secondary concern; the core motivation is surviving context resets, not compliance.
- **C:** While files can be shared, the primary architectural reason for scratchpad files in this context is agent memory persistence across context boundaries, not team collaboration.
- **D:** Audit logging is a distinct concern with different requirements; scratchpad files are specifically for agent working memory, not permanent records.

</details>

---

#### C413

*TS 5.4 · Large-codebase context management*

**Situation:** A developer notices that after reading several large configuration files and auto-generated type definition files, the Claude agent's responses have become less precise and it is struggling to remember earlier findings.

What phenomenon best describes what is happening to the agent's performance?

- **A)** Context degradation caused by large file reads polluting the context window
- **B)** Model hallucination caused by incorrect training data
- **C)** Network latency from too many file read operations
- **D)** Rate limiting from the underlying API provider

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Context degradation occurs when large, low-signal content (like auto-generated files or verbose configs) is read into the context window, displacing earlier high-value findings and reducing the model's effective working memory. Avoiding context pollution from large file reads is a core context management principle.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/context-windows

**Why the others are wrong:**

- **B:** Hallucination refers to generating false information; the described symptom is degraded recall of real findings, which is a context window capacity issue.
- **C:** File read operations are local and have negligible latency; this is not a network problem.
- **D:** Rate limiting would cause request failures or delays, not degraded precision in responses.

</details>

---

#### C414

*TS 5.4 · Large-codebase context management*

**Situation:** An agent has built up a large context window after hours of exploration and needs to continue working but is approaching the context limit.

What does the /compact command primarily accomplish in a long exploration session?

- **A)** Compresses source code files to reduce disk space
- **B)** Restarts the agent process to clear all state
- **C)** Deletes all scratchpad files to free memory
- **D)** Summarizes the current conversation and frees context window space for further work

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

/compact summarizes the accumulated conversation history into a condensed representation, freeing context window tokens while preserving the essential semantic content of the session. This allows the agent to continue working without losing the thread of the exploration.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/context-windows

**Why the others are wrong:**

- **A:** /compact operates on the context window, not on disk files; it has no effect on source code file sizes.
- **B:** Restarting would lose all in-memory state; /compact is a non-destructive operation that preserves semantic continuity.
- **C:** Scratchpad files are persistent storage and are not deleted by /compact; they are intentionally kept to survive exactly this kind of compaction.

</details>

---

#### C415

*TS 5.4 · Large-codebase context management*

**Situation:** An architect agent is orchestrating a full system audit of a 500,000-line codebase. The audit has three phases: (1) dependency graph construction requiring broad file scanning, (2) security vulnerability analysis requiring deep reading of auth and crypto modules, and (3) performance bottleneck identification requiring profiling data correlation. The architect has limited context budget and must ensure findings from all three phases are synthesized into a final report.

Which orchestration strategy best balances context efficiency, finding quality, and synthesis capability?

- **A)** Spawn three parallel specialized subagents, each writing structured findings to named scratchpad files, then have the orchestrator synthesize by reading only the scratchpad files
- **B)** Delegate all three phases to a single subagent with a large context window
- **C)** Have the orchestrator read all 500,000 lines itself to maintain full context for synthesis
- **D)** Run all three phases sequentially in the orchestrator's context, using /compact between phases

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

This is the canonical multi-agent context management pattern. Parallel specialized subagents each have fresh, focused context windows optimized for their specific task domain. Writing structured findings to named scratchpad files creates a clean interface between exploration and synthesis. The orchestrator only reads the compact scratchpad outputs — not raw source files — keeping its synthesis context clean and high-signal. This pattern scales to arbitrarily large codebases.

References:
- https://docs.anthropic.com/en/docs/claude-code/sub-agents
- https://docs.anthropic.com/en/docs/build-with-claude/conversation-management
- https://docs.anthropic.com/en/docs/build-with-claude/context-windows

**Why the others are wrong:**

- **B:** A single subagent with a large context window handling all three phases still suffers from cross-domain context pollution between phases, and does not benefit from parallel execution or specialization.
- **C:** Reading 500,000 lines directly is impossible within any current context window and represents the most extreme form of context pollution; this option is architecturally invalid.
- **D:** Sequential execution with /compact between phases loses the ability to perform cross-phase synthesis within the same context; /compact may discard findings needed for final synthesis, and sequential execution is slower than parallel.

</details>

---

#### C416

*TS 5.4 · Large-codebase context management*

**Situation:** During a long exploration session, an agent used /compact and then realized that a critical architectural insight about a circular dependency — discovered 2 hours earlier — was lost in the compaction summary. The insight had not been written to any scratchpad file because the agent was mid-analysis when /compact was run.

Which combination of practices would have prevented this information loss?

- **A)** Running /compact more frequently so each compaction summary is smaller and more detailed
- **B)** Avoiding /compact entirely and always spawning a new agent session when context is full
- **C)** Using a larger context window model to avoid needing /compact
- **D)** Writing findings to scratchpad files immediately upon discovery, before context accumulates to the point where /compact becomes necessary

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

The failure mode here is relying on /compact to preserve in-progress findings. /compact is a lossy compression — it preserves semantic summaries, not complete analytical details. The correct discipline is to externalize findings to scratchpad files eagerly and continuously, treating the context window as volatile working memory and files as durable storage. This way, /compact can run safely at any time without risk of information loss.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/conversation-management
- https://docs.anthropic.com/en/docs/build-with-claude/context-windows

**Why the others are wrong:**

- **A:** More frequent /compact runs result in more compaction events, each of which can lose in-progress findings; without the scratchpad discipline, this increases rather than decreases risk of loss.
- **B:** Avoiding /compact entirely and spawning new sessions still results in information loss unless findings are externalized to files — the new session starts with no memory of prior discoveries.
- **C:** A larger context window delays but does not eliminate the problem; eventually all context windows fill, and the same information loss occurs if the scratchpad discipline is absent. This is an architectural patch, not a solution.

</details>

---

#### C417

*TS 5.4 · Large-codebase context management*

**Situation:** An agent must determine whether a newly added `PaymentService` class correctly integrates with three existing services: `UserService`, `AuditService`, and `NotificationService`. Each service has 5–10 files. The agent has 40% context remaining. The agent already has deep knowledge of the payment domain from earlier exploration in this session.

How should the agent reason about the delegate-vs-explore decision, and what is the optimal action?

- **A)** Use remaining context to explore all three service integrations directly, reading all files from each service
- **B)** Delegate all integration analysis to three parallel subagents since the context is below 50%
- **C)** Run /compact immediately to free context, then delegate to a subagent
- **D)** Use targeted Grep to find cross-service interface points and constructor signatures without reading full service implementations, leveraging existing payment domain knowledge to interpret results

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

This scenario requires nuanced judgment. The existing payment domain knowledge is directly relevant to interpreting integration patterns, arguing against delegation. However, 40% context is insufficient to read all 15–30 service files. The optimal approach uses targeted Grep to extract only the interface-level information (method signatures, public API, dependency injection points) needed to assess integration correctness, without reading full implementations. This leverages existing context knowledge while staying within context bounds.

References:
- https://docs.anthropic.com/en/docs/claude-code/sub-agents
- https://docs.anthropic.com/en/docs/claude-code/common-workflows
- https://docs.anthropic.com/en/docs/build-with-claude/context-windows

**Why the others are wrong:**

- **A:** Reading all files from all three services at 40% remaining context would almost certainly cause context exhaustion mid-analysis, producing incomplete and potentially misleading findings.
- **B:** Delegating based solely on a 40% context threshold ignores the value of existing payment domain knowledge that is directly relevant to the analysis; the delegate decision should be based on task independence, not an arbitrary context percentage.
- **C:** Running /compact before exploring risks losing the payment domain knowledge that makes direct exploration superior to delegation in this case; the timing of /compact must account for what existing context is worth preserving.

</details>

---

#### C418

*TS 5.4 · Large-codebase context management*

**Situation:** A team is designing the context management strategy for a Claude-based code review agent that will autonomously review pull requests in a repository with 300,000 lines of code. The agent must: understand the full diff, check for consistency with surrounding code patterns, verify test coverage, and produce a structured review report. PRs range from 50 to 5,000 lines changed.

Which architectural design best handles the full range of PR sizes while maintaining review quality?

- **A)** Always use a single agent session with /compact as needed, reading the full diff plus all surrounding context files
- **B)** Limit the agent to reviewing only the diff lines without reading any surrounding code context, to avoid context pollution
- **C)** Implement a tiered strategy: for small PRs use direct exploration; for large PRs spawn specialized subagents per concern (diff analysis, pattern consistency, test coverage), with all agents writing findings to structured scratchpad files for orchestrator synthesis
- **D)** Always spawn a dedicated subagent per PR regardless of size

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

A tiered strategy is architecturally correct because PR complexity varies by orders of magnitude. Small PRs (50 lines) can be handled efficiently in a single focused session. Large PRs (5,000 lines) with multi-concern analysis benefit from parallel specialized subagents, each with a fresh focused context window, converging on scratchpad files for orchestrator synthesis. This design scales gracefully, avoids unnecessary overhead for simple cases, and maintains quality for complex cases.

References:
- https://docs.anthropic.com/en/docs/claude-code/sub-agents
- https://docs.anthropic.com/en/docs/build-with-claude/conversation-management
- https://docs.anthropic.com/en/docs/claude-code/common-workflows
- https://docs.anthropic.com/en/docs/build-with-claude/context-windows

**Why the others are wrong:**

- **A:** A single-session approach with /compact is inadequate for large PRs where the diff itself plus surrounding context may exceed context window limits even after compaction; quality degrades unpredictably.
- **B:** Reviewing only diff lines without surrounding context produces low-quality reviews that miss pattern violations, naming inconsistencies, and architectural concerns; this sacrifices review quality in an unacceptable way.
- **D:** Always spawning a dedicated subagent per PR regardless of size adds unnecessary orchestration overhead for trivially small PRs and fails to address the multi-concern decomposition needed for large PRs.

</details>

---

#### C419

*TS 5.4 · Large-codebase context management*

**Situation:** You are orchestrating an exploration of a 200,000-line codebase. You need to: (1) map all API endpoints, (2) document database schema relationships, and (3) trace the authentication middleware chain. Each task requires reading 20–40 files.

Which approach best manages context while completing all three tasks?

- **A)** Use /compact after each task to free context, then continue in the same agent
- **B)** Read all 40–120 files upfront to build complete context before beginning any analysis
- **C)** Complete all three tasks sequentially in a single agent session, reading files as needed
- **D)** Spawn three focused subagents in parallel, each tackling one task and writing findings to separate scratchpad files

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Spawning focused subagents in parallel is ideal when tasks are independent and each requires substantial file reading. Each subagent has a fresh, focused context window dedicated to its specific task, eliminating cross-contamination between concerns. Writing to scratchpad files ensures findings persist and can be aggregated by the orchestrator.

References:
- https://docs.anthropic.com/en/docs/claude-code/sub-agents
- https://docs.anthropic.com/en/docs/build-with-claude/conversation-management

**Why the others are wrong:**

- **A:** Using /compact between tasks risks losing nuance from earlier tasks; subagent delegation is preferred when tasks are independent and substantial enough to justify fresh context windows.
- **B:** Reading all files upfront maximally pollutes the context window before any analysis begins, almost guaranteeing context exhaustion and degraded reasoning quality.
- **C:** A single sequential session will accumulate context from all three tasks, causing degradation by the time the third task begins and making it hard to maintain precision across all findings.

</details>

---

#### C420

*TS 5.4 · Large-codebase context management*

**Situation:** You are deciding whether to explore a complex data transformation pipeline directly in the current agent session or delegate it to a subagent. The pipeline spans 15 files, you have already explored the surrounding domain, and the insights from the surrounding domain are likely needed to interpret the pipeline.

Based on the described scenario, should you explore directly or delegate to a subagent, and why?

- **A)** Always use /compact first, then delegate to a subagent regardless of existing context
- **B)** Delegate to a subagent and pass the full context window as input so it has all the background
- **C)** Explore directly in the current session, because the existing domain context is valuable for interpreting the pipeline and would be lost in a fresh subagent
- **D)** Delegate to a subagent, because subagents are always more efficient for file exploration

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

The key decision criterion is whether existing context adds value to the new task. When domain knowledge already in the context window is directly relevant to interpreting the target files, exploring directly preserves that cross-cutting insight. Subagent delegation is optimal for independent tasks where fresh context is an advantage, not a disadvantage.

References:
- https://docs.anthropic.com/en/docs/claude-code/sub-agents

**Why the others are wrong:**

- **A:** Blindly applying /compact and delegating ignores the situational analysis that should drive this decision; sometimes continuing in the same session is the right choice.
- **B:** Passing the full context window to a subagent negates the benefit of delegation (fresh focused context) and is architecturally unsound — subagents should receive concise, scoped instructions.
- **D:** Subagents are not universally more efficient; the correct choice depends on whether the task is independent of existing context. When existing context is load-bearing, delegation loses that advantage.

</details>

---

#### C421

*TS 5.4 · Large-codebase context management*

**Situation:** An agent has been exploring a Node.js backend and has written detailed scratchpad notes about route handlers and middleware. The context window is now at 75% and a new request arrives to analyze the React frontend codebase, which is entirely independent.

What is the optimal sequence of actions before beginning the frontend analysis?

- **A)** Verify scratchpad files contain complete backend findings, then run /compact or spawn a fresh subagent for the frontend analysis
- **B)** Delete the scratchpad files to free disk space, then start fresh
- **C)** Run /compact to summarize the backend findings, then continue with frontend analysis in the same session
- **D)** Begin reading frontend files immediately to maximize session continuity

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Before compacting or switching tasks, you must verify that scratchpad files contain all critical findings — /compact may discard nuance that hasn't been externalized yet. Once findings are safely persisted, /compact or a fresh subagent for the independent frontend task are both valid options, with the choice depending on context levels and task independence.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/conversation-management
- https://docs.anthropic.com/en/docs/build-with-claude/context-windows

**Why the others are wrong:**

- **B:** Deleting scratchpad files destroys the persisted findings that are the entire point of the scratchpad strategy; this is the opposite of correct behavior.
- **C:** Running /compact without first verifying scratchpad completeness risks losing backend findings that haven't been written to files yet; the verification step is mandatory.
- **D:** Proceeding immediately at 75% context with an entirely new domain risks exhausting context mid-analysis and potentially corrupting findings from both the backend and frontend tasks.

</details>

---

#### C422

*TS 5.4 · Large-codebase context management*

**Situation:** A subagent has been tasked with exploring a microservices architecture. It needs to understand 8 independent service directories, each containing 30–50 files with their own domain logic.

How should the subagent structure its exploration to minimize context degradation?

- **A)** Read all files from all 8 services before beginning any analysis
- **B)** Spawn 8 further sub-subagents, one per service, and have them report back simultaneously
- **C)** Explore one service at a time, write findings for each service to a scratchpad file before moving to the next, using Glob → Read → Grep per service
- **D)** Explore all 8 services simultaneously by interleaving file reads

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Systematic service-by-service exploration with incremental scratchpad writing is the correct pattern. Each service gets focused attention using the Glob → Read → Grep workflow, and findings are externalized to files before the next service begins. This prevents cross-service context pollution and ensures no findings are lost as context fills up.

References:
- https://docs.anthropic.com/en/docs/claude-code/common-workflows
- https://docs.anthropic.com/en/docs/build-with-claude/conversation-management
- https://docs.anthropic.com/en/docs/build-with-claude/context-windows

**Why the others are wrong:**

- **A:** Reading all files upfront from all 8 services immediately saturates the context window before any analysis can begin, making this approach unworkable.
- **B:** While further delegation is theoretically possible, spawning 8 sub-subagents adds orchestration overhead and complexity; the single-subagent incremental approach is sufficient and more practical for this scale.
- **D:** Interleaving reads across services pollutes context with mixed concerns, making it harder to reason about any individual service and increasing the risk of confusion between similar patterns.

</details>

---

#### C423

*TS 5.4 · Large-codebase context management*

The coordinator needs to resume a research workflow after a system crash. The web search and document analysis phases completed before the crash, but synthesis hadn't started. How should crash recovery be implemented?

- **A)** Save a `manifest.json` where each agent exports structured state. On resume, the coordinator loads the manifest and injects completed agent states into prompts.
- **B)** Implement a message queue that replays unprocessed tasks from a persistent log.
- **C)** Use `--resume` to automatically restore the coordinator's conversation context from before the crash.
- **D)** Re-run the entire workflow from scratch since partial state may be corrupted.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Structured manifests enable crash recovery.

**Why the others are wrong:**

- **B:** Message queues are over-engineered for this pattern.
- **C:** `--resume` may not persist agent state across crashes.
- **D:** Re-running everything is wasteful.

</details>

---

#### C424

*TS 5.4 · Large-codebase context management*

Your agent maintains a long-running support session with a customer. After 20 turns of troubleshooting, the agent starts giving contradictory advice — recommending steps it previously said to avoid. The customer loses trust. What's the architectural fix?

- **A)** Have the agent maintain a scratchpad file recording key decisions, steps taken, and steps explicitly ruled out. Re-read the scratchpad each turn to maintain consistency across the session.
- **B)** Increase the model's context window to prevent information loss.
- **C)** Use `/compact` after every 5 turns to keep context small.
- **D)** Limit all support sessions to 10 turns maximum.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

A scratchpad file persists key decisions and ruled-out steps across context boundaries, preventing contradictions.

**Why the others are wrong:**

- **B:** Larger context delays but doesn't solve degradation.
- **C:** Frequent `/compact` loses important context.
- **D:** Turn limits frustrate customers.

</details>

---

#### C425

*TS 5.4 · Large-codebase context management*

Your agent is exploring a large codebase to understand the refund processing flow. After reading 25 files, context starts degrading — the agent references "typical patterns" instead of specific classes it discovered. What should you do?

- **A)** Have the agent maintain a scratchpad file recording key findings. Re-read the scratchpad for subsequent questions instead of re-discovering information.
- **B)** Use `/compact` to free up context space and continue exploration.
- **C)** Increase the model's context window by using a higher-tier model.
- **D)** Start a new session and re-read the most important files.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Scratchpad files persist findings across context boundaries. The agent writes key findings to a file and re-reads it instead of re-discovering.

**Why the others are wrong:**

- **B:** `/compact` helps but doesn't prevent future degradation.
- **D:** New session loses all context.

</details>

---

#### C426

*TS 5.4 · Large-codebase context management*

Your agent receives tool results accumulating across a 20-turn conversation. By turn 15, the context window is 85% full with verbose order lookups and account data. The agent's responses become increasingly unreliable. What's the best mitigation?

- **A)** Set a hard limit of 10 turns per conversation.
- **B)** Use `/compact` at turn 10 to compress the history, but first extract critical case facts (IDs, amounts, dates, statuses) into a persistent block that survives compression.
- **C)** Discard tool results older than 5 turns.
- **D)** Route long conversations to human agents after 12 turns.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Extract critical facts before `/compact` so they survive compression. Hard turn limits (A) and discarding results (C) are too aggressive.

**Why the others are wrong:**

- **D:** Routing to humans fails the 80% resolution target.

</details>

---

#### C427

*TS 5.4 · Large-codebase context management*

Your knowledge agent reads 100+ policy documents during initial setup, consuming most of the context window. When employees ask specific policy questions, the agent gives generic answers referencing "company policies" without citing specific documents. What's the better approach?

- **A)** Increase the model's context window by upgrading to a higher-tier plan.
- **B)** Use `/compact` after every employee question to free up context space.
- **C)** Build a searchable index: use Grep to find relevant documents per query, then Read only those specific documents — avoid loading everything upfront.
- **D)** Store all policy content in a database and have the agent query the database instead of reading files.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Incremental investigation: Grep → find relevant docs → Read only those. Loading everything upfront floods context.

**Why the others are wrong:**

- **A:** Larger context doesn't fix the pattern.
- **B:** `/compact` loses details.

</details>

---

#### C428

*TS 5.4 · Large-codebase context management*

After researching 30 policy documents for an employee query, the agent's context is 90% full. Subsequent questions in the same session get increasingly vague. What should you implement?

- **A)** Start a new session for every employee question — this ensures fresh context.
- **B)** Upgrade to a model with a larger context window for all knowledge queries.
- **C)** Use `/compact` immediately after each document is read to minimize context usage.
- **D)** Have the agent maintain a scratchpad file with key findings (document IDs, relevant excerpts, citations). Re-read the scratchpad for follow-ups instead of re-discovering information.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Scratchpad files persist findings across context boundaries.

**Why the others are wrong:**

- **A:** New sessions lose all context.
- **B:** Larger context delays but doesn't solve the issue.
- **C:** `/compact` helps but doesn't prevent degradation.

</details>

---

#### C429

*TS 5.4 · Large-codebase context management*

After reviewing 40 contracts, context starts degrading — the agent references "typical clauses" instead of specific terms it discovered. What should you do?

- **A)** Start a new session and re-read the most important contracts.
- **B)** Increase the model's context window by using a higher-tier model.
- **C)** Have the agent maintain a scratchpad file recording key findings (clause types, risk levels, party obligations). Re-read the scratchpad for subsequent reviews.
- **D)** Use `/compact` to free up context space and continue review.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Scratchpad files persist findings across context boundaries. The agent writes key findings to a file and re-reads it.

</details>

---

#### C430

*TS 5.5 · Human review & confidence calibration*

**Situation:** A document-processing pipeline assigns each extracted document a single confidence score (e.g., 0.72) before deciding whether to send it for human review. A senior architect suggests switching to field-level confidence instead.

What is the primary advantage of field-level confidence scoring over document-level confidence scoring?

- **A)** It eliminates the need for a review queue because each field can auto-approve independently.
- **B)** It reduces the total number of documents that require any human review.
- **C)** It allows reviewers to focus only on the specific fields that are uncertain rather than re-examining the entire document.
- **D)** It standardises confidence values across all document types so calibration is simpler.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Field-level confidence (e.g., 'name: high, date: low') isolates uncertainty to the precise fields that need attention. Reviewers can then inspect and correct only those low-confidence fields rather than re-processing the whole document, which saves time and reduces cognitive load. Document-level scores aggregate uncertainty and can mask which specific fields are problematic.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/workflows-and-pipelines

**Why the others are wrong:**

- **A:** Low-confidence fields still require review; field-level granularity directs reviewers to the right place but does not eliminate the need for human oversight of uncertain extractions.
- **B:** Field-level confidence does not inherently reduce the number of documents reviewed; it may actually route more documents to review because any low-confidence field triggers review. The benefit is targeted effort, not volume reduction.
- **D:** Field-level confidence increases calibration complexity because each field type may have a different accuracy distribution; it does not standardise values across document types.

</details>

---

#### C431

*TS 5.5 · Human review & confidence calibration*

**Situation:** A team is designing a review queue for an invoice-extraction system. They want to ensure the highest-risk items are reviewed first so that errors with large financial impact are caught before downstream processing.

Which two factors should primarily drive review queue prioritisation?

- **A)** The model version used and whether the document was processed in batch or real-time mode.
- **B)** Document file size and the time it took the model to generate the extraction.
- **C)** Model confidence score and the business impact of an error in that document.
- **D)** The number of fields extracted and the length of the document.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Review queue priority should reflect both how uncertain the model is (confidence score) and how costly an error would be (business impact). A low-confidence extraction for a high-value invoice should jump to the front of the queue, while a low-confidence extraction for a trivial record may wait. Combining these two dimensions ensures reviewer effort has maximum protective value.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/workflows-and-pipelines

**Why the others are wrong:**

- **A:** Model version and processing mode have no consistent relationship to the urgency of human review; a newer model processed in batch can still produce a high-impact error.
- **B:** File size and processing time are operational metrics with no direct relationship to error risk or business impact; they should not drive prioritisation.
- **D:** Field count and document length are structural properties, not indicators of extraction accuracy or downstream cost of errors.

</details>

---

#### C432

*TS 5.5 · Human review & confidence calibration*

**Situation:** During a post-deployment audit, the team discovers that the model claims 90% confidence on a class of medical-record fields, but actual accuracy on those fields is only 65%. The model is consistently overconfident.

What does this discrepancy between stated confidence and actual accuracy indicate?

- **A)** The review threshold is set too low, causing too many correct extractions to be sent for review.
- **B)** The model is under-performing due to insufficient context window size.
- **C)** The human reviewers are introducing errors that lower the measured accuracy.
- **D)** The model has a calibration problem and its confidence scores cannot be trusted at face value.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Calibration measures whether a model's stated confidence matches its actual accuracy. If the model claims 90% confidence but is only 65% accurate, it is overconfident and poorly calibrated. This means the confidence scores should not be used directly for auto-approve thresholds without recalibration — for example, applying Platt scaling or isotonic regression to map raw scores to true probabilities.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/workflows-and-pipelines

**Why the others are wrong:**

- **A:** A threshold set too low would send more items to review than necessary, but it would not explain why stated confidence exceeds actual accuracy — that is a model calibration issue, not a threshold issue.
- **B:** Context window size affects the model's ability to process long inputs, not the relationship between its stated confidence and ground-truth accuracy on extracted fields.
- **C:** The scenario describes measured accuracy (ground truth comparison), which accounts for reviewer corrections; it is a property of the model's output quality, not reviewer error.

</details>

---

#### C433

*TS 5.5 · Human review & confidence calibration*

**Situation:** An architect is defining the auto-approve policy for a contract-extraction system. Most extracted clauses are low-risk boilerplate, but a small subset involve liability caps and indemnification language that carry significant legal weight.

What is the most appropriate criterion for requiring mandatory human review rather than auto-approving an extraction?

- **A)** The extraction took more than two seconds to produce.
- **B)** The document is longer than ten pages.
- **C)** The confidence score falls below a defined threshold OR the field type carries high business or legal impact.
- **D)** The user who submitted the document does not have administrator privileges.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Human review should be required whenever confidence is low (indicating model uncertainty) OR when the field's impact is high regardless of confidence. Even a high-confidence extraction of a liability cap clause warrants review because the cost of an undetected error is severe. Combining both criteria ensures that neither uncertain nor high-stakes extractions slip through without oversight.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/workflows-and-pipelines

**Why the others are wrong:**

- **A:** Processing latency is an infrastructure metric with no bearing on extraction accuracy or the downstream risk of an error.
- **B:** Document length may correlate loosely with complexity but is not a reliable proxy for extraction accuracy or business impact of specific fields.
- **D:** User privilege level is an access-control concern, not a signal about extraction quality or field risk; it should not drive the review decision.

</details>

---

#### C434

*TS 5.5 · Human review & confidence calibration*

**Situation:** A government agency deploys a document-classification pipeline that routes benefit applications to one of five processing tracks. Field-level confidence is available. Regulatory requirements mandate that any classification error affecting a citizen's benefit eligibility must be detectable within 30 days. The system processes 10,000 applications per day. Human review staff can review 500 applications per day. The team must design a monitoring and review workflow that satisfies the regulatory detectability requirement.

Which combination of strategies best satisfies the 30-day detectability requirement while operating within the 500-review-per-day staff constraint?

- **A)** Use confidence-based auto-approve for high-confidence applications, route low-confidence applications to the daily review queue, and implement a stratified random audit (sampling proportionally from all confidence buckets daily) to provide statistical detectability coverage across auto-approved applications — sizing the audit sample so that a 1% error rate in the auto-approved tier would be detected within 30 days at 95% confidence.
- **B)** Route all 10,000 daily applications to human review and hire additional staff to meet capacity.
- **C)** Auto-approve all applications where document-level confidence exceeds 0.85, and review the remaining 15% within daily capacity, with no further monitoring of auto-approved applications.
- **D)** Implement a 30-day rolling review where every application processed on day 1 is reviewed on day 30, creating a one-month lag review cycle.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Meeting a 30-day statistical detectability requirement requires both operational review (catching low-confidence cases in near-real-time) and systematic audit sampling of the auto-approved tier. Without auditing high-confidence auto-approvals, errors in that tier are invisible until a citizen complains. The stratified audit uses the daily review budget for a dual purpose: reviewing urgent low-confidence cases AND sampling across the confidence distribution to maintain statistical guarantees. Sample sizing (using statistical power calculations) ensures a minimum detectable error rate within the 30-day window. This is the only option that satisfies both the capacity constraint and the regulatory detectability mandate through principled design.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/workflows-and-pipelines

**Why the others are wrong:**

- **B:** Routing all 10,000 applications to human review requires 20× the available staff capacity; it is operationally infeasible and does not represent a design — it is an admission that no automated system is being used.
- **C:** Auto-approving the 85% high-confidence tier with no further monitoring creates a blind spot: errors in that tier are undetectable until downstream impact occurs, which may violate the 30-day detectability requirement if no systematic audit exists.
- **D:** Re-examining every application one month later would mean reviewing 10,000 applications per day (a month's backlog) on top of daily new intake — far beyond the 500-review capacity — and adds a systematic one-month delay before any error is caught, exactly at the regulatory limit.

</details>

---

#### C435

*TS 5.5 · Human review & confidence calibration*

**Situation:** A pipeline processes three document types: invoices (60% of volume, 94% model accuracy), contracts (30% of volume, 78% accuracy), and purchase orders (10% of volume, 88% accuracy). The team has a fixed weekly budget of 500 manual reviews. They want to allocate review effort to maximise error detection.

Which review strategy best allocates the 500 manual reviews across document types?

- **A)** Distribute reviews proportionally to volume: 300 invoices, 150 contracts, 50 purchase orders.
- **B)** Review all documents randomly regardless of type, since random sampling is unbiased.
- **C)** Concentrate all 500 reviews on contracts because they have the lowest accuracy.
- **D)** Allocate reviews inversely proportional to accuracy, weighting toward document types with higher error rates while maintaining coverage of all types.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Accuracy tracking by document type should inform where review effort goes. Contracts have a 22% error rate — more than double the invoice rate — so they deserve a disproportionately large share of the review budget. However, completely ignoring invoices and purchase orders leaves blind spots. The correct approach is accuracy-weighted allocation: more reviews where accuracy is lowest, but some baseline coverage of every document type. This is distinct from simple proportional (volume-based) allocation which ignores accuracy differences.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/workflows-and-pipelines

**Why the others are wrong:**

- **A:** Volume-proportional allocation ignores that contracts are far less accurate than invoices; it wastes most of the budget on document types that rarely produce errors.
- **B:** Pure random sampling is unbiased but inefficient: it allocates reviews in proportion to volume, which — like option A — underweights the high-error contract type.
- **C:** Concentrating all reviews on contracts catches contract errors but provides zero ongoing monitoring of invoices and purchase orders, creating undetected drift risk in those categories.

</details>

---

#### C436

*TS 5.5 · Human review & confidence calibration*

**Situation:** A team uses stratified sampling to draw 200 documents for weekly accuracy audits. Their confidence buckets are: low (0–0.5, 15% of volume), medium (0.5–0.8, 50% of volume), and high (0.8–1.0, 35% of volume). They want the sample to reflect the distribution of each bucket.

How many documents should be sampled from each confidence bucket in a proportional stratified sample of 200?

- **A)** Low: 67, Medium: 67, High: 66 (equal allocation across buckets).
- **B)** Low: 30, Medium: 100, High: 70 (proportional to bucket volume).
- **C)** Low: 200, Medium: 0, High: 0 (only sample uncertain documents).
- **D)** Low: 100, Medium: 70, High: 30 (inverse-proportional to bucket volume).

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Proportional stratified sampling draws from each stratum in proportion to that stratum's share of the total population. With 15% low, 50% medium, and 35% high: 0.15 × 200 = 30 low, 0.50 × 200 = 100 medium, 0.35 × 200 = 70 high. This ensures the sample mirrors the actual confidence distribution, giving unbiased accuracy estimates for each bucket and for the overall system.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/workflows-and-pipelines

**Why the others are wrong:**

- **A:** Equal allocation (roughly 67 each) oversamples the low-confidence bucket and undersamples the medium bucket relative to their actual proportions, introducing selection bias into accuracy estimates.
- **C:** Sampling only the low-confidence bucket provides no information about accuracy in the medium or high buckets, making it impossible to validate whether the auto-approve threshold is appropriate.
- **D:** Inverse-proportional allocation deliberately oversamples rare or uncertain buckets — this is a valid technique for error discovery but is not proportional stratified sampling; it would bias overall accuracy estimates upward.

</details>

---

#### C437

*TS 5.5 · Human review & confidence calibration*

**Situation:** A healthcare document pipeline extracts patient demographics, diagnoses, and billing codes. The compliance team mandates that any extraction that will directly update a patient record without clinician review must meet a strict accuracy standard. The model reports field-level confidence for each extracted value.

Which auto-approve policy correctly applies the 'when to require human review vs auto-approve' principle for this regulated context?

- **A)** Auto-approve all fields where model confidence exceeds 0.75, regardless of field type.
- **B)** Auto-approve low-impact demographics fields above a high confidence threshold (e.g., ≥0.95), but require human review for all diagnosis and billing-code fields due to clinical and financial risk.
- **C)** Require human review for all fields unconditionally because the domain is regulated.
- **D)** Auto-approve if the document-level average confidence exceeds 0.80, ignoring individual field confidence.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

In regulated healthcare contexts, the auto-approve vs. review decision should be stratified by both field-level confidence and field-level impact. Demographic fields (name, address) carry lower clinical risk and can be auto-approved at a high confidence threshold. Diagnosis and billing codes carry direct clinical and financial consequences — errors can harm patients or trigger fraud scrutiny — so they warrant mandatory review regardless of confidence. This tiered policy minimises reviewer workload without compromising safety on high-impact fields.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/workflows-and-pipelines

**Why the others are wrong:**

- **A:** A uniform 0.75 threshold ignores field-type risk; a 0.76-confidence diagnosis code could be auto-approved and directly harm a patient without any clinician seeing it.
- **C:** Universal review is maximally safe but operationally unsustainable at scale and defeats the efficiency goal of automated extraction; it is not a 'design' — it is the absence of one.
- **D:** Document-level average confidence masks field-level variance: a document could average 0.85 while containing a critically miscategorised diagnosis at 0.55 confidence, which would be auto-approved under this policy.

</details>

---

#### C438

*TS 5.5 · Human review & confidence calibration*

**Situation:** A team tracks accuracy over time for their document extraction pipeline. After a model update three weeks ago, they observe that accuracy on 'contract expiry date' fields dropped from 93% to 71%, while all other fields remain stable. The model's confidence scores on this field have not changed.

What does the divergence between stable confidence scores and falling accuracy on 'contract expiry date' fields indicate, and what action should the team take?

- **A)** The drop is within normal statistical variance; no action is needed.
- **B)** The accuracy tracking system is miscalculating accuracy for date fields due to a date-format mismatch.
- **C)** The model update introduced a regression on date parsing for this field type, and the confidence scores are not capturing this degradation — the team should lower the auto-approve threshold for date fields and trigger retraining or rollback.
- **D)** Human reviewers are making more errors on date fields, causing the measured accuracy to fall.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

When accuracy drops on a specific field type after a model change but confidence scores remain unchanged, it signals a calibration regression: the model's self-reported confidence no longer tracks actual performance for that field. Accuracy tracking by document type and field type is precisely designed to catch this pattern. The immediate response is to lower the auto-approve threshold for contract date fields (routing more to human review) and investigate the model update for a regression in date parsing, with retraining or rollback as remediation.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/workflows-and-pipelines

**Why the others are wrong:**

- **A:** A 22-percentage-point drop that correlates with a model update and persists for three weeks is not statistical noise; dismissing it risks a large backlog of incorrect date extractions propagating into contracts.
- **B:** While a format mismatch is worth investigating, the scenario's correlation with the model update and stability of all other fields points to a model regression rather than a measurement artefact.
- **D:** The scenario specifies that accuracy is measured against ground truth, not reviewer decisions; reviewer error would not systematically lower measured accuracy unless reviewers are the ground-truth source, which is not indicated.

</details>

---

#### C439

*TS 5.5 · Human review & confidence calibration*

Your extraction system processes 500 invoices nightly. Confidence scores route extractions: high confidence (≥90%) to auto-approval, low confidence (<90%) to human review. After 3 months, you discover that 12% of auto-approved extractions contain errors. What's the issue?

- **A)** The 90% threshold is too low — increase it to 99%.
- **B)** Model confidence is not well-calibrated out of the box. Calibrate thresholds using a labeled validation set — measure actual accuracy at each stated confidence level, then set empirically-grounded thresholds.
- **C)** Replace confidence scoring with rule-based validation.
- **D)** Remove auto-approval entirely and send all extractions to human review.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Out-of-the-box model confidence is poorly calibrated — stated 90% confidence may correspond to only 88% actual accuracy. Calibrate using labeled data. Rule-based validation (C) and full human review (D) are overreactions.

**Why the others are wrong:**

- **A:** Arbitrary threshold changes don't fix calibration.

</details>

---

#### C440

*TS 5.5 · Human review & confidence calibration*

You extract data from two types of documents: formal contracts (structured, consistent formatting) and informal agreements (emails, handwritten notes, varied layouts). Overall accuracy is 95%. Stakeholders are satisfied. However, your operations team reports frequent errors in informal agreements causing downstream failures. What happened?

- **A)** The 95% aggregate accuracy masks poor performance on informal agreements. You need stratified accuracy analysis by document type — the formal contract accuracy may be 99%+ while informal agreements may be 70%.
- **B)** The downstream system needs better error handling for malformed data.
- **C)** Informal agreements shouldn't be processed by the extraction system.
- **D)** The model needs additional training on informal documents.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Aggregate accuracy masks per-type performance. 95% overall might be 99% formal + 70% informal. Stratified analysis by document type reveals hidden failures.

**Why the others are wrong:**

- **C:** Excluding documents loses business value.
- **D:** Additional training isn't applicable.

</details>

---

#### C441

*TS 5.5 · Human review & confidence calibration*

Your extraction system has been running for 3 months with 98% accuracy on high-confidence extractions routed to auto-approval. Your quality team suggests stopping human review of high-confidence extractions entirely. What should you do first?

- **A)** Agree — 98% over 3 months proves the system is reliable.
- **B)** Implement stratified random sampling of high-confidence extractions to continuously measure error rates and detect novel error patterns. Don't eliminate human review; reduce it while maintaining ongoing validation.
- **C)** Increase the confidence threshold to 99.5% before auto-approving.
- **D)** Add a second Claude instance to review high-confidence extractions instead of humans.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Stratified random sampling for ongoing validation. Never eliminate human review entirely — reduce it while maintaining ongoing measurement. 3 months of 98% doesn't guarantee future performance. Novel patterns could emerge.

**Why the others are wrong:**

- **D:** AI review has its own biases.

</details>

---

#### C442

*TS 5.5 · Human review & confidence calibration*

Your extraction system needs to output field-level confidence scores for routing to human review. After deployment, you find that extractions marked "0.95 confidence" are wrong 30% of the time. What's needed?

- **A)** Lower the auto-approval threshold to 0.99.
- **B)** Replace model confidence with a rule-based confidence system.
- **C)** Remove confidence scoring and send all extractions to human review.
- **D)** Calibrate confidence thresholds using a labeled validation set. Out-of-the-box model confidence is not well-calibrated; you need to measure actual accuracy at each confidence level and set thresholds based on empirical data.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Model confidence is not well-calibrated out of the box. Calibrate using labeled validation sets — measure actual accuracy at each stated confidence level, then set empirically-grounded thresholds.

**Why the others are wrong:**

- **A:** Arbitrary threshold changes don't fix calibration.

</details>

---

#### C443

*TS 5.5 · Human review & confidence calibration*

Your extraction system processes both MLS listings (structured, consistent formatting) and private sale documents (emails, handwritten notes, varied layouts). Overall accuracy is 94%. Stakeholders are satisfied. However, your compliance team reports frequent errors in private sale documents causing valuation gaps. What happened?

- **A)** The 94% aggregate accuracy masks poor performance on private sale documents. You need stratified accuracy analysis by document type — MLS listings may be 98%+ while private sales may be 75%.
- **B)** The downstream valuation system needs better error handling for malformed data.
- **C)** The model needs additional training on informal documents.
- **D)** Private sale documents shouldn't be processed by the extraction system.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

Aggregate accuracy masks per-type performance. 94% overall might be 98% MLS + 75% private sales. Stratified analysis reveals hidden failures.

</details>

---

#### C444

*TS 5.6 · Provenance & multi-source synthesis*

**Situation:** A Claude-based research assistant synthesizes information from three articles. One article states a drug was approved in 2021, a second says 2022, and the third does not mention a date. The assistant outputs: 'The drug was approved in 2022.'

What provenance principle does this response violate?

- **A)** Citation tracking — the assistant failed to include hyperlinks to the original articles
- **B)** Uncertainty expression — the assistant should have refused to answer due to conflicting data
- **C)** Temporal data handling — the assistant should note when each source's information was current
- **D)** Conflict annotation — the assistant silently picked one source instead of surfacing the disagreement between sources

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

When two sources provide conflicting factual claims, the correct behavior is to surface the conflict explicitly (conflict annotation) rather than silently choosing one value. The response should say something like 'Source A states 2021, Source B states 2022' so the user can evaluate the discrepancy.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/reduce-hallucinations

**Why the others are wrong:**

- **A:** Citation tracking is about maintaining source links through synthesis chains; while important, the immediate violation here is picking one of two conflicting values without disclosure.
- **B:** Refusing to answer is not required — expressing uncertainty by presenting both values is preferred over refusal, which would deprive the user of the available information.
- **C:** Temporal data handling concerns flagging when information was current or has become stale over time — it is not the primary issue when two sources give different values for the same fact.

</details>

---

#### C445

*TS 5.6 · Provenance & multi-source synthesis*

**Situation:** A developer asks Claude to summarize a market report. The report was published in January 2023, and the summary will be used in December 2025. Claude produces a summary without mentioning when the underlying data was collected.

Which information provenance practice is missing from this summary?

- **A)** Conflict annotation between competing market forecasts
- **B)** Claim-source mapping for every statistic
- **C)** Flagging that the data is from January 2023 and may be stale relative to current conditions
- **D)** Expressing uncertainty by listing all possible market outcomes

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Temporal data handling requires noting when the underlying information was current and flagging data that may have become stale. A summary used two years after the source was published should clearly state the data's publication date so consumers can assess its relevance.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/reduce-hallucinations

**Why the others are wrong:**

- **A:** Conflict annotation applies when multiple sources disagree; the scenario describes a single report, so no inter-source conflict exists to annotate.
- **B:** Claim-source mapping is important but is a separate concern; the scenario does not indicate that individual claims are unattributed — the primary gap is the missing temporal context.
- **D:** Listing all market outcomes is not a provenance practice; uncertainty expression means disclosing the source and its limitations, not enumerating every possible scenario.

</details>

---

#### C446

*TS 5.6 · Provenance & multi-source synthesis*

**Situation:** A Claude pipeline performs three sequential summarization steps: raw documents → intermediate summary → final brief. By the final step, no source identifiers remain in the output.

What term describes the failure mode illustrated in this pipeline?

- **A)** Temporal degradation
- **B)** Conflict collapse
- **C)** Provenance loss in summarization chains
- **D)** Citation drift

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Provenance loss in summarization chains describes the progressive disappearance of source attribution as information passes through multiple synthesis or summarization steps. Each step can strip context until the final output is completely disconnected from its origins.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/reduce-hallucinations

**Why the others are wrong:**

- **A:** 'Temporal degradation' refers to data becoming outdated over time, not to the loss of source information during processing.
- **B:** 'Conflict collapse' is not a standard term; it might suggest conflicts being hidden, but the scenario describes source attribution disappearing rather than conflicting claims being merged.
- **D:** 'Citation drift' is not a standard information provenance term; while it evokes the idea, the recognized term for this failure mode is provenance loss in summarization chains.

</details>

---

#### C447

*TS 5.6 · Provenance & multi-source synthesis*

**Situation:** An architect is designing a high-stakes medical information system. The system uses a four-agent pipeline: (1) Retrieval Agent fetches PubMed abstracts, (2) Extraction Agent pulls specific claims with their PMID identifiers, (3) Synthesis Agent merges claims into a clinical summary, (4) Formatting Agent produces the final output for clinicians. The team discovers that the Synthesis Agent's prompt does not require it to preserve PMID identifiers when merging claims.

What is the most architecturally rigorous fix, and why does it matter more in a medical context than a general one?

- **A)** Instruct the Formatting Agent to add a footer stating 'Based on PubMed research' to satisfy citation requirements for clinical use
- **B)** Add a post-processing step that re-queries PubMed to reconstruct citations from the final summary text, because medical information requires verified citations
- **C)** Replace the four-agent pipeline with a single large-context Claude call that receives all abstracts at once, eliminating the inter-agent handoff where provenance is lost
- **D)** Redefine the Synthesis Agent's output schema to require a structured claims array where each claim object mandates a source_pmids field, and validate this schema before passing output to the Formatting Agent — because in medical contexts a missing or incorrect citation can directly contribute to patient harm

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

The architecturally rigorous fix is to enforce provenance at the data model level — requiring a source_pmids field in every claim object ensures that provenance cannot be accidentally dropped during synthesis. Schema validation before the next agent receives the data creates a hard contract. In medical contexts this matters more because incorrect or missing attribution can lead clinicians to misapply evidence, with direct patient safety consequences rather than merely informational ones.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/reduce-hallucinations
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use

**Why the others are wrong:**

- **A:** A generic footer attribution provides no granular claim-source mapping; it does not tell clinicians which specific claim comes from which study, and it does not satisfy the standard for evidence-based clinical information.
- **B:** Post-hoc citation reconstruction from summary text is unreliable: the Synthesis Agent may paraphrase claims in ways that cannot be matched back to their original abstract, and the reconstruction step itself can introduce new errors — particularly dangerous in medical contexts.
- **C:** A single large-context call may help with context coherence but does not structurally enforce claim-source mapping — the model could still produce a summary that blends sources without attribution, and it trades the provenance problem for a scalability and auditability problem.

</details>

---

#### C448

*TS 5.6 · Provenance & multi-source synthesis*

**Situation:** A financial intelligence platform builds a daily briefing using a three-step pipeline. Step 1 collects 200 news articles (with publication timestamps). Step 2 runs a Claude summarization that produces 20 topic summaries but discards publication timestamps. Step 3 runs a second Claude call that merges the 20 topic summaries into a single briefing. A compliance officer asks: 'Can you tell me which claim in the briefing comes from an article older than 48 hours?' The engineering team cannot answer.

Which root cause analysis most accurately explains this failure, and what single architectural change would prevent it?

- **A)** Root cause: Publication timestamps are discarded in Step 2, causing provenance loss that propagates forward and makes temporal tracing impossible. Fix: Require Step 2 to output structured topic summaries that include a sources array, where each source entry carries its publication_timestamp.
- **B)** Root cause: Step 3 context window is insufficient. Fix: Increase token limit for Step 3.
- **C)** Root cause: Claude summarization inherently loses metadata. Fix: Replace Claude with a deterministic extraction algorithm that preserves all metadata fields.
- **D)** Root cause: The pipeline lacks an audit log. Fix: Log all Claude inputs and outputs to a database.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: A**

The root cause is architectural: Step 2 receives articles with timestamps but its output schema does not carry those timestamps forward. Once discarded, the timestamps cannot be reconstructed downstream. The fix is to enforce provenance preservation at the data model level in Step 2 — structured outputs with a sources array and publication_timestamp ensure that every subsequent step has the temporal metadata it needs to answer compliance questions.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/reduce-hallucinations
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use

**Why the others are wrong:**

- **B:** Context window size is unrelated to this failure; the problem is that timestamps were discarded by design in Step 2, not that they were present but could not fit in Step 3.
- **C:** Replacing Claude with a deterministic algorithm is an extreme overreaction; the failure is not inherent to LLM summarization but to an output schema that does not mandate metadata preservation. LLM summarization with a structured output requirement can preserve provenance.
- **D:** An audit log captures inputs and outputs but does not, by itself, link specific claims in the final briefing back to specific source articles — it would require complex matching logic and would still fail if Step 2 paraphrased claims beyond recognition.

</details>

---

#### C449

*TS 5.6 · Provenance & multi-source synthesis*

**Situation:** A policy research assistant synthesizes 12 government reports on climate adaptation spending. Three reports (from 2019, 2021, 2023) use different methodologies to measure 'adaptation spending,' making their figures not directly comparable. Two of the 12 reports are from the same country but give contradictory totals for 2022. The remaining seven reports are consistent. The assistant produces a single paragraph stating an aggregate global figure.

Which provenance and uncertainty handling strategy is most appropriate for this synthesis?

- **A)** Average all figures to produce a single defensible aggregate, then add a footnote: 'Figures may vary due to differing methodologies'
- **B)** Exclude the three methodologically inconsistent reports and the two conflicting national reports from the synthesis, and base the output only on the seven consistent reports
- **C)** Produce the synthesis in a structured format that: (i) presents the seven consistent reports' aggregate with source citations; (ii) explicitly notes the three reports use non-comparable methodologies with their specific methodological differences described; (iii) presents both contradictory national totals side-by-side with their source dates; and (iv) recommends expert review before policy use
- **D)** Present the most recent report's figures as definitive, since newer data supersedes older data on the same topic

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

This scenario requires three distinct provenance practices simultaneously: claim-source mapping (citing the seven consistent reports), conflict annotation (surfacing the contradictory national totals), and uncertainty expression (describing the methodological incompatibilities rather than hiding them). Option C is the only choice that addresses all three dimensions without silently discarding or blending conflicting information. The recommendation for expert review appropriately flags the synthesis as insufficient for high-stakes policy decisions.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/reduce-hallucinations

**Why the others are wrong:**

- **A:** Averaging figures with incompatible methodologies produces a misleading number; a footnote about 'differing methodologies' does not constitute the specific conflict annotation and uncertainty expression required — it is a generic disclaimer rather than a substantive disclosure.
- **B:** Silently excluding 5 of 12 reports without informing the user constitutes provenance loss and hides relevant (even if problematic) information; the user cannot make an informed judgment about the exclusions if they are unaware of them.
- **D:** Recency does not resolve methodological incompatibility or factual contradiction; 'the newest report' may use a narrower definition of spending than older reports, making a direct comparison invalid regardless of date.

</details>

---

#### C450

*TS 5.6 · Provenance & multi-source synthesis*

**Situation:** An architect is designing a multi-agent RAG system where Agent A retrieves documents, Agent B extracts claims, and Agent C writes the final report. The product owner requires that every claim in the final report can be traced back to its original document.

Which architectural pattern best ensures end-to-end claim-source mapping across all three agents?

- **A)** Log all agent inputs and outputs to an audit database and generate citations post-hoc from the logs
- **B)** Have Agent B output structured claim objects that include a source_id field, and require Agent C to preserve and surface those source_ids in its output
- **C)** Store all retrieved documents in a shared vector store and let Agent C re-query as needed
- **D)** Instruct Agent C via system prompt to mention 'according to retrieved documents' before each claim

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Claim-source mapping requires that every factual claim carries its source identifier throughout the processing pipeline. By having Agent B emit structured claim objects with source_id fields and requiring Agent C to propagate those fields, the provenance chain is maintained by design rather than reconstructed after the fact.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/reduce-hallucinations
- https://docs.anthropic.com/en/docs/build-with-claude/tool-use

**Why the others are wrong:**

- **A:** Post-hoc citation generation from logs is fragile, requires additional processing, and can fail if log entries are incomplete or if Agent C paraphrases claims in ways that are hard to reconcile with source text.
- **C:** Re-querying the vector store does not guarantee that Agent C retrieves the same document that Agent B used; it creates a potential attribution mismatch rather than a traceable chain.
- **D:** A generic phrase like 'according to retrieved documents' is not claim-source mapping — it provides no granular link between a specific claim and its originating document.

</details>

---

#### C451

*TS 5.6 · Provenance & multi-source synthesis*

**Situation:** A legal research tool powered by Claude aggregates case law from three databases. Database X says a precedent was overturned in 2019, Database Y says it remains valid, and Database Z has no entry. The tool must surface this to the attorney user.

Which response structure best satisfies the conflict annotation requirement?

- **A)** Present the most recently updated database's position as the authoritative answer
- **B)** Present a blended synthesis: 'The precedent's validity is uncertain'
- **C)** Explicitly state: 'Database X indicates this precedent was overturned in 2019; Database Y indicates it remains valid; Database Z contains no record. Attorney verification is recommended.'
- **D)** Return an error to the user because conflicting sources prevent a reliable answer

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Conflict annotation requires surfacing disagreements explicitly, attributing each position to its source. Option C names each source and its specific claim, making the conflict visible so the attorney can exercise professional judgment. This is superior to synthesizing away the conflict, picking a winner, or refusing to respond.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/reduce-hallucinations

**Why the others are wrong:**

- **A:** Selecting the most recently updated source silently resolves the conflict without informing the user that a conflict exists, violating conflict annotation principles.
- **B:** 'The precedent's validity is uncertain' is a blended synthesis that hides the specific nature and source of the disagreement, making it harder for the attorney to investigate.
- **D:** Returning an error is an overcorrection; the system has useful information from two sources that the attorney needs, and withholding it in the presence of conflict is more harmful than surfacing the conflict clearly.

</details>

---

#### C452

*TS 5.6 · Provenance & multi-source synthesis*

**Situation:** A five-step summarization pipeline reduces a 50-page technical report to a one-page executive summary. Each step is performed by a separate Claude call with no structured metadata passed between steps. At the final step, Claude is asked to add citations.

Why will the citation step likely fail to produce accurate claim-source mappings?

- **A)** The final Claude call's context window is too small to hold the original 50-page document
- **B)** The intermediate summaries have already stripped the granular source context needed to map specific claims back to original sections, causing provenance loss
- **C)** Citations require structured data types that plain-text summarization cannot produce
- **D)** Claude cannot generate citations without access to the internet

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

Provenance loss in summarization chains occurs when each step discards source metadata. By the time citations are requested, the intermediate summaries contain claims but no record of which original section or page they came from. Adding citations at the end cannot recover information that was never passed through the pipeline.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/reduce-hallucinations

**Why the others are wrong:**

- **A:** Context window size is a real concern for very large documents, but the architectural flaw described is provenance loss — even with a large context window, unstructured summaries lack the source-mapping metadata needed for accurate attribution.
- **C:** Citations can be generated from plain-text pipelines if source identifiers are explicitly tracked; the problem is that this pipeline does not track them, not that plain text is structurally incapable of carrying provenance.
- **D:** Citation generation does not require internet access; the limitation here is the absence of preserved source metadata within the pipeline, not network connectivity.

</details>

---

#### C453

*TS 5.6 · Provenance & multi-source synthesis*

**Situation:** A Claude-based news aggregator pulls articles from 15 sources daily. Some articles were published hours ago; others are three days old. The system produces a single 'current events' brief without noting article publication dates.

Which combination of provenance practices is most critical to add to this system?

- **A)** Conflict annotation and uncertainty expression
- **B)** Temporal data handling and claim-source mapping
- **C)** Citation tracking across synthesis steps and conflict annotation
- **D)** Provenance loss prevention and uncertainty expression

<details>
<summary><b>Show answer</b></summary>

**Correct answer: B**

The scenario has two primary gaps: (1) no publication dates are surfaced (temporal data handling), meaning users cannot assess whether information is still current; and (2) claims are not linked to their originating articles (claim-source mapping), preventing verification. These two practices together address the identified deficiencies.

References:
- https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/reduce-hallucinations

**Why the others are wrong:**

- **A:** Conflict annotation and uncertainty expression are valuable but secondary here — the scenario does not describe inter-source disagreements, and the primary structural gaps are temporal and attribution-related.
- **C:** Citation tracking across synthesis steps is important in multi-step pipelines, but this scenario describes a single synthesis step; the bigger gaps are the absence of dates and source attribution on individual claims.
- **D:** Provenance loss prevention is relevant but vague as a standalone practice; the scenario's specific gaps are temporal context and claim attribution, which are better addressed by the named practices in option B.

</details>

---

#### C454

*TS 5.6 · Provenance & multi-source synthesis*

Two credible sources provide conflicting statistics: Forbes reports the AI market at $150B (January 2025) and McKinsey reports $180B (March 2025). How should the synthesis subagent handle this?

- **A)** Average the two values and report: "The AI market is valued at approximately $165B."
- **B)** Use the more recent source (McKinsey, March 2025) as the authoritative figure.
- **C)** Preserve both values with source attribution— noting dates and potential methodological differences — and structure the report to distinguish established from contested findings.
- **D)** Flag this as a data quality issue and exclude the statistic from the report entirely.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Preserve both values with attribution, dates, and methodological notes. Averaging (A), picking newer (B), and excluding (D) all lose information.

</details>

---

#### C455

*TS 5.6 · Provenance & multi-source synthesis*

The document analysis subagent returns findings as unstructured prose: "There's interesting research showing AI market growth. Several studies confirm this trend." Downstream synthesis loses all source attribution. What change to the subagent's output format fixes this?

- **A)** Post-process the synthesis output with a separate citation-extraction step.
- **B)** Have the coordinator extract citations from the prose output before passing to synthesis.
- **C)** Require the subagent to output structured claim-source mappings: each finding includes a claim, evidence excerpt, source URL/name, page number, and publication date.
- **D)** Add "include citations" to the document analysis subagent's system prompt.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Structured claim-source mappings preserve provenance through the pipeline.

**Why the others are wrong:**

- **A:** Post-processing cannot recover information that was lost during analysis.
- **B:** Post-processing cannot recover information that was lost during analysis.
- **D:** Prompt instructions are vague.

</details>

---

#### C456

*TS 5.6 · Provenance & multi-source synthesis*

A contract extraction produces: `"governing_law": "State of California"` but a parallel extraction from a rider amendment to the same contract says: `"governing_law": "State of New York"`. The rider supersedes the original contract. How should your system handle this?

- **A)** Use the original contract's value since it's the primary document.
- **B)** Use the rider's value since it was processed more recently.
- **C)** Preserve both values with source attribution, annotate the conflict, and include document dates so a reviewer can determine which takes precedence. Flag the extraction as needing human verification.
- **D)** Concatenate: `"governing_law": "State of California; State of New York"`.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: C**

Preserve both values with source attribution and dates. The rider may supersede, but that's a legal determination — the extraction system should preserve both and flag the conflict.

**Why the others are wrong:**

- **A:** Picking either makes assumptions.
- **B:** Picking either makes assumptions.
- **D:** Concatenation is meaningless.

</details>

---

#### C457

*TS 5.6 · Provenance & multi-source synthesis*

Your synthesis report combines data from 15 extracted contracts into a portfolio summary. Sources are compressed into a paragraph summary during aggregation. A stakeholder asks "Where did the $2.3M total come from?" and no one can trace it to specific contracts. What architectural change prevents this?

- **A)** Store raw extractions in a separate database for manual lookup.
- **B)** Include all raw extraction data in the final report.
- **C)** Add a disclaimer that figures are approximations.
- **D)** Require the aggregation step to maintain claim-source mappings: each figure in the summary must be traceable to specific contracts with contract IDs, page references, and extraction dates.

<details>
<summary><b>Show answer</b></summary>

**Correct answer: D**

Claim-source mappings maintain provenance through aggregation. Every figure must be traceable to specific contracts.

**Why the others are wrong:**

- **A:** Separate databases require manual lookup.
- **B:** Raw data dumps are unreadable.
- **C:** Disclaimers are evasions.

</details>

---

## Licenses and attribution

Parts B and C reproduce or adapt MIT-licensed works. They are unofficial and not affiliated with or endorsed by Anthropic.

- **Part B:** [hculap/claude-certified-architect-practice-exam](https://github.com/hculap/claude-certified-architect-practice-exam) — Copyright (c) 2026 Szymon Paluch
- **Part C:** curated by [quizmill/pack-claude-cert](https://github.com/quizmill/pack-claude-cert) from [haytamAroui/Claude-Certified-Architect](https://github.com/haytamAroui/Claude-Certified-Architect) — Copyright (c) 2026, and [Connectry-io/connectrylab-architect-cert-mcp](https://github.com/Connectry-io/connectrylab-architect-cert-mcp) — Copyright (c) 2026 Connectry LABS

```
MIT License

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
