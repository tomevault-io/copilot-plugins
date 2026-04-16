## grind

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

```bash
bun install              # Install dependencies
bun run dev -- <args>    # Run CLI in dev mode (e.g., bun run dev -- new idea "test")
bun run build            # Compile to single binary named 'grind'
bun run typecheck        # Type check (tsc --noEmit)
bun run test             # Run tests (jest)
```

Tests use Jest with `ts-jest`. Test files live in `tests/` mirroring the `src/` structure (e.g., `tests/utils/config.test.ts`).

## Architecture

**Runtime:** TypeScript on Bun. Compiles to a single standalone binary via `bun build --compile`. Uses Bun's shell (`$` from "bun") for running git commands.

**CLI framework:** Commander.js. Entry point is `src/index.ts` which wires all commands. Commands are organized as async functions in `src/commands/`, one file per command group.

**Core concept — git worktrees as project isolation:** The central abstraction is using a bare git repo (`.grind.repo.git/`) with worktrees to give each project its own directory and branch while sharing history. The workspace layout is:

```
workspace-root/
├── .grind.repo.git/    # bare repo (shared git database)
├── grind/              # main worktree ("main" branch) — holds ideas, project configs, .grind.json
└── project-name/       # project worktree (one per project, own branch)
```

**Workspace discovery:** `src/utils/workspace.ts` walks up the directory tree looking for `.grind.repo.git` to locate the workspace root. All commands rely on this to find configs and project data regardless of where the user runs `grind`.

**Configuration hierarchy:**
- Workspace level: `grind/.grind.json` — billing defaults (rate, rounding)
- Project level: `grind/projects/{name}/.project.json` — per-project metadata, time sessions, billing overrides

**Time tracking:** Sessions are stored as start/end ISO timestamps in `.project.json`. Duration is calculated in seconds and rounded (quarter-hour/half-hour/hour) per `src/utils/time.ts`. Sessions can be marked as invoiced.

**Key types:** All domain types live in `src/types/index.ts` — `ProjectConfig`, `Session`, `GrindConfig`, `BillingConfig`, `RoundTo`, `ProjectType`, `NewCommandOptions`, `ProfessionalInfo`, `ClientInfo`. Project types are defined as a const array (`PROJECT_TYPES`); extend that array to add new types. `ProjectConfig` includes optional `client` (ClientInfo), `repo` (git remote URL), and `longTerm` (boolean, shows ★ icon) fields. `GrindConfig` includes optional `my` (ProfessionalInfo), `currency`, and `paymentTerms` fields for invoicing.

**Utilities:**
- `src/utils/git.ts` — git command wrappers using Bun shell; `getActiveWorktrees()` lists project worktrees; uses `execSync` for interactive editor spawning
- `src/utils/config.ts` — JSON config file read/write
- `src/utils/files.ts` — filesystem helpers
- `src/utils/workspace.ts` — workspace/worktree location logic; `requireWorkspace()` returns `{ workspaceRoot, mainWorktree, bareRepo }` or exits with error
- `src/utils/repo.ts` — parses git remote URLs (SSH/HTTPS) for GitHub and GitLab into `{ platform, repo }` format
- `src/utils/time.ts` — timestamp generation, duration calculation, rounding, `timeAgo`/`formatDate` helpers
- `src/utils/colors.ts` — shared ANSI color constants (`DIM`, `RED`, `GREEN`, `RESET`)

**Invoicing:** `src/commands/invoice.ts` generates both markdown and PDF (via pdfkit) invoices from tracked time sessions.

## Conventions

- Async/await throughout; file I/O uses `node:fs/promises`
- Config files are JSON with 2-space indentation
- Ideas are timestamped markdown files (`YYYYMMDDHHmmss.md`) in `grind/ideas/`
- Rejected ideas are prefixed with `rejected-` in the filename
- Error handling uses early-exit pattern with `console.error()` + `process.exit(1)`
- The `config` command (`src/commands/config.ts`) supports `configList`, `configGet`, and `configSet` for both workspace-level (`-g`) and project-level configs
- Short aliases exist: `grind ideas` = `grind list ideas`, `grind projects` = `grind list projects`
- All `path` imports use the `node:path` prefix (Node.js built-in module convention)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/leebrandt) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
