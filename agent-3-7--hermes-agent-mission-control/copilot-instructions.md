## hermes-agent-mission-control

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Minions is an autonomous task management system with a Kanban board UI. Users create tasks via a chat interface; each task is a Hermes agent session that autonomously decides how to execute — doing the work itself, spawning child sessions, or creating cron jobs. A periodic heartbeat checks in on in-progress tasks, prompting them to self-report status. The user never talks to child sessions or cron jobs directly; the task agent manages its own sub-resources.

## Prerequisites

- Node.js v18+
- Hermes agent installed with its Python venv (default location: `~/.hermes/hermes-agent/`)

The server spawns a Python worker subprocess that imports Hermes `AIAgent` directly — no `hermes gateway` process or HTTP API is involved. The Python executable is resolved in this order:

1. `HERMES_PYTHON` env var (explicit path)
2. `HERMES_AGENT_DIR/venv/bin/python` (if `HERMES_AGENT_DIR` is set)
3. `~/.hermes/hermes-agent/venv/bin/python` (default)
4. `python3` (system fallback)

## Commands

```bash
npm run dev          # dev mode: tsx watch + Vite dev server on :6969
npm run build        # production build: server (tsc) + client (vite) + copy .sql/.py assets
npm run start        # run compiled production build
npm run prod         # build + run production in one command
```

No test suite or linter is configured.

## Architecture

```
Browser (React/Vite :6969)
  ↕ HTTP + SSE
Express API + Vite middleware (:6969)
  ↕ JSONL over stdin/stdout
Python worker (hermes_worker.py)
  ↕ direct Python import
Hermes AIAgent
```

- **Server** (`server/`): Express + SQLite (better-sqlite3, WAL mode). All timestamps are epoch milliseconds.
- **Python worker** (`server/workers/hermes_worker.py`): JSONL bridge that imports Hermes `AIAgent` directly. Spawned as a subprocess by `HermesWorkerAdapter`, auto-restarts on crash. Manages concurrent agent runs via semaphore (default: 10).
- **Client** (`client/`): React 19 + Vite + Tailwind CSS + Zustand + react-router. `@shared` path alias resolves to `../shared/`.
- **Shared** (`shared/types.ts`): TypeScript types used by both client and server.

### State directory

All persistent state lives under `MINIONS_HOME` (default: `~/.minions/`):
- `data/minions.db` — SQLite database
- `logs/` — log files
- `workspace/` — default working directory for Hermes task artifacts

## Key Design Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Agent communication | Python subprocess + JSONL | Imports Hermes AIAgent directly — no HTTP gateway overhead, structured streaming events, per-task model/reasoning control |
| Task execution | Autonomous agent session | Each task IS a Hermes session. The agent decides execution strategy (self, child session, cron job). Our backend doesn't manage sub-resources. |
| Heartbeat | DB-configured scheduler in Express (15 min default) | Sends check-in to task sessions. Agent reports status honestly; system updates task accordingly. |
| Source of truth | Hermes SessionDB for chat history; Minions SQLite for task metadata; in-memory LiveChatRun for active streams | Hermes owns all transcripts and replay. Minions has no message table. `tasks.id` is the Hermes root session ID; Minions stores task metadata, per-task settings, heartbeat logs, and `last_agent_response_at` for heartbeat idle gating. During active streaming, `live-chat.ts` holds an in-memory `LiveChatRun` with accumulated messages. After streaming ends and the run TTL expires, chat history is projected from Hermes SessionDB on demand. |
| Status ownership | AI moves to `blocked`/`in_review` via heartbeat; human moves everything else via drag-drop | Clean separation: AI reports status, human controls all manual transitions. |

## Key Patterns

