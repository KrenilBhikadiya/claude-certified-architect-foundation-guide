# 05 change log (study_guide/05_context_management_reliability.md)

Evidence files are the local copies named in pages_05.txt. Quotes are short (under 20 words).

## Corrections

1. **Mental model, idea 3 (Position matters)**: "Models attend best to the start and end of long inputs" stated as fact -> now attributed to the exam guide ("The exam guide's wording: ...").
   Evidence: official_utf8.txt Task 5.1: "models reliably process information at the beginning and end of long inputs but may omit findings from middle sections". No docs page says this.

2. **5.1 "Why long contexts degrade"**: said the fetched docs "describe the same family of effects as context rot". -> now says the docs do not use "lost in the middle" or describe a middle-of-input effect; they describe context rot and long-prompt layout advice.
   Evidence: context-windows.md: "As token count grows, accuracy and recall degrade, a phenomenon known as context rot" (no position claim); prompting best-practices "Long context prompting" has no middle-of-input claim.

3. **5.1 trimming table, row 2**: "The guide's 'PostToolUse-style hook'" (implied Task 5.1 says it) -> now "The exam guide's PostToolUse hook pattern (Task Statement 1.5 ...) applied to trimming".
   Evidence: exam guide 5.1 never mentions hooks; Task 1.5: "Hook patterns (e.g., PostToolUse) that intercept tool results for transformation".

4. **5.1 PostToolUse paragraph**: "An older field, `updatedMCPToolOutput`, is MCP-only" -> "A second field ... MCP tools only ... prefer `updatedToolOutput`" ("older" is not in the docs). Added: field works for all tools; PostToolUse runs only after successful completion (failures go to `PostToolUseFailure`); `decision: "block"` does not replace output.
   Evidence: hooks.md: "Replaces the output for MCP tools only. Prefer `updatedToolOutput`, which works for all tools"; "Runs immediately after a tool completes successfully"; "Claude still sees the original output; to replace it, use `updatedToolOutput`".

