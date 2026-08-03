## test6

> Guidance for Claude Code working in this repo.

# CLAUDE.md

Guidance for Claude Code working in this repo.

## Project
A minimalist **productivity website** (tasks, time tracking, goals, notes) with a dark/light theme toggle and responsive layout. Static vanilla-JS frontend backed by a Node + Express + SQLite API.

## Stack
- **Backend:** Node.js (ESM) + Express, SQLite via the **built-in `node:sqlite`** module (`DatabaseSync`). No native deps — `better-sqlite3` will NOT build here (no Python/node-gyp).
- **Frontend:** plain HTML5/CSS3/vanilla JS. No build step, no framework.
- **Tests:** jsdom integration harness that boots the server and drives the real UI.

## Layout
```
index.html / styles.css / script.js   # static frontend
server/index.js                        # Express app, mounts routes, serves static
server/db.js                           # SQLite connection + schema (singleton)
server/routes/{tasks,timers,goals,notes}.js  # REST routers
server/data/app.db                     # runtime DB (gitignored)
test/frontend.test.mjs                 # integration test suite
```

## Commands
- `npm start` — start backend on http://localhost:3000 (then open `index.html`, or visit http://localhost:3000/)
- `npm test` — boot server + run jsdom integration suite (self-contained)
- `npm run dev` — start with `--watch`

## Conventions (see `.claude/skills/project-conventions/SKILL.md` for full rules)
- ESM only. Use `node:sqlite`, not `better-sqlite3`.
- Backend validation is manual: trim, `Number.isFinite`, range checks → `400 { error }`.
- Frontend talks to the API only through the `fetchJSON` helper in `script.js` (auto-selects `http://localhost:3000/api` vs relative `/api`).
- **Security:** render backend/user strings into `innerHTML` only via `escapeHtml(...)`; backend uses parameterized queries only.
- UI state (theme, active tab) in `localStorage`; data in the SQLite backend.

## API
REST under `/api/{tasks,timers,goals,notes}` with full CRUD; `/api/health` for liveness. CORS is enabled so `index.html` opened via `file://` works against the running server.

## Claude Code automations installed in this repo
- **Hooks** (`.claude/settings.json`): PreToolUse blocks edits to `*.db`/`server/data/`; PostToolUse runs `npm test` after edits to `server/**`, `script.js`, `styles.css`, or `test/**`.
- **Skills:** `gen-test` (user-invocable, `/gen-test`), `project-conventions` (Claude-only).
- **Subagents:** `api-reviewer`, `xss-checker`.

---
> Source: [aaronlam1234/test6](https://github.com/aaronlam1234/test6) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
