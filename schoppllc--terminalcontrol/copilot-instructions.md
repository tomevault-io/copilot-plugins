## terminalcontrol

> A single browser window holding a **grid of real terminals** (one `node-pty` shell

# FleetView — agent guide

A single browser window holding a **grid of real terminals** (one `node-pty` shell
per box), each typically running `claude`. Click a box to zoom it; when a Claude
needs approval or finishes, its box glows/dings and a chip appears in the top bar.
Local-only tool: a Node server on `localhost` spawns the shells.

## Prerequisites (on any machine that runs this)
- **Node.js** (project uses ESM; `node-pty` ships prebuilds for macOS/Linux x64/arm64).
- **tmux** — *strongly recommended*. Terminals run inside `fleet_<id>` tmux sessions
  so they survive server restarts/crashes. Without it, shells die with the server.
  Install: `brew install tmux` / `apt install tmux`.
- **curl** — the Claude Code hooks use it to phone home. Usually preinstalled.
- **`claude`** (Claude Code CLI) — for the per-box agents (untick "run claude" for a
  plain shell).
- macOS or Linux. `$SHELL` should be set (falls back to `/bin/zsh`; see
  `server/pty-manager.js`).

## Run / build / test
- `npm install` — installs deps; a postinstall (`scripts/fix-pty-helper.js`) chmods
  the node-pty spawn helper.
- `npm run doctor` — preflight: checks tmux/curl/claude, prints the exact per-platform
  install command, and can install tmux (interactive, or `FLEET_AUTO_INSTALL=1`).
  `scripts/preflight.js` also exports `preflight()`, which `server/index.js` calls
  FIRST at startup (warn-only) so missing-dep warnings are the most prominent output.
- `npm run go` — build client + start server (this also **restarts** it). Re-runs
  Vite every time; use `npm start` for a plain restart without rebuilding.
- `npm run service:install` / `service:uninstall` — opt-in auto-start on login
  (launchd on macOS, systemd --user on Linux). `scripts/install-service.js` captures
  a working PATH into the service env (launchd/systemd give a minimal PATH, which
  would silently break tmux/claude/curl). A reboot returns the *server*, not live
  sessions (reboot kills tmux → fresh shells). The server exits readably on
  EADDRINUSE so the service + a manual `npm start` don't fight silently.
- `npm run build` — build the client bundle only (vite → `dist/`). Safe: does NOT
  restart the server; a running server serves the new bundle to the next page load.
- `npm start` — start the server without rebuilding.
- `FLEET_PORT=5000 npm run go` — different port. `FLEET_HOST=0.0.0.0` — bind beyond
  loopback (see Security).
- **Typecheck: `npm run typecheck`** (= `tsc --noEmit`). ⚠️ `vite build` uses
  esbuild and does NOT typecheck — always run `tsc` separately. CI runs typecheck
  + build + `node --check` (`.github/workflows/ci.yml`).
- **No test framework / no committed tests.** Verify changes with throwaway Node
  scripts that exercise modules in isolation (temp state files, unique ports,
  isolated tmux sockets), run ad-hoc with `node`; don't commit them.
  When testing HTTP in-process, use `fetch` (async) — a synchronous `spawnSync`
  curl against an in-process server **deadlocks** the event loop.

## Layout
| Path | Responsibility |
|------|----------------|
| `server/pty-manager.js` | owns panes: spawn/kill shells, tmux sessions, scrollback, attention, **dormant recovery**, persistence |
| `server/layout-store.js` / `task-store.js` | named layouts / the global task tree |
| `server/setup-hooks.js` | idempotently install the guarded Claude Code hooks |
| `server/index.js` | HTTP + WS wiring, REST, hook endpoints, static client |
| `client/src/terminal.ts` | one xterm box bound to one PTY socket (auto-reconnects) |
| `client/src/main.ts` | grid, zoom, tray, drag, picker, attention queue, control socket |
| `client/src/tasks.ts` | the task-list sidebar (tree, drag, debounced save) |
| `client/src/tab.ts` | browser-tab title + favicon attention indicator |

## Key invariants / model (don't break these)
- **tmux durability:** each pane is a detached `fleet_<id>` tmux session on a STABLE
  socket `~/.fleetview/tmux-<port>.sock` (NOT tmux's default `/tmp` socket, which the
  OS sweeps). Sessions outlive the server; on boot `PtyManager._restore` reattaches.
  The liveness check retries (a transient miss right after restart must NOT demote a
  live pane). Per-pane `sock` is tracked so old default-socket sessions still reattach.
- **Dormant recovery:** when a pane's tmux session dies (sleep/crash) or a layout
  "Replace" sets it aside, the pane goes **dormant** (metadata kept) and shows in the
  recovery bar — never silently dropped. `respawn` reattaches if alive, else spawns fresh.
- **Security:** the server runs shells with **no auth** → reaching the port = RCE.
  It binds `127.0.0.1` by default. `FLEET_HOST` opts into wider binding (warns).
  Never expose it publicly; front remote access with a tunnel/VPN (SSH, Tailscale).
- **Hooks:** `setup-hooks.js` merges three guarded hooks into `~/.claude/settings.json`
  on every server start: **Notification**→needs-you, **Stop**→done, **UserPromptSubmit**
  →pins the prompt. All no-op unless `$FLEET_PANE_ID` is set (i.e., inside a FleetView
  shell). Stripped-and-re-added idempotently (marker = `FLEET_PANE_ID`).
- **WebSockets:** `/term?pane=` (per box, reconnects after sleep) and `/control?session=`
  (per browser window; grid events + initial panes/dormant/tasks snapshot).
- **Tasks** are one global tree in `tasks.json`, broadcast to ALL windows on change.

## Gotchas
- `index.js` hardcodes `sessions.json`/`layouts.json`/`tasks.json` under repo ROOT.
  **Do NOT boot a real server for a test** — it reads/writes those live files (and
  reattaches real tmux sessions). Test modules directly with isolated temp files.
- `sessions.json`, `layouts.json`, `tasks.json`, `dist/`, `node_modules/` are
  gitignored local state — they regenerate; a fresh clone starts empty.
- Restarting the server reattaches tmux sessions, but an already-open browser tab
  must be refreshed to load new client code.

## Conventions
- Small, single-purpose modules; match the surrounding style and comment density.
- Commit only when asked; branch off `main` for non-trivial work.

---
> Source: [schoppllc/Terminalcontrol](https://github.com/schoppllc/Terminalcontrol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
