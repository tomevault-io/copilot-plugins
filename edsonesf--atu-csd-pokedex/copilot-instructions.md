## atu-csd-pokedex

> Agent skills provide task-specific context and instructions following the open [Agent Skills](https://agentskills.io/specification) standard. Skills are only loaded when relevant to the current task.

# Skills Index

Agent skills provide task-specific context and instructions following the open [Agent Skills](https://agentskills.io/specification) standard. Skills are only loaded when relevant to the current task.

## Available Skills

| Skill | Description | Path |
|---|---|---|
| testing | Pre-push quality gates (ruff, format, mypy, pytest) | [`.agents/skills/testing/SKILL.md`](.agents/skills/testing/SKILL.md) |
| git-workflow | Branch management, rebasing, and push safety | [`.agents/skills/git-workflow/SKILL.md`](.agents/skills/git-workflow/SKILL.md) |
| github-issues | Read, create, and comment on issues with full context | [`.agents/skills/github-issues/SKILL.md`](.agents/skills/github-issues/SKILL.md) |
| github-prs | PR creation and review feedback loops | [`.agents/skills/github-prs/SKILL.md`](.agents/skills/github-prs/SKILL.md) |
| cicd-aws-deployment | AWS ECS Fargate deployment via GitHub Actions | [`.agents/skills/cicd-aws-deployment/SKILL.md`](.agents/skills/cicd-aws-deployment/SKILL.md) |

## Structure

```
.agents/skills/
├── testing/
│   └── SKILL.md
├── git-workflow/
│   └── SKILL.md
├── github-issues/
│   └── SKILL.md
├── github-prs/
│   └── SKILL.md
└── cicd-aws-deployment/
    ├── SKILL.md
    ├── scripts/
    └── references/
```

## Adding Skills

To add a new skill, create a folder under `.agents/skills/` with a `SKILL.md` file and then update the [Available Skills](#available-skills) table above. The `SKILL.md` should contain:

```yaml
---
name: skill-name
description: What this skill does and when to use it.
---

# Instructions here
```

See the [specification](https://agentskills.io/specification) and [examples](https://github.com/anthropics/skills) for details.

---
> Source: [edsonesf/ATU-CSD-POKEDEX](https://github.com/edsonesf/ATU-CSD-POKEDEX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
