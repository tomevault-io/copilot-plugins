## pokegents

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is pokegents

pokegents is a multi-session Claude Code launcher and session manager. It wraps the `claude` CLI with profile-based configuration, terminal theming (iTerm2 on macOS), session tracking, and Claude Code hook integration. Users run multiple concurrent Claude Code sessions across different projects (e.g. 3 client sessions + 1 personal session simultaneously).

## Architecture

**Entry point:** installed `pokegents`/`pokegent` shims set `POKEGENTS_ROOT`, source `pokegent.sh`, and dispatch to the `pokegent()` function. Users should not need to edit or source shell rc files.

**Code lives in the repo, user data lives in `~/.pokegents/`:**
- `~/.pokegents/profiles/*.json` — per-profile config (cwd, system_prompt, emoji, color, iterm2_profile, add_dirs)
- `~/.pokegents/running/*.json` — active session registry (profile, session_id, pid, tty, display_name)
- `~/.pokegents/status/*.json` — structured state from hooks (state, detail, cwd, timestamp, last_summary)
- `~/.pokegents/history/*.json` — last 5 sessions per profile with timestamps and first-message summaries

**Hooks (in `hooks/`):**
- `status-update.sh` — registered on all Claude Code lifecycle events (PreToolUse, PostToolUse, Stop, SessionStart, SessionEnd, etc.). Writes structured JSON to `~/.pokegents/status/<session_id>.json` with state (running/idle/error/permission/waiting).
- `statusline.sh` — renders profile emoji + title in profile RGB color for Claude's status bar.

**Hook paths are set in `~/.claude/settings.json`** and point to this repo's `hooks/` directory. The `install.sh` script wires this up.

**Session lifecycle:** launch → register in running/ → set terminal tab color/title (iTerm2) → run claude → on exit: cleanup running file, save to history, restore terminal profile.

**Duplicate handling:** if a profile is already running, pokegent prompts to rename existing sessions and name the new one (supporting patterns like 3 concurrent "client" sessions named "pinecone", "int chroma", "int pine").

## Build / Install / Test

```bash
# Install (creates data dirs and CLI shims; no shell rc mutation)
./install.sh

# Developer/source build
POKEGENTS_DEV_BUILD=1 ./install.sh
```

## Roadmap and Goals

The owner wants pokegents to evolve into a full agent orchestration platform. Priority areas:

1. **Hooks integration** — the status-update hook captures events but nothing reads/reacts to the data yet. Goal: richer hook logic (notifications, auto-actions, cross-session triggers).

2. **Dashboard UI** — a viewer for all active agents and their progress. TBD whether terminal TUI or web. The data is already there in `running/` + `status/`. Existing ecosystem references: claude-code-monitor (TUI + mobile web), claude-cockpit (VS Code sidebar), claude-code-ui (web dashboard).

3. **Advanced profile customization** — sub-project support within profiles, profile inheritance, composable config layers (e.g. "client profile + pinecone sub-project overrides").

4. **Git worktree + cross-agent communication** — agents on different worktrees of the same repo sharing context and coordinating. The `-w` flag exists but just passes through to claude. Key patterns from the ecosystem:
   - Claude Code's experimental Agent Teams feature (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`) uses file-based mailbox inboxes with append-and-poll messaging under `~/.claude/teams/`
   - HCOM uses hook-based messaging: `agent → hooks → db → hooks → other agent` with direct messages, broadcasts, @-mentions, collision detection
   - agent-collab-mcp uses an MCP server as coordination layer so agents communicate via tool calls
   - Community consensus: 3-5 concurrent agents is the sweet spot, decomposition quality matters more than agent count

5. **Broader extensibility** — plugin system, MCP server integration, multi-machine coordination (currently an open gap in the ecosystem).

## Key Conventions

- `--dangerously-skip-permissions` is configurable per-profile (`skip_permissions` field) and globally in `~/.pokegents/config.json`
- `POKEGENTS_PROFILE_NAME` env var is set on launch so hooks/statusline can read the active profile
- `POKEGENTS_ROOT` and `POKEGENTS_DATA` env vars are passed through to claude so hooks can reference both repo code and user data
- iTerm2 restore profile is configurable in `~/.pokegents/config.json` (`iterm2_restore_profile`)
- Session IDs are lowercase UUIDs from `uuidgen`

## Deep Architecture Reference

### Hook System (`hooks/status-update.sh`)

**State machine:** `idle → busy → done → needs_input → error`

| Transition | Trigger |
|-----------|---------|
| `* → busy` | UserPromptSubmit, PreToolUse, PostToolUse |
| `busy → done` | Stop, Notification(idle_prompt) when current=busy |
| `* → needs_input` | PermissionRequest only (never from idle_prompt) |
| `* → error` | StopFailure |
| `* → idle` | SessionStart (unless previous state was busy/compacting) |

**Critical rule:** `set -e` is NOT used. Every jq call has `2>/dev/null || fallback`. A crashing hook blocks ALL Claude operations — we learned this from a 7-hour error loop.

