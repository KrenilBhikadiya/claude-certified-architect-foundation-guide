# Claude Certified Architect — Foundations: Study Guide

A read-and-learn guide to **everything the exam tests**, one file per exam domain. Each file explains the concepts, shows worked examples (code, config, prompts, transcripts), points out the typical exam traps, and links to the **current official documentation** (fetched 2026-10-30).

It is built on the **official exam guide v1.0 (July 2026, code CCAR-F)**: every one of the 29 task statements is covered, in the guide's order, with every *Knowledge of* / *Skills in* bullet.

## The files

| # | File | Domain | Weight | Task statements | Lines |
|---|---|---|---|---|---|
| 1 | [01_agentic_architecture_orchestration.md](01_agentic_architecture_orchestration.md) | Agentic Architecture & Orchestration | **27%** | 1.1 – 1.7 | ~1,590 |
| 2 | [02_tool_design_mcp_integration.md](02_tool_design_mcp_integration.md) | Tool Design & MCP Integration | **18%** | 2.1 – 2.5 | ~1,380 |
| 3 | [03_claude_code_configuration_workflows.md](03_claude_code_configuration_workflows.md) | Claude Code Configuration & Workflows | **20%** | 3.1 – 3.6 | ~1,270 |
| 4 | [04_prompt_engineering_structured_output.md](04_prompt_engineering_structured_output.md) | Prompt Engineering & Structured Output | **20%** | 4.1 – 4.6 | ~1,650 |
| 5 | [05_context_management_reliability.md](05_context_management_reliability.md) | Context Management & Reliability | **15%** | 5.1 – 5.6 | ~1,470 |

Each task-statement section has the same six parts: **What the exam tests → Concepts explained → Worked examples → Exam traps → Check yourself (answers hidden) → Official docs.** Each file also opens with a *Mental model* and closes with a *Decision cheat-table* ("if the question says X, prefer Y because…") and the list of pages it was built from.

## The exam in brief (from the official guide)

| | |
|---|---|
| Items | **60**, multiple-choice **and multiple-response** (each item says how many to select) |
| Structure | **4 scenarios drawn at random from 6** |
| Time | 120 minutes, proctored |
| Passing score | scaled **720** on 100–1,000 |
| Fee / validity | $125 USD · 12 months |

### The 6 scenarios and the domains they lean on

| Scenario | Primary domains |
|---|---|
| 1. Customer Support Resolution Agent | D1 Agentic · D2 Tools/MCP · D5 Context |
| 2. Code Generation with Claude Code | D3 Claude Code · D5 Context |
| 3. Multi-Agent Research System | D1 Agentic · D2 Tools/MCP · D5 Context |
| 4. Developer Productivity with Claude | D2 Tools/MCP · D3 Claude Code · D1 Agentic |
| 5. Claude Code for Continuous Integration | D3 Claude Code · D4 Prompting |
| 6. Structured Data Extraction | D4 Prompting · D5 Context |

Because only 4 of 6 scenarios appear, you cannot skip any domain: D1, D2 and D5 together cover three scenarios, D3 covers three, D4 covers two.

## How to study with this folder

1. **Read in order, one file at a time.** Domain 1 carries the most weight and introduces the vocabulary (agentic loop, subagents, hooks, sessions) that the other four domains build on. Budget a few focused hours per file — my estimate; the files are 1,300–1,650 lines each, mostly examples.
2. **Read each section's "Exam traps" before "Check yourself".** The exam is about trade-offs: *most effective*, *first step*, *root cause*. The traps list shows the distractors that keep appearing (prompt-only fixes where enforcement is needed, over-engineering, treating symptoms).
3. **Do the "Check yourself" questions with the answer hidden**, then the practice questions for the same domain (links below).
4. **Finish each file with its decision cheat-table.** When you can predict the right row before reading it, move on.
5. **Type the examples out at least once** (the agentic loop, a hook, a `.mcp.json`, an extraction schema). The exam describes real configurations, and writing them cements the details.

## Practice questions (in the repo root)

- [`../practice_questions_repo.md`](../practice_questions_repo.md) — the 12 official sample questions + aligned repo questions, tagged by task statement.
- [`../practice_questions_extra.md`](../practice_questions_extra.md) — 33 multiple-response drills + ~500 more single-answer questions.
- [`../practice_questions_audit.md`](../practice_questions_audit.md) — what was removed and why, and question counts per task statement.

Question tags like `TS 3.2` match the section numbers in these files, so you can jump from a missed question to the section that explains it.

## Where the current docs differ from the exam's wording

The exam guide predates some documentation changes. **On the exam, answer with the guide's wording**; the notes below explain what you will see in the real product and docs. Each domain file flags these where they matter.

