---
name: reviewing-code
description: >
  Use this skill when the user asks to 'review my code', 'review this PR',
  'check my pull request', 'audit code quality', "what's wrong with this code",
  'review this diff', 'kiểm tra code', 'review PR số', 'đánh giá code của tôi',
  or needs a thorough review covering quality, security, performance, and correctness.
  Also use when given a branch name, file path(s), or PR number to review.
  Do NOT use for: explaining code behavior without quality judgment, debugging a
  specific runtime error, or writing/generating new code.
argument-hint: PR number, file path(s), git diff, or branch name to review
model: opus
version: 1.2.0
---

# Code Review

## 1. PURPOSE

Review code thoroughly across quality, security, performance, and correctness dimensions, then produce a prioritized, actionable report. The output is a structured Markdown report saved to `docs/agent-docs/reviews/`.

**Announce at start:** "I'm using the code-review skill to review this code."

## 2. WHEN TO USE

✅ Use when:
- User asks to review a PR, file, diff, or branch
- User asks "what's wrong with this code?" or "audit my code"
- User wants quality, security, or performance feedback on existing code
- User provides a PR number, file path, branch name, or git diff

❌ Do NOT use for:
- Explaining what code does without quality judgment (use regular chat instead)
- Debugging a specific runtime error or exception
- Writing or generating new code

## 3. EXPECTED INPUTS

**Required (one of):**
- PR number (e.g., `#42`)
- File path(s) to review
- Git diff or branch name
- Inline code snippet

**Optional:**
- Focus area (e.g., "focus on security" or "only check performance")
- Project context or linked ticket

## 4. WORKFLOW

Complete these steps in order:

**Step 1 — Understand context:**
- If given a PR number: run `gh pr view <number> --json title,body,files` and `gh pr diff <number>`
- If given file paths: read each file in full with context from surrounding modules
- If given a branch: run `git diff main...<branch>` to load all changes
- Assess scope: number of files changed, type of change (feature/bugfix/refactor/config)
- Read the PR description or commit message — understanding intent is essential to a good review

**Step 2 — Read project technical docs:**
Check `CLAUDE.md` for the `TECHNICAL_DOCS_DIRS` variable — it lists architecture docs, coding standards, and conventions. Read those files before proceeding. This ensures the review judges code against the project's actual standards, not generic assumptions. If `TECHNICAL_DOCS_DIRS` is not defined, skip this step.

**Step 2b — Plan alignment check (if applicable):**
Check `docs/agent-docs/plans/` for a plan document matching this feature or PR. If found, read it and note whether the implementation matches the plan's requirements, architecture decisions, and scope. Distinguish between:
- **Justified deviations** — implementation improved on the plan for good technical reasons (note these as observations)
- **Problematic deviations** — missing planned functionality, scope creep, or architectural drift (flag these as 🟡 Important findings)

**Step 3 — Architecture review:**
- Does the approach fit the existing patterns in the codebase?
- Is separation of concerns maintained? Are responsibilities clearly bounded?
- Are abstractions at the right level — not too generic, not too specific?
- Does the change introduce unnecessary coupling or circular dependencies?
- For existing codebases: explore the surrounding code to understand conventions before judging consistency

**Step 4 — Code inspection:**

*Naming and structure:*
- Variables and functions have descriptive, intention-revealing names
- Functions follow single responsibility principle and are under ~50 lines
- No dead code, commented-out blocks, or unused variables

*Error handling:*
- Errors are caught at appropriate boundaries, not swallowed silently
- Failure modes are explicit and recoverable

*Anti-patterns to flag:*

| Anti-Pattern | What to Look For |
|---|---|
| **God Class/Function** | One class/function handling many unrelated responsibilities |
| **Magic Numbers** | Hardcoded numeric or string values with no named constant |
| **Deep Nesting** | 3+ levels of nested conditions — suggest early returns |
| **Duplication** | Logic repeated across files that should be extracted |
| **Dead Code** | Unreachable branches, unused imports, obsolete feature flags |

**Step 5 — Security audit:**
- **Input validation:** Is user-controlled input validated and sanitized before use?
- **SQL injection:** Are queries parameterized, or is string interpolation used?
- **XSS:** Is user input rendered as HTML without escaping (e.g., `innerHTML` with user data)?
- **CSRF:** Are state-changing endpoints protected?
- **Hardcoded secrets:** API keys, tokens, or passwords in source code?
- **Auth/authz:** Are access controls present and applied at the right layer?
- **Dependencies:** Are new packages well-maintained and free of known vulnerabilities?

**Step 6 — Performance analysis:**
- Are algorithms appropriate for the expected data size (O(n²) in a hot path is a red flag)?
- Are database queries efficient? Are N+1 query patterns present?
- Is caching used appropriately, or are expensive operations repeated unnecessarily?
- Are file handles, connections, and other resources closed after use?
- Are there obvious memory leak patterns (unbounded lists, retained closures)?

