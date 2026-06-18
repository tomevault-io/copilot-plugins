## power-fixer

> This file provides guidance to WARP (warp.dev) when working with code in this repository.

# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Repository Structure

PowerFixer is split across two repositories:
- **power-fixer** (this repo) - TUI client application
- **power-fixer-server** (`../power-fixer-server/`) - Backend API server

## Build & Run Commands

```bash
# Start the TUI client (requires server to be running)
./script/run                                     # Connects to localhost:3001
./script/run --server http://localhost:3001      # Explicit server URL
./script/run --server https://your-ngrok.domain  # Connect to remote server

# Development commands
cargo check     # Check compilation
cargo fmt       # Format
cargo clippy    # Lint
cargo build --release  # Build only

# To run the server, see ../power-fixer-server/
```

## System Architecture

Power Fixer is a **client-server system** for GitHub issue triage with AI agent support:

### PowerFixer TUI (This Repository)
A ratatui/crossterm TUI application that runs locally. **Key principle: the TUI stores NO persistent state to disk**. All agent state is kept in an **in-memory cache** during the running session. The TUI is purely a view and controller layer.

**In-Memory Cache Layer:**
The TUI maintains a local cache of all agent status information for instant access without network calls. This cache is stored in the `App` struct and includes:
- `cached_agents` - All agents (fix, dedupe, triage) with their status
- `cached_triage_runs` - Power triage batch runs
- `inbox_states` - Read/archived state for inbox items

**Cache Population & Updates:**
1. **Startup**: `initial_agent_sync()` fetches full state from server via `GET /api/v1/state` (retries up to 10 times if server is not ready)
2. **Real-time WebSocket**: Server pushes updates via WebSocket, handled by `handle_agent_event()` in main.rs
3. **Server-side polling**: Server runs a background loop every 5 seconds, polling Warp's REST API for all active tasks and broadcasting changes via WebSocket
4. **Client-side polling (backup)**: `AgentClient` also polls the server periodically as a redundant mechanism
5. **Manual refresh**: User can trigger sync via 's' key in agent view

**Responsibilities:**
- Render the UI from the in-memory cache (instant, no network calls)
- Receive and apply WebSocket updates to the cache
- Make HTTP calls to server only for write operations (launch agent, update inbox state, etc.)
- Directly fetch GitHub issue data via `gh` CLI (exception to the "server-only" rule for read operations)

### PowerFixer Server (Separate Repository)
The server is located at `../power-fixer-server/`. See that repository's WARP.md for server-specific documentation.

**Key server responsibilities:**
- Single source of truth for all agent-related state (Postgres DB)
- WebSocket server for real-time updates to TUI clients
- Background polling of Warp's REST API for task status
- Callback API for cloud agents to report status

## Data Flow

```
┌──────────────────────────────────────┐
│    PowerFixer TUI (this repo)        │
│  ┌────────────────────────────────┐  │
│  │    In-Memory Cache             │  │
│  │  (cached_agents, etc.)         │◄─┼─── UI reads from cache (instant)
│  └────────────────────────────────┘  │
│           │              ▲           │
│     startup sync    WebSocket push   │
│           │              │           │
└───────────┼──────────────┼───────────┘
            │              │
            ▼              │
┌──────────────────────────────────────┐
│  PowerFixer Server (separate repo)   │
│     ../power-fixer-server/           │
│  ┌─────────────┐  ┌───────────────┐  │
│  │ REST API    │  │ WebSocket Srv │  │
│  └─────────────┘  └───────────────┘  │
│                    ...               │
└──────────────────────────────────────┘
```

**Cache Data Flow:**
1. **TUI reads from cache** - All UI rendering reads from `cached_*` fields, never making network calls
2. **Cache miss triggers fetch** - If cache is empty/stale, TUI calls server to populate it
3. **Server pushes to cache** - WebSocket connection receives real-time updates that update the cache
4. **Server polls external APIs** - Server's Warp API Poller and Callback API receive updates, persist to DB, then broadcast via WebSocket

**Startup Flow:**
1. TUI starts, creates empty cache
2. `initial_agent_sync()` calls `GET /api/v1/state` to populate cache
3. TUI establishes WebSocket connection for real-time updates
4. Server broadcasts updates as agents report progress → cache is updated
5. UI renders from cache with zero latency

**Manual Sync:** TUI can call the server at any time to get a fresh state snapshot (e.g., 's' key in agent view), but the normal flow relies on WebSocket pushes keeping the cache current.

## User Flows

See [userflows.md](userflows.md) for comprehensive navigation diagrams and state machine documentation.

### Primary Triage Flow
Go issue-by-issue through untriaged GitHub issues. For each issue:

1. **Skip** - Move to next issue
2. **Triage** - Add comment, label, or send Slack message for a Warper to handle
3. **Assign Agent** - Launch an cloud agent to attempt an automatic fix
4. **Dedupe Search** - Launch an agent to find similar/duplicate issues; results flow back via API and displayed for user to close duplicates or ignore
5. **Send Slack** - Ask about the issue in Warp's Slack

### Power Triage Flow
A hero feature: launch multiple agents in parallel, each analyzing N different issues to determine if they are good candidates for automatic agent fixes. Results reported back via the callback API, stored in DB, pushed to TUI via WebSocket.

