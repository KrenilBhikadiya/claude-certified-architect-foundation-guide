# Change log: study_guide/03_claude_code_configuration_workflows.md (Domain 3)

Verified against the complete local copies of memory.md, skills.md (read end to end), headless.md, cli-reference.md (flag rows), common-workflows.md, best-practices.md, github-actions.md, gitlab-ci-cd.md, features-overview.md (first ~300 lines), claude-directory.md (first ~260 lines), context-window.md, code-review-local.md, plus targeted reads of sub-agents.md, permission-modes.md and commands.md, and the exam guide (Domain 3 task statements, scenarios 2/4/5, Q4/Q5/Q6/Q10).

## Corrections

| # | Section / location | Was | Now | Evidence |
|---|---|---|---|---|
| 1 | Intro paragraph | "two of the six scenarios lean on it directly ... Scenario 4 also lists it" (self-contradictory count) | three of six scenarios list it as primary domain (2, 4, 5) | exam guide: scenario 2, 4, 5 "Primary domains: ... Claude Code Configuration & Workflows" |
| 2 | 3.1 locations table + Example 1 tree | `CLAUDE.local.md` "(gitignored)" (implies automatic) | "(you add it to `.gitignore`)" | memory.md: "Add `CLAUDE.local.md` to your `.gitignore` so it isn't committed" |
| 3 | 3.1 AGENTS.md bullet | read AGENTS.md when "no CLAUDE.md in the working directory or above" | adds `.claude/CLAUDE.md` and `CLAUDE.local.md` count, `~/.claude/CLAUDE.md` does not, setting `Project instructions` in `/config`, v2.1.277+ | memory.md: "no `CLAUDE.md` or `CLAUDE.local.md` in your working directory or above"; "v2.1.277 or later" |
| 4 | 3.1 `/memory` vs `/context` bullets | `/init` never explained (hotspot) | added a bullet: `/init` generates a starting CLAUDE.md, suggests improvements if one exists | memory.md: "`/init` suggests improvements rather than overwriting it" |
| 5 | 3.1 Example 2 transcript | presented a made-up `/memory` screen as the real output | labelled illustrative; real `/memory` is an interactive list; `/context` shows Memory files | memory.md: "`/memory` command lists your CLAUDE.md ... Select any file to open it"; "run `/context` ... Memory files" |
| 6 | 3.2 sample-Q4 caveat | "(verify in docs)" with the alias claim only half-sourced | cites both docs passages; states the docs do not say what happens for a file literally named `review` | skills.md: "Your skill replaces the bundled command, but not its aliases"; code-review-local.md: "the `/review` alias keeps running the bundled review" |
| 7 | 3.2 `context: fork` section | forked subagent runs in background by default (no exceptions) | adds narrower tool set and that `-p`/Agent SDK runs wait for the result | skills.md: "Claude Code also waits for the result ... In non-interactive mode, with the `-p` flag" |
| 8 | 3.2 allowed-tools note | "values are space- or comma-separated lists in the docs' examples" | accepts a space- or comma-separated string or a YAML list | skills.md frontmatter table: "Accepts a space- or comma-separated string, or a YAML list" |
| 9 | 3.2 `argument-hint` | examples `[component] [from] [to]` presented as the docs' | docs' own examples `[issue-number]`, `[filename] [format]` | skills.md: "Example: `[issue-number]` or `[filename] [format]`" |
| 10 | 3.2 Example 2b lead-in | "limit to writes, no shell" (implies `allowed-tools` removes Bash) | states it only pre-approves Write/Edit; Bash stays available unless `disallowed-tools`/deny rules | skills.md: "It does not restrict which tools are available: every tool remains callable" |
| 11 | 3.3 size limits bullet | "40,000 characters on a 200K window; verify exact current limits" | full statement: 40,000 chars (200K window), 150,000-200,000 (1M, model dependent), applies to on-demand files (path-scoped rules, subdirectory CLAUDE.md), only the start is delivered; launch files: 4 MiB | memory.md "Claude ignores the end of my subdirectory CLAUDE.md": "200K context window: 40,000 characters" |
| 12 | 3.4 plan mode details | no statement of what plan mode does not block | added: blocks kept in `-p`; not enforced in interactive sessions with bypass permissions available | permission-modes.md: "Claude Code also doesn't enforce plan mode's blocks" (bypass sessions); "Plan mode keeps its blocks ... with `-p`" |
| 13 | 3.4 combine plan + direct, step 1; Example 1 | plan mode delegates discovery to "Explore" | exam names Explore; docs say plan mode delegates research to the Plan subagent; Example 1 made generic | sub-agents.md: "When you're in plan mode ... it delegates research to the Plan subagent" |
| 14 | 3.4 "Costs to know" paragraph | best-practices "notes that ... 1M windows make 'protect your context window' less urgent" | attributed to the "What you can stop doing" table, wording corrected | best-practices.md: "Running out of context is rarely the problem anymore" |
| 15 | 3.4 workflow sentence | "The docs' Best practices describe the same explore -> plan -> implement loop" (page has no such section) | Common workflows "Plan before editing" recipe + Best practices "give Claude a check"; plan-then-execute framing attributed to the exam guide | grep of best-practices.md: no "plan mode" text; common-workflows.md "## Plan before editing" |
| 16 | 3.4 Official docs line for best-practices | "explore/plan/implement, subagent costs" | actual content: "Know what subagents cost", independent review, "What you can stop doing"; notes no plan-mode section | best-practices.md headings |
| 17 | 3.5 interview prompt | full 29-word quote | shortened to the final sentence (13 words) plus paraphrase | best-practices.md: "Interview me until you could write the spec, then write it to SPEC.md." |
| 18 | 3.5 input/output examples | implied doc backing | marked as exam-guide technique not presented as a named technique in the docs | grep of best-practices/common-workflows: no input/output-examples advice |
| 19 | 3.5 one message vs sequential | no attribution | marked "Exam-guide pattern; design advice" | same grep: no docs rule on batching fixes |
| 20 | 3.5 steering sentence | "advice from the docs" | "from the docs' best-practices page" | best-practices.md "Steer the session" |
| 21 | 3.6 flags table, `--output-format` row | stream-json with no extra flag | notes docs' examples add `--verbose` | headless.md: "`--output-format stream-json` with `--verbose` and `--include-partial-messages`" |
| 22 | 3.6 flags table, permission row | `dontAsk` only | adds that `-p` starts in Manual mode (v2.1.302+) so unapproved calls are denied | permission-modes.md: "`claude -p` ... start in Manual mode ... Calls needing approval are denied" |
| 23 | 3.6 flags table, `--max-turns` row | caps only | adds exits with an error at the limit | cli-reference.md: "Exits with an error when the limit is reached" |
| 24 | 3.6 `--bare` note | no auth caveat | bare mode never reads OAuth/keychain; needs `ANTHROPIC_API_KEY` etc. | headless.md: "bare mode doesn't use your subscription login" |
| 25 | 3.6 CLAUDE.md as CI context | docs say Claude follows CLAUDE.md "when reviewing or creating PRs" | "when creating PRs/MRs and responding to requests" | github-actions.md: "Claude follows these guidelines when creating PRs and responding to requests" |
| 26 | 3.6 reviewer-subagent quote | spliced quote "report only the gaps that affect correctness and skip style" | exact fragment "report only the gaps that affect correctness"; skip style paraphrased | best-practices.md reviewer.md: "report only the gaps that affect correctness. Skip style." |
| 27 | 3.6 platform integrations | both pages say every change flows through PR/MR with branch protection | only GitLab says that; GitHub says review changes before merging | gitlab-ci-cd.md: "Every change flows through an MR ... approvals still apply"; github-actions.md: "review Claude's changes before merging" |
| 28 | 3.6 Example 5 workflow | `pull-requests: write` presented as adapted docs example | `pull-requests: read` (as the docs' example) | github-actions.md review workflow: "pull-requests: read" |
| 29 | 3.6 Example 2 `gh api` comment | "shown for illustration" | flagged ILLUSTRATIVE, not from Claude docs, in the code comment | grep: no `gh api` in any provided page |
| 30 | Sources fetched | said middle of skills, tails of sub-agents/permission-modes unread | updated to the verification actually done; commands.md still search-only | this review |

## Claims checked and found accurate (no change)
CLAUDE.md locations/managed paths, load order and concatenation, "Claude may pick one arbitrarily", lazy subdirectory loading, `@import` four hops/relative to importing file/code-span skip/external approval dialog, HTML comment stripping, `claudeMdExcludes` (managed cannot be excluded), `.claude/rules/` recursion, `paths` as the only rule field, YAML list or comma string, glob table, brace expansion, user rules before project rules, symlinks, rules reload on demand after compaction, plan re-injected after compaction, 200-line guidance, auto memory 200 lines/25 KB; skills frontmatter keys and meanings (name, description, argument-hint, allowed-tools, disallowed-tools, context, agent, disable-model-invocation, user-invocable, model, effort, hooks, paths, shell, when_to_use, arguments, background), commands merged into skills, skill wins over same-named command, enterprise > personal > project, namespacing `/frontend:component`, `$ARGUMENTS`/`$0`/`$1`/ARGUMENTS-append, <500 lines, `context: fork` isolated/no history/Explore and Plan lack CLAUDE.md; plan mode entry (Shift+Tab, `/plan`, `--permission-mode plan`), approval options, Ctrl+G, `defaultMode: plan`; Explore read-only (Write/Edit denied) with quick/medium/very thorough; Task renamed Agent in v2.1.63; `-p`/`--print`, exit codes, 10 MB stdin cap, `--output-format` values, `--json-schema` (print mode only, `structured_output`, absent on no valid output, `format` annotation only), `--allowedTools`, `--append-system-prompt(-file)`, `--continue`/`--resume`/`--fork-session`, `--no-session-persistence`, `--bare` skips CLAUDE.md; GitHub Action `anthropics/claude-code-action@v1`, `prompt` automation mode, `claude_args`, secrets, `/install-github-app`, checkout@v6, inline-comment allowedTools; GitLab beta/maintained by GitLab, acceptEdits + allowedTools command, masked `ANTHROPIC_API_KEY`; Q4/Q5/Q6/Q10 answers vs exam guide.

## Claims checked: approximate counts by category
- A (product/API facts in prose, tables, decision rows): ~150 checked, 24 corrected/softened
- B (code and config examples): ~20 blocks checked (frontmatter blocks, import examples, CI script, GitHub workflow, JSON schema); 3 corrected/labelled
- C ("Exam wording vs current docs" notes): 7 checked; 1 corrected (/review alias), others accurate
- D (doc links/descriptions): ~30 links checked; 1 description corrected (best-practices)
- E (Check yourself / Exam traps): ~40 checked; none contradict exam guide or docs; no change needed
- F (attribution): ~10 statements adjusted (3.4 best-practices claim, 3.5 pattern attributions, GitHub/GitLab merge claim, illustrative gh api)
- G (quote length): 1 quote shortened (interview prompt); 1 spliced quote fixed

## Left as unverifiable
- Sample-Q4 `/review` precedence when a user file is literally named `review`: docs silent; left marked `(verify in docs)` and phrased as untested.
- "A file can match several rules; all matching rules load" (3.3): follows from per-rule matching described in memory.md but not stated verbatim; left as is (low risk).
- Explore's model: not stated in the guide; docs say it follows the main conversation's model with provider-dependent exceptions; not added.
- The CI example script and its `--allowedTools "Read,Grep,Glob"` flags are illustrative (not a docs example); flags exist but the combination was not executed.
- `/memory` UI exact appearance: labelled illustrative.

## Not read in full (be candid)
- commands.md: only the rows for /plan, /memory, /context, /init, /code-review, /review, /clear, /compact and the intro were read.
- permission-modes.md: read lines 1-363 and 625-725; skipped auto-mode classifier detail and critical-path sections (not Domain 3).
- sub-agents.md: read 1-95, 419-510, 834-1473 (includes the previously unread tail); skipped quickstart, frontmatter reference, model choice, MCP scoping, hooks, memory sections.
- features-overview.md: read to line ~300 (context-cost tabs for subagents/hooks tail not needed); claude-directory.md read to line ~260 (application-data/purge tail not relevant).
