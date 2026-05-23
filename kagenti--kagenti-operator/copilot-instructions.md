## kagenti-operator

> This file provides guidance for AI assistants working with this repository.

# CLAUDE.md - Kagenti Operator

This file provides guidance for AI assistants working with this repository.

## Commit Attribution Policy

When creating git commits, do NOT use `Co-Authored-By` trailers for AI attribution.
Instead, use `Assisted-By` to acknowledge AI assistance without inflating contributor stats:

    Assisted-By: Claude (Anthropic AI) <noreply@anthropic.com>

Never add `Co-authored-by`, `Made-with`, or similar trailers that GitHub parses as co-authorship.

A `commit-msg` hook in `scripts/hooks/commit-msg` enforces this automatically.
Install it via pre-commit:

```sh
pre-commit install --hook-type pre-commit --hook-type commit-msg
```

## Claude Code Skills

This repo includes skills in `.claude/skills/` for guided workflows:

### Orchestration

Orchestrate skills for enhancing related repositories:

| Skill | Description |
|-------|-------------|
| `orchestrate` | Entry point — run `/orchestrate <repo-url>` to start |
| `orchestrate:scan` | Assess repo structure, tech stack, and gaps |
| `orchestrate:plan` | Create phased enhancement plan |
| `orchestrate:precommit` | Add pre-commit hooks and linting |
| `orchestrate:tests` | Add test infrastructure and initial coverage |
| `orchestrate:ci` | Add CI workflows (lint, test, build, security) |
| `orchestrate:security` | Add security governance files |
| `orchestrate:review` | Review orchestration PRs before merge |
| `orchestrate:replicate` | Bootstrap skills into target repo |

### Skill Management

| Skill | Description |
|-------|-------------|
| `skills:scan` | Audit repository skills |
| `skills:write` | Create or edit skills following the standard |
| `skills:validate` | Validate skill format and structure |

---
> Source: [kagenti/kagenti-operator](https://github.com/kagenti/kagenti-operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