**Activity log:** On Stop, appends a 1-liner to `~/.pokegents/activity/{project_hash}.log` with changed files (from `recent_actions`) and summary. On UserPromptSubmit, injects new entries from OTHER agents via `systemMessage`, with file overlap warnings.

**Quick reconciliation (lines 37-68):** Runs on every event except SessionStart/SessionEnd. If no running file exists for the current `$SESSION_ID`, searches by `pokegent_id` field ONLY (never `session_id` — that would steal the original's file during clone). Patches and renames the running file to match.

**Pending message delivery:** On UserPromptSubmit only (not Stop — that caused 4x spam per turn). Checks dashboard API, injects via `systemMessage`. Combined with activity log in a single notification.

### Clone/Fork Lifecycle

**Flow:** `pokegent <profile> --resume <id> --fork-session`

1. pokegent.sh generates fresh UUID for `session_id` and `pokegent_id`
2. Creates running file: `{profile}-{fresh_uuid}.json`
3. Launches: `claude --resume <original_id> --fork-session --name <display_name>`
4. Claude creates a NEW conversation session ID (different from both the fresh UUID and the original)
5. SessionStart fires with the NEW conversation session ID
6. Hook's Pass 1 matches by `pokegent_id` → patches running file's `session_id` and renames file

**Three-pass matching in SessionStart:**
- **Pass 1:** Match `pokegent_id` field only (safe for clones)
- **Pass 2:** Match `session_id` field — ONLY if `POKEGENTS_SESSION_ID` env is empty (legacy sessions)
- **Pass 3:** TTY fallback — ONLY if no pokegents context (prevents TTY collisions)

**Collision guard:** Before renaming a running file, check `[ ! -f "$_NEW_RF" ]`. If the target exists, another agent owns that session ID — leave the file untouched.

### iTerm2 Integration (macOS only)

All iTerm2 code is guarded by `POKEGENTS_HAS_ITERM` checks in `pokegent.sh` and behind `platform_darwin.go` build tags in Go.

- **Tab colors:** Set via iTerm2 escape sequences `\033]6;1;bg;{red|green|blue};brightness;N\a`
- **Tab titles:** Set via `\033]0;TITLE\007` (works on most terminals, not just iTerm2)
- **Per-session sprite icons:** Dynamic Profile with `Custom Icon Path` pointing to sprite PNG
- **Session matching:** Primary by `iterm_session_id` (from `$ITERM_SESSION_ID` env var), fallback by TTY
- **Tab focus from dashboard:** AppleScript finds session by UUID, activates window/tab/session
- **Auto-nudge:** Types "check messages" into idle agent terminals via AppleScript with Ctrl+U (newline NO) prefix

### Naming System

**Two sources of truth (kept in sync):**
1. Running file `display_name` — what the dashboard shows for active agents
2. JSONL transcript `custom-title` — what Claude's resume picker and the "Previous sessions" page shows

**On rename (via dashboard):** Updates running file, JSONL transcript (appends `custom-title` entry), search index, AND iTerm tab title. All four stay in sync.

**On resume:** pokegent.sh reads `custom-title` from the JSONL transcript and uses it as `display_name` in the new running file.

### Dashboard Server (`dashboard/server/`)

**Core loop:**
- File watcher (fsnotify) on `running/` and `status/` dirs — handles Write, Create, Remove, and Rename events
- Transcript poller (2s): backfills context tokens, user prompt, trace for busy agents; detects compaction (token decrease)
- CleanStale: 30s grace period for new files, 3-tier liveness (claude_pid → shell PID → session registry TTY match)
- SSE pushes state updates to all connected frontends

**Persistent maps surviving `rebuildAgents()`:** `contexts` (token usage), `activityFeeds`. These aren't in running/status files, so they'd be lost on rebuild without separate storage.

**Platform abstraction:** `TerminalIntegration` interface with `ITerm2Terminal` (darwin) and `StubTerminal` (!darwin) implementations. Affects: focus, write text, set tab name, clone, resume, close.

**Nudger:** Queues "check messages" for idle agents after message delivery. 2s batch delay, 3s minimum idle time, 10s debounce per agent.

### Dashboard Frontend Layout (`dashboard/web/src/hooks/useLayout.ts`)

**5 layout modes** computed dynamically from window size, agent count, and profile count:

| Mode | When | Cards show |
|------|------|-----------|
| `max` | All cards fit at 250px height | Grouped by profile with headers, full output box, prompt, input, sprite |
| `standard` | Fits at 220px height | Flat grid, prompt + output + input |
| `standard-short` | Fits at 180px height | Flat grid, shorter output box |
| `compact` | Fits at 128px height | Flat grid, no prompt box, smaller sprites (`noGlow=true`) |
| `compact-minimal` | Fallback | Flat grid, no prompt, no input box |

**CRITICAL: Any prop passed to `<AgentCard>` must be passed in ALL THREE render paths** in `App.tsx` (max mode at ~line 195, standard at ~line 220, compact at ~line 240). This is a recurring bug source — features work in one layout mode but silently break in others because the prop was only added to one render path. Examples: `hideSprite`, `mutedCtx`, `isReading`, `cardRef`.

**Grid columns** are computed from window width: `MIN_CELL_WIDTH=280px`, `MAX_CELL_WIDTH=500px`. The `max` mode groups agents by profile with section headers; all other modes use a flat grid sorted by `(profile_name, created_at)`.

### MCP Messaging Server (`mcp/server.js`)

**Tools:** `list_agents`, `send_message(to, content)`, `check_messages(my_session_id)`

**Routing key: `pokegent_id`.** All mailbox storage is at `~/.pokegents/messages/{pokegent_id}/{msg_id}.json`. `pokegent_id` is the abstraction-layer identifier: backend-agnostic, unique per running agent, and used consistently by the dashboard, hooks, and MCP messaging.

**ID resolution.** Both client (`resolveAgent` in `mcp/server.js`) and server (`resolveToPokegentID` in `dashboard/server/server.go`) accept any agent ID hint (8-char prefix or full UUID, matching `pokegent_id` or `session_id`) and resolve to the agent's `pokegent_id` for routing. The hook's `MSG_LOOKUP_ID` and `BUDGET_LOOKUP` prefer `$POKEGENT_ID` and keep `$POKEGENTS_SESSION_ID` as a compatibility alias with the same value.

**Message lifecycle:**
1. `send_message` → stored in `~/.pokegents/messages/{pokegent_id}/{msg_id}.json` with `delivered: false`
2. Hook fires (UserPromptSubmit) → `GetPending` reads undelivered messages → injects notification via `systemMessage` → marks `delivered: true`
3. Agent calls `check_messages` → `ConsumePending` reads AND deletes message files
4. If agent is idle, nudger types "check messages" into terminal after 2s delay

**Message budget:** 15 per turn (per `POKEGENTS_MESSAGE_BUDGET` env var, default in `mcp/server.js`), tracked in `~/.pokegents/messages/{pokegent_id}/_msg_budget`, reset on UserPromptSubmit.

### Configuration (`~/.pokegents/config.json`)

Single source of truth for: `port` (default 7834), `default_profile`, `skip_permissions`, `iterm2_restore_profile`. Read by pokegent.sh, hooks (via `POKEGENTS_DASHBOARD_URL` env), Go server (`DefaultConfig()`), MCP server (`getPort()`).

### Known Pitfalls

- **Never use `set -e` in hooks.** A single jq failure crashes the hook, which blocks ALL Claude operations on every subsequent event. We had a 7-hour error loop from one `--argjson` with empty input.
- **Clone session IDs are tricky.** There are THREE IDs: the pokegent ID (UUID from pokegent.sh), the original's Claude session ID (passed to --resume), and the clone's NEW Claude session ID (assigned by Claude). The running file starts with the pokegent ID and gets patched to the Claude ID by the hook.
- **SessionStart fires for resumed sessions too.** The session ID in the event is the conversation ID, which may differ from the running file's session_id. The hook reconciles this.
- **SessionStart can overwrite busy agents.** When a clone does `--resume <id>`, SessionStart fires for the original's session ID. If the original is busy, the hook must NOT overwrite with "idle". Fix: `STATE="SKIP"` when previous state is "busy".
- **PostToolUse race after Stop.** Slow PostToolUse hooks (python3 trace extraction) can finish AFTER Stop, overwriting "done" with "busy". Guard: busy cannot overwrite done/error/idle except via UserPromptSubmit.
- **`custom-title` in JSONL gets overwritten by Claude on resume.** Dashboard renames persist to both JSONL and search index, and `enrichDisplayNames` overrides with running file names for active sessions.
- **iTerm2 sprite profiles must inherit correctly.** Per-session Dynamic Profiles for sprite icons must inherit from the Pokegents profile (e.g. "Pokegents: Client SDK"), NOT "General". Otherwise tab colors reset to default when the sprite profile activates.
- **Sprite list mismatch between shell and frontend.** pokegent.sh must use the same sprite list as the dashboard frontend. Both use 32-bit signed integer overflow hashing to pick sprites deterministically from session IDs.
- **`pokegent reload` must use profile-specific iTerm tabs.** Must use `create tab with profile "$iterm_prof"` not `create tab with default profile`, otherwise relaunched sessions lose their colors.
- **File watcher must handle Rename events.** fsnotify.Rename fires when SessionStart renames a running file. The old filename triggers Rename (not Remove), and the new filename doesn't trigger Create — only Rename. Both must be handled.
- **Activity feed and context maps must survive `rebuildAgents()`.** These are stored in separate maps (`activityFeeds`, `contexts`) outside the main `agents` map, because `rebuildAgents()` reconstructs agents from running+status files which don't contain feed/context data.
- **Message animation seeding.** `seenIdsRef` in the frontend must be populated with ALL existing message IDs on first load, otherwise 100+ historical messages all animate simultaneously.
- **Port 7834 appears in 4+ places as fallback.** The single source of truth is `config.json`, but fallback values are hardcoded in pokegent.sh, hooks, MCP server, and Go server for resilience when config is missing.

---
> Source: [tRidha/pokegents](https://github.com/tRidha/pokegents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
