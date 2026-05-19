## opencode-supreme-setup

> Unified Spec-Driven Development agent — complete 7-step workflow.

# AGENTS

## SpecKit Agent

### @speckit
Unified Spec-Driven Development agent — complete 7-step workflow.
Runs autonomously: constitution → specify → clarify → plan → validate → tasks → implement.
Uses real /speckit.* commands via the official spec-kit CLI.

### Workflow
1. User: @speckit [idea or requirement]
2. Agent executes all 7 steps automatically using /speckit.* commands
3. Agent delivers production-ready code + full .specify/ artifact set
4. Agent prompts you to test the running app

## OpenCode Tool Mapping
Some skills reference `TodoWrite` from Claude Code. In OpenCode, it does NOT exist. Use:

| Skill Reference | OpenCode Tool |
|-----------------|---------------|
| `TodoWrite` | `task_create` (create todo) |
| `TodoWrite` mark complete | `task_update(status="completed")` |
| `TodoWrite` mark in_progress | `task_update(status="in_progress")` |
| List `TodoWrite` | `task_list()` |
| Get `TodoWrite` | `task_get(id)` |

NEVER call `todoWrite` - it will fail. Use the task_* tools above.

---
> Source: [skeletorflet/opencode-supreme-setup](https://github.com/skeletorflet/opencode-supreme-setup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
