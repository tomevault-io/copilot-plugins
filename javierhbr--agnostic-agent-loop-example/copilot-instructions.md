## agnostic-agent-loop-example

> - Use `agentic-agent` CLI for all task and context operations.

# CLAUDE.md - Agnostic Agent Rules

## Base Rules
No base rules found.

## Claude-Specific Rules
- Use `agentic-agent` CLI for all task and context operations.
- When starting a task, run `agentic-agent task claim <TASK_ID>`.
- Before editing files in a directory, run `agentic-agent context generate <DIR>`.
- After completing work, run `agentic-agent task complete <TASK_ID>`.

## Workflow Commands
- `agentic-agent task list` — View backlog and in-progress tasks
- `agentic-agent task claim <ID>` — Claim a task (records branch + timestamp)
- `agentic-agent task complete <ID>` — Complete a task (captures commits)
- `agentic-agent validate` — Run quality validators before completing
- `agentic-agent status` — View project dashboard
- `agentic-agent plan show <track-id>` — View plan checkboxes for a track
- `agentic-agent context build --task <ID>` — Build full context bundle
- `agentic-agent skills ensure` — Ensure skills/rules are set up

## Capabilities
- You can directly edit files. Use `Edit` for surgical changes, `Write` for new files.
- You can run CLI commands via `Bash`. Prefer `agentic-agent` commands over manual file manipulation.
- When claiming a task, the CLI auto-records your git branch and timestamp.
- After completing, the CLI captures associated commits automatically.

## Key Files
- `.agentic/tasks/backlog.yaml` — Available tasks
- `.agentic/tasks/in-progress.yaml` — Current work
- `.agentic/context/global-context.md` — Project overview
- `agnostic-agent.yaml` — Project configuration

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/javierhbr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
