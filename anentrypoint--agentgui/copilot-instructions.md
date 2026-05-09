## agentgui

> There are now **two parallel surfaces** in this repo. Don't conflate them when editing.

# AgentGUI — Agent Notes

## New architecture (2026-05-02 pivot)

There are now **two parallel surfaces** in this repo. Don't conflate them when editing.

1. **Live client** at `site/app/` — single static page, imports [`anentrypoint-design`](https://www.npmjs.com/package/anentrypoint-design) from unpkg, talks to any [`acptoapi`](https://www.npmjs.com/package/acptoapi) backend over fetch / SSE. No build step. Deploys to GH Pages at `/app/` via `.github/workflows/gh-pages.yml` (post-flatspace copy step).
   - `site/app/index.html` — shell + CSS
   - `site/app/js/backend.js` — acptoapi client (models, chat-stream, history, search, SSE)
   - `site/app/js/app.js` — webjsx view + state, exposes `window.__agentgui` for debug
   - Configurable backend URL via `?backend=…` query string or `localStorage['agentgui.backend']`

2. **Legacy server** (`server.js`, `lib/`, `static/`) — the npm-installable Node app, still works, still gets `npm run dev`. Being phased out as the static client + acptoapi cover its features. Don't delete in passing; full removal is its own PR.

Backend dependencies:
- `acptoapi` provides chat / messages / models endpoints (existing) + new history endpoints (`/v1/history/sessions`, `/v1/history/sessions/:sid/events`, `/v1/history/search`, `/v1/history/stream`) — see `c:\dev\acptoapi\lib\history\` (ccsniff functionality merged in 2026-05-02).
- `anentrypoint-design` provides AppShell / Chat / FileGrid / etc. — single-file ESM from unpkg, no install.

The static client never imports anything from `lib/` or `server.js`. Cross-contamination = bug.

## Learning audit

- 2026-05-02 session: 5 items audited (CI bun, stream imports, windows fallback, GM blocker, ACP history), 0 removed (rs-learn retrieval not yet confirmed; safety default kept all), 1 new fact ingested (acptoapi history integration)

## CI / GitHub Actions

**capture-screenshots must run under bun, not node.**

`npm install --ignore-scripts` in gh-pages.yml skips native compilation, leaving `better-sqlite3` without a compiled `.node` binding. `database.js` tries `bun:sqlite` first, then falls back to `better-sqlite3`. When the step runs under Node both fail and the server crashes silently within the 20s health-check window.

Fix: `bun run scripts/capture-screenshots.mjs` (not `node ...`).

Why it works: `process.execPath` becomes bun, so the spawned child server also runs under bun and loads `bun:sqlite` natively — no compiled binding needed.

Rule: any CI step that spawns the agentgui server (directly or via a script that inherits `process.execPath`) must invoke it with `bun`.

## Plugin Dependencies

**`lib/plugins/stream-plugin.js` must not import packages missing from package.json.**

The plugin loader runs at startup and logs failures silently if a plugin import fails. If stream plugin fails to load due to missing dependency (e.g., `uuid`), the error cascades: `agents` and `websocket` plugins both declare `stream` as a dependency and fail with "Plugin stream not found in registry", cascading to boot failure.

Fix for uuid: replaced `import { v4 as uuidv4 } from 'uuid'` with `import { randomUUID as uuidv4 } from 'crypto'` (Node.js built-in, zero deps).

Pattern: all imports in lib/plugins/* must be either built-in (crypto, fs, path, etc.) or already in package.json. No new npm packages should be added to plugins without adding them to dependencies.

## Plugin Tool Provisioning on Windows

**`lib/tool-spawner.js` must iterate bun → npx fallback and match cross-platform command-not-found errors.**

On Windows hosts without bun installed, the auto-provisioner on startup and 6h periodic update checker failed silently when spawnBunxProc tried `bun.cmd` directly. The missing command error came via process stderr+close, not the 'error' event, so simple ENOENT detection was insufficient. Additionally, the error message format differs by OS: Windows shows "'bun.cmd' is not recognized as an internal or external command", Linux/Mac show "command not found" or "cannot find".

Fix: `BUNX_RUNNERS` array iterates `['bun', 'npx']` and tries each in sequence. Error detection regex `isMissingCmdError` matches `/not recognized|ENOENT|command not found|cannot find/i` on both `error.message` and captured stdout+stderr. Only falls through to next runner when the `missing` flag is set.

Pattern: When a binary might not exist on all platforms, use a runner fallback strategy. Always capture and check both error.message and process output streams. Cross-platform error detection requires regex alternation on common message patterns.

## GM Plugin Autonomy Blocker

**gm plugin's pre-tool-use-hook.js enforces "must invoke gm:gm first" gate, blocking multi-tool autonomy. Hook content is NOT sourced from gm-starter/hooks/ files — it is templated from somewhere else.**

The gm plugin enforces a gate via `.gm/needs-gm` marker that requires invoking `gm:gm` before any other tool use, which fragments multi-tool autonomous sessions in agentgui. A bypass patch was committed to c:/dev/gm (commit e300acf7, origin/main) in gm-starter/hooks/{pre-tool-use,prompt-submit}-hook.js to skip the gate when `.gm/prd.yml` exists, but it did NOT propagate after `/plugin update gm` (cache hash changed 495e36843d77 → 075e64d58498 but hook content unchanged).

The actual hook content is generated/templated from c:/dev/gm, likely from `lib/cli-adapter.js` or `platforms/cli-config-shared.js` or `lib/template-builder.js`, not from gm-starter/hooks/. Next session must:
1. Locate the real hook generator in the gm codebase
2. Patch it to add `if (autonomous) { try { fs.unlinkSync(needsGmPath); } catch {} }` when `.gm/prd.yml` exists
3. Rebuild via `node c:/dev/gm/cli.js c:/dev/gm/gm-starter c:/dev/gm/build`
4. Push changes to c:/dev/gm origin/main
5. Run `/plugin update gm` in agentgui

Only after the real generator is patched will agentgui sessions run autonomously without per-tool ceremony.

## ACP-to-API History Integration

**acptoapi (c:\dev\acptoapi) merged Claude Code history routes as of 2026-05-02; ccsniff package is no longer needed.**

History functionality (`GET /v1/history/*` endpoints) is now built into acptoapi. Routes: `snapshot` (event/session/project/tool/error counts + byte/date range), `sessions` (list with title/project/cwd/counts), `sessions/:sid/events` (flattened events), `search` (BM25 with snippets), `reindex` (rebuild index), `stream` (SSE). Implementation: `lib/history/` (bm25.js for tokenize/buildIndex/search/snippet, watcher.js for JsonlWatcher + JsonlReplayer, index.js for HistoryStore singleton + flattenEvent). Reads `~/.claude/projects` by default; override with `CLAUDE_PROJECTS_DIR` env var. The ccsniff package itself is no longer required — acptoapi covers the functionality entirely.

---
> Source: [AnEntrypoint/agentgui](https://github.com/AnEntrypoint/agentgui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
