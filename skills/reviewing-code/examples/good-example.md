# Good Example — Code Review of PR #42 (User Auth Refactor)

## Input

User: "Review PR #42"

## Process

1. **Load context:** `gh pr view 42 --json title,body,files` → PR title: "Refactor user authentication to JWT", 3 files changed, PR body explains migration from session cookies to stateless JWT tokens.

2. **Check project docs:** `CLAUDE.md` → `TECHNICAL_DOCS_DIRS=docs/architecture`. Read `docs/architecture/auth.md` — confirms the team uses RS256 signing and requires token expiry under 1 hour.

3. **Architecture review:** JWT middleware is properly separated into `middleware/auth.go`. No circular deps introduced. ✅

4. **Code inspection:** Found hardcoded expiry of `86400` (24h) in `auth/token.go:34` — violates the 1-hour standard from architecture docs. Also found a dead import in `handlers/user.go:5`.

5. **Security audit:** Token validation is parameterized correctly. But found: `user_id` is taken directly from the JWT payload without verifying the signature first (`auth/validate.go:61`) — critical auth bypass risk.

6. **Performance:** Token parsing done per-request with no caching — acceptable for this scale.

7. **Testing:** New unit tests cover happy path but miss the expired-token case.

8. **Documentation:** Public `ValidateToken()` function has no docstring.

## Output

```markdown
# Code Review: Refactor user authentication to JWT

**Date:** 2026-04-06
**Reviewed by:** Claude (code-review skill)
**Scope:** PR #42 — 3 files (middleware/auth.go, auth/token.go, auth/validate.go)
**Change type:** Refactor

---

## Summary

This PR migrates authentication from session cookies to stateless JWT tokens using RS256. The approach fits the existing architecture pattern, but a critical auth bypass vulnerability was found where the user ID is extracted before signature verification.

---

## Findings

### 🔴 Critical — Must Fix

- [ ] `auth/validate.go:61` **User ID extracted before signature verification** — `claims["user_id"]` is read before `token.Valid` is checked. An attacker can craft an unsigned token and bypass auth entirely. Fix: check `token.Valid` first, then extract claims.

### 🟡 Important — Should Fix

- [ ] `auth/token.go:34` **Token expiry exceeds architecture standard** — hardcoded `86400` (24h) violates `docs/architecture/auth.md` which requires ≤1h. Replace with a named constant `TokenExpirySeconds = 3600`.
- [ ] `auth/token_test.go` **Missing expired-token test case** — add a test that asserts `ValidateToken()` returns an error for a token past its expiry.

### 🟢 Nice-to-Have — Consider

- [ ] `handlers/user.go:5` **Dead import** — `"fmt"` is imported but unused. Remove it.
- [ ] `auth/validate.go` **Missing docstring on `ValidateToken()`** — public function should document its parameters and return values.

---

## What's Done Well

- Clean separation of JWT logic into `middleware/auth.go` — easy to test in isolation.
- RS256 signing correctly implemented; symmetric HS256 was avoided (common mistake).

---

## Verdict

**Request Changes** — Critical auth bypass must be fixed before merge.
```

## Why this is a good example

- **Step 2 (project docs):** Architecture doc was read and used to catch the 24h expiry violation — a finding that generic rules wouldn't catch.
- **Every finding has file:line + concrete fix** — not just "this looks wrong".
- **"What's Done Well" is specific** — names actual patterns, not generic praise.
- **Critical issue is correctly prioritized** — auth bypass gets 🔴, style issue gets 🟢.
