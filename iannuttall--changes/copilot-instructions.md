## changes

> - This app watches pages and stores changes in SQLite.

# Agent Notes

- This app watches pages and stores changes in SQLite.
- The API is local only and used by the TUI.

## Do first

- Run `git log -n 5 --oneline`.
- Use `rg` to search.
- Read files you touch in chunks of 250 lines or less.
- If the user gives URLs, fetch them.
- Run `bun typecheck` and `bun test` before handoff.

## Rules

- Do not use `any`.
- Do not add new deps unless asked.
- Do not hand-write SQL.
- Keep changes small and focused.

## Database changes

- Backup first:

```bash
mkdir -p data/backups
cp "${DATABASE_PATH:-./data/changes.db}" "data/backups/changes.$(date +%Y%m%d-%H%M%S).db"
```

- Edit `src/db/schema.ts`.
- For local work, use `bun db:push` until the schema is ready for a migration.
- Run `bun db:generate`.
- Run `bun db:migrate`.

## Rollback a DB change

- Stop the app and workers.
- Restore backup:

```bash
cp "data/backups/<backup>.db" "${DATABASE_PATH:-./data/changes.db}"
```

- Revert `src/db/schema.ts`.
- Delete the migration files in `drizzle/`.
- Run `bun typecheck` and `bun test`.

## Background work

- Use the `monitor.check` job.
- Use `job_locks`.
- Write `job_events`.
- Do not block HTTP handlers.

## Logs

- Format: `[YYYY-MM-DD HH:MM:SS] message`.
- No emojis.
- Log only real events or errors.

## TUI

- Give human steps like `bun changes` -> menu path.

## Commands

- `bun install`
- `bun dev`
- `bun start`
- `bun typecheck`
- `bun test`
- `bun build`
- `bun setup:vps`
- `bun changes --remote`

## VPS setup

- `bun setup:vps` creates the server, installs the app, and saves the host alias.
- After it finishes, use `bun changes --remote` from your computer.
- Do not ask users to log in to the server for normal use.

## API (TUI only)

- `GET/POST /api/monitors`
- `GET/PUT/DELETE /api/monitors/:id`
- `POST /api/monitors/:id/check`
- `GET /api/monitors/:id/changes`
- `GET /api/changes`
- `GET /api/changes/:id`
- `GET /api/job-events`
- `POST /api/check-all`
- `GET /health`

---
> Source: [iannuttall/changes](https://github.com/iannuttall/changes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
