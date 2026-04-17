## dreamteam

> Subagent dispatch rules for Left/Right orchestrators


# Subagent Dispatch

When Left or Right need to dispatch a subagent:

**Load the appropriate agent prompt from `.cursor/agents/`** — each agent has its own file.
**Full dispatch rules and agent list:** `.cursor/AGENTS.md`

## Dispatch via mcp_task

Use `mcp_task` with the correct `subagent_type`:

| Agent | subagent_type |
|-------|--------------|
| Developer | `developer` |
| Reviewer | `reviewer` |
| Git-Ops | `git-ops` |
| Planner | `planner` |
| Sub-Planner | `planner-sub` (Planner only, not Left/Right) |
| Researcher | `researcher` |
| Meta Planner | `meta-planner` |
| Auditor | `auditor` |
| Learning | `learning` |
| FixPlanner | `fix-planner` |
| DevExperiencer | `dev-experiencer` |
| Terminal | `shell` |
| Left | `orchestrator-left` (Dispatcher only) |
| Right | `orchestrator-right` (Dispatcher only) |

## Left/Right Terminal Commands (Orchestration Only)

Left and Right run Terminal (`shell`) subagent ONLY for:
`set-goal`, `verify-tasks`, `verify-integrity`, `task-counter`, `run-next`, `update-task`, `sync-tasks`, `recover`, `memory-to-files`, `check-memory`, `vector-index`, `subagent-calls`

**All other work goes to the appropriate specialized agent.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/budagov-lab) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
