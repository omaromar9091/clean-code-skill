---
name: clean-code
description: "Review and refactor code against 11 clean-code rules (clear naming, single-purpose functions, DRY, single responsibility, consistency, no duplicated logic, early return, why-comments, right-sized abstraction, explicit error handling, separation of concerns) while preserving behavior. Use this skill whenever the user asks to review, audit, clean up, refactor, simplify, or improve the quality, readability, or maintainability of code, or mentions clean code, code smells, technical debt, spaghetti code, duplicated code, long or messy functions, or organizing a project into routes/services/data layers, even if they never say 'clean code' explicitly. Arabic triggers - مراجعة الكود، تنظيف الكود، تحسين الكود، إعادة هيكلة، ريفاكتور، كود نضيف، كود مكرر، الكود مش مرتب. Works with any language or framework. Replies in the user's language (Arabic or English)."
license: MIT
compatibility: opencode, claude-code
metadata:
  version: "1.0.0"
  languages: "en, ar"
---

# Clean Code: Review and Refactor

Apply 11 clean-code rules to real code **without changing what it does**.

Reply in the user's language (Arabic or English). Explain in that language, but keep code, identifiers, file names, and technical terms exactly as they are in the project. If the user writes Arabic, write the report and explanations in Arabic.

## Golden rules (read these first)

1. **Behavior is preserved.** Refactoring changes structure, never behavior. Anything that would change behavior (including adding error handling) is listed separately and needs approval.
2. **The project's conventions beat this skill.** If the codebase consistently does X, follow X. That is the Consistency rule applied to the skill itself.
3. **Don't invent problems.** If the code is already clean, say so in one or two lines. Report only violations you can point to with `file:line`.
4. **Small, reviewable steps.** One concern per change, so the diff is easy to read and easy to revert.
5. **Verify.** Run tests, linter, or the app after each step. If something breaks, revert that step and report it.
6. **If duplicated copies of a rule disagree, stop.** Do not silently pick one. Report it as a probable bug and ask which version is correct.

## Modes

- **Review (report only, default when unclear):** the user says review, audit, check, "what's wrong", "راجع". Produce the report, make no edits, then ask what to apply.
- **Refactor (apply changes):** the user says fix, clean, refactor, apply, "نضف", "صلّح", "ريفاكتور". Show a short plan, then apply it. If the user says "just do it", skip waiting for approval, but still keep the guardrails below.

If the user names specific rules (for example "only early return and naming"), apply only those.

## Workflow

**Step 0: Scope.** Identify which files or folders are in scope. If the scope is huge (a whole large repo), ask which area to start with, or start with the files that change most or hurt most. Detect the language and framework and note the existing conventions (naming style, folder layout, error style, formatter).

**Step 1: Baseline.** Find how to verify behavior: tests, linter, type checker, or a way to run the app. Run them once before touching anything so you know the starting state. If there are no tests, say so, be extra conservative, and prefer mechanical changes (renames, early returns, extractions).

**Step 2: Scan.** Check the code against the 11 rules using the four groups below. Read the matching reference file when you find a violation or need the before/after pattern.

**Step 3: Report.** Use the report format at the end of this file. Sort by severity. Show at most about 15 findings, the most valuable first. Mention that more exist if there are more.

**Step 4: Apply (Refactor mode).** Use the safe order below. After each step, verify.

**Step 5: Summary.** List what changed, what was verified, what you deliberately did not touch, and any behavior-changing suggestions waiting for approval.

## The 11 rules at a glance

