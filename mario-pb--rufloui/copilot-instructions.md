## rufloui

> **RuFloUI** is a React 19 web frontend for managing [claude-flow v3](https://github.com/ruvnet/claude-flow) multi-agent orchestration. It wraps the CLI (`npx @claude-flow/cli@latest`) behind an Express + WebSocket backend and presents a full dashboard with real-time agent monitoring, task execution via multi-agent pipelines, and persistent state.

# Claude Code Configuration - RuFloUI

## Project Overview

**RuFloUI** is a React 19 web frontend for managing [claude-flow v3](https://github.com/ruvnet/claude-flow) multi-agent orchestration. It wraps the CLI (`npx @claude-flow/cli@latest`) behind an Express + WebSocket backend and presents a full dashboard with real-time agent monitoring, task execution via multi-agent pipelines, and persistent state.

- **Repo**: https://github.com/Mario-PB/rufloui.git
- **Tech Stack**: React 19 + Vite 6 + TypeScript + Express + WebSocket + Zustand
- **Charts**: Recharts
- **Icons**: Lucide React
- **OS**: Windows 11 (use Unix shell syntax in bash commands)

## Quick Start

```bash
npm install
npm run dev          # Starts both frontend (Vite :28588) and backend (Express :28580)
# Or individually:
npm run dev:frontend # Vite dev server on port 28588
npm run dev:backend  # Express API on port 28580 (tsx watch, auto-reloads)
```

The frontend proxies `/api/*` and `/ws` to `localhost:28580` via Vite config.

## Project Structure

```
src/
├── backend/
│   ├── server.ts          # Express API + WebSocket server (~2100 lines)
│   └── jsonl-monitor.ts   # JSONL session file monitor for Agent Viz (~430 lines)
└── frontend/
    ├── main.tsx            # Entry point
    ├── App.tsx             # Router + WebSocket handler + initial data fetch
    ├── api.ts              # API client (fetch wrapper + all endpoint namespaces)
    ├── store.ts            # Zustand global state with sessionStorage persistence
    ├── types.ts            # TypeScript interfaces (Agent, Task, SwarmAgent, etc.)
    ├── styles/
    │   └── global.css      # CSS variables, dark theme, animations
    ├── components/
    │   ├── Layout.tsx       # App shell: sidebar nav (grouped) + header + activity panel
    │   ├── ErrorBoundary.tsx
    │   └── ui/
    │       ├── Button.tsx   # Reusable button with loading/variants
    │       ├── Card.tsx     # Card container with title/actions
    │       └── StatusBadge.tsx # Colored status pill
    └── pages/
        ├── Dashboard.tsx        # System health, agent overview, activity chart
        ├── AgentsPanel.tsx      # Spawn, list, terminate agents
        ├── SwarmPanel.tsx       # Init/shutdown swarm, topology graph
        ├── SwarmMonitorPanel.tsx # Real-time agent cards with live output modal
        ├── AgentVizPanel.tsx    # JSONL tree visualization of Claude sessions
        ├── HiveMindPanel.tsx    # Consensus, broadcast, shared memory
        ├── TasksPanel.tsx       # Kanban task board with live execution output
        ├── SessionsPanel.tsx    # Save/restore sessions
        ├── PerformancePanel.tsx # Benchmarks, latency/throughput charts
        ├── MemoryPanel.tsx      # Store/search/retrieve memories
        ├── HooksPanel.tsx       # Hook configuration
        ├── NeuralPanel.tsx      # Neural network status
        ├── WorkflowsPanel.tsx   # Workflow management with step tracking
        ├── ConfigPanel.tsx      # Configuration editor
        ├── LogsPanel.tsx        # Live activity logs
        └── MemoryGame.tsx       # Easter egg game
```

## Backend Architecture (server.ts)

The backend is a single Express file that wraps CLI commands, manages state, and persists to disk.

### Key Patterns

1. **CLI Wrapper**: `execCli(command, args)` runs `npx @claude-flow/cli@latest <command> <args>` and returns `{ raw, parsed? }`. Supports `--format json` for structured output.

2. **Table Parser**: `parseCliTable(raw)` extracts rows from ASCII tables (`| col1 | col2 |` format) into `Record<string, string>[]`. Column names are lowercased with spaces replaced by underscores.

3. **Persistence Layer** (`.ruflo/state.json`):
   All critical state is persisted to disk as JSON and restored on server startup:
   - `taskStore`, `workflowStore`, `sessionStore` — user-created data
   - `agentRegistry`, `terminatedAgents`, `agentActivity` — agent tracking
   - `swarmConfig` (id, topology, strategy, maxAgents, shutdown flag)
   - `perfHistory`, `lastPerfMetrics`, `benchmarkHasRun` — performance data
   - `currentSwarmAgentIds` — current swarm membership

   Persistence triggers:
   - **Debounced save** (2s) on every `broadcast()` of significant events
   - **Periodic save** every 30s as safety net
   - **Shutdown save** on SIGINT/SIGTERM
   - **Load on startup** via `loadFromDisk()`

4. **Multi-Agent Pipeline** (`launchSwarmPipeline()`):
   When a task is assigned with an active swarm:
   - Phase 1: Coordinator plans subtasks using ALL available agent roles
   - Phase 2: Workers execute in parallel waves respecting dependencies
   - Phase 3: Results synthesized, task marked complete
   Each agent runs as a separate `claude -p` process with `--output-format stream-json`.

5. **Agent Output Buffers** (`agentOutputBuffers: Map`):
   Stores last 500 lines of Claude output per agent. Streamed to frontend via `agent:output` WebSocket events for the live output modal.

6. **In-Memory Stores** (persisted to `.ruflo/state.json`):
   - `agentRegistry: Map<createdTime, {id, name, type}>` — tracks spawned agents
   - `terminatedAgents: Set<createdTime>` — marks agents as terminated
   - `taskStore: Map<id, TaskRecord>` — tasks created via UI
   - `workflowStore: Map<id, WorkflowRecord>` — workflow execution state with steps
   - `sessionStore: Map<id, SessionRecord>` — sessions created via UI
   - `agentActivity: Map<agentId, AgentActivity>` — real-time agent status
   - `perfHistory[]` + `lastPerfMetrics` + `benchmarkHasRun` — performance data

7. **Time Matching**: Agent spawn returns UTC timestamps, but `agent list` shows local time. The registry converts UTC->local for matching.

8. **Health Checks**: Parsed by matching known check names (`Version Freshness`, `Daemon Status`, etc.) rather than Unicode symbols (Windows codepage mangles check/warning marks).

9. **Env Var Cleanup**: When spawning child `claude -p` processes, ALL env vars starting with `CLAUDE` are deleted to prevent nested session conflicts.

10. **Telegram Bot** (`telegram-bot.ts`): Optional polling-based Telegram integration for remote monitoring/control. Enabled via `TELEGRAM_ENABLED=true`, `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID`. Returns `null` when disabled — zero overhead. Hooks into `broadcast()` for fire-and-forget notifications.

### API Routes

| Route | Methods | Description |
|-------|---------|-------------|
| `/api/system` | GET health/info/metrics/status, POST reset | System monitoring |
| `/api/agents` | GET list, POST spawn, GET :id/status, POST :id/terminate, POST terminate-all | Agent lifecycle |
| `/api/swarm` | GET status/health, POST init/shutdown | Swarm management |
| `/api/hive-mind` | GET status, POST init/shutdown/join/leave/consensus/broadcast/memory | Hive mind |
| `/api/tasks` | GET list/summary, POST create, POST :id/assign/:id/complete/:id/cancel | Task management |
| `/api/sessions` | GET list, POST save, POST :id/restore, DELETE :id | Session management |
| `/api/performance` | GET metrics/bottleneck/profile/report, POST benchmark/optimize | Performance |
| `/api/memory` | GET list/search/stats, POST store, GET :key, DELETE :key | Memory storage |
| `/api/hooks` | GET list, POST init, GET metrics, GET :name/explain | Hook management |
| `/api/neural` | GET status/patterns, POST train/compress/optimize | Neural network |
| `/api/workflows` | GET list/templates, POST create/execute, DELETE :id | Workflows |
| `/api/coordination` | GET metrics/topology, POST sync/consensus | Coordination |
| `/api/config` | GET list, GET/PUT :key, POST reset, GET export, POST import | Configuration |
| `/api/ai-defence` | GET stats, POST scan/analyze | AI defence |
| `/api/swarm-monitor` | GET snapshot/activity/agents/health/metrics, GET output/:agentId, POST purge | Swarm Monitor |
| `/api/webhooks` | POST github, GET/PUT github/config, GET github/events | GitHub webhook receiver + config |
| `/api/config/telegram` | GET status, PUT update config | Telegram bot config + status |
| `/api/config/telegram/test` | POST | Send test message to Telegram |
| `/api/config/telegram/log` | GET | Activity log (last 50 messages) |
| `/api/viz` | GET sessions, GET sessions/:id, GET sessions/:id/logs/:nodeId | Agent Viz (JSONL) |

### WebSocket Events

- Path: `ws://localhost:28580/ws`
- **Broadcast events**: `swarm:status`, `agent:activity`, `agent:output`, `task:added`, `task:updated`, `task:output`, `workflow:added`, `workflow:updated`, `session:list`, `performance:metrics`, `viz:update`, `swarm-monitor:update`, `log`
- Frontend connects in `App.tsx` and dispatches to Zustand store

## Frontend Architecture

### State Management (Zustand + sessionStorage)

Global store in `store.ts` with `persist` middleware using `sessionStorage`:
- Survives page reloads within the same browser tab
- Clears when tab is closed (no stale data from old sessions)
- `partialize()` excludes transient state (connected, selectedVizNode)
- Logs capped at 100 entries in storage

### Initial Data Loading

`App.tsx` fetches core data (health, tasks, agents, workflows, sessions) on mount via `Promise.allSettled`. This means all pages have data immediately without waiting for individual page fetches. API responses are normalized (handles both `{tasks: [...]}` and `[...]` formats).

### API Client (api.ts)

Thin `fetch` wrapper: `request<T>(path, options)` -> auto-adds JSON headers, logs to store, throws on non-OK. Namespaces: `system`, `swarm`, `agents`, `tasks`, `memory`, `sessions`, `hiveMind`, `neural`, `performance`, `hooks`, `workflows`, `coordination`, `config`, `viz`, `swarmMonitor`, `aiDefence`.

### Key UI Patterns

- **No CSS framework** — all styles are inline CSSProperties objects or CSS variables in `global.css`
- **Dark theme** — CSS variables like `--bg-primary`, `--text-primary`, `--accent-blue`, etc.
- **Modals** — custom React overlays via `createPortal(modal, document.body)`, with backdrop click-to-close
- **Status-colored cards** — SwarmMonitorPanel agent cards use per-status background/border colors (orange=working, green=active, red=error, blue=idle)
- **Live output modal** — Click agent card "Output" button to see real-time Claude output streamed via WebSocket
- **Defensive rendering** — all `.slice()`, `.toLowerCase()` calls guarded with `?? ''` or `|| 'unknown'` fallbacks
- **Lazy loading** — all pages use `React.lazy()` with Suspense spinner

### Sidebar Navigation (Layout.tsx)

Grouped into: Overview (Dashboard), Orchestration (Swarm, Agents, Agent Viz, Swarm Monitor, Tasks), Intelligence (Hive Mind, Neural, Memory), Operations (Workflows, Hooks, Sessions), Monitoring (Performance, Config, Logs), Games (Memory Game).

## Multi-Agent Task Execution

When a task is assigned with an active swarm:

1. `launchWorkflowForTask()` detects active swarm agents
2. `launchSwarmPipeline()` orchestrates the 3-phase pipeline:
   - **Phase 1**: Coordinator plans subtasks, explicitly assigning to different roles
   - **Phase 2**: Workers execute in dependency waves (parallel where possible)
   - **Phase 3**: Mark complete, persist results
3. Each agent runs as `claude -p` with role-specific system prompts
4. Output is buffered per-agent and streamed to frontend via WebSocket
5. Agent activity broadcasts update Swarm Monitor cards in real-time
6. Fallback: if no swarm active, single `claude -p` handles the whole task

### Agent Roles in Pipeline

The planning prompt forces the coordinator to use ALL available roles:
- `researcher` — explore codebase, find files, understand patterns (Read/Grep/Glob only, no modifications)
- `coder` — implement code changes (Edit/Write tools)
- `tester` — write tests, run test suites, validate
- `reviewer` — review code quality, security, best practices

## Persistence Architecture

```
Frontend (React)                    Backend (Express)                     Disk
┌──────────────┐                   ┌──────────────────┐                ┌─────────────┐
│ Zustand store │◄── WebSocket ──►│ In-memory Maps    │◄── save/load─►│.ruflo/      │
│ sessionStorage│◄── REST API ───►│ taskStore         │               │ state.json  │
│ (per tab)     │                  │ workflowStore     │               └─────────────┘
└──────────────┘                   │ agentRegistry     │               ┌─────────────┐
                                   │ agentActivity     │               │.claude-flow/│
                                   │ sessionStore      │               │ agents/     │
                                   │ perfHistory       │               │ tasks/      │
                                   └──────────────────┘               │ workflows/  │
                                          │                            └─────────────┘
                                          ▼                            ┌─────────────┐
                                   claude-flow CLI ──────────────────►│.swarm/      │
                                   (npx @claude-flow/cli)              │ memory.db   │
                                                                       │ hnsw.index  │
                                                                       └─────────────┘
```

**What persists across server restarts**: tasks, workflows, sessions, agent registry, swarm config, performance history, agent activity.

**What persists across page reloads**: all store state via sessionStorage (within same tab).

**What is lost**: running processes (claude -p), WebSocket connections, agent output buffers (rebuilt on next execution).

## CLI Quirks & Workarounds

These are critical to understand when modifying the backend:

| Issue | Workaround |
|-------|-----------|
| CLI outputs ASCII tables, not JSON | `parseCliTable()` + regex extraction; prefer `--format json` |
| `agent list` doesn't show agent IDs or names | `agentRegistry` Map keyed by created time |
| `agent stop` doesn't remove agent from list | `terminatedAgents` Set, filtered in list endpoint |
| `agent stop --all` doesn't work | List agents first, stop individually in parallel batches of 10 |
| Spawn returns UTC time, list shows local time | Convert UTC->local with `new Date()` for registry key |
| `swarm shutdown` doesn't clear CLI state | `swarmShutdown` boolean flag, checked in status endpoint |
| Windows codepage mangles UTF-8 (check/warning marks) | Match known check names, not Unicode symbols |
| `performance metrics` has different columns than `benchmark` | Metrics: `Metric/Current/Limit/Status`; Benchmark: `Operation/Mean/P95/P99/Status` |
| Agent types must match CLI exactly | Valid: `coder`, `researcher`, `tester`, `reviewer`, `architect`, `coordinator`, `analyst`, `optimizer`, `security-architect`, `security-auditor`, `memory-specialist`, `swarm-specialist`, `performance-engineer`, `core-architect`, `test-architect` |
| Agent spawn flag is `--type` not `-t` | `['spawn', '--type', type, '--name', name]` |
| Nested `claude -p` fails with CLAUDE env vars | Delete ALL env vars starting with `CLAUDE` before spawn |
| `--output-format stream-json` needs `--verbose` | Always include `--verbose` flag with stream-json |
| Zombie agents accumulate across swarm inits | `purgeAllCliAgents()` on swarm init + manual purge button |

## Behavioral Rules

- Do what has been asked; nothing more, nothing less
- NEVER create files unless absolutely necessary
- ALWAYS prefer editing existing files over creating new ones
- NEVER proactively create documentation files unless explicitly requested
- ALWAYS read a file before editing it
- NEVER commit secrets, credentials, or .env files
- Keep files under 500 lines where possible (server.ts is an exception at ~2200 lines)
- After editing `server.ts`, the backend must be restarted: `npx kill-port 28580; npx tsx src/backend/server.ts`
- Frontend hot-reloads automatically via Vite
- When API returns `{ items: [...] }` vs `[...]`, always normalize before setting store

## Build & Deploy

```bash
npm run build        # TypeScript check + Vite production build -> dist/
npm run preview      # Preview production build
npx tsc --noEmit     # Type check without building
```

## Git & Repository

```bash
git remote origin -> https://github.com/Mario-PB/rufloui.git
# Branch: main
# .gitignore excludes: node_modules/, dist/, .env*, .claude/, .claude-flow/, .ruflo/, .swarm/, .mcp.json
```

## Local Data Directories (not in git)

- `.ruflo/` — RuFlo persistence (state.json with tasks, workflows, agents, etc.)
- `.claude-flow/` — claude-flow CLI state (agents/store.json, tasks/store.json, config.yaml, etc.)
- `.swarm/` — claude-flow swarm data (memory.db SQLite + HNSW vector index)

## Support

- claude-flow docs: https://github.com/ruvnet/claude-flow
- claude-flow issues: https://github.com/ruvnet/claude-flow/issues

---
> Source: [Mario-PB/RuFloUI](https://github.com/Mario-PB/RuFloUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