- **Session lifecycle**: `tasks.id` is the Minions task ID and the Hermes root session ID. Chat, history, metadata reads, and heartbeat check-ins all use `task.id`; Minions does not persist Hermes-returned child or continuation session IDs.
- **Chat projection**: `GET /tasks/:id/messages` loads raw rows from Hermes `SessionDB.get_messages()` via the Python worker, which filters out heartbeat check-ins (`[AUTOMATED CHECK-IN]`), status blocks (`<status_report>`), and tool-call-only turns. The client shows optimistic messages during streaming and loads the projected history from Hermes on page load/task switch.
- **Per-task model/reasoning**: Each task can override the default Hermes model and reasoning effort (`agent_model`, `reasoning_effort` columns on `tasks`). Settings logic lives in `server/agent-settings.ts`. The Python worker resolves the final model/provider from Hermes config + per-task overrides.
- **Heartbeat status parsing**: Agent responses must contain a `<status_report>` / `</status_report>` block with JSON. Unparseable responses are logged but don't crash.
- **Heartbeat visibility**: `summary` and optional `user_summary` are saved to `heartbeat_log` and shown only in the Activity tab. Heartbeat turns remain in Hermes session history for agent continuity, but the chat projection filters them out.
- **Heartbeat eligibility**: Heartbeat checks `in_progress` tasks whose last activity (`last_agent_response_at`, falling back to `created_at`) is older than the DB-backed heartbeat idle-delay setting. The heartbeat interval and idle delay both default to 15 minutes and are editable from Settings. Background heartbeat stays below total Python worker capacity (`HERMES_AGENT_RUN_LIMIT`) so user chat always has at least one slot available, and skips tasks that already have an active agent run.
- **Live chat**: `POST /api/tasks/:id/messages` returns `202` immediately with a `runId`. The server consumes the agent stream in the background via `consumeChatRun()` in `server/routes/chat.ts`. Clients subscribe to `GET /api/tasks/:id/live` SSE for real-time `text_delta`, `thinking_delta`, `tool_progress`, `done`, and `error` events. On connect, the client receives a snapshot of the current in-memory `LiveChatRun` if one exists. Runs are kept in memory briefly after completion (30s normal, 5min on error) so late-connecting clients can catch up.
- **Live-chat state** (`server/live-chat.ts`): In-memory `Map<taskId, LiveChatRun>` accumulates streaming events into structured messages (user + assistant with tools/thinking/usage). This is ephemeral — on server restart, active run state is lost, but the Hermes session history remains in SessionDB.
- **SSE board events**: `/api/events` broadcasts board-level events (task CRUD) to all clients. Separate from per-task live chat SSE.
- **Disconnect resilience**: If the browser disconnects during a stream, the server continues draining the worker stream to completion. On successful completion, `last_agent_response_at` is recorded for the task.
- **Cron jobs**: Hermes manages cron job state internally. Minions exposes endpoints to list, pause, resume, trigger, and remove jobs. Cron jobs link back to their originating task via `origin.platform === 'minions'`.
- **Server imports**: Use `.js` extensions in import paths (ESM with tsx).

## Task State Machine

```
                         user creates
                              │
                              ▼
                      ┌──────────────┐
           ┌─────────│  IN_PROGRESS  │◄────────────────┐
           │         └───┬───────┬───┘                 │
           │             │       │                     │
           │       agent │       │ agent               │ human moves
           │     reports │       │ reports             │ (drag-drop)
           │    "blocked"│       │ "completed"          │
           │             │       │                     │
           │       ┌─────▼──┐  ┌─▼──────────┐          │
           │       │BLOCKED │  │ IN_REVIEW   │          │
           │       │ needs  │  │ human       │          │
           │       │ human  │  │ verifies    │          │
           │       │ input  │  │ the work    │          │
           │       └───┬────┘  └──────┬──────┘          │
           │           │              │                 │
           │           │ human moves  │ human moves     │
           │           └──────────────┴─────────────────┘
           │
           │         ┌──────────────┐
           └────────►│    DONE      │
             human   │ human        │
             moves   │ verified     │
                     └──────────────┘
```

**AI moves to**: `blocked`, `in_review` (via heartbeat status reports)
**Human moves to**: `in_progress`, `done` (via drag-and-drop or action buttons)

## Data Flows

### New Task → Agent Session

```
User creates task via UI
  → POST /api/tasks (creates task in SQLite; tasks.id is the Hermes root session ID)
  → Task appears on board in "In Progress" column
  → User sends first message via POST /api/tasks/:id/messages
  → Server returns 202 with runId; client subscribes to GET /tasks/:id/live SSE
  → Server calls consumeChatRun() with session ID equal to task.id
  → LiveChatRun accumulates text_delta/thinking/tools into in-memory messages
  → Each event is broadcast to /live SSE subscribers in real time
  → Python worker loads prior Hermes history, runs AIAgent.run_conversation()
  → Hermes persists the full turn (user + assistant) in SessionDB
  → Successful completion records tasks.last_agent_response_at
  → Run state expires from memory after TTL; history loads from Hermes on next page load
```

### Heartbeat Check-In

