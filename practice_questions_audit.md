# Audit log — alignment with the official exam guide

**Reference:** *Claude Certified Architect – Foundations Exam Guide*, v1.0, effective July 2026 (`Claude+Certified+Architect+–+Foundations+Exam+Guide.pdf`).

## Method

1. Extracted every question (**717** excluding the 12 official samples, which are used verbatim).
2. Twelve independent reviewers each read the full official guide and answered ~60 questions **without seeing the answer key**, recording: their own pick, the task statement tested, whether the topic is in scope / out of scope / beyond the blueprint, and any flaw.
3. Their picks were compared with the keys. I read every disagreement myself and adjudicated it.
4. A question is **kept** only if: in scope, a task statement covers it, the blind pick matched the key, and no flaw was flagged.

**Limits:** reviewers are AI models, so agreement is corroboration, not proof; this audit checks consistency with the guide, not with the real exam's hidden items. The guide is the authority.

## Result

| | Count |
|---|---|
| Audited | 717 |
| **Kept** | **573** |
| Removed | 144 |
| &nbsp;&nbsp;↳ Flawed question | 63 |
| &nbsp;&nbsp;↳ Beyond the blueprint | 50 |
| &nbsp;&nbsp;↳ Key disagrees with guide / blind reviewer | 23 |
| &nbsp;&nbsp;↳ Out of scope per guide | 8 |
| Official samples (verbatim) | 12 |
| Original multiple-response | 33 |
| **Total in the study files** | **618** |

## Where the community guide differs from the official guide

"Community guide" = `guide_en.md` in the open-source repo `paullarionov/claude-certified-architect`, the source of the repo practice questions audited here.

| Topic | Community guide (`guide_en.md`) | Official guide v1.0 |
|---|---|---|
| Scenarios | 8 (incl. *Conversational AI Architecture Patterns*, *Agentic AI Tools*) | **6**; 4 are drawn at random |
| Item format | "1 correct out of 4" | multiple-choice **and multiple-response**; item states how many to select |
| Guessing penalty | "None" | not stated |
| Sample questions | 12, shortened/paraphrased (e.g. "inventory tracking" vs "stock tracking" in Q12) | 12, full text |
| Scenario names | "Customer Support Agent", "Multi-agent Research System" | "Customer Support Resolution Agent", "Multi-Agent Research System" |
| Exam time / fee / validity | not given | 120 min · $125 · 12 months |

## Coverage of the official task statements (kept questions + multiple-response)

| TS | Topic | Questions |
|---|---|---|
| 1.1 | Agentic loops | 19 |
| 1.2 | Coordinator-subagent orchestration | 15 |
| 1.3 | Subagent invocation & context passing | 23 |
| 1.4 | Workflow enforcement & handoff | 19 |
| 1.5 | Agent SDK hooks | 20 |
| 1.6 | Task decomposition | 14 |
| 1.7 | Session state, resumption & forking | 17 |
| 2.1 | Tool descriptions & boundaries | 22 |
| 2.2 | Structured error responses | 22 |
| 2.3 | Tool distribution & tool_choice | 27 |
| 2.4 | MCP server integration | 20 |
| 2.5 | Built-in tools (Read/Write/Edit/Bash/Grep/Glob) | 19 |
| 3.1 | CLAUDE.md hierarchy & modularity | 16 |
| 3.2 | Slash commands & skills | 21 |
| 3.3 | Path-specific rules | 16 |
| 3.4 | Plan mode vs direct execution | 20 |
| 3.5 | Iterative refinement | 12 |
| 3.6 | Claude Code in CI/CD | 26 |
| 4.1 | Explicit criteria & false positives | 23 |
| 4.2 | Few-shot prompting | 21 |
| 4.3 | Structured output via tool use & schemas | 26 |
| 4.4 | Validation, retry & feedback loops | 11 |
| 4.5 | Batch processing | 21 |
| 4.6 | Multi-instance & multi-pass review | 15 |
| 5.1 | Context preservation | 30 |
| 5.2 | Escalation & ambiguity resolution | 30 |
| 5.3 | Error propagation in multi-agent systems | 24 |
| 5.4 | Large-codebase context management | 26 |
| 5.5 | Human review & confidence calibration | 15 |
| 5.6 | Provenance & multi-source synthesis | 18 |

