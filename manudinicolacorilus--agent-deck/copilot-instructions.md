## agent-deck

> A self-hosted Node.js web app for orchestrating multiple AI coding agents (Claude CLI, Copilot CLI)

# Agent Deck — Claude Code Context

## What this project is
A self-hosted Node.js web app for orchestrating multiple AI coding agents (Claude CLI, Copilot CLI)
toward a shared software goal. Users submit a mission; a Super-Master agent coordinates a pipeline
of specialized agents (Explorer → Architect → Developer → Reviewer → Tester → Integrator → Releaser).
Every agent is a real PTY terminal session running inside the browser via xterm.js.

## Stack — DO NOT deviate from this
| Layer        | Choice              | Notes                                      |
|--------------|---------------------|--------------------------------------------|
| Server       | Node.js 20 + Express 4 | Single process, no worker threads        |
| WebSocket    | ws (not socket.io)  | Lightweight, no polling fallback needed    |
| Terminal     | node-pty            | Spawns real PTY for each agent             |
| Persistence  | better-sqlite3      | Synchronous, zero setup, one .db file      |
| Frontend     | Vite + React 18     | SPA, proxies /api and /ws to :3001 in dev  |
| Terminal UI  | xterm.js 5          | + xterm-addon-fit + xterm-addon-web-links  |
| State        | Zustand 4           | Two stores: useMissionsStore, useAgentsStore |
| Styling      | TailwindCSS 3       | No CSS-in-JS                               |
| Dev runner   | concurrently        | npm run dev starts server + vite together  |

## Repository layout
```
agent-deck/
  package.json          root scripts: dev, build, start
  .env.example          CLAUDE_CLI_PATH, COPILOT_CLI_PATH, AUTH_TOKEN, DB_PATH
  data/
    agent-deck.db       SQLite database (auto-created)
  server/
    index.js            Express + ws bootstrap
    routes/             REST handlers
    ws/                 WebSocket message handlers
    pty/                PTY manager (spawn/write/kill/stream)
    orchestrator/       Super-Master loop + pipeline state machine
    db/                 SQLite schema + query helpers
    agents/             Agent registry, role configs, prompt templates
  client/
    src/
      components/
      stores/
      views/
    index.html
    vite.config.js      proxies /api and /ws to localhost:3001
```

## npm scripts
```
npm run dev    → concurrently runs server (port 3001) + vite (port 5173)
npm run build  → vite builds SPA into server/public/
npm start      → production: one process, port 3001
npm run db:reset → drops and recreates SQLite schema
```

## Key architectural rules
1. node-pty is the ONLY way agents run. No Claude API calls. No child_process.exec.
2. The orchestrator is a setInterval(tick, 10000) loop in the same process as Express.
3. Agents communicate ONLY via stdout signals — never direct agent-to-agent calls.
4. SQLite reads/writes are synchronous (better-sqlite3). Keep queries simple.
5. WebSocket sends full state:snapshot on every new connection.
6. Frontend never holds optimistic state for mission/agent status — server is truth.

## Agent signal protocol (stdout scanner watches for these)
| Signal prefix   | Meaning                        |
|-----------------|--------------------------------|
| DONE: <summary> | Task complete                  |
| BLOCKED: <why>  | Cannot proceed                 |
| STATUS: <what>  | Heartbeat, no action needed    |
| VERDICT: PASS   | Reviewer approved              |
| VERDICT: FAIL   | Reviewer rejected              |
| MEMORY: {json}  | Agent requests memory update   |
| CONFLICT: <why> | Integrator found merge conflict|

## Agent roles
SM (Super-Master), EX (Explorer), AR (Architect), DE (Developer),
RV (Reviewer), TE (Tester), IN (Integrator), RL (Releaser)

## SQLite tables
missions, stages, agents, messages, agent_logs
See docs/PRD04_backend.md §5 for full schema.

## Full specifications
- docs/PRD01_master.md     — goals, stack rationale, milestones
- docs/PRD02_agents.md     — roles, CLI invocation, prompts, memory, signals
- docs/PRD03_orchestration.md — pipeline state machine, watchdog, escalation
- docs/PRD04_backend.md    — REST API, WebSocket protocol, PTY manager, SQLite schema
- docs/PRD05_frontend.md   — all views, xterm.js setup, Zustand stores, components

## Current milestone
See TASKS.md for the active task list.
When a task is complete, mark it [x] in TASKS.md before moving to the next.

---
> Source: [manudinicolacorilus/agent-deck](https://github.com/manudinicolacorilus/agent-deck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
