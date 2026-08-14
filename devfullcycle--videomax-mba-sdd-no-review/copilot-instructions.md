## videomax-mba-sdd-no-review

> - **Next.js port:** the default port `3000` is usually in use — pick the first free port starting at `3001` and set `APP_PORT` in `.env.local`.

## Environment

- **Next.js port:** the default port `3000` is usually in use — pick the first free port starting at `3001` and set `APP_PORT` in `.env.local`.
- **Postgres:** use the default port `5432`. Reuse any Postgres container that is already running; if none is running, start one using the repository's `docker-compose.yml`.
- **Per-session database:** inside the container, create a fresh database named `videomax_claude_<short-task-slug>` (e.g., `videomax_claude_f02_auth`). Never run migrations against existing databases that are not yours.
- **Data removal:** only remove databases/schemas you created in this session (those with the `videomax_claude_` prefix). Any other database belongs to the user or another session.
- **`.env.local`:** before starting the environment, copy `.env.example` to `.env.local` and update `APP_PORT` and the database name in `DATABASE_URL`.
- **`launch.json`:** if you create it, use the same `APP_PORT` as `.env.local`. Never commit `launch.json` (it is already in `.gitignore` — verify).

## Development

- Load the `next-best-practices` skill at the start of any session that will touch code (TS/TSX, routes, server actions, Prisma). 

## Testing

- **Unit tests (Vitest):** run whenever you change production code touched by the task.
- **E2E / frontend tests:** required when an acceptance criterion depends on DOM, navigation, forms, or visual behavior (anything phrased as "user can click / see / submit / navigate"). Criteria that are purely API or DB do not require E2E.
- **How to run E2E:** try via `launch.json` with `preview_start`. If that fails, load the `playwright-cli` skill and run in headless mode.
- **Before running E2E:** make sure the session's database exists (migrations applied) and the dev server is running on `APP_PORT`.
- **Video Sample:** If you need a video sample for any type of test or simulation, use the `video-sample/video-test.mkv`.

## Git

- After resolving merge or rebase conflicts, re-run unit tests for the affected area. If any acceptance criterion impacted by the conflict requires frontend interaction, also run the corresponding E2E tests before finishing.

---
> Source: [devfullcycle/videomax-mba-sdd-no-review](https://github.com/devfullcycle/videomax-mba-sdd-no-review) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