### Agent Status Views
- View all agent-assigned issues and their current status
- Jump to session links for running agents
- Review completed agent results (PRs, branches)

## Module Structure

### `app/` - Core TUI Application Layer
- `state.rs` - Application state (`App` struct) with **in-memory cache fields** and state machine enum (`AppState`)
- `handlers.rs` - Keyboard input handling for each `AppState`
- `ui.rs` - ratatui rendering logic per state (**reads from cache, never network**)
- `background.rs` - Background task spawning, result handlers, **`initial_agent_sync()` for cache population**
- `theme.rs` - Color theme support (light/dark/auto modes)

### `server/` - PowerFixer Server Communication
- `client.rs` - HTTP client for PowerFixer Server API, **background polling that updates cache via events**
- `websocket.rs` - WebSocket client for real-time updates → **pushes to cache via `AgentEvent`**
- `types.rs` - Request/response types for server API
- `ws_types.rs` - WebSocket message types (shared with server)

### `services/` - Third-Party External Services
- `github.rs` - GitHub CLI (`gh`) wrapper for issues, comments, labels
- `slack.rs` - Slack API integration

### `features/` - Feature-Specific Logic
- `triage.rs` - Power triage batch orchestration

### Root-Level Files
- `main.rs` - Event loop, terminal setup, connects to server, **handles WebSocket events to update cache**
- `lib.rs` - Library crate exports
- `utils.rs` - Logging, clipboard, mention autocomplete helpers

**Note:** Server code (API endpoints, database, callbacks) lives in the `power-fixer-server` repository.

## State Machine Pattern

The TUI uses `AppState` enum to track current view/mode. Each state has:
1. Dedicated key handler in `handlers.rs`
2. Dedicated render function in `ui.rs`
3. Background tasks communicate via `mpsc` channel (`bg_tx`/`bg_rx`)

## External Dependencies

- **GitHub CLI (`gh`)** - Must be authenticated. Issue fetch operations shell out to `gh`
- **PowerFixer Server** - Must be running (locally or cloud-hosted)
- **Slack** - Optional, requires `SLACK_BOT_TOKEN` env var

## Environment Variables

- `POWERFIXER_CALLBACK_URL` - URL of the PowerFixer Server (default: `http://localhost:3001`)
- `SLACK_BOT_TOKEN` - Slack bot token for notifications (optional)

**Note:** Server-specific environment variables (DATABASE_URL, WARP_API_KEY, etc.) are documented in the server repository.

## Label Workflow

Issues are tracked via `pf:` prefixed labels on GitHub for visibility:
- `pf:waiting-user` - Waiting for user response
- `pf:waiting-warper` - Waiting for Warp team action
- `pf:triaged` - Issue has been manually triaged by a PowerFixer user (examined, possibly commented, no further action needed at this time)

The `IssueFilter` enum in `services/github.rs` maps to these labels.

**Filtering Behavior:**
- **Untriaged view**: Shows issues with NO `pf:` labels (excludes waiting-user, waiting-warper, and triaged)
- **Triaged view**: Shows only issues with `pf:triaged` label
- **All view**: Shows all issues regardless of labels

Agent assignment and other states are tracked in the database, not via GitHub labels.

## Critical Invariants

1. **TUI has no disk state** - All persistent state lives in the server's Postgres DB
2. **Server is source of truth** - TUI's cache reflects server state, never the other way around
3. **UI reads from cache, not network** - All rendering reads from in-memory `cached_*` fields for instant response
4. **Cache updated via WebSocket** - Server pushes updates in real-time via its background polling loop
5. **Server polls Warp API** - Server independently polls Warp's REST API every 5 seconds for active tasks, ensuring all TUI clients stay in sync
6. **Writes go through server** - Any state changes (launch agent, archive item) go via HTTP to server, then flow back to cache via WebSocket
7. **All agent communication goes through server** - TUI never talks directly to cloud agents

## Development Guidelines

### Import Style
- **Always use `use` statements at the top of files** - Do not use inline full paths like `crate::module::Symbol` in code. Instead, add `use crate::module::Symbol;` at the top of the file and reference `Symbol` directly.
- **Function-scoped `use` is acceptable** - When a type is only used in one function, a `use` statement inside the function body is fine for clarity.
- **Only use full paths for name conflicts** - If two modules have the same symbol name, use full paths with a comment explaining the conflict.
- **Exception: log macros** - Always use the `log::` prefix for log macros (e.g., `log::debug!()`, `log::info!()`, `log::error!()`). Do not import these macros directly.

### Compilation Warnings
- **Always fix unused variable warnings** - Unused variable warnings often indicate that code was accidentally deleted or that functionality was broken during refactoring. Always investigate and fix these rather than ignoring or suppressing them.
- **Fix Warnings Introduced by Agent** - Any build that results in warnings (unused code, dead code, etc.) that the agent is initiating should be fixed by the agent. Do not leave warnings in the codebase.
- **Run `cargo check` after changes** - Verify compilation before considering a change complete.
- **Run `cargo fmt`** - Format code before committing.

## UI Guidelines

### Colors
- **Do NOT use `Color::DarkGray`** - It renders too dim to be readable in most terminal themes. Use `Color::Gray` instead for secondary/muted text.

---
> Source: [warpdotdev/power-fixer](https://github.com/warpdotdev/power-fixer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
