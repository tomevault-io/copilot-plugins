## agent-center

> Guidance for AI agents working in this repository.

﻿# CLAUDE.md

Guidance for AI agents working in this repository.

## What this is

Agent Center / Antigravity Manager — a Wails v2 desktop app (Go backend + Svelte 5
frontend) that orchestrates AI coding sub-agents. Users describe a project → an AI
decomposes it into Kanban tasks → the orchestrator dispatches tasks to sub-agents
(Claude CLI or Antigravity/Gemini CLI) that run against a chosen workspace folder.

## Build & test

```bash
go build ./...            # backend
go vet ./...
go test ./backend/...     # backend tests
cd frontend && npm ci && npm run build   # frontend
wails dev                 # full app hot-reload (needs Wails CLI)
wails build -platform windows/amd64 -clean
```

CI runs the first block on every push/PR (`.github/workflows/ci.yml`).

## Architecture map

- `app.go` — Wails-bound `App` methods (the frontend API surface).
- `backend/orchestrator/` — parallel worker pool. `orchestrator.go` dispatches
  dependency-satisfied tasks up to `maxConcurrency`, each in its own goroutine with
  a cancellable context (Stop/Retry per task). `dispatcher.go` matches tasks→agents.
- `backend/cli/` — CLI runners. `claude.go` uses `--output-format stream-json` to
  parse REAL token usage/cost and surface tool calls; `antigravity.go` handles
  Gemini with key-pool rotation on 429. Both honor `TaskTimeout()`.
- `backend/database/` — SQLite repos (agents, tasks, memory).
- `frontend/src/components/pages/` — main UI; `KanbanView.svelte` is the board +
  Task Inspector drawer; `TaskBoardPage.svelte` is the container + AI Plan Builder.
- `frontend/src/lib/stores/appState.ts` — global stores (`tasksStore`, `agentsStore`,
  `taskLogsStore`).

## Before changing anything that looks redundant

`docs/ARCHITECTURE_DECISIONS.md` lists the decisions that read as unnecessary
detours but are load-bearing — `/c` vs `/k`, the `file:` URL for SQLite, the
dedicated Chrome profile, per-task approval channels, the synchronous webhook
bind. Read the relevant entry before "simplifying" one of them.

## Two frontends, one backend

`app.go` (Wails) and `backend/tui/task_actions.go` (TUI) are parallel adapters
over the same services. A capability must use the **same method name on both**;
`backend/contract` fails the build when it does not. TUI-only methods need an
entry in `tuiOnlyMethods` with a reason.

## Critical conventions & gotchas

- **Wails bindings must stay in sync with `app.go`.** After adding/changing an
  `App` method or a struct crossing the boundary, run:

  ```bash
  wails generate module
  ```

  This regenerates `frontend/wailsjs/go/main/App.js`, `App.d.ts` and
  `wailsjs/go/models.ts` from the Go source. Prefer this over hand-editing —
  hand-maintained bindings have repeatedly drifted (missing methods/struct
  fields), which breaks the frontend build or fails silently at runtime. Note
  that a running `wails dev` may briefly delete these generated files while it
  rebuilds; regenerate rather than restoring stale copies.
- **Event bus** (Wails `EventsEmit`/`EventsOn`): `board_updated`, `agent_updated`,
  `log_entry`, `task_log` (per-task, carries `task_id`), `ask_approval` (carries
  `taskId`). `board_updated` is handled centrally in `App.svelte` (single source
  refreshing `tasksStore`) — do not re-add per-component board fetch listeners.
- **Secrets**: never hardcode credentials. GCP OAuth is read from env
  (`AGENT_CENTER_GCP_CLIENT_ID` / `_CLIENT_SECRET`) or `gcp_oauth.json` in the data
  dir (both gitignored).
- Sub-agents run with `--dangerously-skip-permissions` against the workspace — they
  can create/modify files directly. `AutoSnapshot` git-commits before each task so
  the current task's changes stay as an uncommitted diff (`GetWorkspaceDiff`).
- Keyword→agent matching uses `keywordMatch` (word boundaries) — never plain
  `strings.Contains` for short tags, or "DATABASE" matches "BA".

## Reporting a defect you found

Do not write findings straight into a PR comment. State them so they can be
proven wrong:

```bash
agent-center-claim --checks                    # what you may point at
agent-center-claim --host ws://HOST:9111 --session ID --token T   --author you/your-agent --provider claude   --subject "backend/cli/process_windows.go:17"   --assert  "cmd.Wait() never returns when the console is visible"   --falsify "console-window-hidden"
```

`--falsify` names an entry in `.agent-center/checks.json`. **A falsifier passes
when your claim is wrong**, so a failing check confirms the defect.

Omit `--falsify` and the claim is recorded as an opinion: it is kept and shown,
but it cannot block a merge. That is deliberate — a finding nobody can check is
a suggestion, and being asked for the command that would disprove it is what
separates a real defect from a confident guess.

Afterwards read `.agent-center/session-<id>/verdict.json` for the result.

### No agent-center-claim on this machine?

Two more doors into a session, in order of friction (the app's Claims page
hands out ready-made copies of all three):

- **Download the tool** (Windows, PowerShell): fetch the published exe into
  `%TEMP%` and run it from there — the release URL is stable across versions:

  ```powershell
  $c = "$env:TEMP\agent-center-claim.exe"; if (!(Test-Path $c)) { curl.exe -L -o $c https://github.com/thydynh03/Agent_Center/releases/latest/download/agent-center-claim.exe }
  ```

- **MCP, nothing downloaded**: the host serves each session as an MCP server;
  register it and the whole protocol becomes tools:

  ```bash
  claude mcp add --transport http agent-center-debate "http://HOST:9111/mcp/SESSION?token=T"
  ```

  Then `join_session` (save the `participant_key` it returns — every later
  call needs it with your author), `list_checks`, `submit_claim` per finding,
  `finish_reporting`, and poll `get_session_state` for verdicts. The collect
  phase is blind and adjudication waits for your `finish_reporting`, but a
  participant silent past the collect window is treated as departed — keep
  polling. See `backend/claims/mcp.go`.

Sessions also carry free-form chat with the other agents and the human
arbiter. Over MCP the loop is `wait_for_chat` (long-poll with `after_seq` =
last seq you saw) → answer with `say` only when a message names you, asks you
something, or disputes your claim → repeat until the phase is `record`. Over
the CLI: `agent-center-claim --listen 15m` streams chat as JSON lines while
`--say "..."` replies. Talk is not evidence — only a falsifier settles a claim.

---
> Source: [thydynh03/Agent_Center](https://github.com/thydynh03/Agent_Center) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-01 -->
