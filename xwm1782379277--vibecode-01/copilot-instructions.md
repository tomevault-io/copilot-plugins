## vibecode-01

> This file is intentionally mirrored in `CODEX.md`. Keep both files identical so Claude Code and Codex receive the same project guidance.

# Vibecode-01 Agent Notes

This file is intentionally mirrored in `CODEX.md`. Keep both files identical so Claude Code and Codex receive the same project guidance.

## Project Shape

- `frontend/`: Vite + React + TypeScript + TailwindCSS.
- `backend/`: Express + TypeScript + Prisma + SQLite.
- `docs/runbook.md`: the canonical setup and run guide for a new machine.

Before running or modifying the project on a new computer, read `docs/runbook.md` first.

## Current Product Direction

This project is becoming a personal AI workbench:

- Account login/register/logout.
- Apple-style Liquid Glass frontend.
- Dashboard entry point.
- Read-only Obsidian/GitHub knowledge vault integration.

The knowledge vault sync currently pulls Markdown from:

```txt
https://github.com/xwm1782379277/knowledge-vault
```

## Local Data Rules

Do not commit local runtime data:

- `*.db`
- `*.db-journal`
- `.env`
- `.env.local`
- `node_modules/`
- `dist/`
- `*.log`

SQLite data is local per machine. New machines must create their own database and sync their own vault data.

## Setup Checklist

Use `docs/runbook.md` for exact commands. The short version:

1. Install dependencies in `backend/` and `frontend/`.
2. Generate Prisma client.
3. Initialize SQLite with Prisma migrations.
4. If Prisma schema engine fails on Windows, use the Python sqlite fallback from `docs/runbook.md`.
5. Start backend.
6. Start frontend.
7. Register a local user.
8. Open Knowledge page and click GitHub sync.

## Ports

Defaults:

- Backend: `http://localhost:3001`
- Frontend: `http://localhost:5173`

If ports are occupied:

- Set backend `PORT`, for example `3002`.
- Set frontend `BACKEND_URL`, for example `http://localhost:3002`.
- See `docs/runbook.md` for PowerShell and macOS/Linux examples.

## Vault Design Constraints

Do not hard-code the current vault folder structure as business logic.

The vault may change over time. Treat folder names like `daily/`, `topics/`, and `templates/` as configurable signals, not permanent rules.

The durable model is:

- Markdown files are source records.
- Frontmatter is stored as JSON.
- Raw content is preserved.
- Tags, headings, tasks, and wikilinks are derived indexes.
- Unresolved wikilinks are valid sync warnings, not fatal errors.

## Verification

Before handing off code changes, run:

```bash
cd backend
npm run build

cd ../frontend
npm run build
```

If Vite requires a newer Node.js than the system default, use Node.js 20.19+ or Node.js 22/24.

## Git Notes

The repository remote is:

```txt
https://github.com/xwm1782379277/vibecode-01.git
```

Prefer small, meaningful commits. Do not include generated local databases or dependency folders.

---
> Source: [xwm1782379277/vibecode-01](https://github.com/xwm1782379277/vibecode-01) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
