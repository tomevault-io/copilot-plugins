## no-wait-skipped

> Prevent Cursor Agent from skipping waits and losing task context


# No Wait Skipped Rule

## Core Rules

1. Do not use parallel Explore tasks unless explicitly requested.
2. Do not spawn sub-agents for repository exploration.
3. Execute repository analysis sequentially.
4. If a tool, search, file read, or terminal command is stopped, do not continue as if it succeeded.
5. If "Wait skipped" happens, stop the task and write the failure reason to specs/current-agent-log.md.
6. Never produce an implementation plan based on incomplete exploration.
7. Do not ask the user to paste the full requirement again.
8. Always resume from specs/current-agent-state.md.

## Required Files

For every non-trivial task, use:

- specs/current-agent-task.md
- specs/current-agent-state.md
- specs/current-agent-log.md

## Execution Flow

1. Read specs/current-agent-task.md.
2. Read specs/current-agent-state.md.
3. Execute only the first pending stage.
4. Mark completed stages as done.
5. Mark failed stages as failed.
6. After each stage, update specs/current-agent-state.md.
7. Do not repeat stages marked as done.

## Terminal Rules

1. Do not run long-running dev servers.
2. Do not run watch mode.
3. Do not use paginated commands.
4. Use non-interactive commands only.
5. Prefer:
   - git --no-pager log --oneline -n 20
   - git diff --stat
   - pnpm exec tsc --noEmit
   - pnpm run lint
6. Avoid:
   - git log
   - pnpm dev
   - npm run dev
   - docker compose up
   - interactive installers

---
> Source: [loudon84/ai-os-desktop](https://github.com/loudon84/ai-os-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