```
Heartbeat scheduler fires (DB-configured, 15 minutes by default)
  → Query: all tasks WHERE status = 'in_progress'
      AND COALESCE(last_agent_response_at, created_at) <= now - configured idle delay
  → For each task (batched by HEARTBEAT_CONCURRENCY, skipping busy tasks):
      → Fetch last 3 heartbeat summaries for context
      → Send check-in prompt to agent via adapter.chat(task.id, ...)
      → Parse <status_report> JSON block
      → Record last_agent_response_at for every successful heartbeat response
      → IF progressing → stay in_progress (log summary)
      → IF completed → move to 'in_review' (broadcast update)
      → IF blocked → move to 'blocked' (broadcast update)
      → Log summary + optional user_summary to heartbeat_log (visible only in Activity)
```

## Worker Protocol

The Python worker communicates via JSONL (one JSON object per line) over stdin/stdout.

**Request types**: `health`, `chat`, `session.messages.get`, `session.get`, `settings.get`, `models.list`, `cron.jobs.list`, `cron.jobs.get`, `cron.jobs.runs`, `cron.jobs.run.content`, `cron.jobs.pause`, `cron.jobs.resume`, `cron.jobs.run`, `cron.jobs.remove`, `cron.tick`

**Stream events** (emitted during `chat` requests):

| Event | Description |
|-------|-------------|
| `text_delta` | Agent's visible response text |
| `thinking_delta` | Agent's reasoning content |
| `tool_progress` | Tool lifecycle (`running` / `completed` / `error`) with name and duration |
| `done` | Stream complete — carries `sessionId` and `usage` |
| `error` | Error with message, optional `code` and `hint` |

**Thinking content**: Live reasoning streams as `thinking_delta`. Historical reasoning is projected from Hermes `reasoning_content` / `reasoning` into `msg.thinking` when available.

## Client Routes

| Path | Component | Description |
|------|-----------|-------------|
| `/` | `Board` | Kanban board (4 columns + drag-and-drop) |
| `/tasks/new` | `NewTaskPage` | Create task + initial chat with agent |
| `/tasks/:taskId` | `TaskDetailPage` | Task detail + chat thread |
| `/cron` | `CronPage` | View and manage Hermes cron jobs |
| `/settings` | `SettingsPage` | App settings including theme and heartbeat timing |

## Environment Variables

All optional — defaults work for local development.

```bash
PORT=6969                        # Web server port
HERMES_PYTHON=                   # Path to Python with Hermes deps (auto-detected if unset)
HERMES_AGENT_DIR=                # Path to Hermes agent dir (default: ~/.hermes/hermes-agent)
HERMES_AGENT_RUN_LIMIT=10        # Max concurrent agent runs in Python worker
HEARTBEAT_CONCURRENCY=2          # Max concurrent heartbeat checks (capped below AGENT_RUN_LIMIT)
MINIONS_HOME=~/.minions          # State directory (DB, logs, backups, workspace)
DB_PATH=~/.minions/data/minions.db  # SQLite database path
```

Heartbeat interval and idle-delay settings are stored in `app_settings` and default to 15 minutes.

## Hermes Python Library

The Python worker imports Hermes `AIAgent` from the local agent directory (not via pip). Key API surface used:

- **`AIAgent(model, provider, session_id, session_db, ...)`**: Constructor takes model/provider config, session tracking, callbacks for streaming deltas, and tool configuration. Create one instance per conversation turn — not thread-safe across concurrent calls.
- **`agent.run_conversation(user_message, system_message, conversation_history, task_id)`**: Full control method returning `{"final_response", "messages", "input_tokens", "output_tokens", ...}`. The worker passes sanitized history from `SessionDB.get_messages_as_conversation()` as `conversation_history`.
- **`SessionDB`**: From `hermes_state` module. Used for `get_messages()` (raw rows for chat projection), `get_messages_as_conversation()` (formatted history for agent replay), `get_session()` (session metadata with token counts/cost), and `resolve_resume_session_id()` (follow session chains).
- **Callbacks**: `stream_delta_callback` (text), `reasoning_callback` (thinking), `tool_progress_callback` (tool lifecycle events). These are passed to the constructor and fire during `run_conversation()`.

Docs: https://hermes-agent.nousresearch.com/docs/guides/python-library

## Future

- **Additional adapters**: The `AgentAdapter` interface (`server/adapters/types.ts`) is pluggable. Other OpenAI-compatible backends (OpenClaw, LiteLLM, etc.) could implement it.

---
> Source: [Agent-3-7/hermes-agent-mission-control](https://github.com/Agent-3-7/hermes-agent-mission-control) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-08 -->
