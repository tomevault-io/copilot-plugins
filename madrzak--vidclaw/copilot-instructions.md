## vidclaw

> - `src/`: React frontend. Feature UI lives under `src/components/*` (for example `src/components/Kanban/Board.jsx`), shared hooks in `src/hooks/`, and utilities in `src/lib/`.

# Repository Guidelines

## Project Structure & Module Organization
- `src/`: React frontend. Feature UI lives under `src/components/*` (for example `src/components/Kanban/Board.jsx`), shared hooks in `src/hooks/`, and utilities in `src/lib/`.
- `server/`: Express backend (`server/index.js`, `server/routes.js`) with shared server helpers in `server/lib/`.
- `data/`: Runtime JSON state (`tasks.json`, `activity.json`, `heartbeat.json`).
- `docs/`: Static docs site assets.
- `dist/`: Frontend build output from Vite.
- `API.md`: Source of truth for API endpoint documentation.

## Build, Test, and Development Commands
- `npm run start`: Start backend on `127.0.0.1:3333`.
- `npm run dev`: Start Vite frontend with `/api` proxy to `http://localhost:3333`.
- `npm run build`: Production frontend build into `dist/`.
- `npm run preview`: Preview the built frontend.
- `./setup.sh`: One-time setup (install deps, build, and install service).
- `./update.sh`: Pull latest changes, rebuild, and restart service.

Local dev pattern:
```bash
# terminal 1
npm run start
# terminal 2
npm run dev
```

## Coding Style & Naming Conventions
- Use ES modules and single quotes.
- Frontend style in `src/`: 2-space indentation, PascalCase component files, hooks prefixed with `use` (example: `useSocket.jsx`).
- Backend style in `server/`: keep existing semicolon-based style and route organization.
- No formatter/linter config is enforced yet; match the surrounding file’s style and keep diffs focused.

## Testing Guidelines
- No automated test suite is configured yet.
- Minimum validation before opening a PR:
  - `npm run build`
  - Start app and smoke-test key flows (Kanban, calendar, file browser, settings).
  - Spot-check API health, for example: `curl http://127.0.0.1:3333/api/tasks`.
- If adding tests, use `*.test.js` / `*.test.jsx` naming and colocate near the module or in a dedicated `tests/` folder.

## Commit & Pull Request Guidelines
- Recent history uses short imperative subjects (for example `Fix ...`, `refactor: ...`); follow that pattern and avoid empty messages like `no message`.
- Keep commits scoped to one logical change.
- PRs should include:
  - Clear summary and rationale
  - Linked issue/task
  - UI screenshots/GIFs for frontend changes
  - `API.md` updates when endpoints or payloads change

## Security & Configuration Tips
- Keep the service bound to localhost (`127.0.0.1:3333`); use SSH tunneling for remote access.
- Prefer `OPENCLAW_DIR` and `OPENCLAW_API` environment variables over hardcoded machine-specific paths.

---
> Source: [madrzak/vidclaw](https://github.com/madrzak/vidclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
