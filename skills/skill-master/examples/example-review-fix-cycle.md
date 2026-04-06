# Example: Review + Fix of a Rudimentary Skill

## INPUT — Original skill (single file only)

```markdown
---
name: email-skill
description: This skill helps with emails and marketing.
---
# Email Skill
Write professional emails for business use.
When user asks for email, write one that is professional and clear.
Use formal tone. Include subject line.
```

## OUTPUT — Section A: Review Report

# 📋 REVIEW REPORT: email-skill

## Overview
- Name: email-skill | Version: none | File count: 1
- Score: 7/30 — Failing

## Layer-by-layer assessment
| Layer | Name | Status | Notes |
|-------|------|--------|-------|
| 0 | Use Case | ❌ | Too broad: "emails and marketing" |
| 1 | Metadata | ❌ | Description vague, no triggers, no version |
| 2 | SKILL.md | ❌ | Only 3 lines, missing 7 of 8 sections |
| 3 | References | ❌ | None |
| 4 | Examples | ❌ | None |
| 5 | Scripts | ⬜ | Not required |
| 6 | Assets | ⬜ | Not required |
| 7 | Output Contract | ❌ | None |
| 8 | Governance | ❌ | No version |

## 6-axis score
| Axis | Score | Notes |
|------|-------|-------|
| Strategy | 2/5 | "emails and marketing" too broad |
| Activation | 1/5 | Will trigger incorrectly |
| Operation | 1/5 | No workflow |
| Quality | 1/5 | No output contract |
| Resources | 1/5 | No resources |
| Governance | 1/5 | No governance |

## Top 3 items to fix
1. Rewrite metadata — Layer 1 — Urgent
2. Restructure SKILL.md into all 8 sections — Layer 2 — Urgent
3. Create ≥2 examples — Layer 4 — High

---

## OUTPUT — Section B: Fix Report

# 🔧 FIX REPORT: email-skill

## Changelog
| # | Change | Layer | Details |
|---|--------|-------|---------|
| 1 | Rewrote metadata | 1 | Added triggers, negative trigger, version |
| 2 | Restructured SKILL.md | 2 | Split into 8 standard sections |
| 3 | Added output contract | 7 | Format + criteria + final check |
| 4 | Created examples/ | 4 | 1 good + 1 anti-example |
| 5 | Bumped version | 8 | → 1.0.0 |

## Before/after comparison
| Axis | Before | After | Δ |
|------|--------|-------|---|
| Strategy | 2 | 4 | +2 |
| Activation | 1 | 4 | +3 |
| Operation | 1 | 4 | +3 |
| Quality | 1 | 4 | +3 |
| Resources | 1 | 3 | +2 |
| Governance | 1 | 3 | +2 |
| **Total** | **7** | **22** | **+15** |

Rating: Failing → Good