**Step 7 — Testing validation:**
- Are unit tests present for the new/changed logic?
- Are integration or edge-case tests included where the behavior is non-obvious?
- Are tests readable — does the test name describe what behavior it verifies?
- Are tests deterministic (no reliance on time, random values, or external state without mocking)?
- Is test coverage proportional to the risk of the code being changed?
- Does the implementation stay within the planned scope — no unplanned features added?
- Are breaking changes (API, schema, interface) explicitly documented?

**Step 8 — Documentation review:**
- Does complex logic have an explanatory comment (not just restating what the code does)?
- Are public functions and classes documented (docstrings with Args/Returns/Raises)?
- Is any user-facing documentation (README, API docs, changelogs) updated if needed?

**Step 9 — Write and save report:**
- Follow the template in Section 5
- Save to `docs/agent-docs/reviews/YYYY-MM-DD-<topic>-review.md`
- Notify user: "Review complete and saved to `docs/agent-docs/reviews/<filename>.md`. Found [N critical / N important / N nice-to-have] issues."

## 5. OUTPUT FORMAT

Every report MUST use this structure:

```markdown
# Code Review: [PR title / feature name]

**Date:** YYYY-MM-DD
**Reviewed by:** Claude (reviewing-code skill)
**Scope:** [What was reviewed — PR #N / files / branch]
**Change type:** [Feature / Bugfix / Refactor / Config]

---

## Summary

[2-4 sentences: what does this change do, and what is the overall assessment?]

---

## Findings

### 🔴 Critical — Must Fix
> Security vulnerabilities, data loss risks, broken functionality

- [ ] [File:line] **[Issue title]** — [what's wrong] · [why it matters] · [how to fix]

### 🟡 Important — Should Fix
> Performance problems, poor error handling, significant maintainability issues

- [ ] [File:line] **[Issue title]** — [what's wrong] · [why it matters] · [how to fix]

### 🟢 Nice-to-Have — Consider
> Style, naming, minor improvements, refactoring suggestions

- [ ] [File:line] **[Issue title]** — [what's wrong] · [why it matters] · [how to fix]

---

## Anti-Patterns Detected

| Pattern | Location | Severity |
|---|---|---|
| [Pattern name] | [File:line] | 🔴/🟡/🟢 |

*(Leave this section out if none found)*

---

## What's Done Well

[Specific callouts for good patterns, clean code, or clever solutions. Genuine and specific — not filler.]

---

## Checklist

- [ ] Architecture fits existing patterns
- [ ] No security vulnerabilities
- [ ] Performance is acceptable
- [ ] Tests cover new/changed logic
- [ ] Documentation is updated

## Production Readiness

- [ ] Schema migrations present and reversible (if DB changes)
- [ ] Backward compatibility maintained, or breaking changes documented
- [ ] No hardcoded environment-specific values (URLs, credentials, ports)

---

## Verdict

**Ready to merge: Yes / Yes with fixes / No**

[One sentence: what must change before this can merge, or why it's clear to go.]
```

**Output quality criteria:**

| Level | Criteria |
|---|---|
| ✅ PASS | All steps completed (incl. plan alignment check); every finding has file:line + why it matters + fix; report saved to correct path |
| ⚠️ NEEDS WORK | Missing 1–2 steps OR findings lack file:line references OR "why it matters" is absent |
| ❌ FAILING | Skipped security or architecture step; no report saved; findings are vague with no actionable suggestions |

## 6. RESOURCE USAGE

- **`CLAUDE.md`** (`TECHNICAL_DOCS_DIRS`): Read in Step 2 to locate project-specific coding standards
- **Surrounding modules**: Read in Step 1 to understand conventions before judging consistency
- **`examples/good-example.md`**: Reference for what a well-structured review looks like
- **`examples/anti-example.md`**: Reference for patterns to avoid in reviews

## 7. GUARDRAILS

- **Intent first** — understand what the change is trying to do before judging how it does it
- **Specific and actionable** — every finding must include file, line, and a concrete suggestion; vague comments like "this could be better" are not findings
- **Prioritize ruthlessly** — not every issue deserves equal weight; focus on what actually matters
- **Acknowledge quality** — call out what is done well, not just what needs fixing
- **No style nitpicking as blockers** — 🟢 issues should never block a merge
- **One issue per finding** — don't bundle multiple problems under one bullet
- **Never skip Step 5 (Security)** — security issues must always be checked, even for "small" changes
- **Never save to a different path** — always use `docs/agent-docs/reviews/YYYY-MM-DD-<topic>-review.md`

## 8. FINAL CHECK

Before delivering the review, verify:

☐ Loaded the full code (PR diff / files / branch) before starting?
☐ Checked `CLAUDE.md` for project coding standards (or confirmed it's absent)?
☐ Checked `docs/agent-docs/plans/` for a matching plan doc and noted deviations?
☐ Completed all 7 review dimensions (architecture → documentation)?
☐ Every finding has file:line + "why it matters" + concrete fix?
☐ "What's Done Well" section is genuine and specific — not filler?
☐ Report saved to `docs/agent-docs/reviews/YYYY-MM-DD-<topic>-review.md`?
☐ Notified user with finding counts (N critical / N important / N nice-to-have)?
