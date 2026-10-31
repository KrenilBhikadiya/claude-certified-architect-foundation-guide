# Domain 4 change log (04_prompt_engineering_structured_output.md)

Checked against complete local copies of the official pages. Evidence quotes are <= 20 words.

## Corrections (what it said -> what it says now)

1. **4.3 "Schema design: required, optional, nullable" paragraph (~line 595).** Said `["string","null"]` is "cheap for strict-mode grammars" and to "prefer required but nullable". Now says both options cost something: optional params count toward the 24 limit; type-array/anyOf params count toward the separate 16 union limit, "especially expensive"; the example uses 7 nullable fields.
   Evidence: structured-outputs.md "Parameters with union types | 16 ... type arrays (for example, `"type": ["string", "null"]`)... especially expensive".
2. **4.5 "Why tool loops do not fit" (~line 1190) - the exam-vs-docs note on "no multi-turn tool calling".** Said batch requests cannot do a client-side round trip, implying the docs say so. Now: quotes the exam guide's limit, states plainly the batch docs never say it in those words, lists what the docs do say is supported, explains the limit as following from async independent processing, marks the "result is a tool_use block, continue in a new request" step as an inference, and keeps the server-tool/pause_turn fact.
   Evidence: batch-processing.md "Tool use, including all server tools"; "Multi-turn conversations"; "The batch worker runs the same server-side agentic loop".
3. **4.5 facts table, "SLA" row.** Quoted "may take up to 24 hours" (not verbatim) and implied the docs speak of an SLA. Now: "might take up to 24 hours"; "no guaranteed latency SLA" flagged as the exam guide's phrasing; added that unprocessed requests come back `expired`, so 24 h caps the wait, not completion.
   Evidence: batch-processing.md FAQ "Batches might take up to 24 hours for processing"; "Batches expire if processing does not complete within 24 hours".
4. **4.5 facts table, "Supported" row.** Listed "tool definitions, system prompts, multi-turn message arrays..." Now also vision, all server tools, prompt caching (best-effort).
   Evidence: batch-processing.md "Vision / Tool use, including all server tools / Multi-turn conversations"; "cache hits are provided on a best-effort basis".
5. **4.5 failure-handling table, `errored` rows.** Presented "document exceeded the context limit -> invalid_request_error" as fact and "other errors = transient". Now: the context-limit example is marked illustrative (docs do not say which error type); other error types: docs sample treats them as retryable, but the type list includes authentication/billing/permission/not-found so check the type.
   Evidence: batch-processing.md sample "Request can be retried directly"; results.md lists AuthenticationError, BillingError, PermissionError, NotFoundError.
6. **4.5 SLA arithmetic.** Said "That is why the guide uses 4-hour windows" (exam guide gives no reasoning). Now: exam guide gives only the 4-hour figure; the margin explanation is our reading.
   Evidence: exam guide Task 4.5 "4-hour windows to guarantee 30-hour SLA with 24-hour batch processing".
7. **4.5 exam trap "Use batch for a workflow that needs your tools executed mid-request - not supported".** Now attributes the limit to the exam guide and notes batches accept tool definitions and server tools.
8. **4.3 `tool_choice` bullets.** "`tool_choice` accepts an optional `disable_parallel_tool_use`" was listed under "from the Define tools page"; that page does not contain it. Now attributed to the API reference and states with `any`/`tool` = exactly one tool call.
   Evidence: batches/create.md "ToolChoiceAny ... If set to `true`, the model will output exactly one tool use."
9. **4.3 strict/JSON-schema limits paragraph.** Added: unsupported features -> 400; limits are counted across strict tool schemas and JSON-output schemas; refusal returns HTTP 200 and is billed; `max_tokens` fix; `reasoning_extraction` warning for reasoning-style schema properties; available on every model.
   Evidence: structured-outputs.md "You'll receive a 200 status code. You'll be billed for the tokens generated."; "might lead to a `reasoning_extraction` refusal"; "available on every model".
