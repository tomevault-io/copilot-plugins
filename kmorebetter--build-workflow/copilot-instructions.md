## build-workflow

> Orchestrates gstack and compound-engineering into a three-phase feature pipeline: **think → make → ship**.

# Build Workflow Plugin

Orchestrates gstack and compound-engineering into a three-phase feature pipeline: **think → make → ship**.

## Commands

| Command | Phase | Steps | Source |
|---------|-------|-------|--------|
| `/build` | All | Full 11-step workflow with gates | Orchestrator |
| `/build:think` | Think | Clarify → Challenge → CEO Review → Eng Review | Custom + gstack |
| `/build:make` | Make | Brainstorm → Plan → Work → Review | CE |
| `/build:ship` | Ship | QA → Capture Learnings → Deploy | gstack + CE |

## Dependencies

| Tool | Type | Required Skills/Commands |
|------|------|------------------------|
| **gstack** | Skills (`~/.claude/skills/gstack/`) | plan-ceo-review, plan-eng-review, qa, ship |
| **compound-engineering** | Plugin | brainstorming, workflows:plan, workflows:work, workflows:review, workflows:compound |

## Data Flow Between Phases

```
/build:think → docs/briefs/YYYY-MM-DD-*-brief.md
                    ↓
/build:make  → docs/brainstorms/ → docs/plans/ → feature branch + PR
                    ↓
/build:ship  → docs/solutions/ → merged + deployed
```

Each phase writes artifacts that the next phase auto-detects.

## Customization

The think phase (steps 1-2) contains the custom interview and challenge logic. Edit `commands/build-think.md` to adjust:
- Interview questions and confidence threshold
- Challenge topics and pushback style
- Gate criteria between steps

---
> Source: [kmorebetter/build-workflow](https://github.com/kmorebetter/build-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
