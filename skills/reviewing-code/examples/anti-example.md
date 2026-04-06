# Anti-Example — Vague Code Review

## Input

User: "Review PR #42"

## What went wrong

The reviewer skipped Steps 2 (project docs) and 5 (security audit), and produced vague findings without file:line references.

## Bad output

```markdown
# Code Review: PR #42

Overall this looks okay. A few things to consider:

- The JWT implementation could be improved
- There might be some performance issues
- Tests could be more comprehensive
- Code style is generally fine

**Verdict: Approve**
```

## Error analysis

| Problem | Impact | What was missed |
|---|---|---|
| No file:line references | Findings are unactionable — developer doesn't know where to look | auth/validate.go:61 had a critical auth bypass |
| "Could be improved" without specifics | Developer has no idea what to fix | Token expiry was 24h, violating arch standard |
| Skipped security step | A critical vulnerability was approved | Signature not verified before claims extracted |
| Skipped project docs | Generic review instead of project-aware review | Architecture doc had explicit requirements |
| "What's Done Well" is absent | Missed opportunity to reinforce good patterns | RS256 signing was correctly done |
| Report not saved | No audit trail | Should be in docs/agent-docs/reviews/ |

## The rule

A finding without `file:line` + a concrete suggested fix is not a finding — it's a comment. Comments don't get fixed. Findings do.

Never approve code with unverified security — the security step is mandatory even for "small" refactors.
