# Domain 2 change log (02_tool_design_mcp_integration.md)

Line numbers refer to the file before editing (approximate; section named for navigation).

## Corrections

1. **Terminology table, Task/Agent row (L53).** Added: do not confuse `Agent` with the task-tracking tools `TaskCreate`/`TaskList`. Evidence (tools-reference.md): "`TaskCreate` | Creates a new task in the task list."
2. **Terminology table, tool-search row (L56).** "defers by default" now carries the model condition. Evidence (mcp.md): "On earlier models, Claude Code loads all MCP tools upfront".
3. **Terminology table, built-ins row (L57) and Platform note (L1163).** Grep/Glob "come back" conditions were incomplete (omitted "subagent leaves out Bash" and "Bash removed from session"). Evidence (tools-reference.md): "lists `Glob` or `Grep` in its `tools` field and leaves out `Bash`"; "removes `Bash` from the session".
4. **Terminology table, isError row (L58).** "MCP defines `isError` only" -> "defines the `isError` flag for tool failures" (MCP also defines `structuredContent`/`outputSchema`, so "only" was misleading). Evidence (server/tools.md): "`outputSchema`: Optional JSON Schema defining expected output structure".
5. **Define tools / "description is the interface" (L84).** "they compete" was presented as docs fact. Now: docs say one assembled prompt; competition is exam-guide reasoning. Evidence (define-tools): "constructs a special system prompt from the tool definitions, tool configuration, and any user-specified system prompt".
6. **input_examples / tool fields (L96).** Added the 400 on invalid examples and the `name` regex. Evidence (define-tools): "Invalid examples return a 400 error"; name "Must match the regex `^[a-zA-Z0-9_-]{1,128}$`".
7. **Keyword example and 2.1 Example 2/5 (L135, L215, L305).** Scenario data (customer-support prompt line, both tools described as "Analyzes content...", misrouting log) marked illustrative; exam guide gives only tool names. Evidence: exam guide 2.1 text lists only names `analyze_content` / `analyze_document`.
8. **Protocol vs execution error table (L363).** "models rarely recover" -> spec wording. Evidence (server/tools.md): "Clients **MAY** provide protocol errors to language models, though these are less likely to result in successful recovery."
9. **L365 wording.** "protocol-level channel is isError" was confusing next to "protocol error"; now "reporting channel".
10. **structuredContent bullet (L372).** Added: spec SHOULD also return serialized JSON in a text block; `outputSchema` MUST/SHOULD semantics; Agent SDK in-process server forwards structuredContent but not text blocks. Evidence (server/tools.md): "a tool that returns structured content SHOULD also return the serialized JSON in a TextContent block"; "Servers MUST provide structured results that conform to this schema"; (custom-tools.md): "Text blocks in `content` are not forwarded".
11. **isRetryable definition (L385).** Presented as the definition; now labelled a common design practice. Added note that the exam guide's `errorCategory` list is transient/validation/permission with business via `retriable: false`; `business` as a category value is a convention. Evidence: exam guide 2.2 Skills lines.
12. **Currency note, forced tool use (L691).** Added that only manual extended thinking blocks forced use. Evidence (define-tools): "Adaptive thinking itself doesn't block forced tool use".
13. **tool_choice Example 3 (L783).** Snippet used `json.dumps` without `import json`; added.
14. **Env expansion, credential variables (L915).** Said "does not expand" this exact list. Docs: read as empty, list is examples ("such as"), `:-default` ignored. Evidence (mcp.md): "Claude Code reads credential variables from your environment as empty rather than expanding them"; "a `:-default` fallback on it is ignored".
15. **Tool search description limits (L924).** Marked as defaults, added the adjusting variable. Evidence (mcp.md): "default limits of 4,096 and 16,384 characters"; `CLAUDE_CODE_MAX_MCP_DESCRIPTION_LENGTH`.
16. **Shell expansion of `${GITHUB_TOKEN}` (L987).** Not on any docs page; labelled ordinary shell behaviour. (No docs passage found by grep for "single quote"/"shell expand".)
17. **User-scope JSON (L999).** Added what docs do say (top level of `~/.claude.json`; local scope under `projects.<path>.mcpServers`); kept "verify in docs" for the key name. Evidence (mcp.md): "the same stdio server name at the top level of `~/.claude.json` (user scope)".
18. **Exam trap (L1110).** "Treating a pending or failed server as working" -> failed/needs-auth; `pending` can be cached/connect-on-first-use. Evidence (agent-sdk/mcp.md): "don't treat `"pending"` as a failure on its own".
19. **Read row (L1156).** Added `allow_large: true` (v2.1.296). Evidence (tools-reference.md): "setting `allow_large: true` when it needs to".
20. **Sources fetched (L1353).** Said mcp.md "read in two parts"; replaced with an accurate statement of the review pass.

## Verified and left unchanged (examples)

tool_choice four values and shapes, auto/none defaults, prefill, skips thinking, `disable_parallel_tool_use` inside `tool_choice`, cache invalidation, 400 models list (define-tools, parallel-tool-use); `tool_result` / `is_error` (handle-tool-calls); protocol vs execution error, `-32602`, empty `contents` rule, resource templates, control hierarchy (MCP tools/resources/prompts/server-concepts/server-index); scopes, locations, precedence, `.mcp.json`, `${VAR}`/`${VAR:-default}` and where they expand, `--transport`/`--scope`/`--env`/`--`, SSE deprecated, approvals in `-p`, tool search values, `alwaysLoad`, @-mentions, `/mcp__server__prompt` (mcp.md); built-in tool behaviour (tools-reference); AgentDefinition fields, `Agent`/`Task` naming, allowedTools vs tools (subagents.md, custom-tools.md); init statuses (agent-sdk/mcp.md); 30-50 tools, 3-5 non-deferred (tool-search-tool); exam Q1/Q2/Q8/Q9 and all five task statements.

## Left as unverifiable

- User-scope JSON key name in `~/.claude.json` (kept "(verify in docs)").
- Claim that `pattern` in a non-strict `input_schema` constrains formats (Example 1 "uses `pattern` to constrain formats"); not checked against strict-tool-use (only grepped). Left unchanged.
- "Forcing the same tool every turn would loop forever" is reasoning, not a docs statement; left as author reasoning.

## Claims checked (approximate)

A product/API facts: ~95; B code/config examples: ~25 blocks; C currency notes: 6 rows; D doc links: 27 URLs/descriptions; E check-yourself/traps: ~45; F attribution: ~15; G quote length: all quotes under 25 words.

## Coverage honesty

Read in full: exam guide Domain 2 + Q1/2/8/9, define-tools, handle-tool-calls, how-tool-use-works, manage-tool-context, troubleshooting-tool-use, MCP spec tools/resources/prompts/server-index, MCP architecture and server-concepts, writing-tools-for-agents, agent-sdk custom-tools, agent-sdk mcp, agent-sdk tool-search. By section/grep only: code.claude.com mcp.md (lines 1-440, 640-1330, 1490-1900 read; NOT read: 490-640 MCP tasks, 1330-1490 claude.ai connectors / Claude Code as MCP server), tools-reference.md (all except Monitor/PowerShell/LSP/WebFetch/WebSearch sections), agent-sdk subagents.md (tool/AgentDefinition sections), parallel-tool-use.md (disable section only), strict-tool-use.md and tool-search-tool.md (grep for specific claims).
