# 9-Layer Checklist — Skill Evaluation Criteria

## Layer 0 — Use Case & Trigger Map
- [ ] Skill solves a specific task, not too broad
- [ ] Primary user is clearly identified
- [ ] Positive triggers defined (when to activate)
- [ ] Negative triggers defined (when NOT to activate)

## Layer 1 — Metadata
- [ ] YAML frontmatter with `name` and `description`
- [ ] `description` is specific, uses real user language
- [ ] Trigger phrases present (both Vietnamese and English if needed)
- [ ] Negative trigger present (Do NOT use for...)
- [ ] `version` present
- [ ] Description is 50–100 words total

## Layer 2 — Core SKILL.md
- [ ] Section 1: PURPOSE — 2–3 sentences
- [ ] Section 2: WHEN TO USE — positive + negative triggers
- [ ] Section 3: EXPECTED INPUTS — required / optional
- [ ] Section 4: WORKFLOW — step by step, referencing resources
- [ ] Section 5: OUTPUT FORMAT — structure, template
- [ ] Section 6: RESOURCE USAGE — when to read which file
- [ ] Section 7: GUARDRAILS — limits and restrictions
- [ ] Section 8: FINAL CHECK — self-check checklist
- [ ] Under 500 lines / ~3,000 words
- [ ] No long knowledge blocks that should be in references/

## Layer 3 — References
- [ ] references/ directory exists if background knowledge is needed
- [ ] SKILL.md clearly states when to read each file
- [ ] No duplicated content with SKILL.md

## Layer 4 — Examples
- [ ] ≥1 good example (input → output)
- [ ] ≥1 anti-example (bad pattern + error analysis)
- [ ] Recommended: annotated example

## Layer 5 — Scripts & Tools
- [ ] If repetitive logic exists → script is provided
- [ ] Script has a clear, stated purpose

## Layer 6 — Assets & Templates
- [ ] Templates separated from references/
- [ ] If skill produces files → template in assets/

## Layer 7 — Output Contract & QC
- [ ] Output structure is defined
- [ ] PASS / NEEDS WORK / FAILING criteria present
- [ ] Self-check checklist of 5–7 items

## Layer 8 — Governance
- [ ] Version number present
- [ ] Recommended: owner, changelog
- [ ] For personal skills: version alone is sufficient