5. **5.1 context-editing paragraph**: added `trigger` can be `tool_uses`; `keep` default is "3 tool uses"; `clear_at_least` semantics corrected ("avoids clearing too little" -> if the API can't clear that much the strategy is not applied); `clear_tool_inputs` default false; added the separate `clear_thinking_20251015` strategy and its `keep` parameter (previously absent, risk of conflation).
   Evidence: context-editing.md options table: "If the API can't clear at least the specified amount, the strategy will not be applied"; "You can specify this value in either `input_tokens` or `tool_uses`"; "`clear_thinking_20251015` strategy manages `thinking` blocks".

6. **5.1 compaction paragraph (conflation fix)**: "Server-side compaction (on-demand, header compact-2026-09-04) is the API's own summariser" -> now distinguishes on-demand compaction (app-triggered, `compaction` parameter, signed block, cannot combine with `context_management`) from token-threshold compaction (API-triggered, `compact_20260112`, separate page not fetched); `instructions` limited to on-demand and up to 16,384 characters. Added a short "Do not conflate" paragraph (compaction vs context editing vs memory tool vs Claude Code `/compact`, `/clear`).
   Evidence: compaction.md table: "You, by sending a request" vs "The API, when input tokens reach the trigger you set"; compaction-on-demand.md: "You can't combine `compaction` with `context_management` on one request"; "up to 16,384 characters"; context-editing.md: "pass the `compact_20260112` edit in the request's `context_management` parameter".

7. **5.1 "Position effects" layout paragraph**: "The prompting best-practices doc agrees on the layout side" -> docs give related but different advice (documents first, query last, tags, quotes); the key-findings-summary-first rule and the position rationale are the exam guide's; the combination is labelled the guide author's suggestion. Also "5-10 facts" marked illustrative; "Two layout techniques from the guide" -> "exam guide".
   Evidence: prompting best-practices: "Place your long documents and inputs near the top of your prompt, above your query"; no mention of key-findings summaries.

8. **5.1 Example C intro and trailer**: matcher naming (`mcp__<server>__<tool>`) noted, server/script names marked illustrative; "'PostToolUse hook' is the same idea registered in code" in the Agent SDK -> reworded to cite exam guide Task 1.5 and mark SDK field names `(verify in docs)` (SDK hooks page not among the assigned pages).
   Evidence: hooks.md "MCP tools follow the naming pattern `mcp__<server>__<tool>`".

9. **5.1 Official docs bullets**: context editing bullet now lists `clear_tool_inputs` and `clear_thinking_20251015`; compaction bullet says on-demand vs token-threshold and that `instructions` is for on-demand; hooks bullet states `updatedToolOutput` is all tools and `updatedMCPToolOutput` MCP-only.

10. **5.2 AskUserQuestion in subagents**: "...not currently available inside subagents, so a coordinator has to relay questions" -> scoped to "subagents spawned via the Agent tool", added the subagents-page fact, and labelled the "coordinator relays" consequence as this guide's inference.
    Evidence: user-input.md: "`AskUserQuestion` is not currently available in subagents spawned via the Agent tool"; sub-agents.md tool-removal list includes `AskUserQuestion`.

11. **5.3 retry classification paragraph**: "Server-side/transient conditions (500, 504, 529) are the ones to retry with exponential backoff" -> only 500 is explicitly "retry with exponential backoff"; 504 docs suggest streaming for long requests; 529 is "temporarily overloaded"; request-side non-retryable conclusion explicitly labelled as reasoning, not a docs label. SDK default retry sentence now matches the docs (backoff, twice, `retry-after`; rate limits also listed but out of scope). Official docs bullet reworded accordingly (no longer says the docs label classes "transient").
    Evidence: errors.md: "500 - `api_error` ... Retry the request with exponential backoff"; "529 - `overloaded_error`: The API is temporarily overloaded"; 504 "Consider using the streaming Messages API".

12. **5.3 Official docs, Subagents bullet**: added the verified "API errors in subagents" behaviour (foreground returns partial output with a note; background marked failed with last output).
    Evidence: sub-agents.md "API errors in subagents".

13. **5.4 subagent delegation paragraph**: Explore described as "fast, read-only"; added that a fork does inherit the conversation; added that subagent results return to the main conversation and many detailed returns can consume significant context. Added a "current docs differ in emphasis" note: best-practices lists "Use subagents to protect your context window" under advice you can stop doing (1M windows, auto compaction), while the exam still teaches isolation.
    Evidence: sub-agents.md "A fast, read-only agent optimized for searching"; "Running many subagents that each return detailed results can consume significant context"; best-practices "Running out of context is rarely the problem anymore".

14. **5.4 crash-recovery paragraph (manifest attribution)**: "which is exactly state export plus injection" -> "in the same spirit ... but it is not the manifest pattern"; added an explicit statement that no docs page describes agents exporting state or a coordinator loading a manifest; added that SDK session files are local to the machine (session-store adapter or moving the file for other hosts).
    Evidence: agent-sdk/sessions.md: "Session files are local to the machine that created them"; "Capture the results you need ... as application state and pass them into a fresh session's prompt".

15. **5.4 Example E**: added "(Exam-guide pattern; the file names and JSON shape below are illustrative, not a documented format.)"

16. **5.4 Official docs bullets**: memory tool bullet now says client-side (Claude requests, application runs); Agent SDK sessions bullet notes it documents session persistence only, not a manifest.
    Evidence: memory-tool.md "The memory tool operates client-side: Claude requests file operations, and your application runs them."

17. **5.5 Q2 answer**: "Around 0.97 (the lowest level that meets the 98% target...)" had a defensible second answer (a level between 0.90 and 0.97 might also meet the target) -> "0.97: the only level in the data that meets 98%; measure intermediate bands before going lower".

18. **5.5 Example B**: threshold comment "calibrated on a labelled validation set" -> "illustrative values; in practice calibrated ..." (values differ from the worked calibration example).

19. **Sources fetched**: removed the stale "not read" caveats (context-editing, compaction-on-demand, sub-agents, hooks) and replaced with what was actually checked; memory-tool entry states which parts were not read line by line; added a note that the token-threshold compaction page was not fetched.

## Verified without change (spot list, not exhaustive)
- Context windows: "everything counts" list, context rot, preserved turns (context-windows.md).
- clear_tool_uses_20250919: header `context-management-2025-06-27`, trigger default 100,000 input tokens, keep default 3, exclude_tools, memory-tool pairing (context-editing.md).
- On-demand compaction header `compact-2026-09-04` and `instructions` replacing the default prompt (compaction-on-demand.md).
- Claude Code what-survives-compaction facts, five files, 5,000-token path references, `/compact` focus example, `/clear` reopen via `/resume` (context-window.md, sessions.md).
- Large-codebase settings: per-directory CLAUDE.md, `claudeMdExcludes`, `Read` deny rules, code-intelligence plugins, plan file re-injected after compaction (large-codebases.md).
- Sessions: resumed sessions do not re-run tool calls interrupted by a crash (sessions.md); sessions persist the conversation not the filesystem; `fork_session`/`forkSession` (agent-sdk/sessions.md).
- Subagents: own context window, "returns only the summary", non-fork sees no parent history, `maxTurns` partial marking and resume, transcripts under `subagents/`, Task renamed Agent in v2.1.63 (sub-agents.md).
- user-input.md: `canUseTool` fires for approvals and `AskUserQuestion`; never fires for auto-approved tools; pending indefinitely; `PreToolUse` `defer`.
- Engineering articles: "smallest possible set of high-signal tokens", "attention budget", structured note-taking, sub-agent summaries 1,000-2,000 tokens, resume-from-where-the-agent-was plus retry logic and checkpoints, CitationAgent, lightweight references to stored artifacts, evaluation section claims.
- Exam sample Q3/Q7/Q8 quotes and option letters; scenario primary-domain list; 5.5 arithmetic (97.06%, 41%, 29%, 97.2%, 45%).
- Prompting best practices: documents at top, "up to 30 percent", tags, quote-first, `tests.json`/`progress.txt`/git, fresh window vs compaction.
- reduce-hallucinations: allow "I don't know", word-for-word quotes, retract unsupported claims, best-of-N, "don't eliminate them".

## Left as unverifiable (and how marked)
- Agent SDK hook API field names for PostToolUse output (SDK hooks page not in my page list): marked `(verify in docs)` in Example C trailer.
- Exact `tool_response` shape for an MCP tool: left as the file's existing "(verify in docs)" comment; the hooks page I read does not specify it for MCP tools.
- Token-threshold compaction details (`compact_20260112` parameters, trigger minimums): page not fetched; the file now says so and gives only what compaction.md and context-editing.md state.
- Auto-memory loading limits (200 lines / 25KB): the file makes no such claim; nothing to verify.
- Statements that are the guide author's own design advice (case-facts block mechanics, `fields` parameter on tools, phase-summary injection, thresholds tables) are exam-guide patterns or illustrations; they were not changed, and the 5.5 note at the end of that section already says the procedures are the exam guide's patterns.
- Memory tool handler implementation and per-command reference (middle ~600 lines of memory-tool.md): not read line by line; the file makes no claims about them.

## Claims checked (approximate)
- A (product/API facts in prose, tables, decision rows): ~70 checked, 12 corrected or tightened.
- B (code/config examples): ~10 blocks checked (hook settings JSON, hook output JSON, subagent frontmatter, scratchpad prompt); 1 marked illustrative (thresholds), hook example matches `updatedToolOutput` shape and settings layout.
- C ("current docs differ" notes): 5 checked; 2 corrected (lost-in-the-middle note, layout "agrees" note); 1 added (subagent-protection emphasis).
- D (doc links / "what to read there"): 25 checked; 5 descriptions adjusted. All URLs are in pages_05.txt.
- E (Check yourself / traps): 18 questions and ~35 traps checked; 1 corrected (5.5 Q2).
- F (attribution): ~12 statements checked; 6 re-attributed (items 1, 3, 7, 8, 14, 15).
- G (quote length): all quotes under about 22 words; none shortened.