| Topic | Exam guide says | Current docs say | Detail in |
|---|---|---|---|
| Subagent-spawning tool | the **`Task`** tool; `allowedTools` must include `"Task"` | renamed **`Agent`** (Claude Code v2.1.63); `Task(...)` still works as an alias | D1, D2, D5 |
| `fork_session` | fork a saved session into an independent branch | same; but the docs also have a separate "forked subagent" (inherits the parent conversation) — a different feature | D1 |
| Session names | `--resume <session-name>` | CLI: `--name/-n`, `/rename`, `--resume <name>`; the **SDK `resume` option takes a session ID** | D1 |
| Hook "redirect to escalation" | block the call and redirect | a hook can **deny with a reason** or rewrite arguments (`updatedInput`); it cannot itself call another tool — the reason tells Claude to escalate | D1 |
| `tool_choice` | `auto`, `any`, forced tool | also a fourth value `none`; on the newest models (and with manual extended thinking) `any`/forced return a 400 — docs advise `auto` + `strict: true` or `output_config.format` | D2, D4 |
| "Tool use removes syntax errors" | strict JSON schemas via tool use | schema compliance is *guaranteed* only with `strict: true` (or `output_config.format`) | D4 |
| MCP error fields | `isError`, `errorCategory`, `isRetryable` | only **`isError`** is in the protocol; the others are design conventions you add | D2 |
| MCP config scopes | project `.mcp.json` vs user `~/.claude.json` | three scopes: **local** (default), project, user; precedence local → project → user | D2 |
| Tool discovery | all servers' tools available at connection | Claude Code now **defers** MCP tool definitions by default (tool search) — the exam's "all available simultaneously" still holds in effect | D2 |
| Grep / Glob | built-in tools | on macOS, Linux and WSL they are **absent from the default tool set** (Claude uses `find`/`grep` via Bash); default on Windows | D2 |
| `allowed-tools` in a skill | "restricts" tool access | it **pre-approves** tools for that turn and does not restrict others; `disallowed-tools` or deny rules restrict | D3 |
| `argument-hint` | prompts for required parameters | an **autocomplete hint** only; arguments arrive via `$ARGUMENTS` | D3 |
| Commands vs skills | `.claude/commands/` and `.claude/skills/` | commands were **merged into skills**; a skill beats a same-named command; a personal skill beats a same-named project skill (the guide's advice — use a *different* name — still avoids the collision) | D3 |
| CLAUDE.md hierarchy | user / project / directory | also managed policy and `CLAUDE.local.md`; files are **concatenated, not overridden** | D3 |
| Check what's loaded | `/memory` | `/memory` lists/edits memory files; **`/context`** shows what loaded at launch | D3 |
| CI and CLAUDE.md | CLAUDE.md gives CI its context | plain `-p` loads CLAUDE.md; **`--bare`** (recommended for fast scripted runs) skips it | D3, D4 |
| Few-shot count | 2–4 targeted examples | docs suggest 3–5 | D4 |
| Prefill | — | assistant prefill is **no longer supported** on newer models | D4 |
| Batch API and tools | no multi-turn tool calling inside one request | tool definitions are allowed in batches; the limit is no client-side tool round trip within a single request. Current limits: 100,000 requests or 256 MB per batch | D4 |
| `allowedTools` (Agent SDK) | must include `"Task"` for a coordinator to spawn subagents | still the exam answer; but the docs describe it as an **auto-approve list** — unlisted tools stay available; only `disallowedTools` or deny rules block a tool | D1 |
| Who spawns whom | hub-and-spoke: the coordinator manages all subagent communication | subagents **can spawn subagents** by default (up to 3 layers); hub-and-spoke is the design the exam wants, not a product restriction | D1 |
| Hooks and subagents | hooks intercept tool calls | hooks **do fire inside subagents** (input carries `agent_id` / `agent_type`); a hook `deny` holds even in `bypassPermissions`; command/HTTP hooks that crash or time out **don't block** unless `onFailure: "block"` | D1 |
| Plan-mode research | the **Explore** subagent isolates verbose discovery | docs say plan mode delegates research to the read-only **Plan** subagent; Explore is the built-in read-only discovery agent | D3 |
| Context compaction | `/compact` | also API-level **compaction** (on-demand vs token-threshold) and **context editing** (`clear_tool_uses_*`) — separate mechanisms; API details are not exam content | D5 |
| "Use subagents to protect context" | subagent isolation for verbose exploration | the Claude Code best-practices page now calls this advice less important with 1M windows + auto-compaction; the exam still tests the isolation rationale | D5 |
| "Lost in the middle" | the term used by the guide | the docs call the effect **context rot** and give layout advice (documents first, query last) | D5 |
| Self-reported confidence | an unreliable proxy for escalation (5.2) yet used for review routing (4.6, 5.5) | consistent once you separate them: not a substitute for explicit criteria; usable for routing **after calibration** on a labeled set | D4, D5 |

## Verification status

Done on 2026-10-30, after the first drafts:

1. **Full pages, not summaries.** All **65 official pages** cited by the five files were downloaded complete and verbatim (e.g. the 283K-character hooks reference), including the Claude Code docs, the Claude platform docs, the MCP specification (2026-07-28) and four Anthropic engineering articles.
2. **Automated term check.** Every flag, path, field, tool name and identifier written in inline code (about 1,270 terms) was searched for in the full pages and the exam guide. Everything that makes a claim about the real product was found; the remainder are the guide's own example names (e.g. `refund_amount`, `src/billing/refunds.ts`). Specific numbers and versions (v2.1.63, 100,000 requests / 256 MB per batch, 29-day result retention, the 4,096 / 16,384 description cut-offs, 40,000-character rule limit, compaction header, and so on) were confirmed individually.
3. **Five independent fact-checks**, one per file, reading the full pages and editing the files in place: about **1,040 claims** checked and about **120 edits** made (corrections, attribution fixes, softened or relabelled statements, additions). Typical fixes: a claim attributed to "the docs" that only the exam guide makes; an overstated "cannot / only"; a missing condition; a wrong doc link description; an example that implied an API shape. The **evidence for every edit** (what it said → what it says now, the doc file and a short supporting quote) is in [`verification/`](verification/): [D1](verification/01_changes.md) · [D2](verification/02_changes.md) · [D3](verification/03_changes.md) · [D4](verification/04_changes.md) · [D5](verification/05_changes.md).

**What this does *not* guarantee**

- **Some pages were read by section, not end to end** (each log has a "coverage" statement). Largest gaps: `hooks.md` (only the sections Domain 1 needs), `permission-modes.md` (~40%), parts of `mcp.md` (MCP tasks, claude.ai connectors), the Monitor/PowerShell/LSP/WebFetch/WebSearch parts of `tools-reference.md`, `commands.md` (rows only), the code tabs of the API pages, and the middle of `memory-tool.md`. The guide makes no claims that depend on the unread parts, but I cannot prove that.
- **Still marked "(verify in docs)":** the user-scope key shape in `~/.claude.json`; the shape of an MCP `tool_response` inside a PostToolUse hook; what happens when your own skill is literally named `review`; Agent SDK hook field names in one Domain 5 example; which error an oversized batch document produces.
- **Exam-guide patterns have no doc page to check** (case-facts blocks, scratchpads, crash-recovery manifests, stratified sampling, claim-source mappings, escalation criteria). They were checked for consistency with the exam guide and are attributed to it.
- **Examples are illustrations.** Scenario data, tool descriptions in code blocks, retry classifiers and similar are teaching examples; the checks confirmed that API shapes and flags in them match the docs, not that the scenarios exist anywhere.
- **Docs change.** Anything here may be out of date by your exam date.

## About these notes — provenance and limits

- **Checked against the full pages — see [Verification status](#verification-status) above.** The first drafts were written from summarized, partly-read pages; every file was then fact-checked against complete local copies of the official pages (2026-10-30) and corrected. It is still an AI-written study aid: the checks reduce error, they do not make it infallible.
- **Some exam topics have no doc page.** The guide's patterns for *case-facts blocks, scratchpad files, manifest-based crash recovery, stratified-sampling calibration, claim-source mappings and escalation criteria* are architecture practices, not product features. Those sections are built from the exam guide and Anthropic's engineering articles, and say so.
- **Illustrations are illustrations.** A few code samples (retry classification, SLA arithmetic, placeholder server names) are teaching examples written for this guide, not copied from the docs; they are labelled.
- **Paraphrased with links.** These are explanations in my own words with short quotes where wording matters; the official pages remain the authoritative source for exact syntax and current behavior.
- **Out-of-scope topics are not taught** (computer use, vision, streaming, pricing/rate limits, auth, cloud-provider config, prompt-caching internals, token counting, fine-tuning, embeddings/vector DBs, hosting MCP servers). Where docs pages touch them, the files say to skip those parts.
- **Not official.** This folder is an unofficial study aid and is not affiliated with or endorsed by Anthropic. The exam is confidential; nothing here is exam content. If anything conflicts with the official exam guide, **the exam guide wins**.
- **Docs move.** Re-check fast-changing details (model-specific behavior, CLI flags, SDK option names) in the linked pages before test day.
