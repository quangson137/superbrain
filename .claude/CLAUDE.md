# Superbrain — Claude Plugin Project

## Project Purpose

This project builds a **Claude Code plugin** — a curated collection of skills and agents for software development workflows (and potentially other domains in the future). The goal is to produce high-quality, reusable, production-ready skills that follow a rigorous engineering standard.

## Project Structure

```
superbrain/
├── skills/                        # Main skill library
│   ├── brainstorming/             # Idea → spec pipeline
│   ├── writing-plans/             # Spec → implementation plan pipeline
│   ├── reviewing-code/            # Code review pipeline
│   └── executing-plans/           # Plan execution pipeline
├── agents/
│   └── planner.md                 # Orchestrator agent (coordinates skills)
├── .claude/
│   ├── settings.local.json        # Local Claude Code permissions
│   └── skills/
│       └── skill-master/          # Meta-skill for skill engineering
│           ├── SKILL.md
│           ├── examples/
│           └── references/        # 9-layer checklist, rubric, metadata formula
├── CREATE_PLUGIN.md               # Instructions for packaging and publishing
└── temporary/                     # Research/reference materials (gitignored)
```

## Core Development Pipeline

Skills implement a sequential software development pipeline:

```
Raw Idea
  → /brainstorming     → produces: docs/agent-docs/specs/<spec>.md
  → /writing-plans     → produces: docs/agent-docs/plans/<plan>.md
  → /executing-plans   → implements plan task by task
  → /reviewing-code    → produces: docs/agent-docs/reviews/<review>.md
```

The `planner` agent orchestrates this pipeline by selecting which skill to invoke based on current state.

## Skill Engineering Standard

All skills in this project follow the **9-Layer Skill Engineering Framework**. The meta-skill for creating/reviewing/fixing skills is at `.claude/skills/skill-master/SKILL.md`.

### 9 Layers
| Layer | Name | Key Requirement |
|-------|------|-----------------|
| 0 | Use-case map | When to use, when NOT to use |
| 1 | Metadata | YAML frontmatter with trigger-phrase description (CSO formula) |
| 2 | Core SKILL.md | Body ≤500 lines, 8 structured sections |
| 3 | References | Supporting docs in `references/` folder |
| 4 | Examples | Positive + anti-examples in `examples/` folder |
| 5 | Scripts | Automation scripts if needed |
| 6 | Assets | Templates, prompts, companion files |
| 7 | Output contract | Explicit definition of deliverables |
| 8 | Governance | Version, ownership, changelog |

### Quality Scoring (6 axes × 5 points = 30 max)
- **Strategy** — purpose clarity and use-case specificity
- **Activation** — trigger phrase quality (CSO formula)
- **Operation** — workflow completeness and decision logic
- **Quality** — self-check and output contract
- **Resources** — references and examples
- **Governance** — versioning and changelog

Score interpretation: 0–12 Failing → 13–18 Basic → 19–24 Good → 25–30 Mature

### Metadata Description Formula (CSO)
Skill descriptions must use trigger phrases, NOT workflow summaries:
```
"Use this skill when the user asks to '[trigger 1]', '[trigger 2]'. Do NOT use for: [anti-trigger]."
```
50–100 words, 3–6 trigger phrases, at least 1 negative trigger.

## Key Conventions

- **Skills produce artifacts**: Every skill writes output to `docs/agent-docs/<type>/` in the target project.
- **Documentation-first**: Skills read existing project docs before reviewing or planning code.
- **TDD philosophy**: Plans include failing tests first before implementation.
- **No premature implementation**: `brainstorming` skill has a hard gate — no code until user approves the spec.
- **Skills read the codebase**: `reviewing-code`, `writing-plans`, and `executing-plans` all explore the target project structure before acting.

## Working on This Project

### Adding a new skill
1. Use `/skill-master` to create it (CREATE mode)
2. Follow the 9-layer checklist at `.claude/skills/skill-master/references/9-layer-checklist.md`
3. Score it with the rubric before merging — target ≥25/30 for production use
4. Add it to the plugin manifest (see `CREATE_PLUGIN.md`)

### Reviewing/improving an existing skill
1. Use `/skill-master` in REVIEW mode to score the current skill
2. Use FIX mode to standardize it to the 9-layer framework

### Plugin packaging
Refer to `CREATE_PLUGIN.md` for instructions on how to package and distribute the plugin.

## What NOT to Do

- Do not implement features during brainstorming — produce a spec first
- Do not create skills without going through the 9-layer framework
- Do not write metadata descriptions that summarize workflow steps (use trigger phrases only)
- Do not commit anything in `temporary/` — it is gitignored reference material