## Removed questions

Listed so nothing disappears silently. "Source" is `repo` (the community guide's practice test, `paullarionov/claude-certified-architect`), `mock` (Paluch) or `community`.

### Key disagrees with guide / blind reviewer (23)

| ID | Source | Why |
|---|---|---|
| `R36` | repo | Guide task 3.2 says personal skill variants use a *different name* to avoid affecting teammates; the keyed same-name override contradicts that wording. |
| `R47` | repo | Guide 1.4 says decompose multi-concern requests and investigate each with shared context; the keyed few-shot-only answer is not clearly the best and a decomposition option is also defensible. |
| `R62` | repo | Both tool-level retry and structured retryable-flag errors (guide 2.2) are defensible; two answers fit. |
| `R68` | repo | Hinges on embeddings / vector retrieval, which the guide lists as out of scope. |
| `R70` | repo | Reminder injection and few-shot rules are both defensible; no task statement covers persona drift. |
| `R72` | repo | Depends on assistant-message prefill, which newer Claude models no longer support. |
| `cca-q-cn-1-5-hard-4` | community | Relies on SDK hook-composition behavior the guide never specifies; unverifiable. |
| `cca-q-cn-1-6-medium-3` | community | Static chain with conditional branches also satisfies the requirement; keyed answer is not uniquely correct. |
| `cca-q-cn-1-7-hard-4` | community | Guide 1.7 prefers a fresh session with a structured summary over stale context; keyed 'edit messages in a fork' is not an established pattern. |
| `cca-q-cn-2-3-medium-1` | community | Ill-formed premise: forcing each subagent's single tool via tool_choice is dubious. |
| `cca-q-cn-2-4-medium-4` | community | Key is illogical: the placeholder in the premise already uses the correct ${VAR} syntax, so 'wrong syntax' cannot be the cause. |
| `cca-q-cn-3-2-hard-2` | community | Same-name user vs project command precedence is not in the guide and differs across Claude Code versions. |
| `cca-q-cn-3-2-medium-3` | community | Same-name user vs project command precedence is not in the guide and differs across Claude Code versions. |
| `cca-q-cn-3-2-medium-5` | community | No option truly enforces read-only (Bash can still write); `context: fork` is irrelevant to that goal. |
| `cca-q-cn-3-3-easy-4` | community | Two options are equivalent valid frontmatter (inline vs block list); guide itself shows the inline form. |
| `cca-q-cn-4-3-medium-5` | community | Guide 4.3 prescribes an 'other' + detail-string pattern for extensible categories; keyed 'just add to the enum' contradicts it. |
| `cca-q-cn-4-5-easy-2` | community | Batch-size limit trivia; not in the guide and the keyed 10,000 figure is outdated. |
| `cca-q-ha-e1-45` | community | Both tool_choice 'any' and forced selection work for a single-tool case (the explanation admits it). |
| `cca-q-ha-e2-36` | community | Guide 1.4 says investigate multi-concern items in parallel with shared context; keyed 'sequential' conflicts with that wording. |
| `cca-q-ha-e2-56` | community | 'Both B and C' answer format; B alone is also correct, so the key is ambiguous. |
| `cca-q-ha-e3-36` | community | Guide 1.4 says investigate multi-concern items in parallel with shared context; keyed 'sequential' conflicts with that wording. |
| `cca-q-ha-e4-02` | community | 'Both B and C' answer format; each alone is valid, so the key is ambiguous. |
| `cca-q-ha-e4-34` | community | Guide 1.4 says investigate multi-concern items in parallel with shared context; keyed 'sequential' conflicts with that wording. |

### Out of scope per guide (8)

| ID | Source | Why |
|---|---|---|
| `cca-q-cn-1-6-hard-2` | community | Balanced trade-off answer, but analysis hinges on rate limits/concurrency quotas, an out-of-scope topic. |
| `cca-q-cn-2-4-hard-4` | community | Hosting/transport deployment is out of scope; SSE now superseded by streamable HTTP; B best reasoning |
| `cca-q-cn-3-6-easy-3` | community | Correct is env var from secrets; main concept is API key handling, out of scope. |
| `cca-q-cn-3-6-hard-3` | community | Company proxy with logging and secrets-manager rotation meets all; key rotation/auth handling is out of scope. |
| `cca-q-cn-3-6-medium-2` | community | Secret handling is the only real option; API key rotation/auth is out of scope; prompt-injection risk not offered. |
| `cca-q-cn-4-2-hard-1` | community | Quantify cost vs benefit of examples; per-token cost calculation is pricing territory (out of scope). A/B rely on fine-tuning. |
| `cca-q-cn-5-5-hard-3` | community | Retraining model weights/confidence head is fine-tuning (out of scope); D's selection-bias reasoning is sound. |
| `cca-q-ha-e3-14` | community | MCP resource access control/authz is auth territory, not covered; access control by underlying systems is most sound. |

### Flawed question (63)

| ID | Source | Why |
|---|---|---|
| `R52` | repo | Flawed question (multiple-defensible): Self-critique fits gaps varying per case, but few-shot (D) is the guide's consistency technique. Self-critique stage is not a guide topic. |
| `R53` | repo | Flawed question (multiple-defensible): Prompting for parallel tool calls in one turn vs composite tool (C); guide covers parallelism only for Task subagent calls. |
| `R60` | repo | Flawed question (multiple-defensible): Targeted 4-6 ambiguous-case examples with rationale per guide. Option A (richer descriptions) is also strong, though the stem presupposes few-shot. |
| `R66` | repo | Flawed question (multiple-defensible): Hybrid summary plus verbatim recent is standard; D (running summary prepended) is essentially the same technique, so options overlap. |
| `R69` | repo | Flawed question (multiple-defensible): Periodic reinforcement is common drift fix, but D (validate and regenerate) is also defensible; guide never covers system-prompt drift. |
| `R75` | repo | Flawed question (ambiguous): Dilution by accumulated turns is the least-wrong cause, but B overlaps and the premise is loosely worded; guide silent. |
| `M21` | mock | Flawed question (multiple-defensible): Forced tool_choice ensures classify runs first (guide example); a blocking prerequisite hook (D) is also deterministic. |
| `M27` | mock | Flawed question (multiple-defensible): Guide's fallback is Read plus Write when Edit lacks unique anchors; Bash sed is also practical and arguably valid. |
| `cca-q-cn-1-2-medium-5` | community | Flawed question (contradicts-guide-premise): Hub-and-spoke rules out debate; but guide 5.6 favors annotating conflicts over silently picking one value. |
| `cca-q-cn-1-3-easy-1` | community | Flawed question (ambiguous): Only allowed_tools resembles real naming; guide places tool limits in AgentDefinition, so premise imprecise. |
| `cca-q-cn-1-4-easy-4` | community | Flawed question (ambiguous): PostToolUse transforms results before model sees them; scenario (model-generated email body) fits PreToolUse arguments better. |
| `cca-q-cn-1-5-medium-1` | community | Flawed question (ambiguous): Global hooks run in order is the intended premise; real SDK uses matchers and may run hooks in parallel. Hook ordering semantics not in guide. |
| `cca-q-cn-1-6-hard-1` | community | Flawed question (multiple-defensible): Parallel fan-out plus adaptive re-trigger is intended; D's static branch for the conditional re-run is also arguably workable. |
| `cca-q-cn-2-2-easy-2` | community | Flawed question (ambiguous): No official MCP taxonomy; A is only sensible mapping |
| `cca-q-cn-2-2-medium-5` | community | Flawed question (multiple-defensible; contradicts-guide-premise): Y avoids leaking SQL; but B is also right: guide says valid empty results are not errors |
| `cca-q-cn-2-3-easy-3` | community | Flawed question (multiple-defensible): Forced tool is precise; with a single tool, any also guarantees a call |
| `cca-q-cn-2-4-medium-3` | community | Flawed question (outdated): Remote server over HTTP/SSE; SSE transport deprecated for streamable HTTP; hosting-adjacent |
| `cca-q-cn-2-5-hard-2` | community | Flawed question (multiple-defensible): Write new files and rewrite original; Bash script or option C also arguable |
| `cca-q-cn-2-5-medium-3` | community | Flawed question (multiple-defensible): Guide starts with Grep for entry points; D equally arguable, C likely intended |
| `cca-q-cn-3-1-hard-1` | community | Flawed question (ambiguous): Directory-level most specific; but Claude Code concatenates files, no strict precedence |
| `cca-q-cn-3-1-hard-2` | community | Flawed question (multiple-defensible): Path-scoped rules; but @import in root defeats conditional loading; D (dir CLAUDE.md with import) arguable |
| `cca-q-cn-3-1-medium-1` | community | Flawed question (multiple-defensible): Most specific scope wins per common docs; but all files are actually loaded additively, so D is defensible. |
| `cca-q-cn-3-2-hard-1` | community | Flawed question (ambiguous): Only A is a true unaddressed concern; B could also be read as an unaddressed concern. Awkward NOT wording. |
| `cca-q-cn-3-2-hard-3` | community | Flawed question (ambiguous): A is the right design; D rationale partly wrong since Bash can also write files. |
| `cca-q-cn-3-3-easy-3` | community | Flawed question (multiple-defensible): YAML list and inline array (B) are both valid YAML; guide example uses inline array. |
| `cca-q-cn-3-3-hard-3` | community | Flawed question (multiple-defensible): A and B name the same two files; both correct, B only adds explanation. |
| `cca-q-cn-3-3-hard-4` | community | Flawed question (multiple-defensible): C is modularity benefit; A is overstated but partly true (all four sets load on any matching file). |
| `cca-q-cn-3-4-medium-3` | community | Flawed question (ambiguous): Explore then clarify is sound; plan mode could also surface the questions. |
| `cca-q-cn-3-5-hard-3` | community | Flawed question (multiple-defensible): Checkpoint with rubric and rollback targets regression; independent critic (C) also defensible per guide. |
| `cca-q-cn-3-6-medium-3` | community | Flawed question (multiple-defensible): JSON output extraction is guide-aligned, but human-review branch (B) is equally defensible as the most important consideration. |
| `cca-q-cn-4-1-easy-1` | community | Flawed question (contradicts-guide-premise): B is the only sensible option, but guide says confidence-threshold filtering is unreliable versus categorical criteria. |
| `cca-q-cn-4-1-hard-1` | community | Flawed question (multiple-defensible): Explicit criteria plus few-shot for borderline cases is best practice; B (rules only) is arguably defensible. |
| `cca-q-cn-4-3-easy-4` | community | Flawed question (outdated): Assistant prefill; not in guide and prefill is unsupported on newer Claude models. |
| `cca-q-cn-4-3-medium-3` | community | Flawed question (ambiguous): Premise says name is required in author, so {} should be invalid; C (add name to inner required) is closest, B nonsensical. |
| `cca-q-cn-4-4-easy-4` | community | Flawed question (contradicts-guide-premise): Only sensible option, but guide detected_pattern is for dismissal/false-positive analysis, not retry-loop detection. |
| `cca-q-cn-4-4-hard-1` | community | Flawed question (contradicts-guide-premise): Error feedback, retry cap and escalation are right; misuses detected_pattern as loop tracker. |
| `cca-q-cn-4-4-hard-3` | community | Flawed question (contradicts-guide-premise): Only option combining multiple relevant signals; detected_pattern misused versus guide. |
| `cca-q-cn-4-5-hard-1` | community | Flawed question (outdated): Multiple batches with custom_id=account ID and result mapping. Premise of 10,000 cap is outdated (real 100,000), which weakens why B is wrong. |
| `cca-q-cn-4-5-medium-2` | community | Flawed question (outdated): Splitting into batches is sensible; the 10,000 limit premise is outdated (real 100,000), but D is still wrong since no auto-split. |
| `cca-q-cn-5-1-hard-1` | community | Flawed question (ambiguous): Re-pinning rules after the log is best fix, but A's claim that position 0 becomes 'middle' is shaky; C is wrong. |
| `cca-q-cn-5-2-easy-2` | community | Flawed question (contradicts-guide-premise): Trivially A, but guide says self-reported confidence scores are unreliable escalation proxies. |
| `cca-q-cn-5-2-easy-4` | community | Flawed question (ambiguous): Policy gap warrants escalation; only escalation option offered is supervisor agent. Guide implies human escalation; supervisor agents not in guide. |
| `cca-q-cn-5-5-hard-1` | community | Flawed question (ambiguous): Softmax/Platt/temperature scaling is ML calibration detail the guide never covers; D is only option acknowledging rare-class gap, none ideal (per-class calibration). |
| `cca-q-cn-5-5-hard-2` | community | Flawed question (multiple-defensible): B and C both put all 2000 reviews on <0.70 bucket prioritized by impact; difference is trivial wording. |
| `cca-q-cn-5-5-medium-3` | community | Flawed question (multiple-defensible): Recalibrate on labeled data then set threshold; raising threshold to 0.9 (A) is also a defensible quick fix. |
| `cca-q-cn-5-6-hard-4` | community | Flawed question (multiple-defensible): Guide favors passing provenance through; C is the low-stakes exploratory case, but D (auditable side-channel for compliance) is arguable. |
| `cca-q-cn-5-6-medium-5` | community | Flawed question (multiple-defensible): Averaging hides conflict and false precision; but B (no source mapping, no dates) is equally arguable; uncertainty expression is not a guide term. |
| `cca-q-ha-e1-26` | community | Flawed question (ambiguous): Coordinator should select subagents dynamically per query; premise says it already selects dynamically yet runs fixed pipeline. |
| `cca-q-ha-e1-35` | community | Flawed question (multiple-defensible): Guide: project commands in .claude/commands/; but .claude/skills/deploy/SKILL.md also creates /deploy in current Claude Code. |
| `cca-q-ha-e1-55` | community | Flawed question (multiple-defensible): Coordinator should dynamically select subagents; C (prompt listing all as required) is a plausible cause too. |
| `cca-q-ha-e1-57` | community | Flawed question (multiple-defensible): Hook stripping (B) and scoped tool (C) both valid; D's 'both' framing is defensible but B alone also plausible. |
| `cca-q-ha-e2-02` | community | Flawed question (multiple-defensible): Guide fallback is Read+Write (B), but a longer unique Edit anchor (C) is equally valid first try. |
| `cca-q-ha-e2-05` | community | Flawed question (multiple-defensible): Community Jira server plus custom deployment, both project-scoped in .mcp.json; A is nearly identical. |
| `cca-q-ha-e2-14` | community | Flawed question (multiple-defensible): A, C and D overlap; hedged both-answer. Subagents have isolated context, need tools and instructions to investigate. |
| `cca-q-ha-e2-32` | community | Flawed question (multiple-defensible): A and B nearly identical (prompt lacks guidance on retaining verified state); A more precise. |
| `cca-q-ha-e2-44` | community | Flawed question (contradicts-guide-premise): Policy here explicitly says no other matching, so not silent; guide's escalate case is silent policy. Deny is sound. |
| `cca-q-ha-e2-55` | community | Flawed question (multiple-defensible): Few-shot for varied document structures per guide; footnote preprocessing (C) also plausible. |
| `cca-q-ha-e3-02` | community | Flawed question (multiple-defensible): Guide fallback is Read+Write; sed -i for URL swap is also defensible, C is guide-aligned. |
| `cca-q-ha-e3-12` | community | Flawed question (multiple-defensible): C and D essentially identical; B wrong because subagents do not inherit context. |
| `cca-q-ha-e3-18` | community | Flawed question (multiple-defensible): Explicit rules with examples fits guide's explicit-criteria principle; a deterministic pricing service (B) is also defensible for guaranteed consistency. |
| `cca-q-ha-e3-21` | community | Flawed question (multiple-defensible): Pass prior state, report only new items (3.6 analog); server-side filtering (C) is also defensible and more deterministic. |
| `cca-q-ha-e3-32` | community | Flawed question (multiple-defensible): History already holds the data; D is more accurate than B, but B and D overlap (both prompt guidance on session state). |
| `cca-q-ha-e4-32` | community | Flawed question (multiple-defensible): History has the content; C is more precise than A, but A and C overlap as prompt-guidance fixes. |

### Beyond the blueprint (50)

| ID | Source | Why |
|---|---|---|
| `R63` | repo | Clarify conflicting stated goals rather than guess; echoes 5.2 ask-instead-of-heuristic principle but no task statement covers preference conflicts. |
| `R71` | repo | Behavioral guidelines belong in system prompt; trivial and not a specific task statement. |
| `R73` | repo | Prefixing the next user message folds the event in naturally without extra turns or system-prompt churn; guide silent on injecting async events. |
| `R74` | repo | Explicit assumptions plus invited correction balances friction and accuracy; hidden defaults are bad. Near-duplicate of R76. |
| `R76` | repo | State assumptions and offer adjustment reduces abandonment; near-duplicate of R74. |
| `M30` | mock | disable-model-invocation is a real SKILL.md flag giving the described behavior, but the guide never mentions it. |
| `cca-q-cn-1-1-hard-2` | community | Atomic hold-then-commit satisfies all-or-nothing; transactional commit semantics not covered by guide. |
| `cca-q-cn-1-2-hard-3` | community | Idempotency keys with uniqueness constraint prevent duplicate writes; idempotency not in guide. |
| `cca-q-cn-1-2-hard-4` | community | Graceful cancellation: cancel side-effect-free, compensate partial, let atomic commit finish; not in guide. |
| `cca-q-cn-1-4-medium-4` | community | Immutable state snapshots avoid in-place stale mutation; guide does not cover state mutability. Version counter (A) is weaker. |
| `cca-q-cn-1-5-hard-1` | community | If Pre hook blocks, tool never ran so no PostToolUse result exists. Hook lifecycle detail not in guide. |
| `cca-q-cn-1-5-hard-3` | community | Single hook resolving tenant from session context and loading rules is cleanest; multi-tenant hooks and SDK context object not covered by guide. |
| `cca-q-cn-1-5-medium-5` | community | Intended premise is sequential composition; real SDK hook chaining semantics are not covered by guide. |
| `cca-q-cn-2-2-hard-2` | community | Leaks username existence and lockout policy; generic message. Auth security not in guide |
| `cca-q-cn-2-3-easy-2` | community | Least privilege; guide frames as selection reliability but principle is standard |
| `cca-q-cn-2-3-hard-1` | community | Isolation, human confirmation, scoped tools; prompt-injection mitigation beyond guide text |
| `cca-q-cn-2-4-hard-1` | community | Real precedence local>project>user; guide silent on name conflicts |
| `cca-q-cn-2-4-medium-1` | community | Resource templates; guide mentions resources only generally |
| `cca-q-cn-2-4-medium-5` | community | Claude Code manages stdio lifecycle; no auto-restart; handle tool errors. Not in guide |
| `cca-q-cn-2-5-hard-3` | community | run_in_background for Bash; flag not in guide |
| `cca-q-cn-3-2-easy-3` | community | $ARGUMENTS placeholder is real, but guide never mentions it. |
| `cca-q-cn-3-2-medium-4` | community | Placeholder substitutes empty string; not covered by guide. |
| `cca-q-cn-3-5-hard-1` | community | Fix upstream structured output contract before tweaking downstream prompt; guide lacks direct statement. |
| `cca-q-cn-3-5-hard-4` | community | Metric misaligned with readability; add rubric-based human criteria as primary signal. ROUGE not in guide. |
| `cca-q-cn-3-5-medium-5` | community | Consolidate into clean system prompt; guide silent on this prompt-hygiene practice. |
| `cca-q-cn-3-6-hard-4` | community | GitHub Actions concurrency cancel-in-progress is standard; guide covers prior findings in context, not concurrency. B partly defensible. |
| `cca-q-cn-3-6-medium-5` | community | Sandboxed container with read-only mount is sound; other options use invented flags. Sandboxing not in guide. |
| `cca-q-cn-4-2-medium-2` | community | Examples can lose influence over long conversations; re-injecting is plausible. Others clearly wrong; not in guide. |
| `cca-q-cn-4-2-medium-4` | community | Clustered same-class examples create bias; interleave. A reverses attention direction. Example ordering not in guide. |
| `cca-q-cn-4-3-hard-2` | community | Prefill gives no schema enforcement; forced tool_use does. Prefilling itself is outside guide. |
| `cca-q-cn-4-3-medium-4` | community | Prefill does not constrain content count; model decides when to close. Prefill outside guide. |
| `cca-q-cn-4-4-easy-2` | community | 2-3 retries is conventional; guide gives no number. |
| `cca-q-cn-4-4-medium-4` | community | Repeated identical tool-call cycle is a structural loop signal; others are arbitrary caps/NL parsing (1.1 anti-patterns). Loop-detection itself not in guide. |
| `cca-q-cn-4-6-easy-4` | community | Low-stakes small artifact with cost priority; other options are high-stakes. Cost/benefit heuristic not explicit in guide. |
| `cca-q-cn-4-6-hard-1` | community | Phase 2 should independently re-examine artifact with structured findings; others are superficial. Bias-propagation design goes beyond guide. |
| `cca-q-cn-4-6-hard-2` | community | Correlated errors make unanimity uninformative; diversity or adversarial reviewers mitigate. Majority-vote limits not in guide. |
| `cca-q-cn-4-6-hard-4` | community | Systemic fix via specialised reviewer and few-shot calibration is best; structured-disagreement mechanism not in guide. |
| `cca-q-cn-4-6-medium-1` | community | 2-of-3 majority vote surfaces the issue. Guide only discusses independent instances, not voting schemes (its Q12 D warns against consensus filtering). |
| `cca-q-cn-4-6-medium-3` | community | Retaining and surfacing both findings to a human fits the stated requirement; mechanism not in guide. |
| `cca-q-cn-4-6-medium-5` | community | Cost vs error-value tradeoff is sound; others use 'always' or false claims. Cost framing not in guide. |
| `cca-q-cn-5-1-medium-2` | community | Fresh start suits topic changes where old context could mislead; other options are false or absolute. |
| `cca-q-cn-5-2-hard-1` | community | Keep hard risk trigger but personalize UX with lightweight confirmation. A is defensible but less nuanced; confidence-threshold trigger itself is shaky per guide. |
| `cca-q-cn-5-2-hard-2` | community | Circuit breaker with degraded mode and ops alert fits systemic staleness; others mask the problem. Pattern not in guide. |
| `cca-q-cn-5-2-hard-3` | community | Safety case: escalate while giving safe guidance and recommending provider. Safety-trigger handling not in guide. |
| `cca-q-cn-5-2-hard-4` | community | Tiered design covering all four requirements; others suppress or ignore needs. Tiered supervisor architecture not in guide. |
| `cca-q-cn-5-2-medium-3` | community | Transparent degradation with alternative channel; others block, hide, or terminate. |
| `cca-q-cn-5-2-medium-5` | community | Supervisor for policy arbitration, human for judgment/accountability; other options are arbitrary. Supervisor agents not in guide. |
| `cca-q-cn-5-3-hard-4` | community | Circuit breaker, backoff with jitter and retry budget stop retry storms; guide does not cover resilience patterns like this. |
| `cca-q-cn-5-6-easy-4` | community | Date-qualified answer with verification advice is sound, but concerns training-data staleness, not multi-source synthesis. |
| `cca-q-ha-e3-20` | community | Inconsistent results from natural-language queries to search tool; explicit structured filters fix it. Guide has no direct task statement. |
