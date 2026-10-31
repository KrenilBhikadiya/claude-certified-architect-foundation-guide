# Domain 3: Claude Code Configuration & Workflows (20%)

> This domain is about *operating Claude Code well as a team*: where instructions live and who they reach (CLAUDE.md hierarchy, imports, `.claude/rules/`), how reusable workflows are packaged (slash commands and skills), when to plan before acting (plan mode, the Explore subagent), how to steer Claude toward a correct result (iterative refinement), and how to run it unattended in CI/CD (`claude -p`, structured JSON output). It carries 20% of the exam, and three of the six scenarios list it as a primary domain: **Scenario 2 (Code Generation with Claude Code)**, **Scenario 4 (Developer Productivity)** and **Scenario 5 (Claude Code for Continuous Integration)**. Sample questions Q4, Q5, Q6 and Q10 in the official guide are all from this domain.

## Contents

- [Mental model](#mental-model)
- [3.1 Configure CLAUDE.md files with appropriate hierarchy, scoping, and modular organization](#31-configure-claudemd-files-with-appropriate-hierarchy-scoping-and-modular-organization)
- [3.2 Create and configure custom slash commands and skills](#32-create-and-configure-custom-slash-commands-and-skills)
- [3.3 Apply path-specific rules for conditional convention loading](#33-apply-path-specific-rules-for-conditional-convention-loading)
- [3.4 Determine when to use plan mode vs direct execution](#34-determine-when-to-use-plan-mode-vs-direct-execution)
- [3.5 Apply iterative refinement techniques for progressive improvement](#35-apply-iterative-refinement-techniques-for-progressive-improvement)
- [3.6 Integrate Claude Code into CI/CD pipelines](#36-integrate-claude-code-into-cicd-pipelines)
- [Decision cheat-table](#decision-cheat-table)
- [Sources fetched](#sources-fetched)

## Mental model

Think of Claude Code configuration as answering four questions about every piece of guidance you want Claude to follow:

1. **Who must receive it?** Only me (user level), the whole team (project level, committed to git), everyone in the company (managed policy)? The exam's favourite bug is guidance placed at the *wrong scope*, e.g. a rule in `~/.claude/CLAUDE.md` that teammates never receive.
2. **When must it be in context?** Always (CLAUDE.md, unconditional rules), only when touching certain files (`.claude/rules/` with `paths:`), or only when a task is invoked (skills / slash commands)? Loading everything always wastes context and dilutes adherence.
3. **Does it need isolation?** A skill that produces lots of output or explores alternatives can run in a forked subagent (`context: fork`) so the main conversation stays clean. Verbose discovery goes to the Explore subagent.
4. **Is "usually follows it" good enough?** CLAUDE.md, rules and skills are *context* the model reads, not guaranteed enforcement. Anything that must always happen belongs in a hook or permission rule (that is Domain 1/2 territory, but the docs repeat it here: the memory page says CLAUDE.md is "context, not enforced configuration").

Rules of thumb for the exam:

- **Project-shared = in the repo.** `.claude/commands/`, `.claude/skills/`, project `CLAUDE.md` / `.claude/CLAUDE.md`, `.claude/rules/` are version-controlled and reach everyone who clones. `~/.claude/...` is personal and reaches only you.
- **Hierarchy is additive.** CLAUDE.md files at different levels are *concatenated* into context, not overridden. More specific (closer to your working directory) is read later.
- **Use `.claude/rules/` with `paths:` globs for conventions tied to file *types* that are scattered across directories** (all `*.test.tsx`). Use a directory-level CLAUDE.md only when the convention belongs to one directory tree.
- **Skills = on-demand; CLAUDE.md = always-loaded.** Universal standards go in CLAUDE.md; task-specific workflows go in skills / commands.
- **Plan mode for big, ambiguous, architectural, multi-file work; direct execution for small, well-scoped changes.** The complexity is known up front; do not "start direct and switch if it gets hard".
- **Show, don't describe.** When prose is interpreted inconsistently, give 2–3 concrete input/output examples; write tests first and feed failures back; interview first in unfamiliar domains.
- **In CI always use `-p`.** Add `--output-format json` and `--json-schema` for machine-parseable findings; put project standards in CLAUDE.md; review with an *independent* instance, not the session that wrote the code.

---

## 3.1 Configure CLAUDE.md files with appropriate hierarchy, scoping, and modular organization

### What the exam tests

- **Knowledge**: the three hierarchy levels the guide names: user-level (`~/.claude/CLAUDE.md`), project-level (`.claude/CLAUDE.md` or root `CLAUDE.md`), and directory-level (CLAUDE.md files in subdirectories). User-level instructions apply only to that user and are **not** shared through version control. The `@import` syntax keeps CLAUDE.md modular. `.claude/rules/` is the topic-specific alternative to one monolithic CLAUDE.md.
- **Skills**: diagnosing hierarchy problems (the classic: a new teammate does not get the instructions because they live in user-level config); using `@import` so each package's CLAUDE.md pulls in only the standards relevant to it, based on what the maintainer knows about that package; splitting a big CLAUDE.md into focused files under `.claude/rules/` (`testing.md`, `api-conventions.md`, `deployment.md`); using `/memory` to see which memory files are loaded and to debug behaviour that differs between sessions.

### Concepts explained

#### What CLAUDE.md is (and is not)

Each Claude Code session starts with a fresh context window. CLAUDE.md files are markdown files you write; Claude reads them at the start of every session as persistent instructions. The docs are explicit that the content is delivered as a *user message after the system prompt*, so Claude "treats CLAUDE.md files as context, not enforced configuration". Specific, concise, non-contradictory instructions are followed more reliably ("Use 2-space indentation" beats "Format code properly"). If something must happen with zero exceptions, use a hook or a deny rule instead.

Good candidates for CLAUDE.md (per the docs): build/test commands, conventions, project layout, "always do X" rules, things you have had to correct twice. Aim for **under 200 lines per file**; longer files consume context and reduce adherence.

#### The locations and who they reach

The memory docs list the scopes from broadest to most specific (load order):

| Scope | Location | Shared with | Typical content |
|---|---|---|---|
| Managed policy | macOS `/Library/Application Support/ClaudeCode/CLAUDE.md`; Linux/WSL `/etc/claude-code/CLAUDE.md`; Windows `C:\Program Files\ClaudeCode\CLAUDE.md` | Everyone in the organisation (deployed by IT/MDM; cannot be excluded by `claudeMdExcludes`) | Company standards, compliance reminders |
| User | `~/.claude/CLAUDE.md` | **Only you**, across all projects | Personal style preferences, shortcuts |
| Project | `./CLAUDE.md` or `./.claude/CLAUDE.md` | The team, via source control | Architecture, coding standards, workflows, build/test commands |
| Local | `./CLAUDE.local.md` (you add it to `.gitignore`) | Only you, this project | Your sandbox URLs, personal test data |
| Directory (subdirectory) | `some/dir/CLAUDE.md` | The team, if committed | Conventions specific to that subtree |

Key behaviours:

- **Additive, not overriding.** Claude Code loads `CLAUDE.md` and `CLAUDE.local.md` from the working directory and every directory above it; all discovered files are *concatenated*. Ordering runs from the filesystem root down to the working directory, so instructions closest to where you launched are read last. If two files contradict each other, "Claude may pick one arbitrarily", so keep them consistent.
- **Subdirectory files load on demand.** CLAUDE.md files in subdirectories *below* the working directory are not loaded at launch; each loads once Claude reads, writes or edits a file in that subdirectory. This is why a subdirectory CLAUDE.md does not show up under "Memory files" in `/context` at startup.
- **User-level is personal.** Because `~/.claude/CLAUDE.md` lives in your home directory, not in the repo, it is never committed and never reaches teammates. That is the heart of the diagnostic skill below.
- **Block-level HTML comments** (`<!-- notes for maintainers -->`) are stripped before injection, so maintainers can leave notes without spending tokens.
- **AGENTS.md.** By default (the **Project instructions** setting in `/config`; requires Claude Code v2.1.277 or later), Claude Code reads `AGENTS.md` only when there is no `CLAUDE.md`, `.claude/CLAUDE.md` or `CLAUDE.local.md` in the working directory or above (your `~/.claude/CLAUDE.md` does not count); if both kinds exist, only the CLAUDE.md files are read. The usual bridge is a `CLAUDE.md` containing `@AGENTS.md` plus Claude-specific additions. (Peripheral to the exam; know it exists.)
- **Large monorepos.** `claudeMdExcludes` (in a settings file) lets you skip other teams' CLAUDE.md files by path/glob.

> Exam wording vs. current docs: the guide names three levels (user, project, directory). The current docs add **managed policy** and **CLAUDE.local.md** levels. Answer exam questions with the guide's three-level model, but know the extras exist.

#### `@import` for modular CLAUDE.md

Inside any CLAUDE.md you can pull in other files with `@path/to/file`. Imported files are expanded into context alongside the file that imports them.

- Relative paths resolve relative to **the file containing the import**, not the working directory; absolute paths and `@~/...` home paths also work.
- Imports can nest up to **four hops** deep.
- Import parsing skips code spans and fenced code blocks, so wrap a path in backticks to mention it without importing it.
- An import that resolves outside the working directory (e.g. in your home directory) triggers a one-time approval dialog for project-level files.
- **Important nuance:** imports "help organise a long file but don't reduce its context cost", because imported files load together with the importing CLAUDE.md. Imports give *modularity and maintainability*, not token savings. For token savings use path-scoped rules or skills.

Why does the guide stress "selectively include relevant standards files in each package's CLAUDE.md based on maintainer domain knowledge"? In a monorepo, the person who maintains the `payments` package knows it needs `standards/security.md` and `standards/api-versioning.md` but not the React style guide. Each package's CLAUDE.md imports only what applies, so package A's agents are not burdened with package B's rules, and the standards live in one place (single source of truth) rather than being copy-pasted.

#### `.claude/rules/`: topic-specific files instead of one monolith

Put markdown files in `.claude/rules/`, one topic per file (`testing.md`, `api-conventions.md`, `deployment.md`). All `.md` files are discovered recursively, so you can create subfolders such as `frontend/` or `backend/`. Rules **without** a `paths:` frontmatter load at launch with the same priority as `.claude/CLAUDE.md`; rules **with** `paths:` load conditionally (that is task statement 3.3). Personal rules can live in `~/.claude/rules/` (apply to all your projects); user rules load before project rules, and neither overrides the other.

Why prefer rules over a monolith? A 600-line CLAUDE.md is hard to maintain (many authors, merge conflicts, stale sections) and weakens adherence. Splitting by topic gives each file an owner, makes the reviewable diff small, and, with `paths:`, allows loading only when relevant. Symlinks are supported, so a shared rules directory can be linked into many repos.

#### `/memory` and `/context` for diagnosis

- `/memory` lists your CLAUDE.md, CLAUDE.local.md and other memory file locations (user and project scopes, including ones that do not exist yet), lets you open any in your editor, and toggles **auto memory**.
- `/context` shows a live breakdown of context usage including the list under **Memory files** — the files that loaded at launch.
- `/init` generates a starting CLAUDE.md from your codebase (build commands, test instructions, conventions it discovers); if a CLAUDE.md already exists it suggests improvements rather than overwriting it. It creates content; it is not a diagnostic command.
- The `InstructionsLoaded` hook can log which instruction files load, when, and why (useful for path-scoped rules).

> Exam wording vs. current docs: the guide says "use `/memory` to verify which memory files are loaded". Current docs say `/memory` lists/edit memory files and the docs' troubleshooting steps point to **`/context` → Memory files** as the check for "what actually loaded at launch". For exam answers, `/memory` is the expected command; in real life use both.

**Auto memory** (not in the guide's bullets but part of the same page): Claude can write its own notes (`MEMORY.md` index plus topic files under `~/.claude/projects/<project>/memory/`), machine-local, with only the first 200 lines / 25 KB of `MEMORY.md` loaded at session start. It is not shared with teammates either. Not a substitute for committed CLAUDE.md.

#### Diagnosing "new teammate isn't getting the rules"

Symptom: Alice's Claude follows the team's testing conventions; Bob, who just cloned the repo, gets different behaviour. Reasoning:

1. What differs between Alice's and Bob's setups? Not the repo (identical clone). So the rules must live outside the repo.
2. Where would instructions live outside the repo? `~/.claude/CLAUDE.md` (user level), `~/.claude/rules/`, `CLAUDE.local.md` (gitignored), or auto memory — all personal.
3. Fix: move the shared instructions into the **project level** (`./CLAUDE.md`, `./.claude/CLAUDE.md`, or `.claude/rules/`) and commit them. Verify on Bob's machine with `/memory` (or `/context`).

### Worked examples

#### Example 1: The full hierarchy and who it reaches

```text
Machine (Windows/macOS/Linux)
│
├── [managed policy] /etc/claude-code/CLAUDE.md          -> every user on the machine (IT-deployed)
│
├── ~/.claude/                                           -> ONLY ME, all projects (never in git)
│   ├── CLAUDE.md          "I prefer concise answers; use my sandbox URL"
│   └── rules/preferences.md
│
└── ~/code/shop/                       (git repository, shared with the team)
    ├── CLAUDE.md                      -> TEAM: build/test commands, architecture, "always do X"
    ├── CLAUDE.local.md                -> ONLY ME, this repo (you add it to .gitignore)
    ├── .claude/
    │   ├── CLAUDE.md                  -> alternative location for project CLAUDE.md (TEAM)
    │   └── rules/
    │       ├── testing.md             -> TEAM, always loaded (no paths:)
    │       ├── api-conventions.md     -> TEAM, only when matching files touched (paths:)
    │       └── deployment.md
    ├── packages/
    │   ├── payments/
    │   │   └── CLAUDE.md              -> TEAM, loaded on demand when Claude touches files here
    │   └── storefront/
    │       └── CLAUDE.md              -> TEAM, loaded on demand when Claude touches files here
    └── docs/
```

Reading order in context when you start Claude in `~/code/shop/`: managed → user → project CLAUDE.md files from root down to the working directory (with `CLAUDE.local.md` after `CLAUDE.md` at each level). Subdirectory files (`packages/payments/CLAUDE.md`) join later, on demand.

#### Example 2: Diagnosing the "new teammate" case (bad vs good)

Bad (rules in the personal file, so only the author gets them):

```markdown
<!-- ~/.claude/CLAUDE.md  (lives in Alice's home directory, NOT in git) -->
# Team testing standards
- Use Vitest, never Jest.
- Every new endpoint needs an integration test in tests/api/.
- Use factory helpers in tests/factories.ts, never hand-built fixtures.
```

Good (same rules committed at project level):

```markdown
<!-- ./CLAUDE.md  (committed; every clone gets it) -->
# Testing standards
- Use Vitest, never Jest.
- Every new endpoint needs an integration test in tests/api/.
- Use factory helpers in tests/factories.ts, never hand-built fixtures.

# Commands
- Run one test file: pnpm vitest run path/to/file.test.ts
```

Verification transcript on the new teammate's machine (illustrative only: the real `/memory` screen is an interactive list of memory files, not this exact text; `/context` also shows the loaded files under **Memory files**):

```text
> /memory
  User memory       ~/.claude/CLAUDE.md                (not present)
  Project memory    ./CLAUDE.md                        (loaded)
> What test framework should I use here?
  Vitest, per the project's CLAUDE.md. ...
```

#### Example 3: A modular root CLAUDE.md using `@imports`

```markdown
# Shop monorepo

See @README.md for the project overview and @package.json for npm scripts.

## Always-on rules
- TypeScript strict mode everywhere; no `any` without a comment.
- Run `pnpm typecheck` and `pnpm test` before declaring work done.

## Shared standards (single source of truth, imported where relevant)
- Git workflow: @docs/standards/git-workflow.md
- Error handling: @docs/standards/error-handling.md
```

A package-level CLAUDE.md that imports only what that package needs (maintainer knowledge decides what):

```markdown
<!-- packages/payments/CLAUDE.md -->
# Payments package
Handles card charges and refunds. Money is integer cents, never floats.

- Security and PCI rules: @../../docs/standards/security.md
- API versioning rules: @../../docs/standards/api-versioning.md
- Idempotency conventions: @./docs/idempotency.md
<!-- Deliberately NOT importing the React style guide: this package has no UI. -->
```

Note the relative paths resolve from the file that contains the import (`packages/payments/`), not from where Claude was launched.

#### Example 4: Splitting a monolith into `.claude/rules/`

Before: one 700-line `CLAUDE.md` mixing testing, API, deployment and Terraform notes. After:

```text
.claude/
├── CLAUDE.md              (40 lines: overview, build/test commands, links)
└── rules/
    ├── testing.md         (always-on testing standards)
    ├── api-conventions.md (always-on or path-scoped, see 3.3)
    └── deployment.md      (release/deploy checklist notes)
```

```markdown
<!-- .claude/rules/testing.md  (no frontmatter -> loaded at launch, same priority as .claude/CLAUDE.md) -->
# Testing conventions
- Name tests "should <behaviour> when <condition>".
- Use factories from tests/factories.ts; never inline large fixtures.
- A bug fix needs a regression test that fails before the fix.
```

### Exam traps

- **"Put it in `~/.claude/CLAUDE.md` so it is always available"** when the question says the whole team needs it. Wrong: user-level is personal and not version-controlled.
- **"Directory-level CLAUDE.md overrides the project-level one."** Wrong: files are concatenated; conflicts are resolved by the model's judgement, which is why you avoid contradictions.
- **"`@import` reduces token usage."** Wrong: imported files load with the importing file. Imports aid organisation; path-scoped rules and skills reduce always-on context.
- **"Put the standards file path in CLAUDE.md as prose and Claude will read it."** Unreliable; use an explicit `@import` so it is actually loaded.
- **"Use CLAUDE.md to guarantee a rule."** CLAUDE.md is guidance, not enforcement (hooks / deny rules enforce).
- **"Create `.claude/config.json` with a rules array."** No such mechanism (compare sample Q4 option D).
- **Inconsistent behaviour across sessions/people** → the root cause is usually *which files loaded*, so the first diagnostic step is `/memory` (or `/context`), not rewriting the prompt.

### Check yourself

1. Your team agreed that all API handlers must use a shared error helper. Only you see Claude follow this; two colleagues do not. You wrote it in `~/.claude/CLAUDE.md`. What is the fix and how do you verify it?
   <details><summary>Answer</summary>Move the instruction to project level (`./CLAUDE.md`, `./.claude/CLAUDE.md`, or a file under `.claude/rules/`) and commit it. User-level config applies only to you and is not shared via version control. Verify on a teammate's machine with `/memory` (the guide's command; `/context` also lists loaded memory files).</details>
2. A monorepo maintainer wants the `payments` package to follow the security and API-versioning standards but not the React guide, while keeping a single copy of each standard. What mechanism fits?
   <details><summary>Answer</summary>`packages/payments/CLAUDE.md` with `@import` lines pointing at only those two shared standards files. (Remember imports organise content; they do not reduce its context cost.)</details>
3. True or false: when `~/.claude/CLAUDE.md` and the project `CLAUDE.md` give conflicting guidance, the project file wins automatically.
   <details><summary>Answer</summary>False. All discovered files are concatenated into context; neither overrides the other, and with conflicts Claude may follow either. Keep them consistent. (The project file is merely read later.)</details>

### Official docs

- [How Claude remembers your project (memory)](https://code.claude.com/docs/en/memory.md) — locations table, how files load, `@import`, `.claude/rules/`, `/memory`, AGENTS.md, troubleshooting.
- [Explore the .claude directory](https://code.claude.com/docs/en/claude-directory.md) — what lives where, which files to commit, frontmatter fields per file type.
- [Extend Claude Code (features overview)](https://code.claude.com/docs/en/features-overview.md) — CLAUDE.md vs rules vs skills, and how features layer.

---

## 3.2 Create and configure custom slash commands and skills

### What the exam tests

- **Knowledge**: project-scoped commands in `.claude/commands/` (shared through version control) vs user-scoped commands in `~/.claude/commands/` (personal); skills in `.claude/skills/` with a `SKILL.md` whose frontmatter supports `context: fork`, `allowed-tools`, and `argument-hint`; what `context: fork` does (runs the skill in an isolated sub-agent so its output does not pollute the main conversation); creating *personal variants* of a skill under `~/.claude/skills/` with a *different name* so teammates are unaffected.
- **Skills**: creating team-wide commands in `.claude/commands/`; using `context: fork` for verbose output (codebase analysis) or exploratory work (brainstorming alternatives); configuring `allowed-tools` in skill frontmatter to limit tools during the skill (e.g. only file-writing, to avoid destructive actions); using `argument-hint` to prompt for required parameters when the skill is invoked without arguments; choosing between skills (on-demand, task-specific) and CLAUDE.md (always-loaded, universal).

### Concepts explained

#### Commands and skills are one mechanism now

Custom commands have been **merged into skills**. A file `.claude/commands/deploy.md` and a skill `.claude/skills/deploy/SKILL.md` both create `/deploy` and work the same way; existing command files keep working. Skills add: a directory for supporting files, frontmatter controlling who may invoke them, and the ability for Claude to load them automatically when relevant. If a skill and a command file share a name, **the skill wins**. The docs say to prefer skills for new work, but the exam (and many real repos) still use `.claude/commands/`, so know both.

Locations (from the skills and .claude directory docs):

| Kind | Path | Reaches |
|---|---|---|
| Project command | `.claude/commands/name.md` | Everyone who clones/pulls the repo (commit it) |
| Personal command | `~/.claude/commands/name.md` | Only you, all projects |
| Project skill | `.claude/skills/name/SKILL.md` | Everyone on the repo (commit it) |
| Personal skill | `~/.claude/skills/name/SKILL.md` | Only you, all projects on this machine |
| Enterprise/managed skill | managed settings directory | All users where the organisation deploys it |
| Plugin skill | `<plugin>/skills/name/SKILL.md` | Wherever the plugin is enabled, invoked as `/plugin-name:skill-name` |

A subdirectory of `.claude/commands/` becomes a namespace, e.g. `.claude/commands/frontend/component.md` → `/frontend:component`.

**Name collisions.** When two skills share a name: enterprise beats personal, and personal beats project (so with `deploy` in both `~/.claude/skills/` and the project's `.claude/skills/`, `/deploy` runs the *personal* one). A skill replaces a bundled command of the same name but not that command's aliases. Plugin skills never collide because they are namespaced. This is exactly why the guide says to create personal variants under a **different name**: a same-named personal skill would silently shadow the team's version for you (and be confusing), while a differently named one sits beside it and touches nobody else.

> Docs caveat for sample Q4: the docs say `/review` is now an **alias** of the bundled `/code-review`, and that your own skill replaces a bundled command "but not its aliases" (the code-review page says a skill named `code-review` replaces the command while the `/review` alias keeps running the bundled review). The docs do not say explicitly what happens when your own file is literally named `review`. The exam's expected answer for "where do I put a shared `/review` command" is still `.claude/commands/` in the repo. If you actually build one, test that it is what runs, or give it a distinct name like `/team-review` (verify in docs).

#### SKILL.md anatomy and frontmatter

A skill is a directory with a `SKILL.md`: YAML frontmatter between `---` markers, then markdown instructions. All frontmatter fields are optional; `description` is recommended because Claude uses it to decide when to load the skill. Fields relevant to the exam, plus neighbours worth recognising:

| Field | What it does |
|---|---|
| `name` | Command name; defaults to the directory name |
| `description` | What it does and when to use it (key use case first; used for automatic matching) |
| `argument-hint` | Autocomplete hint for expected arguments, e.g. `[issue-number]` |
| `allowed-tools` | Tools Claude may use **without asking** during the turn the skill is invoked |
| `disallowed-tools` | Tools **removed** from the available pool while the skill is active |
| `context` | `fork` runs the skill in a forked subagent context |
| `agent` | Which subagent type to use with `context: fork` (`Explore`, `Plan`, `general-purpose`, or a custom agent from `.claude/agents/`; default `general-purpose`) |
| `disable-model-invocation` | `true` = only you can invoke it (use for side-effect workflows like `/deploy`) |
| `user-invocable` | `false` = only Claude can invoke it (background knowledge) |
| `model`, `effort`, `hooks`, `paths`, `shell`, `when_to_use`, `arguments`, `background` | Other options (not exam-critical) |

Command files accept the same fields except `name` and `paths`.

Arguments reach the skill body through `$ARGUMENTS` (everything the user typed) or positional `$0`, `$1`, ...; if the body has no placeholder, Claude Code appends `ARGUMENTS: <value>`. Keep `SKILL.md` under ~500 lines and move bulky reference material into supporting files that Claude loads only when needed.

#### `context: fork`: isolation

`context: fork` runs the skill in a **subagent with its own context window**. The subagent receives the skill content as its prompt and does **not** see your conversation history, so the instructions must be self-contained. Only the result comes back to the main session. Use it when:

- the skill produces **verbose output** (whole-codebase analysis, long test runs), or
- the skill is **exploratory** (brainstorming alternatives) and you do not want the dead ends sitting in your main context for the rest of the session.

Do **not** fork a skill that is only guidelines ("use these API conventions") with no task: the subagent has nothing to do and returns little useful output. Also note that the built-in Explore and Plan agents start *without* CLAUDE.md and git status, so a forked skill with `agent: Explore` gets only the skill text plus that agent's own prompt; put everything it needs in `SKILL.md`. By default a forked subagent runs in the background (you keep working and its result arrives when it finishes, with a narrower tool set); `background: false` waits for the result in the invoking turn. Claude Code also waits for the result in non-interactive runs (`-p` or the Agent SDK), which matters in CI.

#### `allowed-tools`: what it really does

The guide frames `allowed-tools` as a way to *restrict* tool access during a skill (e.g. limit to file-write operations to prevent destructive actions). The **current docs are more precise**: `allowed-tools` *pre-approves* the listed tools for the turn in which the skill is invoked so Claude does not need to prompt you; the grant clears on your next message; it "does not restrict which tools are available": every other tool remains callable, subject to your normal permission settings. To actually *remove* tools while a skill runs, use `disallowed-tools`; to block tools everywhere, use deny rules.

> Exam wording vs. current docs: answer exam questions with the guide's framing (`allowed-tools` limits what the skill may do without further approval, e.g. only `Write`/`Edit`, not `Bash`). In real configuration, combine it with `disallowed-tools` or permission deny rules if you need a hard restriction. Also, tool-pattern syntax like `Bash(git add *)` uses permission-rule syntax, and the field accepts a space- or comma-separated string or a YAML list.

#### `argument-hint`

Shows a hint in autocomplete for the arguments a skill expects (the docs' examples are `[issue-number]` and `[filename] [format]`). The guide's phrasing is that it prompts developers for required parameters when they invoke the skill without arguments. The docs describe it as an *autocomplete hint* only; the arguments themselves flow through `$ARGUMENTS` / `$0` placeholders. Treat `argument-hint` as the documentation-of-parameters mechanism and make the skill body handle a missing argument (e.g. "if no component name was given, ask for it").

#### Skills vs CLAUDE.md: on-demand vs always-loaded

| | CLAUDE.md | Skill / command |
|---|---|---|
| Loads | Every session, automatically | Description at session start (for model-invocable skills); full content only when invoked or relevant |
| Best for | Universal standards: "always do X", build commands, conventions | Task-specific workflows and reference material: `/review`, `/deploy`, a migration playbook, API docs |
| Context cost | Full content, every request | Low until used (user-only skills with `disable-model-invocation: true` cost nothing until invoked) |
| Can trigger a workflow | No | Yes (`/name`) |

Rule of thumb from the docs: put it in CLAUDE.md if Claude should *always* know it; put it in a skill if it is reference material needed sometimes or a workflow you trigger. Both can combine: CLAUDE.md says "follow our API conventions", a skill holds the full style guide. A long release procedure in CLAUDE.md taxes every unrelated bug fix; in a skill it costs nothing until `/release` is typed.

### Worked examples

#### Example 1: Project command vs personal command (sample Q4)

```text
Goal: every developer gets /review the moment they clone the repo.

CORRECT   <repo>/.claude/commands/review.md      -> committed, shared via git
WRONG     ~/.claude/commands/review.md           -> personal; teammates never receive it
WRONG     a "Review checklist" section in CLAUDE.md -> context, not a command definition
WRONG     .claude/config.json with a commands[] array -> does not exist
```

File `.claude/commands/review.md` (frontmatter must be the first thing in the file):

```markdown
---
description: Run the team's standard code review checklist on the current changes
argument-hint: [path or PR number]
---
Review $ARGUMENTS (or the uncommitted changes if no argument was given) against our checklist:

1. Correctness: logic errors, unhandled null/empty cases.
2. Security: injection, secrets, missing authorization checks.
3. Tests: new behaviour has tests; bug fixes have a regression test.
4. Conventions: see CLAUDE.md for naming and error-handling rules.

Report findings grouped by severity. Do not modify files.
```

A personal command lives at `~/.claude/commands/standup.md` and only you can run `/standup`.

#### Example 2: A SKILL.md with `context: fork`, `allowed-tools`, `argument-hint`

```markdown
---
name: analyze-module
description: Deep analysis of one module (dependencies, public API, risks). Use when asked to understand or audit a module before changing it.
argument-hint: [module-path]
context: fork
agent: Explore
allowed-tools: Read Grep Glob
---
Analyze the module at $ARGUMENTS. If no path was supplied, ask which module to analyze and stop.

1. List its public exports and who imports them (use Grep across the repo).
2. Map its direct dependencies and any circular imports.
3. Identify risky areas: missing tests, TODO/FIXME clusters, large functions.

Finish with a summary of at most 15 bullet points, each citing file paths and line numbers.
Return ONLY the summary, not the intermediate search output.
```

What this buys you: the heavy Grep/Read exploration happens in an isolated Explore subagent (`context: fork`), the main conversation receives only the 15-bullet summary, and the pre-approved read-only tools mean no permission prompts and no write tools involved. (Docs caveat: `allowed-tools` pre-approves; Explore is already read-only, so it is the agent choice that really prevents edits.)

A skill that *must be run deliberately* and writes files (it pre-approves only `Write` and `Edit`, and the body tells Claude not to use the shell; note that `allowed-tools` alone does not remove `Bash`, so add `disallowed-tools: Bash` if the shell must be unavailable):

```markdown
---
name: scaffold-component
description: Create a new React component with test and story files from our templates
argument-hint: [ComponentName]
disable-model-invocation: true
allowed-tools: Write Edit
---
Create src/components/$ARGUMENTS/ with $ARGUMENTS.tsx, $ARGUMENTS.test.tsx and $ARGUMENTS.stories.tsx
following the templates in docs/templates/component/. Do not run any shell commands.
```

#### Example 3: Personal skill variant under a different name

The team ships `.claude/skills/review/SKILL.md` (strict checklist). You want a lighter personal flavour without changing anything for teammates:

```text
Team (committed):   .claude/skills/review/SKILL.md              -> /review
Personal (yours):   ~/.claude/skills/review-quick/SKILL.md      -> /review-quick
```

If you had instead created `~/.claude/skills/review/SKILL.md`, then per the docs' precedence (personal over project) *your* `/review` would run your variant instead of the team's, so you would silently diverge from the team standard and be confused when your results differ from a colleague's. A different name avoids both problems.

#### Example 4: Skill vs CLAUDE.md decision (bad vs good)

Bad: pasting the 120-line "how we cut a release" procedure into CLAUDE.md. It is loaded on *every* task (including a one-line typo fix) and dilutes the rules that matter every time.

Good split:

```markdown
<!-- CLAUDE.md : always-loaded, short, universal -->
- Releases are cut with the /release skill. Never publish from a developer machine.
- Run `pnpm typecheck && pnpm test` before declaring work done.
```

```markdown
<!-- .claude/skills/release/SKILL.md : loaded only when /release is invoked -->
---
name: release
description: Cut a new release. Bumps version, writes changelog entry, tags and pushes so CI publishes.
disable-model-invocation: true
---
1. Ensure you are on main with a clean tree; otherwise stop and ask.
2. Run `pnpm changeset version` and commit.
3. Summarize PRs merged since the last tag into CHANGELOG.md, user-facing changes only.
4. Tag and push the tag; CI publishes from there.
```

### Exam traps

- **Putting a shared command in `~/.claude/commands/`.** That is user-scoped, not shared. Shared = `.claude/commands/` (or `.claude/skills/`) inside the repo.
- **Putting a command definition in CLAUDE.md or inventing `.claude/config.json`.** CLAUDE.md is instructions/context; no config.json command registry exists.
- **Overwriting the team skill by creating a personal skill with the same name.** The exam answer is a *different name*.
- **Using `context: fork` for a skill with no task / only conventions.** Fork is for tasks that generate bulk or exploratory output.
- **Assuming a forked skill sees the chat history.** It does not; the skill text must stand alone.
- **Thinking skills are always in context.** Only their descriptions load (and not even that with `disable-model-invocation: true`); the body loads on invocation. That is the whole point of choosing a skill over CLAUDE.md for rarely needed content.
- **Using a skill to enforce a must-always rule.** Skills are probabilistic guidance; enforcement is a hook/permission rule.

### Check yourself

1. You want a `/migrate-component` workflow that prints 800 lines of analysis while exploring. How do you keep your main conversation readable, and where do you put the file so the team gets it?
   <details><summary>Answer</summary>Create `.claude/skills/migrate-component/SKILL.md` (committed) with `context: fork` (optionally `agent: Explore`) and instruct it to return only a concise summary. The fork isolates the verbose output from the main conversation. (A `.claude/commands/` file also reaches the team but, as a command file, has the same frontmatter options; skills are preferred for new work.)</details>
2. A developer creates `~/.claude/skills/deploy/SKILL.md` to tweak the team's `deploy` skill. Why is that risky, and what is the recommended alternative?
   <details><summary>Answer</summary>A personal skill with the same name takes precedence over the project skill (personal over project), so only that developer's behaviour silently changes. Create the variant under a different name (e.g. `deploy-dry-run`) in `~/.claude/skills/` so teammates are unaffected and both remain available.</details>
3. Which belongs in CLAUDE.md and which in a skill: (a) "Use 2-space indentation and named exports", (b) a 15-step incident-response runbook?
   <details><summary>Answer</summary>(a) CLAUDE.md: a small universal standard needed on every task. (b) A skill: task-specific, long, only needed when invoked, so it should not occupy always-loaded context.</details>

### Official docs

- [Skills](https://code.claude.com/docs/en/skills.md) — frontmatter reference, locations, name collisions, `context: fork`, `allowed-tools`, arguments, content lifecycle.
- [Commands](https://code.claude.com/docs/en/commands.md) — built-in commands (`/memory`, `/context`, `/plan`, `/compact`, `/clear`, `/init`) and bundled skills such as `/code-review`.
- [Explore the .claude directory](https://code.claude.com/docs/en/claude-directory.md) — where `commands/`, `skills/`, `agents/` live and which are committed.
- [Extend Claude Code](https://code.claude.com/docs/en/features-overview.md) — skill vs CLAUDE.md vs subagent vs hook vs MCP comparisons.

---

## 3.3 Apply path-specific rules for conditional convention loading

### What the exam tests

- **Knowledge**: `.claude/rules/` files can carry YAML frontmatter with a `paths` field of glob patterns; such rules activate conditionally. They load only when Claude is working with matching files, which cuts irrelevant context and token use. For conventions that span many directories (the guide's example: test files scattered through the codebase), glob-pattern rules beat directory-level CLAUDE.md files.
- **Skills**: writing a rules file with `paths: ["terraform/**/*"]`; using patterns like `**/*.test.tsx` to target files by *type* regardless of location; choosing path-specific rules over subdirectory CLAUDE.md files when a convention must apply to files spread across the codebase.

### Concepts explained

#### How a path-scoped rule works

A rule file is just markdown in `.claude/rules/`. Adding frontmatter turns it conditional:

```markdown
---
paths:
  - "src/api/**/*.ts"
---

# API Development Rules
- All API endpoints must include input validation
- Use the standard error response format
```

Behaviour documented in the memory page:

- **No `paths` = unconditional.** The rule loads at launch like `.claude/CLAUDE.md`.
- **With `paths` = on demand.** The rule loads when Claude uses the Read, Write or Edit tool on a *matching* file (also when a single-file Bash read like `cat` or `head` counts as a read). Until then it is not in context.
- **Patterns are relative to the rule's base directory**: for project rules, the directory containing the `.claude` folder; for user-level rules (`~/.claude/rules/`), the session's primary working directory.
- **Syntax**: `paths` accepts a YAML list (or a comma-separated string). Standard globs: `**/*.ts` (all TS files anywhere), `src/**/*` (everything under `src/`), `*.md` (Markdown in the project root only), `src/components/*.tsx` (one directory). Brace expansion works: `src/**/*.{ts,tsx}`. Quote patterns that start with `*` or contain special characters so YAML parses them.
- **`paths` is the only frontmatter field** Claude Code reads for a rule; other fields are ignored. Broken YAML means the frontmatter is ignored and the rule loads as if unconditional (`claude --debug` shows the parse error).
- **Survives compaction differently**: path-scoped rules are loaded into message history when triggered, so `/compact` can summarise them away; they reload on demand when a matching file is touched again. If a rule must persist unconditionally, drop `paths:` or move it to the root CLAUDE.md.
- A per-file size limit applies to files that load on demand (path-scoped rules and subdirectory CLAUDE.md files): 40,000 characters on a 200K context window, and 150,000-200,000 characters on a 1M window depending on the model. Past the limit Claude receives only the start of the file, so keep each rule file focused. (Files that load at launch have a different limit: a CLAUDE.md over 4 MiB is skipped.)

Why this saves tokens and improves adherence: a Terraform rule file (variable naming, state backend, tagging) is irrelevant while Claude edits a React component. With `paths: ["terraform/**/*"]` it simply is not loaded. The model sees fewer, more relevant instructions, so it applies them more consistently.

#### Rules with globs vs per-directory CLAUDE.md

A directory-level CLAUDE.md is bound to a *directory*: it loads when Claude touches a file in that subtree. That is perfect when the convention belongs to a place (`packages/payments/`). It is the wrong tool when the convention belongs to a *file type* that is everywhere:

```text
src/components/Button.tsx
src/components/Button.test.tsx        <- test next to code
src/api/orders.ts
src/api/orders.test.ts                <- another test, different directory
src/models/User.ts
src/models/User.test.ts               <- and another
```

To give all tests the same conventions with directory CLAUDE.md files you would need a copy in `src/components/`, `src/api/`, `src/models/` and every new directory (duplication, drift, forgotten folders; and those files would also carry rules for the non-test code beside them). One rule with `paths: ["**/*.test.ts", "**/*.test.tsx"]` covers every present and future test file wherever it lives. That is sample Q6 in a sentence.

Why not the alternatives in Q6?

- **One big root CLAUDE.md with headers per area**: loads everything always and relies on Claude *inferring* which section applies (probabilistic, wasteful).
- **Skills per code type**: skills load on invocation or when Claude decides they are relevant; the requirement is *automatic, path-driven* application, which is deterministic matching on file paths, not on model judgement.
- **Directory CLAUDE.md per subdirectory**: cannot cover files spread across many directories without duplication.

### Worked examples

#### Example 1: Test conventions across the whole codebase (sample Q6)

```markdown
---
paths:
  - "**/*.test.tsx"
  - "**/*.test.ts"
---

# Test conventions (apply to every test file, wherever it lives)
- Use React Testing Library queries by role/label; never query by CSS class.
- One behaviour per `it(...)`, named "should ... when ...".
- Build data with factories from tests/factories.ts; no inline fixtures over 10 lines.
- Mock the network with MSW handlers, never by stubbing fetch directly.
```

Saved as `.claude/rules/testing.md`. When Claude edits `src/components/Button.test.tsx` the rule loads; when it edits `src/api/orders.ts` it does not.

#### Example 2: A small rule set matching the guide's examples

```text
.claude/rules/
├── testing.md          paths: ["**/*.test.ts", "**/*.test.tsx"]   (all tests, anywhere)
├── api-conventions.md  paths: ["src/api/**/*"]                    (everything under src/api)
├── terraform.md        paths: ["terraform/**/*"]                  (infra code only)
├── react.md            paths: ["src/components/**/*.tsx"]         (UI code)
└── general-style.md    (no paths)                                 (always on, keep it tiny)
```

```markdown
---
paths:
  - "src/api/**/*"
---
# API conventions
- Handlers are async; wrap in `withErrorBoundary` and return `{ error: { code, message } }` on failure.
- Validate bodies with the zod schemas in src/api/schemas/.
- Every new endpoint needs an entry in openapi.yaml.
```

```markdown
---
paths:
  - "terraform/**/*"
---
# Terraform conventions
- Resource names: snake_case, prefixed by service (`payments_db`).
- Every resource must set the `owner` and `env` tags.
- Never hard-code credentials; use variables marked `sensitive = true`.
- Run `terraform fmt` and `terraform validate` before presenting changes.
```

Trace of what loads:

```text
Claude edits terraform/main.tf          -> terraform.md loads (+ general-style.md already loaded)
Claude edits src/api/orders.ts          -> api-conventions.md loads
Claude edits src/api/orders.test.ts     -> api-conventions.md AND testing.md load (both match)
Claude edits README.md                  -> only general-style.md (always on)
```

A file can match several rules; all matching rules load, so avoid contradictions between them.

#### Example 3: Bad vs good (Scenario 2 team)

Bad: five `CLAUDE.md` files each repeating the same test conventions in `src/components/`, `src/api/`, `src/models/`, `src/hooks/`, `src/utils/`. Someone adds `src/jobs/` and forgets one; the test style there drifts.

Good: one `.claude/rules/testing.md` with `paths: ["**/*.test.*"]`. New directories are covered automatically.

### Exam traps

- **"Subdirectory CLAUDE.md files are the most maintainable way to apply conventions by file type."** They are directory-bound; the right tool is path-scoped rules.
- **"Rely on Claude to infer which section of a long CLAUDE.md applies."** Probabilistic and token-heavy.
- **"Create a skill per code type."** Skills need invocation or model discretion, so they cannot guarantee automatic path-based application.
- **Forgetting the quotes or the list syntax** is not tested heavily, but remember `paths` is a YAML key in the frontmatter *of the rule file*, not a setting in `settings.json`.
- **Thinking path-scoped rules load at startup.** They load when matching files are read/edited (that is the point).
- **Thinking `*.md` matches all markdown.** Without `**/` it matches only the project root.

### Check yourself

1. Terraform files live in `terraform/` and `modules/*/infra/`. You want infra conventions loaded only when editing those. What do you write?
   <details><summary>Answer</summary>A rule such as `.claude/rules/terraform.md` with frontmatter `paths:` listing `"terraform/**/*"` and `"modules/**/infra/**/*"` (or `**/*.tf`), followed by the conventions. It loads only when matching files are read or edited.</details>
2. Why is a single `paths: ["**/*.test.tsx"]` rule better than a CLAUDE.md in each folder that contains tests?
   <details><summary>Answer</summary>Directory CLAUDE.md files are bound to directories, so each folder needs its own copy (duplication, drift, missed folders), and each copy also loads when editing non-test code. A glob rule matches by file type everywhere, including future directories, and loads only when a test file is touched.</details>
3. A rule has no `paths` frontmatter. When does it load?
   <details><summary>Answer</summary>At launch, unconditionally, with the same priority as `.claude/CLAUDE.md`; it applies to all files and costs context every session.</details>

### Official docs

- [How Claude remembers your project: Organize rules with `.claude/rules/`](https://code.claude.com/docs/en/memory.md) — path-specific rules, glob table, brace expansion, rule frontmatter reference, user-level rules, symlinks.
- [Best practices: Scope rules to one part of the codebase](https://code.claude.com/docs/en/best-practices.md) — the `src/api/**/*.ts` example.
- [Explore the context window](https://code.claude.com/docs/en/context-window.md) — what survives compaction (path-scoped rules reload on demand).

---

## 3.4 Determine when to use plan mode vs direct execution

### What the exam tests

- **Knowledge**: plan mode suits complex tasks: large-scale changes, several valid approaches, architectural decisions, multi-file modifications. Direct execution suits simple, well-scoped changes (adding one validation check to one function). Plan mode lets Claude explore safely and design before committing to changes, preventing costly rework. The **Explore subagent** isolates verbose discovery output and returns summaries, preserving the main conversation's context.
- **Skills**: choosing plan mode for architectural work (microservice restructuring, a library migration touching 45+ files, choosing between integration approaches with different infrastructure needs); choosing direct execution for well-understood changes (single-file bug with a clear stack trace, a date-validation conditional); using Explore for verbose discovery phases to avoid exhausting context in multi-phase tasks; combining plan mode for investigation with direct execution for implementation.

### Concepts explained

#### What plan mode is

Plan mode tells Claude to *research and propose* changes without making them. Claude can read files, run exploratory commands and write a plan, but does not edit your source until you approve the plan. Docs details:

- **Enter**: press `Shift+Tab` (cycle modes until the status bar shows plan mode), type `/plan` (optionally with a task: `/plan fix the auth bug`), or start the session with `claude --permission-mode plan`. Press `Shift+Tab` again to leave without approving.
- **Review**: when the plan is ready Claude presents it and asks how to proceed. Choices include approving with auto mode (or "auto-accept edits" if auto mode is unavailable), approving while manually reviewing each edit, or "No, keep planning" with feedback. Approving exits plan mode and switches to the mode of the option you chose, so Claude begins implementing. `Ctrl+G` opens the plan in your editor so you can edit it first.
- **Plan subagent**: during plan mode a built-in read-only **Plan** subagent gathers codebase context before the plan is presented.
- **What is and is not blocked**: edits stay blocked until you approve the plan, and plan mode keeps those blocks in non-interactive `-p` runs. The exception: in interactive terminal sessions where bypass permissions are available, Claude Code does not enforce plan mode's blocks (Claude is only instructed to plan without editing).
- **Project default**: set `defaultMode` to `plan` in `.claude/settings.json` for plan-first projects.
- **Persistence**: the plan Claude wrote is re-injected from disk after compaction.

Plan mode is one of the *permission modes* (`default`, `acceptEdits`, `plan`, `auto`, `dontAsk`, `bypassPermissions`), which is why the CLI flag is `--permission-mode plan`.

#### Decision framework

Ask: *is the shape of the solution already known and local?*

| Signal in the task | Mode | Why |
|---|---|---|
| Restructure a monolith into microservices (dozens of files, service boundaries to decide) | **Plan** | Architectural decisions; wrong boundaries cause expensive rework |
| Migrate a library affecting 45+ files | **Plan** (then execute the plan) | Multi-file, ordering/dependency questions, risk of inconsistent partial migration |
| Choose between two integration approaches with different infrastructure needs | **Plan** | Multiple valid approaches; trade-offs should be explored before editing |
| Unfamiliar codebase, unclear where the change belongs | **Plan** / explore first | Discovery needed before design |
| Single-file bug fix with a clear stack trace | **Direct** | Scope and cause are known |
| Add a date-validation conditional to one function | **Direct** | Small, well-understood; planning overhead exceeds the work |
| Rename a variable in one file; add a log line | **Direct** | Trivial |

The exam's recurring distractors (see sample Q5): "start with direct execution and let the implementation reveal the boundaries" (rework), "direct execution with comprehensive upfront instructions" (assumes you know the structure without exploring), and "start direct and switch to plan mode if it gets complex" (ignores that the complexity is *already stated* in the requirements).

Plan mode is not a safety switch for tiny edits either: using it for a one-line fix adds ceremony for no benefit. The exam rewards matching effort to complexity.

#### The Explore subagent

Subagents are isolated workers: each has its own context window, does the work there, and returns a summary to the main conversation. The built-in **Explore** subagent is "a fast, read-only agent optimized for searching and analyzing codebases": its tools are read-only (Write and Edit are denied), and its purpose is file discovery, code search and codebase exploration. When Claude invokes it, it picks a thoroughness level: **quick** (targeted lookups), **medium**, or **very thorough**. Explore and Plan also start *without* your CLAUDE.md and git-status snapshot, to stay fast and cheap.

Why this matters in multi-phase tasks: discovery is verbose (grep outputs, dozens of file reads). If done in the main conversation those tokens sit in context for the rest of the session, crowd out the design and implementation phases and push you toward compaction. Delegating discovery to Explore keeps "dozens of file reads" in the subagent's window and returns only the findings. You can ask for it explicitly:

```text
Use a subagent to investigate how our auth system handles token refresh and report which files and functions are involved.
```

Costs to know: each subagent spends tokens of its own, and returned results still occupy main context (keep summaries short). The current best-practices page (its "What you can stop doing" table) says that with 1M-token windows and automatic compaction, using subagents purely to protect your context window matters less than it used to, but that subagents remain good for parallel work, keeping noisy output out of the way, and independent review. The exam still tests Explore as the tool for isolating verbose discovery.

> Terminology: the exam guide and older material call the subagent-spawning tool **Task**. The current docs call it **Agent** (renamed in Claude Code v2.1.63; existing `Task(...)` references still work as aliases). Exam answers follow the guide's wording; in current settings and agent definitions you may see `Agent(...)`.

#### Combining plan mode and direct execution

The guide's pattern is a **two-phase workflow**: plan mode for investigation and design, direct execution for implementation of the approved plan. In practice:

1. Enter plan mode; let Claude explore (the exam names the Explore subagent for verbose discovery; the docs say Claude delegates research to the read-only Plan subagent while in plan mode) and draft the plan.
2. Review and edit the plan; ask questions; reject and refine if needed.
3. Approve; Claude leaves plan mode and executes the plan with edits enabled.
4. Verify with tests (see 3.5), commit.

The docs' "Common workflows" page has a short "Plan before editing" recipe for the review-before-edit part, and the "Best practices" page recommends giving Claude a way to verify its work (tests, linter, a script that diffs output against a fixture). The two-phase "plan, then execute" framing itself is the exam guide's.

### Worked examples

#### Example 1: Library migration, plan then execute (Scenario 4 style)

```text
You: /plan We need to replace moment.js with date-fns across the repo (about 45 files).
     Find every usage pattern, group them, flag risky ones (timezone handling), and propose
     a migration order. Do not edit anything yet.

Claude (plan mode): [delegates discovery to a read-only research subagent]
  Plan:
  1. Add date-fns; create src/lib/dates.ts wrapper exposing the 6 helpers we actually use.
  2. Migrate leaf utilities first (src/utils/*), then services, then UI.
  3. Timezone-sensitive files (billing/, scheduler/) get extra tests BEFORE migration.
  4. Remove moment.js and run the full test suite after each batch.
  Risks: locale formatting differs in 4 files; moment mutates dates, date-fns does not.

You: Looks good. Do step 3's test additions first, then proceed.
[approve plan -> mode switches -> Claude executes batch by batch, running tests after each]
```

Why this is the right call: 45+ files, an ordering question, hidden behavioural differences, and a wrong approach would be discovered late. The plan front-loads the thinking.

#### Example 2: Direct execution is the right choice

```text
You: The stack trace shows validateDeadline() in src/orders/validate.ts line 52 throws on
     a null dueDate. Add a null check that returns a validation error instead of throwing,
     plus a unit test.
Claude: [edits one function, adds one test, runs tests]
```

Scope, location and cause are all known. Asking for a plan here would be over-engineering.

#### Example 3: Plan vs direct decision table with the guide's examples

| Task | Right choice | Wrong choice and its cost |
|---|---|---|
| Monolith to microservices, dozens of files | Plan mode (Q5 answer A) | Direct + "let boundaries emerge": rework when dependencies surface late |
| 45-file library migration | Plan, then execute plan | Direct from the start: inconsistent partial migrations |
| Pick between queue-based vs webhook integration (different infra) | Plan mode: compare options | Direct: committing to one approach unexamined |
| Single-file bug with stack trace | Direct | Plan mode: pure overhead |
| Add a date-validation conditional | Direct | Plan mode: pure overhead |

#### Example 4: Explore for discovery in a multi-phase task

```text
Phase 1 (discovery, isolated):   "Use the Explore subagent: list every module that imports
                                  legacy-auth and summarise how each uses it (max 20 lines)."
   -> 60 files read inside the subagent; main context receives ~20 lines.
Phase 2 (design, main context):   "Given that summary, propose a migration plan."
Phase 3 (implementation):         approve plan, execute, run tests.
```

Without the subagent, phase 1 alone might add tens of thousands of tokens of file contents to the main window.

### Exam traps

- **"Begin with direct execution and switch to plan mode if complexity appears."** The complexity is stated up front; planning first avoids rework.
- **"Direct execution with detailed upfront instructions."** Assumes you already know the right structure without exploring.
- **"Always use plan mode, it is safer."** Over-engineering for small, well-scoped changes.
- **"Explore subagent can edit files / is the same as the general-purpose agent."** Explore is read-only.
- **"Do verbose discovery in the main conversation so nothing is lost."** That is exactly what exhausts context in multi-phase work.
- **Naming the tool.** If an option says "the Task tool spawns subagents", that is the guide's wording; the current docs name it Agent. Do not be thrown by either name.

### Check yourself

1. You must choose between adding a message queue or webhooks for order notifications; each needs different infrastructure. Which mode and why?
   <details><summary>Answer</summary>Plan mode: multiple valid approaches with architectural implications; exploring and comparing before editing prevents costly rework. Implement afterwards by approving the plan.</details>
2. Why use the Explore subagent in the first phase of a long refactoring session?
   <details><summary>Answer</summary>Discovery produces verbose output (searches, many file reads). Explore does that in its own context and returns only a summary, preserving the main conversation's context for design and implementation phases.</details>
3. Stack trace points at one function; the fix is a null check. Plan mode or direct execution?
   <details><summary>Answer</summary>Direct execution: clearly scoped, single-file, cause known. Plan mode would add overhead without reducing risk.</details>

### Official docs

- [Choose a permission mode: plan mode](https://code.claude.com/docs/en/permission-modes.md) — entering/leaving plan mode, approval options, defaults.
- [Common workflows](https://code.claude.com/docs/en/common-workflows.md) — exploring codebases, "Plan before editing", "Delegate research to subagents".
- [Create custom subagents](https://code.claude.com/docs/en/sub-agents.md) — built-in Explore/Plan/general-purpose agents, what loads at startup, why subagents isolate verbose output.
- [Best practices](https://code.claude.com/docs/en/best-practices.md) — "Know what subagents cost", subagents for independent review, and the "What you can stop doing" table (the page has no separate plan-mode or explore/plan/implement section).
- [Explore the context window](https://code.claude.com/docs/en/context-window.md) — how a subagent keeps large reads out of your context.

---

## 3.5 Apply iterative refinement techniques for progressive improvement

### What the exam tests

- **Knowledge**: concrete input/output examples are the most effective way to communicate a transformation when prose is interpreted inconsistently; test-driven iteration (write the test suite first, then iterate by sharing failures); the interview pattern (Claude asks questions to surface considerations you had not thought of before implementing); when to put all problems in one message (interacting problems) versus fix them one at a time (independent problems).
- **Skills**: giving 2–3 concrete input/output examples when natural-language descriptions yield inconsistent results; writing tests covering expected behaviour, edge cases and performance requirements *before* implementation and iterating on failures; using the interview pattern in unfamiliar domains (cache invalidation strategy, failure modes); giving specific test cases with example input and expected output to fix edge cases such as null values in a migration script; addressing multiple interacting issues in one detailed message but independent issues sequentially.

### Concepts explained

The unifying idea: Claude cannot read your mind, and a vague request gets a plausible-but-wrong first draft. Refinement is the discipline of giving the *highest-signal feedback* available at each step. The docs' best-practices page frames it as "tell Claude what it can't see" and "give Claude a check to run".

#### 1. Concrete input/output examples

Prose descriptions of transformations ("normalise the phone numbers", "convert legacy status codes") are ambiguous: every reader (including a model) fills the gaps differently, producing *inconsistent* results across runs and across inputs. Two or three representative input → output pairs pin down the rule far better, including edge cases the prose never mentioned. Choose examples that **cover the variety**: the normal case, a tricky case, and an edge case. (More than 2–3 yields diminishing returns; unrepresentative examples can over-fit.)

Compare with how the docs advise specific prompts: name the file, describe the symptom, say what "fixed" looks like. Examples are the most compact way to say what "right" looks like. (The 2-3 input/output examples technique is the exam guide's; the docs pages do not present it as a named technique.)

#### 2. Test-driven iteration

Write the tests first (expected behaviour, edge cases, performance requirements), confirm they fail, then let Claude implement and iterate. Each failing test is an *objective, specific* signal: you paste the failure output back and Claude fixes exactly that. The docs state the underlying principle: if Claude has something it can run that comes back pass/fail, "it can keep going on its own". Without it, "looks done" is all Claude has, and *you* become the test suite. Ask for evidence (the command and its output), not just "all tests pass".

Benefits: unambiguous spec, regression protection while refactoring, and a natural stopping condition. Include performance cases (e.g. "processes 100k rows in under 2 seconds") when performance is a requirement; otherwise a correct-but-slow solution passes.

#### 3. The interview pattern

When you are in an unfamiliar domain, or have not thought the feature through, do not guess the spec. Ask Claude to interview you first. The best-practices page's example prompt (about adding rate limiting to a public API) ends: *"Interview me until you could write the spec, then write it to SPEC.md."* Claude asks (multiple-choice) questions on the parts you have not pinned down: per-user versus per-key limits, behaviour for requests over the limit, what is out of scope. Then start a **fresh session** to build from `SPEC.md` so the build begins with clean context and a written definition of done.

The guide's examples: cache invalidation strategies and failure modes. These are exactly the "considerations the developer may not have anticipated". The pattern surfaces them *before* code exists, when changing direction is cheap. It is the opposite of "implement then discover" (rework) and differs from plan mode in emphasis: plan mode explores the *codebase* and designs; interviewing elicits *requirements and constraints from you*. They combine well.

#### 4. One message vs sequential fixes

(Exam-guide pattern: the docs pages do not give a rule for batching versus sequencing fixes, so the reasoning below is design advice.)

- **Interacting problems** (fixing one changes or constrains the fix of another; shared root cause; same function/contract): report them **together in one detailed message** so Claude designs one coherent solution. Fixing sequentially can produce a fix for A that breaks B, then a fix for B that reverts A (thrashing).
- **Independent problems** (different modules, no shared state or contract): fix **sequentially** (one at a time), verifying each. Small focused changes are easier to review, test and roll back, and mixing unrelated problems dilutes attention.

Related steering advice from the docs' best-practices page: after two failed corrections on the same thing, run `/clear` and write a better prompt that folds in what you learned, rather than adding a third correction to a conversation already containing two failed attempts. Use `Esc` to interrupt early, and `/rewind` to roll back code/conversation to try a bolder approach safely.

### Worked examples

#### Example 1: Input/output examples fix inconsistent output

Bad (prose only; Claude produced `+1-415-555-0100`, `4155550100` and `(415) 555-0100` across runs):

```text
Normalise the phone numbers in the customers table to a consistent format.
```

Good (2-3 examples covering normal, tricky and edge cases):

```text
Normalise phone numbers to E.164. Examples:

  "(415) 555-0100"      -> "+14155550100"
  "415.555.0100 x23"    -> "+14155550100"   (drop extensions; store them in the ext column)
  "020 7946 0958" (UK)  -> "+442079460958"  (use the row's country_code to resolve the prefix)
  "n/a"                 -> null             (do not guess; log the row id)

Apply to customers.phone and add unit tests using exactly these four cases.
```

#### Example 2: Test-driven iteration

```text
You:    Write tests FIRST for a parseDuration(str) function in src/lib/duration.test.ts.
        Cover: "1h30m" -> 5400, "90s" -> 90, "" -> throws, "1.5h" -> 5400, "-5m" -> throws,
        and a performance test: 100,000 calls finish in under 200 ms. Do not implement yet.
Claude: [writes tests, runs them: all fail as expected]
You:    Now implement parseDuration to make them pass. Run the tests after each change.
Claude: [implements; 2 failing]
You:    Failures:
          FAIL "1.5h" -> expected 5400, got 3600
          FAIL perf: 100000 calls took 640 ms (limit 200)
Claude: [fixes decimal handling; precompiles the regex; all green]
```

Each failure message is a precise, objective instruction.

#### Example 3: Interview pattern (Scenario 4: unfamiliar domain)

```text
You:    I need a caching layer for product-catalog lookups. I haven't designed one for this
        service before. Interview me about the design considerations, then write the spec
        to SPEC.md. Don't write any code yet.
Claude: A few questions:
        1. Staleness tolerance: seconds, minutes, or must be immediately consistent after a price change?
        2. Invalidation trigger: TTL only, explicit invalidation on write, or event-driven from the pricing service?
        3. Failure mode: if the cache is down, fall back to the DB (risk: thundering herd) or fail the request?
        4. Multi-instance: shared cache (Redis) or per-process? Who invalidates peers?
        5. Cache stampede protection needed for hot keys?
You:    [answers]
Claude: [writes SPEC.md with decisions, edge cases and test list]
You:    (new session) Implement SPEC.md, tests first.
```

The questions about invalidation and failure modes are the considerations the developer might not have raised on their own.

#### Example 4: Specific test cases for an edge case (null values in a migration)

Bad: "The migration breaks on some rows, please fix it."

Good:

```text
migrate_orders.py fails on rows where shipping_address is NULL. Required behaviour:

  input  {"id": 7, "shipping_address": null,                "total": "19.99"}
  output {"id": 7, "shipping": {"present": false},          "total_cents": 1999}

  input  {"id": 8, "shipping_address": {"zip": "94107"},    "total": "5.00"}
  output {"id": 8, "shipping": {"present": true, "zip": "94107"}, "total_cents": 500}

  input  {"id": 9, "shipping_address": null,                "total": null}
  output skipped, row id logged to migration_errors.csv (do not insert 0)

Add these as test cases first, show me they fail, then fix the script.
```

#### Example 5: One message vs sequential

Interacting (send together):

```text
Three problems in the checkout flow, and I think they are linked:
1. applyDiscount() rounds per line item, so totals differ by 1 cent from the invoice.
2. invoice generation rounds the grand total, which is why (1) is visible.
3. Refunds recompute the discount from the rounded invoice total and over-refund by 1 cent.
Propose ONE consistent rounding strategy (where and when to round), then update all three.
```

Independent (do one at a time): a typo in the settings page label, a flaky date test in the scheduler, and a missing index on a reports query. Fix and verify each separately, in its own commit.

### Exam traps

- **More prose detail** as the fix for inconsistent interpretation. The better answer is 2–3 concrete input/output examples.
- **"Implement first, add tests later."** Test-driven iteration is write tests first and iterate on failures.
- **"Ask Claude to implement and then review if it considered caching edge cases."** The interview pattern surfaces considerations *before* implementing.
- **Fixing interacting issues one by one** (thrashing) or **batching unrelated issues in one giant message** (dilution). Match the strategy to whether the fixes interact.
- **Adding a third correction after two failed ones.** Better: `/clear` and a sharper prompt.
- **Treating "all tests pass" from Claude as proof.** Ask for the command and output; give Claude a runnable check.

### Check yourself

1. Claude keeps reformatting dates inconsistently even though your description is precise. What is the most effective change?
   <details><summary>Answer</summary>Provide 2–3 concrete input/output examples (normal, tricky, edge case) showing the exact expected transformation, instead of further prose.</details>
2. You are building a rate limiter and are unsure about per-user vs per-key limits and over-limit behaviour. What do you do before any code is written?
   <details><summary>Answer</summary>Use the interview pattern: have Claude interview you about the design considerations and write a spec; then implement (ideally in a fresh session) from that spec, tests first.</details>
3. Two bugs: a rounding error in totals that also causes a refund mismatch (same calculation), and an unrelated broken link in the footer. How do you report them?
   <details><summary>Answer</summary>Report the rounding and refund issues together in one detailed message because the fixes interact; handle the footer link separately/sequentially since it is independent.</details>

### Official docs

- [Best practices](https://code.claude.com/docs/en/best-practices.md) — specific prompts, interview prompt, "give Claude a check to run", steering and `/clear` after two failed corrections.
- [Common workflows](https://code.claude.com/docs/en/common-workflows.md) — fixing bugs, refactoring and testing recipes.
- [Explore the context window](https://code.claude.com/docs/en/context-window.md) — `/clear` and `/compact` between tasks.

---

## 3.6 Integrate Claude Code into CI/CD pipelines

### What the exam tests

- **Knowledge**: the `-p` (or `--print`) flag runs Claude Code non-interactively in automation; `--output-format json` plus `--json-schema` enforce structured output in CI; CLAUDE.md is how you supply project context (testing standards, fixture conventions, review criteria) to CI-invoked Claude Code; **session context isolation**: the session that generated code is a weaker reviewer of its own changes than an independent review instance.
- **Skills**: using `-p` so a pipeline never hangs waiting for interactive input; combining `--output-format json` with `--json-schema` to get machine-parseable findings that a script can post as inline PR comments; including prior review findings in context on re-runs after new commits and telling Claude to report only new or still-unaddressed issues (no duplicate comments); supplying existing test files so generated tests do not duplicate covered scenarios; documenting testing standards, what counts as a valuable test, and available fixtures in CLAUDE.md to raise test quality and cut low-value output.

### Concepts explained

#### `-p` / `--print`: non-interactive mode

Plain `claude "prompt"` starts an *interactive* session. In a CI runner there is no terminal user, so it waits for input and the job hangs. That is sample Q10. The fix is `claude -p "prompt"` (long form `--print`): Claude processes the prompt, prints the result to stdout and **exits**. It exits 0 on success and non-zero on failure, so scripts can branch on the exit code. You can pipe data in (`gh pr diff 42 | claude -p "review this"`; piped stdin is capped at 10 MB, so use a file path for bigger inputs) and redirect the answer out.

The other Q10 options do not exist or are not the documented mechanism: there is no `CLAUDE_HEADLESS` environment variable and no `--batch` flag, and redirecting stdin from `/dev/null` is a Unix workaround, not how Claude Code is meant to be run non-interactively. (The docs page for this mode is titled "Run Claude Code programmatically"; older material calls it "headless" mode and it is the same thing: the CLI face of the Agent SDK.)

Flags you will combine with `-p` (all in the CLI reference):

| Flag | Purpose |
|---|---|
| `--output-format text\|json\|stream-json` | `text` is default; `json` gives one result object with `result`, `session_id`, usage/cost metadata; `stream-json` streams newline-delimited events (the docs' examples add `--verbose`) |
| `--json-schema '<schema>'` | Print mode only. Validates/shapes output to a JSON Schema; the conforming data appears in the response's **`structured_output`** field. Invalid schema = error exit |
| `--allowedTools "Read,Grep,Bash(git diff *)"` | Pre-approve tools so nothing prompts (permission-rule syntax; trailing space-star is prefix match) |
| `--permission-mode dontAsk` (or `acceptEdits`, `auto`) | Baseline for unattended runs; `dontAsk` denies anything not pre-approved, good for locked-down CI. With no mode set, a `-p` run starts in Manual (`default`) mode (v2.1.302 or later) and calls that need approval are denied unless an allow rule or hook approves them |
| `--max-turns N`, `--max-budget-usd X` | Hard caps for runaway runs (print mode only); a run that hits the turn limit exits with an error |
| `--append-system-prompt "..."` | Add run-specific instructions while keeping defaults |
| `--continue` / `--resume <id>` | Continue the latest session, or a specific one by session ID or name; capture the ID from the JSON `session_id` field. `--fork-session` creates a new session ID when resuming |
| `--no-session-persistence` | Do not save the session to disk (print mode) |
| `--bare` | Skip auto-discovery of hooks, skills, plugins, MCP servers, auto memory **and CLAUDE.md** for reproducible scripted runs |

Without `--bare`, `claude -p` loads CLAUDE.md, hooks, skills and MCP servers exactly as an interactive session would, including those in the working directory and `~/.claude`.

> Exam wording vs. current docs: the guide says CLAUDE.md is the mechanism for giving CI-invoked Claude Code its project context. That is true for a normal `-p` run. The current docs recommend `--bare` for scripted calls (reproducibility across machines; "will become the default for `-p` in a future release"), and `--bare` **skips CLAUDE.md**. With `--bare` you must pass context explicitly, e.g. `--append-system-prompt-file ./ci-standards.md`; bare mode also never reads your OAuth login or keychain, so it needs `ANTHROPIC_API_KEY` (or an `apiKeyHelper` or cloud-provider credentials). For the exam, "put the standards in CLAUDE.md" is the expected answer; know the `--bare` caveat for real pipelines.

#### Structured output for automation: `--output-format json` + `--json-schema`

Free-text reviews cannot be posted as inline comments reliably; a script needs file, line, severity and message as fields. `--json-schema` constrains the final output to your schema and exposes it as `structured_output`. Two details from the docs worth remembering:

- If a run ends without valid structured output (for example Claude declined the request), the response has **no `structured_output` field**, so your script must check that the field exists before using it.
- Claude Code accepts schemas using the `format` keyword (e.g. `"format": "email"`) but treats it as an annotation and does not enforce it. A schema guarantees *shape*, not that the content is correct: a finding can be well-formed and still be a false positive. Schema-valid does not mean semantically valid.

The extraction pattern from the docs:

```bash
claude -p "Extract function names from auth.py" \
  --output-format json \
  --json-schema '{"type":"object","properties":{"functions":{"type":"array","items":{"type":"string"}}},"required":["functions"]}' \
  | jq '.structured_output'
```

(Domain 4 covers schema design: required vs optional, enums with "other", nullable fields. Here you only need to know the CLI mechanics.)

#### CLAUDE.md as the CI context mechanism

A CI job starts from a clean checkout and no memory of your team's expectations. CLAUDE.md in the repo root is read in `-p` runs (unless `--bare`), so it is where you write: testing standards, what makes a test *valuable* (and what does not), the fixtures and factories that exist, review criteria and severity definitions, and conventions. The GitHub and GitLab docs both say the same: define code style, review criteria and project rules in CLAUDE.md and Claude follows them when creating PRs/MRs and responding to requests. `/code-review` likewise "follows your CLAUDE.md, including in a non-interactive `-p` run".

Keep it short and verifiable (under ~200 lines). Concise CLAUDE.md also lowers per-run cost because it is read on every CI invocation.

#### Session context isolation: independent reviewer beats self-review

When the same session that wrote the code is asked "now review it", Claude still has its own generation reasoning in context ("I did it this way because..."). It tends to confirm its own choices and miss defects it rationalised away earlier. An **independent review instance** (a new session/process with no access to the generating reasoning) sees only the artifact (diff, tests, spec) and evaluates it fresh, so it is more likely to catch real problems. The docs reinforce this:

- `/code-review` reviews the diff "in a fresh subagent that never saw your conversation", so it "isn't swayed by the reasoning that produced the code".
- "A second session is also the easiest independent reviewer you can get": one session writes, another reviews without having watched it being written.
- A reviewer subagent can be given read-only tools (`tools: Read, Grep, Glob`) and told to "report only the gaps that affect correctness" and skip style (a reviewer asked to find problems will always find *something*; telling it what to ignore keeps findings valuable).

In CI this is natural: a review job is a separate `claude -p` process that sees only the PR diff and repo, never the authoring session. Don't implement "review your own work" as a second prompt in the generation session; and don't try to fix self-review blindness by a bigger context window or a higher-tier model alone. Independence, not capacity, is the lever. (Related, in Domain 4: large multi-file PRs should be reviewed in focused per-file passes plus a cross-file integration pass; sample Q12.)

#### Avoiding duplicate or low-value output

- **Re-runs after new commits**: if each push re-reviews the whole PR from scratch, the reviewer re-reports issues already commented on (noise, comment spam). Give the new run the **prior findings** (from the previous run's JSON, or the PR's existing review comments) and an instruction: "Report only issues that are new, or that appear in the prior findings and are still unaddressed in the current diff. Do not repeat findings that have been fixed." Also include which prior findings were fixed, so Claude can verify them against the new diff.
- **Test generation**: give Claude the existing test files (paths via the checkout, or contents in the prompt) and say "do not propose scenarios already covered by these tests". Otherwise it will happily regenerate the happy path tests that already exist.
- **Quality of tests**: write down in CLAUDE.md what a valuable test is (behaviour, edge cases, regressions) versus low-value tests (testing the framework, trivial getters, snapshot-of-everything), and which fixtures exist, so generated tests reuse them.
- **False positives** (Scenario 5 mentions minimising them): explicit review criteria (what to flag, what to ignore) with severity levels in CLAUDE.md or the prompt; this is the same explicit-criteria idea as Domain 4.1.

#### Platform integrations (what the docs show)

- **GitHub Actions**: the `anthropics/claude-code-action@v1` action. Inputs include `prompt` (plain text or a skill invocation; when present the action runs in *automation mode*, when absent it waits for `@claude` mentions), `claude_args` (any CLI flags such as `--max-turns`, `--model`, `--allowedTools`), `anthropic_api_key` (or `claude_code_oauth_token`). Store keys in GitHub Secrets, never in the repo. `/install-github-app` does quick setup.
- **GitLab CI/CD** (beta, maintained by GitLab): add a job to `.gitlab-ci.yml` that installs Claude Code and runs `claude -p "..." --permission-mode acceptEdits --allowedTools "Bash Read Edit Write mcp__gitlab"`, with `ANTHROPIC_API_KEY` as a masked CI/CD variable.
- Both pages recommend CLAUDE.md for standards and `--max-turns` plus job/workflow timeouts for cost control. The GitLab page adds that every change flows through an MR so reviewers see the diff and your branch protection and approvals still apply; the GitHub page says to review Claude's changes before merging.

(Authentication, cloud-provider setup and pricing details on those pages are out of scope for this exam.)

### Worked examples

#### Example 1: The hang and the fix (sample Q10)

```bash
# BAD: starts an interactive session; the pipeline blocks forever waiting for input
claude "Analyze this pull request for security issues"

# GOOD: non-interactive; prints result to stdout and exits
claude -p "Analyze this pull request for security issues"

# NOT real: CLAUDE_HEADLESS=true, claude --batch ..., or "< /dev/null" tricks
```

#### Example 2: Structured review findings and inline PR comments

Schema file `review-schema.json`:

```json
{
  "type": "object",
  "properties": {
    "findings": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "id":         { "type": "string", "description": "Stable id: file + rule + short hash of the code, so re-runs can dedupe" },
          "file":       { "type": "string" },
          "line":       { "type": "integer" },
          "severity":   { "type": "string", "enum": ["critical", "high", "medium", "low"] },
          "category":   { "type": "string", "enum": ["bug", "security", "test-gap", "performance", "other"] },
          "message":    { "type": "string" },
          "suggestion": { "type": ["string", "null"] }
        },
        "required": ["id", "file", "line", "severity", "category", "message"]
      }
    }
  },
  "required": ["findings"]
}
```

CI script (CLAUDE.md in the repo root supplies review criteria; no `--bare`, so it is loaded):

```bash
#!/usr/bin/env bash
set -euo pipefail
PR_NUMBER="$1"

gh pr diff "$PR_NUMBER" > pr.diff

claude -p "Review the pull request diff provided on stdin. Follow the review criteria in CLAUDE.md.
Report only real defects with file and line from the NEW side of the diff.
Return JSON matching the provided schema." \
  --allowedTools "Read,Grep,Glob" \
  --max-turns 15 \
  --output-format json \
  --json-schema "$(cat review-schema.json)" \
  < pr.diff > result.json

# Guard: the structured_output field is absent if the run produced no valid structured output
jq -e '.structured_output.findings' result.json > findings.json

# Post one inline comment per finding (ILLUSTRATIVE: GitHub REST API, not from the Claude docs; check GitHub's docs for exact parameters)
HEAD_SHA=$(gh pr view "$PR_NUMBER" --json headRefOid -q .headRefOid)
jq -c '.[]' findings.json | while read -r f; do
  gh api "repos/{owner}/{repo}/pulls/$PR_NUMBER/comments" \
    -f body="**$(jq -r .severity <<<"$f")**: $(jq -r .message <<<"$f")" \
    -f commit_id="$HEAD_SHA" \
    -f path="$(jq -r .file <<<"$f")" \
    -F line="$(jq -r .line <<<"$f")" \
    -f side="RIGHT"
done
```

The `jq` extraction mirrors the docs' `jq '.structured_output'` example. The `gh api` call is GitHub's pull-request review-comment API, shown here as a plausible way a script would use the structured fields; it is not Claude documentation, so check GitHub's docs for the exact parameters.

#### Example 3: Re-run after new commits: report only new or unaddressed findings

```bash
# prior-findings.json = findings.json saved from the previous run (e.g. as a CI artifact)

claude -p "You are re-reviewing a pull request after new commits.

Prior findings from the earlier review (JSON):
$(cat prior-findings.json)

Task:
1. For each prior finding, check the current code and decide: fixed or still present.
2. Report ONLY (a) new issues introduced or revealed by the new commits and (b) prior findings that are still present and unaddressed.
3. Do NOT repeat findings that are now fixed, and do not rephrase old findings as new ones. Reuse the same id for a still-present finding.
Return JSON matching the schema." \
  --output-format json --json-schema "$(cat review-schema.json)" < pr.diff > result.json
```

Bad: re-running the original prompt unchanged on every push. Result: the same 12 comments posted again after every commit, and reviewers learn to ignore the bot.

#### Example 4: Test generation without duplicates, standards in CLAUDE.md

CLAUDE.md excerpt supplying test context to CI:

```markdown
# Testing standards (read by CI test-generation and review jobs)
- Framework: Vitest. Tests live next to code as *.test.ts.
- A VALUABLE test checks observable behaviour, an edge case (null, empty, boundary), or a past bug.
  Do NOT write tests that assert framework behaviour, trivial getters/setters, or private implementation details.
- Fixtures: use `buildUser()` and `buildOrder({ status })` from tests/factories.ts; use `mockClock()` from tests/helpers/time.ts.
  Never construct raw database rows by hand.
- Each new test needs a name of the form "should ... when ...".
```

Prompt for generation that avoids duplicates:

```bash
claude -p "Generate additional tests for src/orders/refund.ts.
The existing tests are in src/orders/refund.test.ts (read it first).
Do not propose any scenario already covered there. List only missing scenarios, prioritising
edge cases and error paths, then write them following the standards in CLAUDE.md." \
  --allowedTools "Read,Grep,Glob,Edit" --max-turns 20
```

#### Example 5: GitHub Actions snippet (adapted from the docs)

Automation-mode review that runs when a PR is opened or updated, with project standards coming from the repository's CLAUDE.md:

```yaml
name: Claude PR Review
on:
  pull_request:
    types: [opened, synchronize, ready_for_review, reopened]
jobs:
  review:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: read
      issues: read
      id-token: write
    steps:
      - uses: actions/checkout@v6
        with:
          fetch-depth: 1
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            Review pull request ${{ github.event.pull_request.number }} in ${{ github.repository }}.
            Follow the review criteria in CLAUDE.md. Report only new or still-unaddressed issues.
          claude_args: >-
            --max-turns 10
            --allowedTools "Read,Grep,Glob,mcp__github_inline_comment__create_inline_comment"
```

Notes: the `prompt` input makes it automation mode; `claude_args` forwards CLI flags (the docs list `--max-turns`, `--model`, `--allowedTools`); the docs' own code-review example uses `--allowedTools "mcp__github_inline_comment__create_inline_comment"` because the action starts the inline-comment MCP server only when that tool is named. Secrets come from GitHub Secrets. (This snippet is adapted: the docs' official example installs the `code-review` plugin and passes `/code-review:code-review --comment ...` as the prompt; check the page for the exact current form.)

#### Example 6: Independent reviewer vs self-review

```text
Same-session self-review (weaker):
  Session A: "Implement the discount feature" ... code written, with its own reasoning in context
  Session A: "Now review the code you just wrote"
  -> tends to reaffirm its own design decisions; subtle defects it already rationalised are missed.

Independent review (stronger):
  Session A (or CI job 1): implements the feature, opens PR.
  CI job 2: separate `claude -p` process. Context = PR diff + repo + CLAUDE.md criteria. No access to A's reasoning.
  -> evaluates the artifact fresh; can also be given a different, review-specific prompt and read-only tools.
```

Locally the equivalent is `/code-review` (runs in a fresh subagent) or simply opening a second session to review.

### Exam traps

- **`CLAUDE_HEADLESS=true`, `--batch`, `< /dev/null`** for CI (Q10). Only `-p`/`--print` is the documented non-interactive mode.
- **Parsing free-text output with regex** to post PR comments. The better design is `--output-format json` + `--json-schema` and reading `structured_output`.
- **Assuming schema-valid output is correct.** The schema enforces structure, not truth; false positives still need prompt criteria and verification.
- **Re-running the same review prompt on every commit** (duplicate comments). Include prior findings and ask for only new/unaddressed ones.
- **Not telling the test generator about existing tests**: duplicates. Provide the test files and state the rule.
- **Asking the generating session to review itself**, or believing a bigger model/context fixes it. Independence of session context is the fix.
- **Putting CI standards only in the pipeline's prompt string** (duplicated per job) instead of CLAUDE.md. Standards in CLAUDE.md are versioned, reviewed and shared by local and CI usage. (Remember the `--bare` exception.)
- **Hard-coding API keys in the workflow file.** Always repository secrets.
- **Using the Batch API for the blocking pre-merge check** (Q11, Domain 4 material): batch has no latency guarantee; it fits overnight jobs, not blocking checks.

### Check yourself

1. A nightly CI job calls `claude "Summarize today's merged PRs"` and never finishes. What is wrong and what exact change fixes it?
   <details><summary>Answer</summary>`claude "..."` starts an interactive session that waits for input. Use `claude -p "Summarize today's merged PRs"` (or `--print`), which runs non-interactively, prints to stdout and exits.</details>
2. Your CI review posts the same comments again after every push. How do you change the re-run?
   <details><summary>Answer</summary>Pass the prior review findings into the new run's context and instruct Claude to report only new issues or prior ones that remain unaddressed (and not to repeat fixed ones); optionally keep stable finding ids to dedupe.</details>
3. You want the review output to become inline PR comments automatically. Which flags and what must the script check?
   <details><summary>Answer</summary>`claude -p ... --output-format json --json-schema '<findings schema>'`; read the `structured_output` field with jq and post one comment per finding. The script must check that `structured_output` exists, because the field is absent when no valid structured output was produced.</details>
4. Why does a separate CI review job tend to find more real defects than asking the authoring session to "double-check your work"?
   <details><summary>Answer</summary>The authoring session carries its own reasoning and assumptions in context and is biased toward confirming them; an independent instance sees only the artifact and evaluates it fresh. (Bigger context or a stronger model alone does not remove that bias.)</details>

### Official docs

- [Run Claude Code programmatically (headless / `-p`)](https://code.claude.com/docs/en/headless.md) — `-p`, `--bare`, `--output-format`, `--json-schema`, `--allowedTools`, `--continue`/`--resume`.
- [CLI reference](https://code.claude.com/docs/en/cli-reference.md) — exact flag names and behaviours.
- [Claude Code GitHub Actions](https://code.claude.com/docs/en/github-actions.md) — workflows, `prompt`, `claude_args`, CLAUDE.md best practice, cost controls.
- [Claude Code GitLab CI/CD](https://code.claude.com/docs/en/gitlab-ci-cd.md) — `.gitlab-ci.yml` job, CLAUDE.md, `--max-turns`.
- [Review locally with /code-review](https://code.claude.com/docs/en/code-review-local.md) — independent fresh-context review, follows CLAUDE.md in `-p` runs.
- [Best practices: let Claude run without you / run Claude from a script](https://code.claude.com/docs/en/best-practices.md) — five questions before unattended runs, independent reviewer.

---

## Decision cheat-table

| If the question says ... | Prefer ... | Because ... |
|---|---|---|
| New teammate doesn't get the team's Claude rules | Move the instructions to project-level `./CLAUDE.md` / `.claude/CLAUDE.md` / `.claude/rules/` and commit | `~/.claude/CLAUDE.md` is personal and not shared via version control |
| Verify what instruction files Claude actually loaded; behaviour differs between sessions | `/memory` (guide) / `/context` Memory files (docs) | Loaded-file set is the first diagnostic; you cannot reason about behaviour without it |
| CLAUDE.md has become huge and mixes topics | Split into `.claude/rules/` topic files (and `@import` shared standards) | Modular, owned, mergeable; add `paths:` to cut context |
| Each package needs only some shared standards | `@import` the relevant standards file in that package's CLAUDE.md | Single source of truth, package-specific selection (imports do not save tokens) |
| Conventions for file types scattered across many directories (e.g. `*.test.tsx`) | `.claude/rules/*.md` with `paths:` globs | Glob matches by file type anywhere; directory CLAUDE.md is directory-bound |
| Conventions for one specific directory tree | Directory-level CLAUDE.md (or a rule with that path) | Loads on demand when Claude works in that subtree |
| Team-wide `/review` command available on clone | `.claude/commands/` (or `.claude/skills/`) in the repo | Version-controlled; `~/.claude/commands/` is personal |
| Personal tweak of a team skill | `~/.claude/skills/` under a different name | Same name would shadow the team skill for you only |
| Skill output is huge or exploratory (codebase analysis, brainstorming) | `context: fork` in SKILL.md | Isolated subagent; only the result returns to the main conversation |
| Restrict what a skill may do (e.g. writes only) | `allowed-tools` (guide); add `disallowed-tools`/deny rules for hard limits (docs) | Pre-approval alone does not remove other tools |
| Always-needed universal rule vs rarely-needed procedure | CLAUDE.md vs skill | Always-loaded vs on-demand; keep CLAUDE.md small |
| A rule must hold with zero exceptions | Hook / deny rule / managed setting, not CLAUDE.md | CLAUDE.md is context the model may not follow |
| Dozens of files, service boundaries, multiple valid designs | Plan mode, then execute | Prevents costly rework; complexity is known up front |
| Single-file fix with a clear stack trace | Direct execution | Well-scoped; planning is overhead |
| Discovery output would flood context in a multi-phase task | Explore subagent | Read-only, isolated context, returns a summary |
| Prose instructions yield inconsistent transformations | 2–3 concrete input/output examples | Pins down the rule including edge cases |
| Unfamiliar domain (caching, failure modes) before building | Interview pattern, write SPEC.md, then implement | Surfaces unconsidered requirements before code exists |
| Edge case wrong (nulls in migration) | Specific failing test cases with input and expected output | Objective, targeted feedback |
| Several problems that interact vs unrelated problems | One detailed message vs sequential fixes | Coherent joint fix vs focused, reviewable changes |
| CI job hangs waiting for input | `claude -p "..."` | Documented non-interactive mode; no `CLAUDE_HEADLESS` or `--batch` |
| Need machine-parseable findings for inline PR comments | `--output-format json` + `--json-schema`, read `structured_output` | Deterministic structure for scripts; check the field exists |
| Re-review after new commits | Provide prior findings; report only new/unaddressed issues | Avoids duplicate comments |
| Generate tests without repeats | Supply existing test files; document standards and fixtures in CLAUDE.md | Avoids duplicate scenarios and low-value tests |
| Reviewing code Claude just wrote | Independent review instance (fresh session / `/code-review` / separate CI job) | Self-review is biased by the generating context |

## Sources fetched

All fetched on 2026-10-30 (exam guide v1.0 read from the local text file; docs pages fetched with WebFetch, all returned successfully):

- https://code.claude.com/docs/en/memory.md — 2026-10-30
- https://code.claude.com/docs/en/claude-directory.md — 2026-10-30
- https://code.claude.com/docs/en/features-overview.md — 2026-10-30
- https://code.claude.com/docs/en/skills.md — 2026-10-30 (page is ~119K characters; fact-checked against the complete local copy, read end to end)
- https://code.claude.com/docs/en/commands.md — 2026-10-30 (fact-checked against the complete local copy by searching for the relevant command rows: `/plan`, `/memory`, `/context`, `/init`, `/code-review`, `/review`; not read end to end)
- https://code.claude.com/docs/en/sub-agents.md — 2026-10-30 (fact-checked against the complete local copy: built-in subagents, tool filters, invocation, foreground/background, startup context, forks and the previously unread tail were read; the frontmatter, MCP-scoping, hooks and memory sections were not needed for this domain)
- https://code.claude.com/docs/en/common-workflows.md — 2026-10-30
- https://code.claude.com/docs/en/permission-modes.md — 2026-10-30 (fact-checked against the complete local copy: modes table, start-up mode, switching modes, acceptEdits, plan mode, dontAsk and bypassPermissions were read; the auto-mode classifier and critical-path sections were not needed for this domain)
- https://code.claude.com/docs/en/best-practices.md — 2026-10-30
- https://code.claude.com/docs/en/context-window.md — 2026-10-30
- https://code.claude.com/docs/en/headless.md — 2026-10-30
- https://code.claude.com/docs/en/cli-reference.md — 2026-10-30 (flag table read in full for the print/session/permission flags)
- https://code.claude.com/docs/en/github-actions.md — 2026-10-30
- https://code.claude.com/docs/en/gitlab-ci-cd.md — 2026-10-30
- https://code.claude.com/docs/en/code-review-local.md — 2026-10-30
- Official exam guide v1.0 (July 2026): Domain 3 task statements (lines 517–701), scenarios 2, 4 and 5 (lines 92–157), sample questions Q4, Q5, Q6, Q10 (and Q11, Q12 for cross-reference).
