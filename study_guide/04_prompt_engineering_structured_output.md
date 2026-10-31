# Domain 4: Prompt Engineering & Structured Output (20%)

> This domain is about getting a model to behave **precisely and predictably when it is a component in a pipeline**: review prompts that flag real bugs without drowning developers in noise, few-shot examples that pin down format and judgment, tool-use schemas that make output machine-readable, validation-and-retry loops that fix what schemas cannot, batch processing that trades latency for cost, and multi-pass review designs that stop a model from grading its own homework. It carries 20% of the exam because two of the six scenarios lean on it directly: **Scenario 5 (Claude Code for CI/CD)** - automated review, test generation and false-positive control - and **Scenario 6 (Structured Data Extraction)** - JSON-schema extraction, validation and edge cases. Sample questions Q3 (escalation calibration), Q11 (batch vs real-time) and Q12 (multi-pass review) all test ideas from this file.

## Contents

- [Mental model](#mental-model)
- [4.1 Design prompts with explicit criteria to improve precision and reduce false positives](#41-design-prompts-with-explicit-criteria-to-improve-precision-and-reduce-false-positives)
- [4.2 Apply few-shot prompting to improve output consistency and quality](#42-apply-few-shot-prompting-to-improve-output-consistency-and-quality)
- [4.3 Enforce structured output using tool use and JSON schemas](#43-enforce-structured-output-using-tool-use-and-json-schemas)
- [4.4 Implement validation, retry, and feedback loops for extraction quality](#44-implement-validation-retry-and-feedback-loops-for-extraction-quality)
- [4.5 Design efficient batch processing strategies](#45-design-efficient-batch-processing-strategies)
- [4.6 Design multi-instance and multi-pass review architectures](#46-design-multi-instance-and-multi-pass-review-architectures)
- [Decision cheat-table](#decision-cheat-table)
- [Sources fetched](#sources-fetched)

## Mental model

Think of every prompt-driven component as a **function with a contract**. The exam keeps asking the same underlying question: *what is the cheapest mechanism that actually enforces the contract?* The mechanisms form a ladder, from weakest to strongest guarantee:

| Rung | Mechanism | What it buys you | Where it stops |
|---|---|---|---|
| 1 | Vague instruction ("be accurate", "be conservative") | Almost nothing measurable | Model has no definition to apply |
| 2 | Explicit categorical criteria + severity definitions | Precision: model knows what to report vs skip | Still probabilistic |
| 3 | Few-shot examples (2-4 targeted) | Consistent format, calibrated judgment on ambiguous cases | Still probabilistic |
| 4 | Tool use / JSON schema (`strict: true`) | **Syntax and type guarantees** on the output | Cannot guarantee *meaning* (sums, field placement) |
| 5 | Validation + retry with specific error feedback | Catches semantic/format errors, self-correction | Useless when the information is absent from the source |
| 6 | Independent second pass / human review | Catches what the generator is blind to | Costs more; needs routing |

Rules of thumb for the whole domain:

1. **Explicit criteria beat vague guidance.** "Flag a comment only when its claimed behavior contradicts the code" works; "check that comments are accurate" and "be conservative" do not.
2. **Categorical criteria beat confidence filtering.** Tell the model *which kinds of issue* to report and skip; do not ask it to guess how sure it is.
3. **Few-shot examples are the exam's answer for inconsistent output when detailed instructions alone have failed** (the exam guide calls them the most effective technique for consistently formatted, actionable output), especially for ambiguous cases; 2-4 well-chosen, diverse examples that show *why* a choice was made usually do more than adding more rules.
4. **Schemas guarantee shape, not truth.** Strict tool use removes JSON syntax/type errors; it does not stop a model from putting a value in the wrong field or producing line items that do not sum.
5. **Let the schema say "I don't know".** Nullable/optional fields, an `"unclear"` enum value and `"other"` + detail string stop the model from fabricating values to satisfy a required field.
6. **Retry with the specific error, but only when the information exists.** Format and structural failures are fixable; absent information is not.
7. **Batch = 50% cheaper, up to 24 h, no latency SLA.** Use it only for non-blocking work. Match the API to the latency requirement; resubmit only failures, by `custom_id`.
8. **A model reviewing its own output in the same session is biased by its own reasoning.** Use an independent instance, and split big reviews into per-file passes plus a cross-file integration pass.

Reasoning pattern the exam rewards: *fix the root cause first with the lowest-effort high-leverage change, avoid over-engineering, and use programmatic enforcement when compliance must be guaranteed.* In this domain that usually means: sharpen the prompt (criteria + examples) before building classifiers; use schemas for syntax and code-level validation for semantics; use a second instance rather than a "think harder" instruction.

---

## 4.1 Design prompts with explicit criteria to improve precision and reduce false positives

### What the exam tests

**Knowledge of:**

- Explicit criteria beat vague instructions. The guide's own pair: *"flag comments only when claimed behavior contradicts actual code behavior"* (good) versus *"check that comments are accurate"* (bad).
- General instructions such as "be conservative" or "only report high-confidence findings" do **not** raise precision. Specific, categorical criteria do.
- False positives destroy trust: if one category is noisy, developers start dismissing *everything*, including the categories that are accurate.

**Skills in:**

- Writing criteria that say which issues to **report** (bugs, security) and which to **skip** (minor style, local patterns), instead of confidence-based filtering.
- **Temporarily disabling** a high-false-positive category to restore trust while the prompt for that category is improved.
- Defining **severity levels with a concrete code example for each level**, so classification is consistent.

### Concepts explained

#### Why vague instructions fail

A model told to "check that comments are accurate" has to invent its own definition of "accurate". One run flags a comment because the wording is imprecise, another because the comment is outdated, another because it disagrees with a style guide it imagined. Those are *different tests*, applied inconsistently, and most of them produce findings the team does not care about.

The fix is to **replace the judgment word with an observable condition**. Anthropic's general prompting guidance makes the same point: Claude responds well to clear, explicit instructions, and a useful test is whether a colleague with no context could follow the prompt without confusion. Think of the model as a capable new hire who does not know your team's norms: the more precisely you state what you want, the better the result.

| Vague (bad) | Explicit criterion (good) |
|---|---|
| "Check that comments are accurate." | "Flag a comment only when the behavior it claims contradicts what the code actually does (for example, it says the function returns `None` on a missing user but the code raises). Do not flag outdated wording, typos, missing comments or TODOs." |
| "Look for security problems." | "Report: SQL built by string concatenation or f-strings from request data; secrets committed in source; missing authorization check on a route that mutates data. Skip: theoretical issues that require an attacker to already control the server." |
| "Review this PR." | "Report bugs (incorrect behavior on valid input) and security issues. Skip minor style, naming, formatting, and patterns that match the surrounding file's conventions." |

#### Why "be conservative" and "only high-confidence findings" fail

These sound sensible but they hand the model a **threshold with no definition**:

- The model has no calibrated internal "confidence" to compare against your threshold. Self-reported confidence is poorly calibrated (the exam guide says so explicitly in sample Q3: the model is "already incorrectly confident on hard cases").
- A false positive is usually something the model *is* confident about (a "bug" that is really an accepted local pattern). Raising the confidence bar does not remove it; it only changes how the model words its certainty. (This is the guide's reasoning behind the exam answer, not a statement from a docs page.)
- It can also suppress **true** findings that look unusual, so you lose recall without gaining precision.

What works is a **categorical rule**: name the categories to report and the categories to skip, and say what makes a borderline case count. The decision is then a matter of classification, not of introspection.

```text
BAD
Review the diff. Be conservative: only report issues you are highly confident about.

GOOD
Review the diff.
REPORT only:
  1. Bugs: code that produces wrong results, crashes, or loses data on valid input.
  2. Security: injection, secrets in source, missing authorization on state-changing routes,
     unsafe deserialization of untrusted input.
  3. Comment/code contradiction: a comment whose claimed behavior contradicts the code.
SKIP (do not mention):
  - Style, naming, formatting, import order.
  - Patterns that are consistent with the rest of the file or repo (local conventions),
    even if you would write them differently.
  - Missing tests or docs, unless the PR description promises them.
```

#### Trust is an asset with a shared budget

Developers do not evaluate categories independently. If "comment accuracy" findings are wrong 60% of the time, they learn to scroll past automated review comments, and the security findings get skimmed too. So a **noisy category costs you more than its own false positives**: it lowers the value of every accurate category.

That leads to a practical operating procedure, which the exam tests directly:

1. **Measure per-category precision** (how often developers accept vs dismiss each category; see `detected_pattern` in 4.4).
2. **Temporarily disable** the worst category (remove it from the REPORT list) so developers see only high-precision output again.
3. **Improve that category's prompt** off to the side - tighter criteria, few-shot examples of acceptable patterns (4.2) - and test it against a labeled sample.
4. **Re-enable** it once measured precision is acceptable.

Disabling is a *temporary trust-restoration measure*, not a fix: the fix is the improved prompt.

#### Severity levels need anchors

"Rate severity as critical, high, medium or low" produces inconsistent labels, because each run (and each reviewer) places the lines differently. The fix is the same as everywhere in this domain: **define each level with a rule and a concrete code example**. A model can pattern-match a new finding against five concrete anchors far more consistently than it can apply abstract adjectives.

### Worked examples

#### Example 1 - Scenario 5: review criteria prompt with severity anchors

```text
You review pull-request diffs for a payments service.

## What to report
Report ONLY issues in these categories. Anything else: say nothing.
- bug        : wrong result, crash, data loss or race on VALID input
- security   : injection, secret in source, missing authz on a mutating route, unsafe deserialization
- comment    : a comment whose CLAIMED behavior contradicts what the code does

## What to skip
- Minor style, naming, formatting, import ordering
- Patterns consistent with surrounding code in the same file (local conventions)
- Missing docs/tests, unless the diff itself removes existing tests
- Comments that are merely vague, outdated in wording, or contain typos

## Severity (use exactly these definitions)
critical - exploitable by an unauthenticated user, or silently corrupts money/data.
  Example:
    db.execute(f"SELECT * FROM accounts WHERE id = {request.args['id']}")   # SQL injection

high - wrong behavior on a common valid input; no exploit needed.
  Example:
    for i in range(len(items) + 1):     # off-by-one: IndexError on every call
        total += items[i].price

medium - wrong behavior on an uncommon path, or an error that is swallowed.
  Example:
    try:
        charge(card, amount)
    except Exception:
        pass                              # payment failure silently ignored

low - correct today but a defect trap worth noting; does not block merge.
  Example:
    # returns None when the user is missing
    def get_user(uid): ...                # actually raises KeyError -> comment/code contradiction

## Output
For each finding give: location (file:line), issue, severity, suggested_fix.
```

Why this works: every severity has an observable test ("exploitable by an unauthenticated user", "wrong on a common valid input"), plus an anchor snippet. The "skip" list stops the model from reporting formatting churn, and nothing in the prompt asks the model to guess how confident it is.

#### Example 2 - the comment-accuracy criterion, applied

```python
# Claimed: "Returns None if the order is not found."
def get_order(order_id):
    return ORDERS[order_id]          # raises KeyError instead
# -> FLAG: claimed behavior (returns None) contradicts actual behavior (raises KeyError)

# Claimed: "Fetch the order." (vague, but not contradicted)
def get_order(order_id):
    return ORDERS.get(order_id)
# -> DO NOT FLAG: nothing the comment claims contradicts the code

# Claimed: "Retry up to 3 times." (code retries 5 times)
for attempt in range(5):
    ...
# -> FLAG: concrete claim (3) contradicts concrete behavior (5)
```

#### Example 3 - bad vs good, side by side (the exam's pattern)

| | Prompt | Likely result |
|---|---|---|
| Bad | "Be conservative. Only report findings you are at least 90% sure about." | Same noisy categories, now with confident-sounding wording; real issues that "look odd" get dropped. |
| Bad | "Check that comments are accurate." | Flags typos, outdated wording and style mismatches: low precision, inconsistent. |
| Good | "Flag a comment only when its claimed behavior contradicts the code. Skip typos, vagueness and missing comments." | Narrow, checkable, high precision. |

#### Example 4 - temporarily disabling a noisy category (config-driven prompt)

```python
# review_config.py
ENABLED_CATEGORIES = {
    "bug": True,
    "security": True,
    "comment_mismatch": False,   # disabled 2026-10-12: 58% of findings dismissed.
                                 # Re-enable after few-shot examples raise precision on the labeled sample.
}

CATEGORY_RULES = {
    "bug": "wrong result, crash, data loss or race on valid input",
    "security": "injection, secrets in source, missing authz on a mutating route",
    "comment_mismatch": "a comment whose claimed behavior contradicts the code",
}

def build_report_list():
    lines = [f"- {name}: {CATEGORY_RULES[name]}"
             for name, on in ENABLED_CATEGORIES.items() if on]
    return "\n".join(lines)
```

The prompt template renders `build_report_list()` into the REPORT section, so toggling a category is a one-line change that does not touch the rest of the prompt.

#### Example 5 - criteria in CI (Scenario 5)

In CI you run Claude Code non-interactively with `-p`. Two facts from the docs matter for criteria delivery:

- Plain `claude -p` loads the project's `CLAUDE.md` as an interactive session would, so team review criteria can live there.
- With `--bare` (recommended for scripted calls, because it makes the run identical on every machine), `CLAUDE.md`, hooks, skills and so on are **skipped**; supply context explicitly, for example with `--append-system-prompt` or `--append-system-prompt-file`. Bare mode also does not use your subscription login, so set `ANTHROPIC_API_KEY` in the CI environment.

```bash
# Criteria passed explicitly so the result does not depend on the runner's files (file path is illustrative)
gh pr diff "$PR" | claude --bare -p \
  --append-system-prompt-file ci/review_criteria.md \
  --output-format json
```

### Exam traps

- **"Add 'be conservative' / 'only report high-confidence findings'"** - tempting, always wrong as the primary fix. It is a threshold with no definition. Prefer categorical criteria.
- **"Have the model self-report a confidence score and filter below a threshold"** - the same family of wrong answer (see sample Q3 option B: self-reported confidence is poorly calibrated).
- **"Train/deploy a separate classifier"** when prompt criteria have not been tried - over-engineering; the first step is explicit criteria.
- **"Disable the category permanently"** - the guide says *temporarily*, while the prompt is improved.
- **"Raise severity wording ('CRITICAL: ...') to make it consistent"** - emphasis does not define levels; concrete examples per level do.
- **"Switch to a bigger model"** to reduce false positives - the root cause is unclear criteria, not capability.

### Check yourself

1. A CI reviewer flags too many "inaccurate comment" findings, most of which are about wording. Name the single most effective prompt change.
   <details><summary>Answer</summary>Replace "check that comments are accurate" with an explicit criterion: flag a comment only when its claimed behavior contradicts what the code does, and explicitly skip wording, typos and missing comments. Specific categorical criteria, not a confidence threshold.</details>
2. Developers have started ignoring all automated review comments, even security ones, because the "naming consistency" category is wrong half the time. What should you do in the short term, and what in the long term?
   <details><summary>Answer</summary>Short term: temporarily disable the noisy category so the remaining categories regain credibility. Long term: improve that category's prompt (tighter criteria, examples of acceptable patterns), validate precision on a labeled sample, then re-enable. False positives in one category undermine trust in the accurate ones.</details>
3. Why is "classify severity as critical/high/medium/low" insufficient, and what does the guide recommend?
   <details><summary>Answer</summary>The labels have no shared definition, so classification drifts between runs. Define each level with an explicit rule and a concrete code example, so new findings can be matched against anchors.</details>

### Official docs

- [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) - "Be clear and direct", "Add context to improve performance" (explaining *why* an instruction exists lets Claude generalize), "Control the format of responses" (tell the model what to do, not only what not to do).
- [Prompt engineering overview](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/overview) - define success criteria and tests before prompt-tuning.
- [Run Claude Code programmatically (headless)](https://code.claude.com/docs/en/headless.md) - `-p`, `--bare`, `--append-system-prompt(-file)` for delivering criteria in CI.
- [Review locally with /code-review](https://code.claude.com/docs/en/code-review-local.md) - the built-in reviewer follows your `CLAUDE.md`; effort level `low` reports only its most confident findings (a product control, not a substitute for explicit criteria).

---

## 4.2 Apply few-shot prompting to improve output consistency and quality

### What the exam tests

**Knowledge of:**

- Few-shot examples are the **most effective technique for consistently formatted, actionable output** when detailed instructions alone give inconsistent results.
- Examples demonstrate **ambiguous-case handling**: which tool to choose for an ambiguous request, how to describe a branch-level test coverage gap.
- Examples let the model **generalize judgment to novel patterns** instead of matching only the cases you listed.
- Examples reduce **hallucination in extraction** (informal measurements, varied document structures).

**Skills in:**

- Creating **2-4 targeted** examples for ambiguous scenarios that show the *reasoning* for choosing one action over plausible alternatives.
- Including examples that show the exact **output format** (location, issue, severity, suggested fix).
- Including examples that **distinguish acceptable code patterns from genuine issues**, reducing false positives while still generalizing.
- Using examples that show correct handling of **varied document structures** (inline citations vs bibliographies; methodology sections vs details embedded elsewhere).
- Using examples of extraction from **varied formats** to fix **empty/null extraction of required fields**.

### Concepts explained

#### Why examples beat more instructions

A rule describes a behavior; an example **is** the behavior. When instructions alone give inconsistent output - sometimes a bulleted list, sometimes prose, sometimes a finding without a location - adding more words rarely helps, because the model is still interpreting. One or two worked examples remove the interpretation: format, level of detail, tone and decision boundary are all visible at once. Anthropic's guidance calls examples "one of the most reliable ways" to steer format, tone and structure, and the consistency guide says constraining with examples "is more effective than abstract instructions".

#### What makes a good example set

The docs list three properties: **relevant** (mirrors your real use case), **diverse** (covers edge cases and varies enough that the model does not pick up an unintended pattern), and **structured** (wrapped in `<example>` tags, several inside `<examples>`, so they are not confused with instructions). The docs suggest 3-5 examples in general; the exam guide specifically says **2-4 targeted examples for ambiguous scenarios**. Both agree on the principle: a *few* carefully chosen examples, not dozens. Exam answers follow the guide's 2-4.

Where to spend your examples - target the cases where the model currently goes wrong:

| Target | What the example shows |
|---|---|
| **Ambiguous tool selection** | A request that could plausibly use two tools, with the *reasoning* for the correct pick |
| **Output format** | Exactly one finding in your required shape (location, issue, severity, suggested fix) |
| **Acceptable vs genuine** | A pattern that looks suspicious but is fine (skip), next to a similar one that is a real bug (report) |
| **Varied inputs** | The same field extracted from two different document layouts |
| **Absence** | A document where the field truly is not there, and the correct output is `null` |

#### Reasoning in the example is what generalizes

An example that shows only input and answer teaches a mapping. An example that includes **one sentence of why** teaches a rule the model can apply to new inputs. That is how few-shot "enables the model to generalize judgment to novel patterns rather than matching only pre-specified cases": the model extracts the *principle* behind the examples. (This is the exam guide's framing - "show reasoning for why one action was chosen over plausible alternatives". The prompting best-practices page does not say this about few-shot examples; it says examples should be relevant, diverse and structured, that explaining *why* an instruction exists lets Claude generalize, and that examples shape how Claude approaches similar problems when thinking is on. Keep the "why" to a short explanation: the docs warn that asking for step-by-step reasoning written out in the output can trigger a `reasoning_extraction` refusal on newer models.)

#### How few-shot reduces hallucination in extraction

Fabrication happens when the model feels pressure to fill a field. Examples relieve that pressure in two ways: they show **where** a value hides in unusual layouts (so the model finds it instead of returning empty), and they show **when to emit null/"unclear"** (so it does not guess). Both address the guide's "empty/null extraction of required fields" problem: empty output often means the model did not recognize the value in this layout, not that it is absent.

#### Few-shot vs fixing tool descriptions

Be careful: in sample Q2 (two similar tools, minimal descriptions), the correct *first step* was **expanding the tool descriptions**, not adding 5-8 few-shot examples. Few-shot is right when the criteria are already clear but the *edge-case judgment* is inconsistent (Q3: escalation criteria **plus** few-shot examples). Order of operations: clear descriptions/criteria first, examples to calibrate the ambiguous remainder.

### Worked examples

#### Example 1 - ambiguous tool selection with reasoning (Scenario 1 style)

```text
<examples>
<example>
Request: "Check on order #88213 - I never got my confirmation email."
Chosen tool: lookup_order(order_id="88213")
Why: the customer gives an order number and asks about an order's state. get_customer
would only return profile data, not order status. Verify the account afterwards if a
change (refund, address edit) is needed.
</example>

<example>
Request: "I'm jane@acme.com, can you see why my last two orders are late?"
Chosen tool: get_customer(email="jane@acme.com"), then lookup_order for each returned order
Why: no order number is given, so the account must be identified first; the order IDs
come from the customer record.
</example>

<example>
Request: "Cancel my subscription and also tell me what I owe."
Chosen tool: get_customer first (identity), then escalate if cancellation is not an
available action.
Why: two intents; the identity check comes before anything that changes state, and an
unsupported action is escalated rather than improvised.
</example>
</examples>
```

Three examples, each with a "Why" that names the alternative it beats. None of them lists every possible request; the "Why" lines carry the generalizable rule.

#### Example 2 - consistent review-finding format (Scenario 5)

```text
<examples>
<example>
<diff_excerpt>
+ def apply_discount(price, pct):
+     return price - price * pct
</diff_excerpt>
<finding>
location: pricing.py:41
issue: pct is used as a fraction here, but callers pass whole percents (e.g. 15), so a 15% discount yields a negative price.
severity: high
suggested_fix: Divide by 100 (price * (1 - pct / 100)) or validate 0 <= pct <= 1 at the boundary.
</finding>
</example>
</examples>
```

One example of the exact shape (location, issue, severity, suggested fix) removes most format drift. Add a second only if you need to show a different severity or category.

#### Example 3 - acceptable pattern vs genuine issue (reduces false positives, still generalizes)

```text
<example>
<code>
try:
    cache.set(key, value)
except ConnectionError:
    log.warning("cache unavailable, continuing without cache")
</code>
<verdict>SKIP</verdict>
<reason>Swallowing a cache failure is an intentional degrade-gracefully pattern: the
cache is an optimization, the code logs the failure, and the main path continues
correctly. Do not report.</reason>
</example>

<example>
<code>
try:
    ledger.write(entry)
except Exception:
    pass
</code>
<verdict>REPORT severity=medium</verdict>
<reason>Same try/except shape, but here the swallowed operation is the system's source
of truth and nothing is logged: a failed write is silently lost.</reason>
</example>
```

Pairing two *superficially identical* snippets with opposite verdicts is the point: the model learns the distinguishing feature (what is being swallowed, and whether it is logged), so it also handles a third pattern it has never seen.

#### Example 4 - branch-level test coverage gap (test generation)

```text
<example>
<code>
def shipping_cost(weight_kg, express):
    if weight_kg > 20:
        base = 40
    else:
        base = 10
    return base * 2 if express else base
</code>
<existing_tests>shipping_cost(5, False) == 10 ; shipping_cost(25, False) == 40</existing_tests>
<gap>
location: shipping_cost, branch `express == True` (both weight branches)
issue: The express multiplier is never exercised; 2 of 4 branch combinations untested.
severity: medium
suggested_fix: Add shipping_cost(5, True) == 20 and shipping_cost(25, True) == 80.
</gap>
</example>
```

This demonstrates the guide's "branch-level" notion: coverage reported per decision branch, not "needs more tests".

#### Example 5 - varied document structure (Scenario 6): inline citations vs bibliography

```text
<examples>
<example>
Document A (inline citations): "...reduced error by 12% (Chen et al., 2023) and ..."
Extraction: {"claim": "reduced error by 12%", "source": "Chen et al., 2023", "source_location": "inline"}
</example>

<example>
Document B (numbered bibliography): "...reduced error by 12% [4]... References: [4] Chen, L. et al. (2023). Title. Journal."
Extraction: {"claim": "reduced error by 12%", "source": "Chen, L. et al. (2023). Title. Journal.", "source_location": "bibliography"}
Note: resolve the [4] marker through the reference list; do not return "[4]" as the source.
</example>

<example>
Document C (no citation for the claim): "...error fell sharply in production."
Extraction: {"claim": "error fell sharply in production", "source": null, "source_location": null}
Note: do not attach the nearest citation; if the claim has none, source is null.
</example>
</examples>
```

The third example matters as much as the first two: it teaches the model that `null` is a valid answer.

#### Example 6 - methodology section vs details embedded elsewhere

```text
<example>
Document: sample size is only mentioned in a figure caption ("n = 214 participants").
Methodology section says only "a convenience sample was recruited".
Extraction: {"sample_size": 214}
Why: the methodology section is the usual place, but the value can sit in captions, tables
or footnotes. Search the whole document before returning null.
</example>
```

This example attacks the "empty required field" problem from the other side: the value *is* there, in an unexpected place.

#### Example 7 - informal measurements

```text
<examples>
<example>
Text: "stir in a couple of tablespoons of butter"
Extraction: {"quantity": 2, "unit": "tbsp", "original_text": "a couple of tablespoons", "precision": "approximate"}
</example>
<example>
Text: "season with a pinch of salt"
Extraction: {"quantity": null, "unit": "pinch", "original_text": "a pinch", "precision": "unquantified"}
Why: "pinch" has no standard numeric value; do not invent one.
</example>
<example>
Text: "about 2 kg-ish of flour"
Extraction: {"quantity": 2, "unit": "kg", "original_text": "about 2 kg-ish", "precision": "approximate"}
</example>
</examples>
```

The pattern - normalize when you safely can, keep `original_text`, mark approximations, and use null when there is no principled value - cuts hallucinated precision.

### Exam traps

- **"Add 5-8 (or 20) examples"** - more is not better; the guide says 2-4 targeted examples for ambiguous cases. Also, examples add tokens on every call.
- **"Use few-shot instead of improving tool descriptions"** - for tool-selection failures with minimal descriptions, descriptions come first (sample Q2).
- **"Examples only for the happy path"** - the value is in ambiguous and negative cases (acceptable pattern, null-valued field).
- **"Examples without reasoning"** - they teach a lookup, not a principle; the guide explicitly says to show reasoning for why one action was chosen over plausible alternatives.
- **"Few-shot guarantees format"** - it makes format *likely*; if format must be guaranteed use tool use / strict schemas (4.3). If tool-sequence compliance must be guaranteed, use programmatic enforcement (Q1).

### Check yourself

1. A review prompt with a long rules list yields findings with inconsistent structure. What is the most effective prompt-only fix?
   <details><summary>Answer</summary>Add a small set (2-4) of few-shot examples that each show the exact output format (location, issue, severity, suggested fix), including at least one that shows a skip decision.</details>
2. Why include an example where the correct extraction for a field is `null`?
   <details><summary>Answer</summary>It shows the model that absence is a valid answer, which reduces fabricated values for required-looking fields. Examples of where the value hides in other layouts cover the opposite failure (empty extraction when the value exists).</details>
3. Your two similar tools are mis-selected and both descriptions are one line long. Is "add few-shot examples" the best first step?
   <details><summary>Answer</summary>No. Tool descriptions are the primary mechanism for tool selection; expand them first (inputs, examples, boundaries versus similar tools). Few-shot examples are a later refinement for remaining ambiguous cases.</details>

### Official docs

- [Prompting best practices - Use examples effectively](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) - relevant/diverse/structured; `<example>` / `<examples>` tags.
- [Increase output consistency](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/increase-consistency) - "Constrain with examples", specifying exact output formats.
- [Define tools - Providing tool use examples](https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools) - the optional `input_examples` field and the principle that descriptions matter most.
- [Reduce hallucinations](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations) - allow "I don't know", quote-grounding, citation-backed claims.

---

## 4.3 Enforce structured output using tool use and JSON schemas

### What the exam tests

**Knowledge of:**

- Tool use (`tool_use`) with JSON schemas is the most reliable way to get **schema-compliant structured output**, eliminating JSON **syntax** errors.
- `tool_choice` values: **`"auto"`** (model may return text instead of calling a tool), **`"any"`** (must call a tool, chooses which), **forced** `{"type": "tool", "name": "..."}` (must call that specific tool).
- Strict JSON schemas via tool use eliminate **syntax** errors but **not semantic** errors (line items that do not sum to the total, values in the wrong field).
- Schema design: required vs optional fields; enum fields with `"other"` plus a detail string for extensible categories.

**Skills in:**

- Defining **extraction tools** with JSON schemas as input parameters and reading the data from the `tool_use` response.
- Using `tool_choice: "any"` to guarantee structured output when **multiple extraction schemas** exist and the document type is unknown.
- Forcing a specific tool, `{"type": "tool", "name": "extract_metadata"}`, so a particular extraction runs **before** enrichment steps.
- Making fields **optional/nullable** when the source may not contain the information, so the model is not pushed to fabricate.
- Adding enum values like `"unclear"` for ambiguous cases and `"other"` + detail string for extensible categories.
- Including **format normalization rules in the prompt** alongside strict schemas, to handle inconsistent source formatting.

### Concepts explained

#### The extraction-by-tool trick

You do not need a "real" tool to get structured output. You define a tool whose **`input_schema` is the shape of data you want**, ask Claude to extract, and read the arguments Claude passes to the tool. The `tool_use` block's `input` is already a parsed JSON object, so there is no JSON-in-text to parse, no stray prose, no markdown fences. The tool is never executed; your code just reads `block.input`.

Anatomy of a tool definition (from the Define tools page): `name` (must match `^[a-zA-Z0-9_-]{1,128}$`), `description` (what it does, when to use it, what each parameter means; "by far the most important factor in tool performance"), `input_schema` (a JSON Schema object), and optionally `input_examples`, `strict` and others.

The response contains content blocks. The `tool_use` block looks like:

```json
{
  "type": "tool_use",
  "id": "toolu_01A09q90qw90lq917835lq9",
  "name": "extract_invoice",
  "input": { "vendor_name": "Acme Ltd", "total": 1180.0 }
}
```

and `stop_reason` is `"tool_use"` when the model has called a tool.

#### What "guaranteed" means - and what it does not

Two levels exist, and the exam's wording ("eliminating JSON syntax errors") sits at the first:

1. **Tool use at all.** Output arrives as a structured `input` object rather than free text you must parse. This removes the "model wrapped JSON in prose / broke a comma" class of failure.
2. **`strict: true` on the tool** (current docs: *strict tool use*). The docs state that grammar-constrained sampling guarantees the tool `input` strictly follows the `input_schema` and the tool `name` is valid. Without strict mode, the docs warn the model "might return incompatible types (`"2"` instead of `2`) or omit required fields". There is a separate feature, **JSON outputs** (`output_config.format` with `type: "json_schema"`), that constrains a plain response (not a tool call) to a schema; SDK helper: `client.messages.parse(..., output_format=PydanticModel)` returns `parsed_output`.

Neither level guarantees **meaning**:

| Guaranteed by schema / strict mode (syntax) | NOT guaranteed (semantic) |
|---|---|
| Valid JSON, correct field types, required fields present | Line items that add up to the stated total |
| Value is one of the enum members | The right enum member for this document |
| No extra properties (`additionalProperties: false`) | A value placed in the *right* field (e.g. ship-to address in the bill-to field) |
| | A true value instead of a plausible fabrication when the source is silent |

Closing the semantic gap is the job of prompt design (this section) and validation/retry (4.4).

Practical limits of strict/JSON schema mode (current docs): no recursive schemas; `minimum`/`maximum`/`minLength`/`maxLength` and similar constraints are not supported by the grammar (SDK helpers strip them and, if you use the validating helper such as Pydantic parsing, re-check them locally); `additionalProperties` must be `false`; at most 20 strict tools per request, 24 optional parameters and 16 union-typed parameters (`anyOf` or type arrays such as `["string", "null"]`) counted across all strict tool schemas and JSON-output schemas in the request. Unsupported features return a 400 error. Making parameters required where possible keeps the grammar smaller. Enum string capitalization is not guaranteed (compare case-insensitively). A `refusal` or `max_tokens` stop can yield output that does not match the schema, so check `stop_reason`: a refusal comes back with HTTP 200 and is billed, and a `max_tokens` cut-off is fixed by retrying with a higher limit. Don't add a schema property that asks for the model's step-by-step reasoning; the docs say that can lead to a `reasoning_extraction` refusal, so ask for a short explanation or supporting evidence instead. Structured outputs and strict tool use are available on all current models, per the docs' availability section.

#### `tool_choice` - the three options the exam names (plus one)

| `tool_choice` | Behavior | Use it when |
|---|---|---|
| `{"type": "auto"}` (default when tools are given) | Claude decides: it **may answer in plain text** and call no tool | You want a normal assistant that sometimes uses tools |
| `{"type": "any"}` | Claude **must** call one of the provided tools, but chooses **which** | Several extraction schemas exist (invoice, receipt, contract) and the document type is unknown - you still need structured output |
| `{"type": "tool", "name": "extract_metadata"}` | Claude **must** call that named tool | A specific step has to run first (extract metadata, then enrich) or there is exactly one schema |
| `{"type": "none"}` | Claude cannot use tools (current docs; default when no tools are given) | Tools present in the request but you want text only |

Important behaviors from the Define tools page:

- With `any` or `tool`, the API **prefills** the assistant turn, so the model emits **no natural-language text before the `tool_use` block**, even if asked to explain. If you want commentary, use `auto` with an explicit instruction in the user message.
- Forcing a tool also **skips thinking** on models that accept forced tool use.
- (From the API reference rather than the Define tools page) each `tool_choice` object accepts an optional `disable_parallel_tool_use`; with `any` or `tool`, setting it to `true` means exactly one tool call.

> **Current-docs note (differs from the exam guide's simple three-way picture).** The Define tools page lists restrictions: with **manual extended thinking** (`thinking: {type: "enabled"}`), `any` and `tool` are rejected (use `auto`/`none`); and on **Claude Opus 5.5, Sonnet 5.5, Fable 5.1 and Mythos 5.1**, `any` and `tool` return a **400 error**. On those models the docs recommend `auto` + `strict: true` (guaranteed schema-valid inputs) or structured outputs (`output_config.format`) when you need a fixed JSON shape. The exam's wording is the three-way choice; answer exam questions with it, and know that in production you must check the model's support.

#### Schema design: required, optional, nullable, enums

**Required vs optional.** A `required` field must be present in every output. If the source document may not contain a value, making the field required *forces* the model to put something there - which means fabrication. Make it **optional or nullable** so "not stated" is a legitimate answer.

Two ways to express it:

```json
"po_number": { "type": ["string", "null"] }
```

(`required` includes `po_number`, but `null` is allowed: the key is always present, the value may be null), or leave `po_number` out of `required` (the model may omit it). Under `strict: true` both options have a cost in the current docs: each parameter left out of `required` counts toward the limit of 24 optional parameters and, per the docs, roughly doubles part of the grammar; each type-array/`anyOf` parameter such as `["string", "null"]` counts toward the separate limit of 16 union-typed parameters, which the docs call "especially expensive". So keep both counts small (the `EXTRACT_INVOICE` example below uses 7 nullable fields). For the exam, both are "optional (nullable) fields"; what matters is that **absence is representable**.

**Enums with an escape hatch.** Closed enums force a wrong label whenever a case is not on the list. Two patterns:

- `"unclear"` (or `"unknown"`) for genuinely ambiguous cases: the model reports its uncertainty instead of guessing.
- `"other"` **plus a free-text detail field** (e.g. `category_other_detail`) for extensible categories: a new payment method, a new document type, shows up as `other` + detail and you can later promote it to a real enum value. Use `"other"` when the model *understood* the item but your list lacks the label; use `"unclear"` when the model *could not tell*.

**Format normalization in the prompt.** A schema says "date is a string"; it does not say "ISO 8601". Source documents write `3/4/26`, `4 March 2026`, `2026.03.04`, `$1,180.00`, `1.180,00 EUR`. Put normalization rules **in the prompt next to the schema**: "Dates as `YYYY-MM-DD`; amounts as numbers without currency symbols or thousands separators; currency as ISO 4217 code; leave `null` if the document is ambiguous (for example `03/04/26` with no locale clue)." The schema constrains type; the prompt specifies convention.

### Worked examples

#### Example 1 - a complete extraction tool (Scenario 6)

```python
import anthropic

client = anthropic.Anthropic()

EXTRACT_INVOICE = {
    "name": "extract_invoice",
    "description": (
        "Record the structured data found in ONE invoice document. Call this once per "
        "document. Only report values that appear in the document; use null when a field "
        "is not stated. Do not infer or compute missing values, except calculated_total, "
        "which must be the sum of the line items you extracted."
    ),
    "strict": True,          # schema-valid tool input (see strict tool use docs)
    "input_schema": {
        "type": "object",
        "properties": {
            # --- always present in an invoice: required, non-null ---
            "vendor_name": {"type": "string", "description": "Seller name exactly as printed."},
            "line_items": {
                "type": "array",
                "items": {
                    "type": "object",
                    "properties": {
                        "description": {"type": "string"},
                        "amount": {"type": "number", "description": "Line amount, no currency symbol."}
                    },
                    "required": ["description", "amount"],
                    "additionalProperties": False
                }
            },

            # --- may be absent in the source: required key, nullable value ---
            "invoice_number": {"type": ["string", "null"]},
            "invoice_date": {
                "type": ["string", "null"],
                "description": "ISO 8601 YYYY-MM-DD, or null if absent/ambiguous."
            },
            "po_number": {"type": ["string", "null"]},

            # --- enum with "unclear": the model may admit it cannot tell ---
            "document_type": {
                "type": "string",
                "enum": ["invoice", "credit_note", "pro_forma", "unclear"]
            },

            # --- enum with "other" + detail for extensible categories ---
            "payment_method": {
                "type": "string",
                "enum": ["card", "bank_transfer", "cheque", "cash", "other", "unclear"]
            },
            "payment_method_other_detail": {
                "type": ["string", "null"],
                "description": "Fill only when payment_method is 'other'; else null."
            },

            # --- self-checking fields (see 4.4) ---
            "stated_total": {
                "type": ["number", "null"],
                "description": "The total printed on the document, or null if none."
            },
            "calculated_total": {
                "type": ["number", "null"],
                "description": "Sum of line_items[].amount as extracted."
            },
            "conflict_detected": {
                "type": "boolean",
                "description": "True if the document contradicts itself (e.g. two different totals)."
            },
            "conflict_notes": {"type": ["string", "null"]}
        },
        "required": [
            "vendor_name", "line_items", "invoice_number", "invoice_date", "po_number",
            "document_type", "payment_method", "payment_method_other_detail",
            "stated_total", "calculated_total", "conflict_detected", "conflict_notes"
        ],
        "additionalProperties": False
    }
}

NORMALIZATION_RULES = """Normalization rules:
- Dates: output YYYY-MM-DD. If day/month order is ambiguous (e.g. 03/04/26) and the
  document gives no locale clue, output null rather than guessing.
- Amounts: plain numbers (1180.00), no currency symbols or thousands separators.
  "1.180,00" is 1180.00 (European format).
- Never invent a value. If the document does not state it, use null.
- document_type / payment_method: pick the closest enum value; use "other" (and fill the
  detail field) when you understand the item but no value fits; use "unclear" when you
  cannot tell."""

def extract_invoice(document_text: str):
    response = client.messages.create(
        model="claude-sonnet-4-6",                       # see the tool_choice model note above
        max_tokens=2048,
        tools=[EXTRACT_INVOICE],
        tool_choice={"type": "tool", "name": "extract_invoice"},   # forced
        messages=[{
            "role": "user",
            "content": f"<document>\n{document_text}\n</document>\n\n{NORMALIZATION_RULES}\n\n"
                       "Extract the invoice using the extract_invoice tool."
        }],
    )
    # Read the structured data from the tool_use block (never from text)
    for block in response.content:
        if block.type == "tool_use" and block.name == "extract_invoice":
            return block.input               # already a dict
    raise RuntimeError(f"No tool call returned (stop_reason={response.stop_reason})")
```

Things to notice: the nullable fields (`invoice_number`, `po_number`, ...) so absence is representable; `"unclear"` and `"other"` + `payment_method_other_detail`; `stated_total` / `calculated_total` / `conflict_detected` for self-validation; normalization rules in the prompt **and** types in the schema; document wrapped in `<document>` tags with the instructions after it (long-form data first, query last, per the long-context guidance).

#### Example 2 - `tool_choice: "any"` with several extraction schemas (unknown document type)

```python
TOOLS = [EXTRACT_INVOICE, EXTRACT_RECEIPT, EXTRACT_CONTRACT]   # three different schemas

resp = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=2048,
    tools=TOOLS,
    tool_choice={"type": "any"},        # must call one of them; Claude chooses which
    messages=[{"role": "user", "content": f"<document>\n{doc}\n</document>\n\nExtract this document."}],
)
block = next(b for b in resp.content if b.type == "tool_use")
handler = {"extract_invoice": handle_invoice,
           "extract_receipt": handle_receipt,
           "extract_contract": handle_contract}[block.name]
handler(block.input)
```

Why `any` and not `auto`: with `auto` the model might reply "This looks like an invoice from Acme..." in plain text and you would have nothing to parse. Why not forcing one tool: you do not know the document type in advance.

#### Example 3 - forced tool to guarantee ordering (extract first, enrich after)

```python
# Step 1: guarantee metadata extraction runs first
step1 = client.messages.create(
    model="claude-sonnet-4-6", max_tokens=1024,
    tools=[EXTRACT_METADATA, ENRICH_FROM_CRM, CLASSIFY_RISK],
    tool_choice={"type": "tool", "name": "extract_metadata"},   # forced
    messages=[{"role": "user", "content": f"<document>\n{doc}\n</document>\nExtract metadata."}],
)
metadata = next(b.input for b in step1.content if b.type == "tool_use")

# Step 2: enrichment, now with the metadata as context; Claude may pick among the other tools
step2 = client.messages.create(
    model="claude-sonnet-4-6", max_tokens=1024,
    tools=[ENRICH_FROM_CRM, CLASSIFY_RISK],
    tool_choice={"type": "any"},
    messages=[{"role": "user",
               "content": f"Metadata: {metadata}\nEnrich and classify using the available tools."}],
)
```

Forcing is deterministic: the first call *cannot* skip `extract_metadata`. This is the same "programmatic enforcement beats prompt instructions" principle as sample Q1 - here applied to ordering.

#### Example 4 - "required" field vs nullable: fabrication in action

```json
// BAD schema: po_number required, non-null
"po_number": {"type": "string"}  // in "required"

// Source invoice has NO purchase-order number.
// Model output (forced to satisfy the schema):  "po_number": "PO-0000"   <-- fabricated
```

```json
// GOOD schema: absence is representable
"po_number": {"type": ["string", "null"]}

// Model output: "po_number": null    <-- honest
```

#### Example 5 - enum design

```json
// Closed enum: a wire transfer labelled "SEPA Instant" gets forced into the wrong bucket
"payment_method": {"enum": ["card", "bank_transfer", "cheque", "cash"]}

// Extensible enum
"payment_method": {"enum": ["card", "bank_transfer", "cheque", "cash", "other", "unclear"]},
"payment_method_other_detail": {"type": ["string", "null"]}
// "SEPA Instant" -> payment_method: "bank_transfer"  (if you decide it belongs)
// "PayPal Credit" -> payment_method: "other", payment_method_other_detail: "PayPal Credit"
// Garbled scan -> payment_method: "unclear"
```

Review the `other_detail` values periodically; frequent ones become new enum members.

#### Example 6 - the semantic error that strict mode cannot catch

```json
{
  "line_items": [{"description": "Widget", "amount": 400}, {"description": "Gadget", "amount": 600}],
  "stated_total": 1180.0,
  "calculated_total": 1000.0,
  "conflict_detected": false
}
```

This output is *perfectly schema-valid* - correct types, all required fields. It is also wrong or suspicious: the items sum to 1000 but the document states 1180. Syntax is fine; semantics are not. Detecting this is validation work (4.4).

#### Example 7 - structured output in CI and the Agent SDK (Scenario 5/6)

CLI (headless): `--output-format json` returns session metadata plus the text result; add `--json-schema` for schema-conforming output in a `structured_output` field.

```bash
claude -p "Extract the main function names from auth.py" \
  --output-format json \
  --json-schema '{"type":"object","properties":{"functions":{"type":"array","items":{"type":"string"}}},"required":["functions"]}' \
  | jq '.structured_output'
```

If the run ends without valid structured output (for example the model declined), the `structured_output` field is absent - **check that it is present** before using it. An invalid schema makes `claude` exit with `Error: --json-schema is not a valid JSON Schema`. The `format` keyword is accepted as an annotation but not enforced.

Agent SDK (Python): pass `output_format={"type": "json_schema", "schema": ...}` in `ClaudeAgentOptions`; the final `ResultMessage` carries `structured_output`. Under the hood the SDK adds a `StructuredOutput` tool, validates, and re-prompts on mismatch; if no valid output results, the result subtype is `error_max_structured_output_retries`. Treat a result as successful only when `subtype == "success"` **and** `structured_output` is present. The SDK page's own tip: if the task might not have all the information the schema needs, make those fields optional. (The page also notes that a single-shot `query()` raises after yielding an error result, so wrap the loop in `try/except` as the page's examples do.)

```python
from pydantic import BaseModel
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

class Finding(BaseModel):
    location: str
    issue: str
    severity: str
    suggested_fix: str

class Review(BaseModel):
    findings: list[Finding]

async def review(prompt):
    async for msg in query(prompt=prompt, options=ClaudeAgentOptions(
            output_format={"type": "json_schema", "schema": Review.model_json_schema()})):
        if isinstance(msg, ResultMessage):
            if msg.subtype == "success" and msg.structured_output:
                return Review.model_validate(msg.structured_output)
            raise RuntimeError(f"no structured output: {msg.subtype}")
```

### Exam traps

- **"Ask for JSON in the prompt and parse it"** - least reliable; tool use with a schema is the reliable route. The old workaround of prefilling the assistant turn (for example with `{`) is also gone on Claude 4.6 and later models and Claude Mythos Preview, which return a 400 error; the docs point to structured outputs, or a tool with an enum field for classification.
- **"Strict schema means the extraction is correct"** - it guarantees syntax/types, not semantics (sums, field placement).
- **"Mark everything required so we always get data"** - causes fabrication when the source is silent. Use nullable/optional.
- **`auto` when you need guaranteed structure** - `auto` lets the model answer in text. Use `any` (unknown type) or a forced tool (known tool/ordering).
- **`any` vs forced** - `any` = *some* tool, model picks; forced = *this* tool. Do not swap them.
- **Closed enum** with no `other`/`unclear` - forces mislabels.
- **Normalization via schema alone** - schemas cannot express conventions like "ISO dates"; put rules in the prompt.
- **Expecting explanatory text before a forced tool call** - not emitted with `any`/`tool`.

### Check yourself

1. Your pipeline ingests invoices, receipts and contracts, each with its own extraction tool, and you do not know the type beforehand. Which `tool_choice` ensures you always get structured output?
   <details><summary>Answer</summary><code>{"type": "any"}</code> - the model must call one of the tools and picks the matching one. <code>auto</code> could return text; a forced tool would be wrong for two of the three document types. (On the few models that reject forced tool use, see the current-docs note above: <code>auto</code> plus strict tools or structured outputs.)</details>
2. A downstream system rejects extractions where `purchase_order` is `"PO-0000"` for invoices that have no PO at all. What schema change addresses the root cause?
   <details><summary>Answer</summary>The field was required and non-nullable, forcing fabrication. Make it nullable/optional so null (not stated) is valid, and instruct the model never to invent values.</details>
3. A schema-valid extraction has line items summing to 1,000 while `stated_total` is 1,180. Why did strict tool use not prevent this, and what does?
   <details><summary>Answer</summary>Strict schemas guarantee syntax and types, not semantic consistency. Add `calculated_total`, `stated_total` and `conflict_detected` fields and validate in code (4.4), with retry or human review as appropriate.</details>

### Official docs

- [Define tools](https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools) - `input_schema`, writing descriptions, `tool_choice` (`auto`/`any`/`tool`/`none`), model restrictions on forced tool use.
- [Strict tool use](https://platform.claude.com/docs/en/agents-and-tools/tool-use/strict-tool-use) - what `strict: true` guarantees.
- [Structured outputs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs) - `output_config.format`, supported schema features, complexity limits, refusal and `max_tokens` caveats.
- [Agent SDK structured outputs](https://code.claude.com/docs/en/agent-sdk/structured-outputs.md) - `output_format`, `structured_output`, `error_max_structured_output_retries`.
- [Run Claude Code programmatically](https://code.claude.com/docs/en/headless.md) - `--output-format json`, `--json-schema`.
- [Increase output consistency](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/increase-consistency) - the note that guaranteed JSON conformance calls for structured outputs rather than prompt techniques; prefill is no longer supported on Claude 4.6 and later models or Claude Mythos Preview (use structured outputs or system-prompt instructions instead).

---

## 4.4 Implement validation, retry, and feedback loops for extraction quality

### What the exam tests

**Knowledge of:**

- **Retry-with-error-feedback**: put the specific validation errors in the follow-up prompt so the model can correct itself.
- **Limits of retry**: retrying cannot create information that is absent from the source; it only helps with **format or structural** errors.
- **Feedback-loop design**: record which code constructs trigger findings (`detected_pattern`) so you can analyze **dismissal patterns** systematically.
- **Semantic validation errors** (values do not sum, wrong field) differ from **schema syntax errors**, which tool use eliminates.

**Skills in:**

- Follow-up requests containing the **original document, the failed extraction, and the specific validation errors**.
- Recognizing when retries will be **ineffective** (information exists only in an external document that was not provided) versus **effective** (format mismatches, structural output errors).
- Adding `detected_pattern` to structured findings to analyze false-positive patterns when developers dismiss findings.
- **Self-correction validation flows**: extract `calculated_total` alongside `stated_total` to flag discrepancies; add `conflict_detected` booleans for inconsistent source data.

### Concepts explained

#### The three layers of validation

| Layer | Catches | Done by |
|---|---|---|
| 1. Syntax / schema | invalid JSON, wrong types, missing required keys | Tool use + `strict: true` (eliminated at generation time) |
| 2. Semantic / business rules | totals that do not sum, dates out of order, value in the wrong field, an enum value that is valid but implausible | **Your code** (Pydantic validators, cross-field checks) |
| 3. Truth against the source | a plausible-but-fabricated value | Prompt design (nullable fields, quote grounding), human review sampling |

Layer 2 is what this task statement is about. The model's own arithmetic is not trustworthy for this check either - so extract the numbers and **recompute in code**.

#### Retry with error feedback

When validation fails, do not simply call the same prompt again (a stochastic retry might succeed, or might fail identically). Send a **follow-up that contains**:

1. the **original document** (the model needs the source to re-read),
2. the **failed extraction** (so it sees what it produced), and
3. the **specific validation errors** ("line_items sum to 1000.00 but stated_total is 1180.00"; "invoice_date '31/02/2026' is not a valid date").

Specific errors give the model something to act on; "try again, it was wrong" does not. Cap the number of attempts (two or three), then route to a human or an exception queue.

(API detail: if you continue the same conversation after a `tool_use` block, the next user turn is expected to answer it with a `tool_result`; the easiest robust design is a fresh, stateless follow-up request that embeds document + failed extraction + errors in one user message, as below.)

#### When retry works and when it cannot

| Failure | Example | Retry helps? | Why / what to do |
|---|---|---|---|
| Format mismatch | date `March 4th, 2026` instead of `YYYY-MM-DD`; `$1,180` instead of `1180` | **Yes** | Information is in the source; the model only needs the convention restated |
| Structural error | value placed in the wrong field; array nested wrongly; enum label misspelled; item missing from a list that is in the document | **Yes** | The data exists; point out the error and the model re-reads |
| Arithmetic inconsistency from misread line | a digit misread so items do not sum | **Usually yes** | Re-reading often fixes a misread |
| Source itself inconsistent | document prints two different totals | **No, but not a failure** | `conflict_detected: true` is the *correct* output; route to review |
| Information absent | PO number is in a separate contract that was not supplied; "see attached schedule" | **No** | Nothing to re-read. Correct output is `null`; retrying invites fabrication. Fetch the external document, or flag for human follow-up |
| Ambiguous source | `03/04/26` with no locale clue | **Rarely** | Return `null` / `"unclear"`; escalate |
| Model refusal / `max_tokens` cut-off | `stop_reason: "refusal"` or `"max_tokens"` | Depends | `max_tokens`: retry with a higher limit. Refusal: change the request, not just retry |

The exam trap here: a pipeline that retries indefinitely on "missing PO number" wastes cost and, worse, trains the model to invent one. Distinguish **"the model made a mistake"** from **"the document does not contain it"**.

#### Self-correction fields: `calculated_total`, `stated_total`, `conflict_detected`

Extract both values and let code decide:

- `stated_total`: what the document says the total is.
- `calculated_total`: sum of the extracted line items (and **recompute it yourself**; do not trust the model's addition).
- `conflict_detected`: the model's explicit flag that the *source* contradicts itself.

Decision matrix:

| Stated vs recomputed | `conflict_detected` | Meaning | Action |
|---|---|---|---|
| match | false | healthy | accept |
| mismatch | false | extraction error (misread item, item missing) | **retry with error feedback** |
| mismatch | true | source is genuinely inconsistent | accept as-is, **route to human review** (retry is pointless) |
| `stated_total` is null | n/a | no stated total in source | accept; skip the check |

#### `detected_pattern`: a feedback loop for findings

For review findings (Scenario 5), add a field that records **which code construct triggered the finding**, e.g. `"bare_except_pass"`, `"fstring_in_sql"`, `"missing_await"`, `"comment_claims_return_none"`. Log developer reactions (accepted / dismissed with reason). Then group by `detected_pattern`:

| detected_pattern | reported | dismissed | dismissal rate | action |
|---|---|---|---|---|
| `fstring_in_sql` | 40 | 2 | 5% | keep |
| `bare_except_pass` | 60 | 33 | 55% | add acceptable-vs-genuine few-shot examples |
| `comment_claims_return_none` | 25 | 19 | 76% | disable category temporarily (4.1), refine criteria |

Without the field you only know "developers dismiss a lot of findings"; with it you know **which patterns** to fix. It closes the loop between 4.1 (criteria), 4.2 (examples) and 4.4 (measurement).

### Worked examples

#### Example 1 - Pydantic validation + retry-with-feedback loop

```python
import json
from pydantic import BaseModel, ValidationError, model_validator

class LineItem(BaseModel):
    description: str
    amount: float

class Invoice(BaseModel):
    vendor_name: str
    line_items: list[LineItem]
    invoice_date: str | None
    stated_total: float | None
    calculated_total: float | None
    conflict_detected: bool
    conflict_notes: str | None = None

    @model_validator(mode="after")
    def totals_are_consistent(self):
        recomputed = round(sum(i.amount for i in self.line_items), 2)   # do NOT trust the model's sum
        if self.stated_total is not None and abs(recomputed - self.stated_total) > 0.01:
            if not self.conflict_detected:
                raise ValueError(
                    f"line_items sum to {recomputed:.2f} but stated_total is "
                    f"{self.stated_total:.2f}, and conflict_detected is false. Re-read the line "
                    f"items and the total; if the document itself disagrees, set conflict_detected=true."
                )
        return self

def call_model(prompt: str) -> dict:
    resp = client.messages.create(
        model="claude-sonnet-4-6", max_tokens=2048,
        tools=[EXTRACT_INVOICE],
        tool_choice={"type": "tool", "name": "extract_invoice"},
        messages=[{"role": "user", "content": prompt}],
    )
    return next(b.input for b in resp.content if b.type == "tool_use")

def extract_with_retry(document: str, max_retries: int = 2) -> tuple[Invoice | None, str]:
    base = f"<document>\n{document}\n</document>\n\n{NORMALIZATION_RULES}\n\nExtract the invoice."
    prompt, extraction = base, None
    for attempt in range(max_retries + 1):
        extraction = call_model(prompt)
        try:
            invoice = Invoice.model_validate(extraction)
        except ValidationError as e:
            errors = "\n".join(f"- {'.'.join(map(str, err['loc']))}: {err['msg']}" for err in e.errors())
        else:
            if invoice.conflict_detected:
                return invoice, "needs_human_review"      # source is inconsistent: do not retry
            return invoice, "ok"
        # Follow-up = original document + failed extraction + SPECIFIC errors
        prompt = (
            f"{base}\n\n"
            f"Your previous extraction:\n{json.dumps(extraction, indent=2)}\n\n"
            f"It failed validation with these errors:\n{errors}\n\n"
            "Correct the extraction using the document. Only change what the errors point to. "
            "If the document does not contain a required value, return null for it rather than guessing."
        )
    return None, "failed_after_retries"                    # send to exception queue
```

Key design points: the validator recomputes the sum itself; the error message is specific and actionable; the retry prompt re-includes the document; the last sentence tells the model that null is acceptable (to avoid "retry pressure" fabrication); `conflict_detected=True` is treated as a *successful* extraction that needs a human; retries are capped.

#### Example 2 - routing by error class (do not retry what retry cannot fix)

```python
RETRYABLE = {"date_parsing", "value_error", "enum", "string_type", "float_parsing", "missing"}
# 'missing' here means a structural omission of a key that the document DOES contain.

def should_retry(err: dict, document: str) -> bool:
    if err["type"] in RETRYABLE:
        field = err["loc"][0]
        # Information-absence check: if the field is legitimately null in the source, don't retry.
        return field not in KNOWN_EXTERNAL_FIELDS     # e.g. {"po_number"} lives in a contract not supplied
    return False
```

The point is the classification, not the exact code: format/structure -> retry; absence/external/inconsistent source -> do not.

#### Example 3 - findings schema with `detected_pattern` (Scenario 5)

```python
REPORT_FINDINGS = {
    "name": "report_findings",
    "description": "Report review findings for a diff. Report only categories listed in the instructions.",
    "strict": True,
    "input_schema": {
        "type": "object",
        "properties": {
            "findings": {
                "type": "array",
                "items": {
                    "type": "object",
                    "properties": {
                        "location": {"type": "string", "description": "file:line"},
                        "category": {"type": "string", "enum": ["bug", "security", "comment_mismatch", "other"]},
                        "category_other_detail": {"type": ["string", "null"]},
                        "severity": {"type": "string", "enum": ["critical", "high", "medium", "low"]},
                        "issue": {"type": "string"},
                        "suggested_fix": {"type": "string"},
                        "detected_pattern": {
                            "type": "string",
                            "description": "Short snake_case name of the code construct that triggered this finding, e.g. bare_except_pass, fstring_in_sql."
                        }
                    },
                    "required": ["location", "category", "category_other_detail", "severity",
                                 "issue", "suggested_fix", "detected_pattern"],
                    "additionalProperties": False
                }
            }
        },
        "required": ["findings"],
        "additionalProperties": False
    }
}
```

Dismissal analysis:

```python
from collections import Counter, defaultdict

stats = defaultdict(Counter)
for event in review_events:                 # one row per finding shown to a developer
    stats[event["detected_pattern"]]["shown"] += 1
    if event["developer_action"] == "dismissed":
        stats[event["detected_pattern"]]["dismissed"] += 1

for pattern, c in sorted(stats.items(), key=lambda kv: -kv[1]["dismissed"] / kv[1]["shown"]):
    rate = c["dismissed"] / c["shown"]
    print(f"{pattern:32s} shown={c['shown']:4d} dismissed_rate={rate:.0%}")
```

Patterns with a high dismissal rate are the candidates for new "acceptable pattern" few-shot examples, tighter criteria, or temporary disabling (4.1).

#### Example 4 - end-to-end retry trace

```text
Attempt 0 extraction : line_items 400 + 600, stated_total 1180, calculated_total 1000, conflict_detected=false
Validator            : "line_items sum to 1000.00 but stated_total is 1180.00, conflict_detected is false"
Follow-up prompt     : <document> + failed JSON + that error message
Attempt 1 extraction : line_items 400 + 600 + 180 ("Shipping" was missed), calculated_total 1180
Validator            : ok  -> accepted

Different document:
Attempt 0            : po_number null; downstream rule "po_number required for approval" fails
Decision             : do NOT retry. PO is referenced as "see Master Services Agreement", not provided.
                       Route to human follow-up / fetch the MSA.
```

### Exam traps

- **"Retry up to N times whenever any required field is missing"** - fails when the source lacks the information; it produces fabrication.
- **"Retry with the same prompt"** - without the errors there is nothing to correct; send document + failed extraction + specific errors.
- **"Strict mode makes validation unnecessary"** - semantic errors remain.
- **"Trust the model's `calculated_total`"** - recompute in code; use the model's field to compare, not to certify.
- **"`conflict_detected: true` means the extraction failed"** - it can mean the extraction faithfully reports a contradictory source; send to review.
- **"Log only accept/dismiss"** - without `detected_pattern` you cannot find which constructs cause false positives.

### Check yourself

1. An extraction fails validation because a date was returned as `4 March 2026`. Retry or not?
   <details><summary>Answer</summary>Retry, with the specific error and the normalization rule (YYYY-MM-DD) - it is a format error and the information is in the source.</details>
2. A contract extraction repeatedly returns `null` for `governing_law` because the clause is in an annex that was not supplied. What should the system do?
   <details><summary>Answer</summary>Do not keep retrying; the information is absent from the supplied document. Accept null, flag as incomplete, and obtain the annex or send to human follow-up.</details>
3. What do you add to review findings so you can tell *which* patterns generate the most dismissed findings?
   <details><summary>Answer</summary>A <code>detected_pattern</code> field naming the code construct that triggered the finding, then aggregate dismissals by that field.</details>

### Official docs

- [Increase output consistency](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/increase-consistency) - "Chain prompts for complex tasks", constraining output with examples.
- [Reduce hallucinations](https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations) - allow uncertainty, ground in quotes, verify claims; "don't eliminate them entirely".
- [Structured outputs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs) - what the schema constrains; `refusal`/`max_tokens` caveats.
- [Agent SDK structured outputs - Error handling](https://code.claude.com/docs/en/agent-sdk/structured-outputs.md) - the SDK's built-in validate-and-re-prompt behavior and its failure subtype.
- [Prompting best practices - Chain complex prompts](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) - generate, review against criteria, refine as separate calls so each can be logged or branched.

---

## 4.5 Design efficient batch processing strategies

### What the exam tests

**Knowledge of:**

- **Message Batches API**: 50% cost savings, processing window **up to 24 hours**, **no guaranteed latency SLA**.
- Appropriate for **non-blocking, latency-tolerant** workloads (overnight reports, weekly audits, nightly test generation); **inappropriate for blocking** workflows (pre-merge checks).
- The batch API **does not support multi-turn tool calling within a single request** (it cannot execute your tools mid-request and return the results to the model).
- **`custom_id`** correlates each request with its response.

**Skills in:**

- Matching API choice to latency need: **synchronous** for blocking pre-merge checks, **batch** for overnight/weekly analysis.
- Calculating **submission frequency from an SLA** (e.g. 4-hour windows to guarantee a 30-hour SLA with 24-hour batch processing).
- **Handling failures**: resubmit only the failed documents, identified by `custom_id`, with modifications (e.g. chunking documents that exceeded context limits).
- **Prompt refinement on a sample set** before batch-processing large volumes, to maximize first-pass success and reduce resubmission cost.

### Concepts explained

#### What the Message Batches API is

You submit many independent Messages requests in one call; they are processed asynchronously, each independently, and you collect results later. Facts from the current docs:

| Fact | Current docs |
|---|---|
| Discount | All usage at **50%** of standard prices (input, output, special tokens) |
| Processing time | Most batches finish **within 1 hour**, but batches may take **up to 24 hours**, and **expire** if not complete in 24 h. Results are available when all requests finish, or after 24 h, whichever is first |
| SLA | The docs word it as a maximum window, not a latency promise: batches "might take up to 24 hours" and "it is possible for a batch to expire". ("No guaranteed latency SLA" is the exam guide's phrasing.) Plan for the worst case, never for "often faster". Requests not processed within 24 h come back `expired`, so 24 h caps the wait for a result, not the chance that every request completes |
| Size limit | **100,000 requests or 256 MB** per batch, whichever comes first |
| Result retention | Results downloadable for **29 days** after batch creation |
| `custom_id` | Required per request; unique within the batch; 1-64 chars matching `^[a-zA-Z0-9_-]{1,64}$` |
| Result order | **Not guaranteed to match request order** - always match by `custom_id` |
| Failure isolation | One request failing does not affect the others |
| Modification | A submitted batch **cannot be modified**; cancel and resubmit (cancellation may not take effect immediately) |
| `processing_status` | `in_progress`, then `ended`; (`canceling` after a cancel request) |
| Result types | `succeeded`, `errored`, `canceled`, `expired` - `errored`/`canceled`/`expired` requests are **not billed** |
| Unsupported params | `stream: true`, `speed` (fast mode), `fallbacks`, `max_tokens: 0`; each request needs `max_tokens` of at least 1 |
| Supported | Almost everything else: vision, tool use (including all server tools such as web search), system prompts, multi-turn conversations (message arrays), extended thinking, prompt caching (best-effort cache hits), most betas |

(The exam only requires 50%, 24 h, no SLA, `custom_id`, and the no-mid-request-tool-loop limit; the table gives you the current exact numbers so nothing surprises you.)

#### Why tool loops do not fit

A normal agentic tool loop is: send request -> model returns `tool_use` -> **your code runs the tool** -> you send `tool_result` -> model continues. The exam guide's wording is that the batch API "does not support multi-turn tool calling within a single request (cannot execute tools mid-request and return results)". **Exam-vs-docs note:** the batch docs never state that limit in those words. They list tool use (including all server tools), system messages and multi-turn conversations as batchable. The limit follows from how batches work: each request is processed asynchronously and independently, so there is no channel through which your code could run a client-side tool and hand the `tool_result` back to the same in-flight request. By implication (the docs do not spell this out), a batch request that calls a client tool just returns its `tool_use` block as the result; continuing means submitting a *new* request (batch or synchronous) that contains the `tool_result`. You *can* include tool definitions: that is how extraction-by-tool works in a batch (a single forced tool call whose `input` is your data). What you cannot get is a *client-side* round trip inside one request. (Anthropic-run **server tools** such as web search do work in batches: the docs say the batch worker runs the same server-side agentic loop, and a result that ends with `stop_reason: "pause_turn"` can be continued in a follow-up request.) If your workflow needs iterative client-side tool execution, use the synchronous API, or run each loop step as a separate request.

#### Sync vs batch: the decision

| Question | Synchronous Messages API | Message Batches API |
|---|---|---|
| Is a person or a pipeline step **blocked waiting** for the result? | **Yes -> sync** | No |
| Latency requirement | seconds to minutes | hours acceptable |
| Cost sensitivity | normal price | 50% cheaper |
| Needs client-side tool loop mid-request | Yes | No |
| Typical examples | **pre-merge check**, interactive review, live support agent | **nightly tech-debt report**, weekly security audit, overnight test generation, bulk document backfill |

Sample Q11 is exactly this: manager proposes batching both a blocking pre-merge check and an overnight technical-debt report. Correct answer: batch the overnight report only. Wrong alternatives: switching both and polling ("often faster" is not a guarantee); keeping both real-time because of "ordering issues" (`custom_id` solves correlation); batch with a real-time fallback (extra complexity when you can simply match each API to its workload).

#### SLA arithmetic: how often to submit

A batch can take up to 24 h. If your business promises results within a fixed time of a document **arriving**, you also have to count how long a document waits **before** the next submission:

```text
worst-case turnaround = (submission interval) + (batch processing window) + (post-processing/handoff)
```

Example from the guide: SLA = **30 hours** from arrival, batch window = 24 h.

- Worst case: a document arrives just after a batch was submitted, so it waits one full interval for the next submission.
- Maximum interval = 30 h - 24 h = **6 h**, *before* any post-processing time.
- With a **4-hour submission window**: worst case = 4 h wait + 24 h processing = **28 h**, leaving **2 h** margin for result processing, retries and delivery. The exam guide gives only the 4-hour figure; this margin explanation is our reading of why it works.
- With a 6 h interval you would hit exactly 30 h with zero margin - one slow retrieval breaks the SLA. With 12 h windows the worst case is 36 h, which violates the SLA.

General rule: `interval <= SLA - 24 h - safety margin`. (If the SLA were 48 h, intervals up to about 24 h minus margin would work. If the SLA were under 24 h, a batch cannot be *guaranteed* to meet it at all - use sync for those, or a hybrid.) Batches "often" finish in under an hour, but the guarantee can only be built on the documented 24 h worst case.

#### Failure handling: resubmit only what failed, by `custom_id`

After the batch ends, stream the results and branch on each result type:

| Result | Meaning | Typical action |
|---|---|---|
| `succeeded` | Message produced | Validate it (4.4). A succeeded message can still fail *your* validation |
| `errored` with `invalid_request_error` | The request body is bad | **Fix the request** before resending (docs sample code: "Request body must be fixed before re-sending request"). Illustrative example (the docs do not say which error type an oversized document produces): a document too large for the context window -> **chunk it** |
| `errored` with other error types (server/overloaded etc.) | The docs' sample code treats these as retryable | Retry directly (docs: "Request can be retried directly"); the possible types also include authentication, billing, permission and not-found errors, which will not fix themselves, so check the type |
| `expired` | Batch hit 24 h before this request ran | Resubmit (consider a smaller batch) |
| `canceled` | You canceled | Resubmit if still wanted |

Chunking uses `custom_id` to preserve identity: e.g. `doc-0042` for the whole document, `doc-0042__c01`, `doc-0042__c02`... for chunks, then merge the chunk results back to `doc-0042`. Remember `custom_id` allows only letters, digits, `_` and `-`, up to 64 characters - file paths with `/` or `.` do not fit; use an ID and keep a mapping.

#### Refine the prompt on a sample *before* the big batch

Every failed-and-resubmitted round can cost up to another 24 h and wastes spend on the failing portion. So do the cheap, fast iteration first:

1. Take a **representative sample** (e.g. 50-200 documents covering each layout/type, including awkward ones).
2. Run it through the **synchronous** API (fast feedback), validate with the same checks as production.
3. Fix prompt/schema/few-shot problems; repeat until first-pass validation success is high.
4. Only then submit the full volume as a batch.

Illustration: 10,000 documents at 80% first-pass success -> 2,000 failures need a second batch (another up to 24 h, so ~48 h to finish). At 97% -> 300 failures. Refining on a 100-document sample costs minutes and shifts you from the first world to the second. The goal stated in the guide: "maximize first-pass success rates and reduce iterative resubmission costs".

### Worked examples

#### Example 1 - create a batch with `custom_id`s (Python SDK)

```python
import anthropic
from anthropic.types.message_create_params import MessageCreateParamsNonStreaming
from anthropic.types.messages.batch_create_params import Request

client = anthropic.Anthropic()

def to_request(doc_id: str, text: str) -> Request:
    return Request(
        custom_id=doc_id,                                     # ^[a-zA-Z0-9_-]{1,64}$
        params=MessageCreateParamsNonStreaming(
            model="claude-sonnet-4-6",
            max_tokens=2048,
            tools=[EXTRACT_INVOICE],
            tool_choice={"type": "tool", "name": "extract_invoice"},   # one-shot structured output
            messages=[{"role": "user",
                       "content": f"<document>\n{text}\n</document>\n\n{NORMALIZATION_RULES}\n\nExtract the invoice."}],
        ),
    )

docs = load_documents()                       # {doc_id: text}
batch = client.messages.batches.create(
    requests=[to_request(doc_id, text) for doc_id, text in docs.items()]   # up to 100,000 / 256 MB
)
print(batch.id, batch.processing_status)      # "in_progress"
```

#### Example 2 - poll until `ended`

```python
import time

while True:
    batch = client.messages.batches.retrieve(batch.id)
    if batch.processing_status == "ended":
        break
    time.sleep(60)                            # the docs' own example polls every 60 s
```

Polling is acceptable here because this is a **non-blocking** workload. Nothing is waiting on this loop to finish a pull request.

#### Example 3 - process results by `custom_id`; collect failures

```python
ok, to_fix, to_retry = {}, [], []

for entry in client.messages.batches.results(batch.id):     # streams .jsonl, memory efficient
    doc_id = entry.custom_id                                # NOT the position in the file
    outcome = entry.result
    if outcome.type == "succeeded":
        block = next((b for b in outcome.message.content if b.type == "tool_use"), None)
        invoice, status = validate(block.input) if block else (None, "no_tool_call")
        if status == "ok":
            ok[doc_id] = invoice
        else:
            to_retry.append((doc_id, status))               # semantic failure -> resubmit with feedback
    elif outcome.type == "errored":
        if outcome.error.error.type == "invalid_request_error":
            to_fix.append(doc_id)                           # fix the request first (e.g. chunk it)
        else:
            to_retry.append((doc_id, "transient"))
    else:                                                    # "expired" or "canceled"
        to_retry.append((doc_id, outcome.type))
```

#### Example 4 - resubmit only the failures, chunking oversized documents

```python
def chunk(text: str, max_chars: int = 60_000, overlap: int = 500):
    step = max_chars - overlap
    return [text[i:i + max_chars] for i in range(0, len(text), step)]

retry_requests = []
for doc_id in to_fix:                                       # exceeded context: chunk
    for n, piece in enumerate(chunk(docs[doc_id]), start=1):
        retry_requests.append(to_request(f"{doc_id}__c{n:02d}", piece))

for doc_id, reason in to_retry:                             # others: resubmit as-is (or with an
    retry_requests.append(to_request(doc_id, docs[doc_id])) # error-feedback prompt, see 4.4)

if retry_requests:
    retry_batch = client.messages.batches.create(requests=retry_requests)   # ONLY the failures
```

Only failed IDs are resubmitted; the 97% that succeeded are not reprocessed. Chunk results are merged back to their parent `doc_id` afterwards (split on the `__c` suffix).

#### Example 5 - SLA planner (the guide's example)

```python
BATCH_WINDOW_H = 24
SLA_H = 30
HANDOFF_MARGIN_H = 2              # validation, retries, delivery

max_interval = SLA_H - BATCH_WINDOW_H - HANDOFF_MARGIN_H     # = 4 hours
# Submit a batch every 4 hours (00:00, 04:00, 08:00, ...), containing everything that arrived
# since the previous submission. Worst case per document = 4 + 24 = 28 h, +2 h margin = 30 h.
```

#### Example 6 - workload classification (Scenario 5)

| Workload | API | Reason |
|---|---|---|
| PR pre-merge review gating the merge button | **Synchronous** | Developer is blocked; batch has no latency guarantee |
| Nightly generation of tests for changed modules | **Batch** | Results needed next morning; 50% cheaper |
| Weekly security audit across the monorepo | **Batch** | Latency-tolerant, large volume |
| Interactive refactoring assistant | **Synchronous** | Multi-turn tool loop, human in the loop |
| Backfill extraction of 80,000 archived invoices | **Batch**, after a sample-based prompt refinement | Large, non-urgent; avoid expensive resubmission rounds |

### Exam traps

- **"Switch everything to batch to save 50%"** - blocking workflows (pre-merge) cannot tolerate a 24 h worst case with no SLA.
- **"Batch usually finishes in under an hour, so it is fine for blocking checks"** - "often faster" is not a guarantee.
- **"Poll more frequently to reduce latency"** - polling does not speed processing.
- **"Rely on result order"** - not guaranteed; use `custom_id`.
- **"Resubmit the whole batch when some requests fail"** - resubmit only the failed `custom_id`s, after fixing the cause (chunking, etc.).
- **"Use batch for a workflow that needs your tools executed mid-request"** - the exam guide's limit: no multi-turn tool calling within a single request (client-side tools cannot be run mid-request and their results returned). Batches themselves do accept tool definitions and server tools.
- **SLA math errors** - forgetting the wait-for-next-submission term; using the batch window alone as the turnaround.
- **"Add a real-time fallback when a batch is late"** - works, but is the over-engineered answer when simple workload-to-API matching solves it.

### Check yourself

1. Your compliance team needs results within 36 hours of a document arriving. Batch processing takes up to 24 hours. What is the longest submission interval that still guarantees the SLA, before any safety margin, and what would you actually choose?
   <details><summary>Answer</summary>36 - 24 = 12 hours at most (zero margin). In practice choose a shorter interval such as 8 hours (or less) to leave margin for result handling and retries.</details>
2. 40 of 8,000 batch results come back `errored` with `invalid_request_error` because the documents exceeded the context window. What do you do?
   <details><summary>Answer</summary>Identify them by <code>custom_id</code>, chunk those documents (with chunk-suffixed custom IDs), and resubmit only them in a new batch; do not reprocess the other 7,960. Merge chunk results by parent ID.</details>
3. Why run a 100-document sample through the synchronous API before submitting 50,000 documents as a batch?
   <details><summary>Answer</summary>To refine the prompt/schema and raise first-pass success, since each resubmission round can add up to another 24 hours and extra cost; sync gives fast feedback.</details>

### Official docs

- [Batch processing](https://platform.claude.com/docs/en/build-with-claude/batch-processing) - limits, pricing statement, supported features, polling, results, cancellation, best practices, FAQ.
- [Create a Message Batch (API)](https://platform.claude.com/docs/en/api/messages/batches/create) - request shape, `custom_id` constraints, 1-100,000 requests, `params`.
- [Retrieve Message Batch results (API)](https://platform.claude.com/docs/en/api/messages/batches/results) - `.jsonl` stream; results not guaranteed in request order; result variants.

---

## 4.6 Design multi-instance and multi-pass review architectures

### What the exam tests

**Knowledge of:**

- **Self-review limitation**: a model keeps the reasoning context from generating the output, so it is less likely to question its own decisions in the same session.
- **Independent review instances** (no prior reasoning context) catch subtle issues better than self-review instructions or **extended thinking**.
- **Multi-pass review**: split large reviews into **per-file local passes** plus **cross-file integration passes**, to avoid attention dilution and contradictory findings.

**Skills in:**

- Using a **second independent Claude instance** to review generated code without the generator's reasoning context.
- Splitting multi-file reviews into focused per-file passes for local issues, plus a separate integration pass for cross-file data flow.
- Running **verification passes** where the model **self-reports confidence** with each finding to enable calibrated review routing.

### Concepts explained

#### Why a model should not grade its own work in the same session

When Claude writes code and then reviews it in the same conversation, the review is conditioned on everything that led to the code: its assumptions, its interpretation of ambiguous requirements, its chosen approach, and the reasoning that justified them. Questioning a decision means contradicting its own earlier reasoning that is *sitting in context*. The result is the same blind spot twice: a check that confirms rather than challenges. A prompt line like "now carefully review your work for mistakes" does not remove the anchoring; the reasoning is still present.

Extended thinking does not fix this either. Thinking (docs: Claude works through a problem in `thinking` blocks before answering - billed as output tokens) improves the quality of the *generation* or of a single answer. It does not give the model a **fresh perspective**; a reviewer who reads the same thinking inherits the same framing. The exam's position: an **independent instance** - a new session/API call with only the artifact (code, spec), none of the generator's reasoning - is more effective than self-review instructions or extended thinking.

**Exam-vs-docs note.** This comparison comes from the exam guide; the docs pages we read do not make it. The thinking page only describes what thinking does (thinking blocks before the answer, billed as output tokens), and nothing there compares thinking with independent review. The prompting best-practices page even recommends asking Claude to self-check ("verify your answer against test criteria") because it catches errors in coding and math. That is a cheap in-answer check, not an architecture for reviewing generated code, so it does not change the exam answer: when a question asks how to catch subtle issues in generated code, the independent instance wins.

| Technique | Fresh perspective? | Notes |
|---|---|---|
| "Review your own work" in the same conversation | No | Reasoning context retained |
| Extended thinking before the answer | No (exam position) | Better reasoning, same viewpoint |
| **Second independent Claude call/session given only the code (and requirements)** | **Yes** | The recommended design |
| Same prompt run three times, flag only issues seen in 2 of 3 | Partial | Voting *suppresses* real bugs that are only caught intermittently (sample Q12 option D) |

In Claude Code terms, each separate `claude -p` invocation without `--continue`/`--resume` starts with fresh context, which is exactly what you want for a reviewer stage following a generator stage in CI. (The docs also note that in a terminal session `/code-review` runs the review as a background subagent in its own context window; whichever mechanism you use, the test is "does the reviewer see the generator's reasoning or not?".)

The Anthropic engineering article on workflow patterns names the related **evaluator-optimizer** pattern: one call generates, another evaluates and gives feedback in a loop, which fits when there are clear evaluation criteria and iteration adds measurable value. The separation of generator and evaluator is the point. (The article does not say the evaluator must have a fresh context; "independent instance, no generator reasoning" is the exam guide's requirement.)

#### Attention dilution: why a 14-file single pass goes wrong

Sample Q12 describes the symptoms: a single pass over 14 files gave detailed feedback for some files and superficial feedback for others, missed obvious bugs, and contradicted itself (a pattern flagged in one file and approved identically in another). The root cause named by the guide is **attention dilution** - when too much is analyzed at once, depth and consistency per file fall. A bigger context window does not solve this (the guide: larger windows do not fix attention quality).

The fix is **multi-pass review**:

1. **Per-file local passes**: each file gets its own focused pass for local issues (bugs, security, comment mismatches within the file). Same criteria and same few-shot examples for every file - consistent depth, and the same pattern gets the same verdict.
2. **Cross-file integration pass**: a separate pass for issues that exist only *between* files: a function signature changed in one file but not its callers, data flowing from module A to B with mismatched units/types, an invariant maintained in one file and violated in another. It receives the **summaries/findings and interfaces** from pass 1 (and the relevant diffs), not necessarily every file's full text.

Rejected alternatives from Q12 and why:

| Option | Why not |
|---|---|
| Developers split PRs into 3-4 files | Shifts the burden to developers, does not improve the system |
| Bigger model / larger context | Attention quality is the problem, not capacity |
| Run 3 independent full passes, keep issues found in 2 of 3 | Consensus filtering *drops* real bugs found intermittently |

#### Confidence self-reports for calibrated routing

A **verification pass** asks the model to review findings (ideally an independent instance reviewing another's findings) and attach a **confidence** to each. You use those values for **routing**: high-confidence items flow straight to the developer, low-confidence items go to a human reviewer or a deeper second look. Used for *prioritizing attention* rather than as the sole gate, this reduces the human review load while keeping the uncertain items visible.

Be precise about the two claims in the guide, because they look contradictory:

- **Sample Q3** says a *confidence-threshold auto-escalation* is a poor fix for an agent's escalation calibration: self-reported confidence is poorly calibrated - the agent is already confidently wrong on the hard cases - so use explicit criteria with few-shot examples instead.
- **Task 4.6** says self-reported confidence can enable *calibrated review routing* in a verification pass.

Reconcile: raw self-reported scores are not trustworthy as absolute probabilities; they become useful **after calibration** against labeled data (Domain 5 covers confidence calibration with labeled validation sets and stratified sampling). Use them to *rank and route* review effort, keep a human-review sample of the "confident" bucket to measure error rate, and never treat the number as ground truth. Never use it *instead of* explicit criteria.

### Worked examples

#### Example 1 - generator and independent reviewer (CI)

```bash
# Stage 1: generator session
claude --bare -p "Implement retry logic in payments/client.py per SPEC.md" \
  --allowedTools "Read,Edit" --output-format json > gen.json

# Stage 2: independent reviewer - a NEW session. It gets the requirements and the resulting
# diff, NOT the generator's transcript or reasoning.
git diff main -- payments/ | claude --bare -p \
  --append-system-prompt-file ci/review_criteria.md \
  "Review this diff against SPEC.md. Report findings only." \
  --output-format json
```

The separation is by *context*, not by model: the reviewer could even be the same model, so long as it has not seen the generator's reasoning.

#### Example 2 - per-file passes plus an integration pass (Python skeleton)

```python
import json
from concurrent.futures import ThreadPoolExecutor

def review_file(path: str, diff: str, criteria: str) -> dict:
    """Local pass: one file, same criteria and few-shot examples every time."""
    resp = client.messages.create(
        model="claude-sonnet-4-6", max_tokens=2048,
        system=criteria,                                   # explicit criteria + examples (4.1, 4.2)
        tools=[REPORT_FINDINGS],
        tool_choice={"type": "tool", "name": "report_findings"},
        messages=[{"role": "user", "content": f"<file path=\"{path}\">\n{diff}\n</file>\n\nReview this file only."}],
    )
    return {"path": path,
            "findings": next(b.input for b in resp.content if b.type == "tool_use")["findings"]}

def integration_pass(pr_files: dict, local_results: list[dict], criteria: str) -> dict:
    """Cross-file pass: data flow and contract issues between files."""
    summaries = json.dumps(local_results, indent=2)
    interfaces = "\n".join(extract_signatures(text) for text in pr_files.values())
    resp = client.messages.create(
        model="claude-sonnet-4-6", max_tokens=2048,
        system=criteria,
        tools=[REPORT_FINDINGS],
        tool_choice={"type": "tool", "name": "report_findings"},
        messages=[{"role": "user", "content":
            f"<local_findings>\n{summaries}\n</local_findings>\n"
            f"<interfaces>\n{interfaces}\n</interfaces>\n\n"
            "Report ONLY cross-file issues: signature/caller mismatches, data passed between "
            "modules with inconsistent types or units, broken invariants across files. "
            "Do not repeat local findings."}],
    )
    return next(b.input for b in resp.content if b.type == "tool_use")

with ThreadPoolExecutor(max_workers=8) as pool:           # local passes are independent -> parallel
    local = list(pool.map(lambda kv: review_file(kv[0], kv[1], CRITERIA), pr_diffs.items()))
cross = integration_pass(pr_files, local, CRITERIA)
```

Note the design: local passes run in parallel with identical instructions (consistency), the integration prompt explicitly narrows scope to cross-file issues, and the PR's contradictory-feedback problem disappears because the same criteria are applied per file rather than "remembered" across fourteen.

#### Example 3 - verification pass with confidence for routing

```python
VERIFY = {
    "name": "verify_findings",
    "description": "For each finding, decide whether it is a genuine issue and state your confidence.",
    "strict": True,
    "input_schema": {
        "type": "object",
        "properties": {
            "verdicts": {
                "type": "array",
                "items": {
                    "type": "object",
                    "properties": {
                        "finding_id": {"type": "string"},
                        "verdict": {"type": "string", "enum": ["genuine", "false_positive", "unclear"]},
                        "confidence": {"type": "string", "enum": ["high", "medium", "low"]},
                        "reason": {"type": "string", "description": "One or two sentences of evidence from the code."}
                    },
                    "required": ["finding_id", "verdict", "confidence", "reason"],
                    "additionalProperties": False
                }
            }
        },
        "required": ["verdicts"],
        "additionalProperties": False
    }
}

def route(finding, verdict):
    if verdict["verdict"] == "false_positive" and verdict["confidence"] == "high":
        return "suppress_but_log"          # keep logs: sample these to measure the error rate
    if verdict["verdict"] == "genuine" and verdict["confidence"] == "high":
        return "post_to_pr"
    return "human_review_queue"            # medium/low or unclear: a person looks
```

The verifier is a **separate call** (and ideally separate instance from the one that produced the findings) that sees the code and the finding, not the original reasoning. The `route` function is where the self-reported confidence is used, and the "suppress_but_log" branch keeps a sampling path so you can check the high-confidence bucket for errors.

#### Example 4 - what *not* to do

```text
BAD  : Single prompt: "Review these 14 files, then re-read your own review and double-check it,
        think step by step before answering."
        -> attention diluted across files; self-check anchored on its own reasoning;
           contradictory verdicts on identical patterns.

GOOD : 14 focused per-file passes (same criteria) + 1 integration pass over interfaces and findings
        + (optionally) an independent verification pass with confidence-based routing.
```

#### Example 5 - the prompt-chaining shape behind it

The workflow patterns article lists **prompt chaining** (a task split into fixed steps with programmatic "gates" between them, trading latency for accuracy) and **parallelization** (sectioning independent subtasks; or voting for confidence). Do not confuse the article's code-review voting example (several different prompts review the code and flag it if they find a problem; the article also mentions tuning vote thresholds to balance false positives and negatives) with the Q12 distractor that keeps only issues found in at least two of three runs; the exam answer rejects the consensus filter. Multi-pass review is prompt chaining plus sectioning: per-file passes are the sectioned parallel stage; the integration pass is the next link in the chain; a gate between them can enforce, for example, that every per-file result validates against the findings schema before the integration pass starts.

### Exam traps

- **"Ask Claude to review its own output more carefully / use extended thinking"** - same context, same blind spots. Prefer an independent instance.
- **"Use a larger context window so all files fit"** - capacity is not the issue; attention dilution is.
- **"Run the review three times and require agreement"** - suppresses true findings that are intermittently caught.
- **"Make developers submit smaller PRs"** - pushes the problem onto people instead of fixing the pipeline.
- **"Single integration pass over all files for everything"** - recreates the dilution; keep local and cross-file concerns separate.
- **"Self-reported confidence is always garbage / always the answer"** - it is a routing aid after calibration, not an escalation gate replacing explicit criteria (Q3 vs 4.6).
- **"Pass the generator's reasoning to the reviewer so it understands"** - defeats the independence the design relies on.

### Check yourself

1. A team has the same Claude session write a feature and then review it, and bugs still ship. What is the architectural fix?
   <details><summary>Answer</summary>Use a second, independent Claude instance (fresh context, no access to the generator's reasoning) to review the code; self-review in-session and extended thinking keep the same anchoring.</details>
2. Why does the integration pass exist separately from the per-file passes?
   <details><summary>Answer</summary>Per-file passes give consistent depth for local issues but cannot see problems that live between files (cross-file data flow, interface mismatches). A separate integration pass focused on those avoids attention dilution and contradictory findings.</details>
3. How can self-reported confidence be used safely in a review pipeline?
   <details><summary>Answer</summary>As a routing aid in a verification pass: high-confidence verdicts take the fast path, medium/low go to human review, calibrated against labeled data and with a sample of the confident bucket checked. It should not replace explicit criteria and is not trusted as an absolute probability.</details>

### Official docs

- [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents) - prompt chaining, parallelization (sectioning/voting), orchestrator-workers, evaluator-optimizer.
- [Thinking](https://platform.claude.com/docs/en/build-with-claude/thinking) - what thinking does and what it costs (billed as output tokens), plus the forced-tool-use restrictions behind the 4.3 note. It says nothing about independent review; that contrast is the exam guide's.
- [Review locally with /code-review](https://code.claude.com/docs/en/code-review-local.md) - in a terminal session the built-in reviewer runs as a background subagent with its own context window (in a `-p` run it waits and returns the findings), honors `CLAUDE.md`, supports `--fix`, `--comment`, an effort level, and `--max-findings`.
- [Prompting best practices - Chain complex prompts](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) - draft, review against criteria, refine as separate calls.
- [Run Claude Code programmatically](https://code.claude.com/docs/en/headless.md) - each `claude -p` call is a fresh run unless you pass `--continue` / `--resume`.

---

## Decision cheat-table

| If the question says ... | Prefer ... | Because ... |
|---|---|---|
| Too many false positives in code review | Explicit categorical report/skip criteria | "Be conservative" / confidence thresholds define nothing; criteria define the decision |
| "Only report high-confidence findings" is offered as the fix | Reject it | Self-reported confidence is poorly calibrated; it does not fix the categories that are wrong |
| One review category is much noisier than the rest | Temporarily disable it, improve its prompt, re-enable | False positives in one category erode trust in all categories |
| Severity labels are inconsistent | Define each level with a rule plus a concrete code example | Anchors beat adjectives |
| Output format or ambiguous-case judgment is inconsistent despite detailed instructions | 2-4 targeted few-shot examples with reasoning | Examples show format and generalize judgment |
| Two similar tools are being confused and descriptions are minimal | Expand tool descriptions first | Descriptions drive tool selection; few-shot is the later refinement |
| Need guaranteed schema-compliant structured output | Tool use with a JSON schema (`strict: true` for type guarantees) | Removes syntax/type errors; prose-JSON parsing does not |
| Model may reply in text instead of calling the tool | Use `tool_choice` `any` or forced tool | `auto` permits a text-only reply |
| Several extraction schemas; document type unknown | `tool_choice: {"type": "any"}` | Must call a tool; model picks the right schema |
| A particular extraction must run before enrichment steps | Forced `{"type": "tool", "name": "extract_metadata"}` | Deterministic ordering |
| Model fabricates values for fields the source lacks | Make fields nullable/optional (and add `"unclear"` / `"other"` + detail) | Required fields pressure the model to invent |
| Line items do not sum to the total in an otherwise valid extraction | Extract `calculated_total` + `stated_total` + `conflict_detected`, validate in code | Schemas guarantee syntax, not semantics |
| Extraction failed validation on format/structure | Retry with document + failed extraction + specific errors | Information exists; model can self-correct |
| Required info lives in an external document that was not provided | Do not retry; return null / flag for follow-up | Retry cannot create absent information |
| Need to learn which code constructs developers dismiss | Add `detected_pattern` to findings and aggregate dismissals | Pinpoints which patterns/categories to fix |
| Blocking pre-merge check vs overnight report; "save 50%" | Sync for pre-merge, Batch for overnight | Batch: up to 24 h, no latency SLA |
| 30-hour SLA with 24-hour batch window | Submit every 4 hours (4 + 24 = 28 h worst case, 2 h margin) | Worst case = submission interval + batch window |
| Some batch requests failed | Resubmit only failed `custom_id`s, fixing the cause (e.g. chunk) | Results are independent; order not guaranteed |
| About to batch 50,000 documents | Refine the prompt on a small sample first (sync) | Raises first-pass success, avoids resubmission rounds |
| Single-pass review of a big PR is inconsistent/contradictory | Per-file local passes + a cross-file integration pass | Attention dilution; bigger context does not help |
| Same session generated the code and reviews it | Independent second instance without generator reasoning | Self-review is anchored; extended thinking does not remove that |
| Want to reduce human review volume safely | Verification pass with confidence, calibrated, routed; sample the confident bucket | Confidence is a routing signal, not ground truth |

---

## Sources fetched

All fetched on 2026-10-30. (All pages fetched successfully and were then re-checked against complete local copies. The batch, prompting best-practices, structured-outputs, strict-tool-use, Agent SDK, headless, code-review and engineering-article pages were read in full; the long API-reference pages `batches/create` and `batches/results` and the `thinking` and `define-tools` pages were checked passage by passage via targeted search rather than line by line.)

- https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/overview.md
- https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices.md
- https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/increase-consistency.md
- https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations.md
- https://platform.claude.com/docs/en/build-with-claude/structured-outputs.md
- https://platform.claude.com/docs/en/agents-and-tools/tool-use/strict-tool-use.md
- https://platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools.md
- https://platform.claude.com/docs/en/build-with-claude/batch-processing.md
- https://platform.claude.com/docs/en/api/messages/batches/create.md
- https://platform.claude.com/docs/en/api/messages/batches/results.md
- https://code.claude.com/docs/en/agent-sdk/structured-outputs.md
- https://code.claude.com/docs/en/headless.md
- https://code.claude.com/docs/en/code-review-local.md
- https://platform.claude.com/docs/en/build-with-claude/thinking.md
- https://www.anthropic.com/engineering/building-effective-agents

Also used: the official exam guide v1.0 (task statements 4.1-4.6, Scenarios 5 and 6, sample questions Q3, Q11, Q12, in-scope/out-of-scope lists).