| # | Rule | Main smell | Fix | Details |
|---|------|-----------|-----|---------|
| 1 | Naming (التسمية) | `calc1`, `data`, `tmp`, names that lie | Names that say what it is or does | `references/readability.md` |
| 2 | Comment (التعليقات) | Comments that repeat the code, dead code | Comment the **why**, delete the **what** | `references/readability.md` |
| 3 | Consistency (الاتساق) | Two styles or patterns for the same thing | Follow the dominant convention | `references/readability.md` |
| 4 | Early Return (الخروج المبكر) | Deep nesting, `else` after `return` | Guard clauses first, main path last | `references/readability.md` |
| 5 | Function (الدوال) | Long, does several jobs, flag parameters | One function, one job | `references/structure.md` |
| 6 | Single Responsibility (المسؤولية الواحدة) | File or class with several reasons to change | Split by reason to change | `references/structure.md` |
| 7 | Abstraction (التجريد) | Noisy repeated details, or over-engineering | Hide noisy details, stop at the rule of three | `references/structure.md` |
| 8 | Separation of Concerns (فصل المسؤوليات) | SQL, business rules, and UI in one place | Layers: UI/API, business logic, data | `references/structure.md` |
| 9 | DRY (عدم تكرار الكود) | Copy-pasted blocks and literals | Extract once, call everywhere | `references/duplication.md` |
| 10 | Don't Repeat Logic (عدم تكرار المنطق) | Same business rule in several places | One source of truth per rule | `references/duplication.md` |
| 11 | Error Handling (التعامل مع الأخطاء) | Bare `except`, unchecked input, ignored failures | Validate at boundaries, catch specifically | `references/error-handling.md` |

## Safe order for applying changes

Go from lowest risk to highest. Finish and verify one level before starting the next.

1. **Mechanical and local:** Naming, Comment, Early Return, Consistency.
2. **Within a file:** Function (extract), Abstraction (constants, small helpers).
3. **Across the file or module:** DRY, Don't Repeat Logic.
4. **Across files (highest churn):** Single Responsibility, Separation of Concerns. Extract data access first, then business logic, then thin the entry point.
5. **Behavior-changing:** Error Handling. Adding validation or catching errors changes what happens on failure. List these separately with the before and after behavior, and get approval before applying.

## Severity

- **High:** real bug risk or hard to change safely. Examples: silent failures, the same business rule in several places, UI, SQL, and business rules mixed in one function, hidden side effects.
- **Medium:** hurts readability and maintenance. Examples: function over about 40 lines or whose name needs "and", nesting deeper than 3, duplicated blocks, mixed naming styles.
- **Low:** polish. Examples: vague names, comments that repeat the code, magic numbers.

Numbers here (40 lines, 3 levels, 4 parameters) are heuristics, not laws. Judge by readability.

## Guardrails: never do these without explicit approval

- Rename or alter anything other code or systems depend on: public APIs, route URLs, DB tables and columns, migrations, environment variables, config keys, JSON field names, CLI flags, template variable names.
- Change user-facing text, translations, i18n keys, or text direction (RTL/LTR). Arabic interface strings stay exactly as they are.
- Add dependencies, change formatter or linter settings, or reformat whole files (it creates noisy diffs). Style-only changes go in their own step.
- Touch generated, vendored, minified, or lock files.
- Add an abstraction with fewer than 3 real uses.
- Delete comments that explain **why**, TODOs with context, or license headers.
- When adding error handling: never swallow exceptions silently, never catch a broad exception just to hide it. Log with context or re-raise.
- In code that handles money, dates, permissions, authentication, or deleting data: make the smallest possible change, and require tests or a manual check.

## Report format

```
## Clean Code Review: <scope>

Summary: <2-3 lines: overall state, biggest risk, quick wins>
Verified with: <tests / linter / manual / none>

### Findings
| Sev | Rule | Location | Problem | Suggested fix |
|-----|------|----------|---------|---------------|
| High | Don't Repeat Logic | services.py:42, routes.py:88 | "Refundable" rule written twice, copies differ | One `is_refundable(order)`; confirm which copy is right |
| Medium | Early Return | routes.py:15-60 | 5 levels of nesting | Guard clauses |

### Behavior-changing suggestions (need approval)
- <what changes, before vs after>

### Not touched on purpose
- <public routes, DB schema, UI text, ...>

### Suggested order
1. ... 2. ... 3. ...
```

## Notes for Arabic-speaking users

- Explain in the user's dialect level (Egyptian or Modern Standard), keep technical words in English (`function`, `route`, `service`, `refactor`).
- Never translate identifiers, and never edit Arabic UI strings, labels, or messages as part of a refactor.
- Rule names appear in both languages in this skill so the user can refer to them either way ("طبّق Early Return بس" works).
