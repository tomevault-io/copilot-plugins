## beadsui

> Feature-rich WebUI for the Beads issue tracker — pnpm monorepo with React + Hono.

# Project Instructions for AI Agents

## Project: BeadsUI

Feature-rich WebUI for the Beads issue tracker — pnpm monorepo with React + Hono.

## Stack

- **Monorepo**: pnpm workspaces
- **Frontend** (`packages/web`): React 18 + TypeScript + Vite + Tailwind CSS v4 + TanStack Query + React Flow + cmdk
- **Backend** (`packages/api`): Hono + Node.js + TypeScript + tsx

## Build & Test

```bash
# First time: cp .env.example .env and set BEADS_WORKSPACE
pnpm install
pnpm dev            # Starts web (port 5173) + api (port 3001)
pnpm -F api dev     # API only
pnpm -F web dev     # Web only
pnpm build          # Production build
pnpm typecheck      # Type-check all packages
```

## Architecture Overview

```
packages/
  api/    Hono REST API — wraps bd CLI via BEADS_WORKSPACE env
  web/    React SPA — IssueList, Kanban, DependencyGraph, CommandPalette
```

## Conventions & Patterns

- TypeScript strict + exactOptionalPropertyTypes
- All API responses: `{ data }` success, `{ error }` failure
- `BEADS_WORKSPACE` env var points API at target beads workspace
- Tailwind CSS v4 with `@tailwindcss/vite` plugin (`@import "tailwindcss"` in CSS)

<!-- BEGIN BEADS INTEGRATION v:1 profile:minimal hash:ca08a54f -->
## Beads Issue Tracker

This project uses **bd (beads)** for issue tracking. Run `bd prime` to see full workflow context and commands.

### Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --claim  # Claim work
bd close <id>         # Complete work
```

### Rules

- Use `bd` for ALL task tracking — do NOT use TodoWrite, TaskCreate, or markdown TODO lists
- Run `bd prime` for detailed command reference and session close protocol
- Use `bd remember` for persistent knowledge — do NOT use MEMORY.md files

## Session Completion

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **PUSH TO REMOTE** - This is MANDATORY:
   ```bash
   git pull --rebase
   bd dolt push
   git push
   git status  # MUST show "up to date with origin"
   ```
5. **Clean up** - Clear stashes, prune remote branches
6. **Verify** - All changes committed AND pushed
7. **Hand off** - Provide context for next session

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - that leaves work stranded locally
- NEVER say "ready to push when you are" - YOU must push
- If push fails, resolve and retry until it succeeds
<!-- END BEADS INTEGRATION -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JannikRoesch) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