10. **4.3 exam trap on prompt-JSON.** Added the prefill statement: prefill returns 400 on Claude 4.6+ and Claude Mythos Preview; docs recommend structured outputs / enum tool for classification.
    Evidence: best-practices "Starting with Claude 4.6 models and Claude Mythos Preview, prefilled responses ... no longer supported".
11. **4.3 "Official docs" bullet for increase-consistency.** Said prefill unsupported on "Claude 4.6+ models"; now adds Claude Mythos Preview and the recommended alternatives.
    Evidence: increase-consistency.md "Prefilling is not supported on Claude 4.6 and later models and Claude Mythos Preview."
12. **4.6 extended thinking vs independent review.** Table and prose presented the comparison as if documented. Now an "Exam-vs-docs note": the thinking page makes no such comparison; the best-practices page recommends self-check for coding/math; exam answer unchanged. Table cell marked "(exam position)"; the "Official docs" bullet for the thinking page rewritten to say it says nothing about independent review.
    Evidence: best-practices.md "Ask Claude to self-check... This catches errors reliably, especially for coding and math."; thinking.md overview describes billing/blocks only.
13. **4.6 evaluator-optimizer paragraph.** Added that the article does not require a fresh-context evaluator (that is the exam guide's requirement).
    Evidence: building-effective-agents "one LLM call generates a response while another provides evaluation and feedback in a loop".
14. **4.6 prompt-chaining/voting paragraph.** Added a clarification that the article's code-review voting example is not the same as the Q12 "2 of 3 consensus" distractor.
    Evidence: building-effective-agents "several different prompts review and flag the code if they find a problem".
15. **4.1 rule of thumb 3.** Said few-shot is "the strongest prompt-only lever" and "beat a long list of rules" (overreach). Now states the exam guide's wording and softens the second clause.
    Evidence: exam guide Task 4.2 "most effective technique for achieving consistently formatted, actionable output when detailed instructions alone produce inconsistent results".
16. **4.1 confidence reasoning bullet.** Marked "a false positive is usually something the model is confident about" as the guide's reasoning, not a docs statement.
17. **4.2 "Reasoning in the example is what generalizes".** Added attribution: exam-guide framing; docs only say relevant/diverse/structured, "explaining why lets Claude generalize", and multishot examples shape thinking; warning about reasoning_extraction.
    Evidence: best-practices "Multishot examples work with thinking"; "Claude can generalize from the explanation."
18. **4.1 Example 5 (CI).** Added: `--bare` does not use subscription login, set `ANTHROPIC_API_KEY`; file path marked illustrative.
    Evidence: headless.md "bare mode doesn't use your subscription login".
19. **4.3 Agent SDK paragraph.** Added that single-shot `query()` raises after an error result.
    Evidence: agent-sdk/structured-outputs.md "A single-shot query() raises after yielding an error result".
20. **4.3 Check-yourself Q1 answer.** Added a pointer for the models that reject forced tool use (no change to the exam answer).
21. **4.6 /code-review bullets (two places).** "Runs as a background subagent" qualified: in a terminal session; in `-p` it waits and returns findings.
    Evidence: code-review-local.md "Run it in non-interactive mode, with the `-p` flag... Claude Code waits for the review".
22. **Sources fetched parenthetical.** Replaced "read in parts" with an honest statement of what was read in full vs passage by passage.

## Checked and found correct (no change)
- 50% discount, 100,000 requests / 256 MB, 29-day retention, `custom_id` regex, result order not guaranteed, four result types and "not billed" for errored/canceled/expired, `processing_status` values, unsupported params (`stream`, `speed`, `fallbacks`, `max_tokens: 0`), 60 s polling example (batch-processing.md, create.md, results.md).
- SDK names: `client.messages.batches.create/retrieve/results`, `MessageCreateParamsNonStreaming`, `Request`, `outcome.error.error.type`, `client.messages.parse(..., output_format=Model)` -> `parsed_output`.
- `tool_choice` four modes and defaults; any/tool prefill => no text before `tool_use`; forcing skips thinking; manual extended thinking rejects any/tool; 400 on Opus 5.5, Sonnet 5.5, Fable 5.1, Mythos 5.1 (define-tools.md, thinking.md).
- Tool name regex, description "by far the most important factor", `input_examples` (define-tools.md).
- Few-shot: "3-5 examples", relevant/diverse/structured, `<example>`/`<examples>` (best-practices.md); "more effective than abstract instructions" (increase-consistency.md).
- Headless: `--json-schema`, `structured_output` absent on decline, invalid-schema error text, `format` annotation, `--bare`, `--append-system-prompt(-file)`, `--continue/--resume` (headless.md).
- Agent SDK: `output_format`, `structured_output`, `StructuredOutput` tool, `error_max_structured_output_retries`, "success AND structured_output" check, optional-field tip.
- Exam-guide statements for 4.1-4.6, Q2, Q3, Q11, Q12, Scenarios 5/6 match the exam guide text.

## Left as unverifiable
- That an oversized document surfaces as `errored`/`invalid_request_error` in a batch: marked "illustrative" (docs silent). The 4.5 Check-yourself Q2 scenario still stipulates it as a given.
- That a batch request calling a client tool returns the `tool_use` block as its result: marked "by implication (the docs do not spell this out)".
- `stop_reason: "tool_use"` after a tool call: not in define-tools.md; confirmed in other fetched pages (handle-tool-calls, how-tool-use-works), left as is.
- "Consider a smaller batch" for expired requests and the SLA "2 h margin" reasoning: author advice, now labelled as such for the SLA; the smaller-batch hint is consistent with "breaking very large datasets into multiple batches" but not stated for expiry.
- All design advice not tied to a docs page (severity anchors, `detected_pattern` workflow, per-file/integration pass design, confidence routing) is exam-guide pattern or author design; headings/wording already framed as the exam's, left unchanged.

## Claims checked (approximate)
- A (product/API facts, prose and tables): ~70 checked; 12 corrected/softened.
- B (code/config examples; 14 Python blocks, 5 CLI/bash, JSON shapes): ~30 checked; 0 wrong API shapes; 3 annotations added (CI api key, query() raising, illustrative path).
- C (docs-differ-from-exam notes: tool_choice restrictions, no multi-turn tool calling, thinking vs review): 3 checked; 2 rewritten (batch note, thinking note); tool_choice note correct.
- D (doc links / Sources fetched): 28 URLs and descriptions checked; 3 descriptions fixed (increase-consistency bullet, thinking bullet, code-review bullet), parenthetical in Sources fixed.
- E (Check yourself / Exam traps): ~25 checked; 3 touched.
- F (attribution): ~15 statements re-attributed (items 12, 13, 15, 16, 17, 6, 3).
- G (quote length): all quotes <= 20 words; none shortened.

## Coverage statement
Read in full: batch-processing.md (all non-duplicated-language sections; code tabs skimmed per language), claude-prompting-best-practices.md (all), increase-consistency.md, reduce-hallucinations.md, prompt-engineering overview.md, agent-sdk/structured-outputs.md, headless.md (lines 1-75, 150-200 plus grep of the rest), code-review-local.md, building-effective-agents. structured-outputs.md: lines 1-140, 738-780, 2160-2862 (limitations, invalid outputs, complexity, compatibility) read; the multi-language code tabs/use-case accordions (lines ~140-738, ~780-2160) located by grep only. strict-tool-use.md: prose read, code tabs skipped. define-tools.md: lines 1-180, 572-641, 870-950 read, remainder grep. thinking.md: lines 40-110, 920-970, 1218-1260 read plus grep. batches/create.md and results.md: relevant passages only (params, tool_choice, MessageBatch object, result variants).
