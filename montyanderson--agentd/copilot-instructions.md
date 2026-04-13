## agentd

> Multi-agent thread daemon that manages concurrent coding agent threads using pi-mono. Threads persist to disk as NDJSON. REST+SSE API with a React frontend chat UI.

# agentd

Multi-agent thread daemon that manages concurrent coding agent threads using pi-mono. Threads persist to disk as NDJSON. REST+SSE API with a React frontend chat UI.

## project conventions

- For readme/markdown files, use **lower case** filenames (e.g. `readme.md`, not `README.md`)
- Keep this CLAUDE.md updated as the project evolves — add new architectural decisions, key patterns, and important file paths as they emerge

## code style

Prettier config: tabs, no trailing commas, double quotes, semicolons. See `.prettierrc`.

- Zod schemas on every I/O boundary (API requests, disk reads, SSE payloads)
- No `any` — strict types everywhere, derive from Zod schemas where possible
- Prefer arrow functions (`const foo = () => {}`)

## architecture

```
server/
  index.ts              — Bun.serve entry point (port 4747)
  app.ts                — Hono app: routes, CORS, static serving
  routes/threads.ts     — REST CRUD + send message + abort
  routes/events.ts      — SSE streaming via Hono streamSSE
  thread/types.ts       — Zod schemas, defaults, type definitions
  thread/thread-store.ts    — Disk I/O (data/threads/{uuid}/metadata.json + conversation.ndjson)
  thread/thread-handle.ts  — Wraps pi-mono Agent + AgentSession + SSE fan-out
  thread/thread-manager.ts — In-memory registry, lazy loading, CRUD

web/                    — React 19 + Vite + Tailwind v4
  src/api/client.ts     — Fetch helpers + EventSource SSE
  src/hooks/            — useThreads, useThread
  src/components/       — Sidebar, ThreadView, MessageBubble, StreamingMessage
```

Data dir: `data/threads/{uuid}/` with `metadata.json` + `conversation.ndjson`

## pi-mono integration

Key packages: `@mariozechner/pi-agent-core`, `@mariozechner/pi-coding-agent`, `@mariozechner/pi-ai` (all v0.27.2)

- `createCodingTools(cwd)` — creates tools bound to a specific directory (true parallelism, no process.chdir)
- `SessionManager.open(path)` — opens a specific NDJSON session file
- `SettingsManager.inMemory()` — in-memory settings (no disk config)
- `AgentSession.prompt(text)` when idle, `.queueMessage(text)` when streaming
- `AgentSession.abort()` to cancel, `.dispose()` to clean up
- `session.subscribe(listener)` for event fan-out

## running

```sh
bun run dev          # server on :4747 (with --watch)
bun run dev:web      # vite dev server on :5173 (proxies /api to :4747)
bun run build:web    # build frontend to web/dist
bun run start        # production: serves API + static on :4747
```

## defaults (overridable via env)

- `AGENTD_PROVIDER` = anthropic
- `AGENTD_MODEL` = claude-opus-4-20250514
- `AGENTD_PORT` = 4747
- `AGENTD_DATA_DIR` = data

## api endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/threads` | List all threads |
| POST | `/api/threads` | Create thread |
| GET | `/api/threads/:id` | Thread detail + messages |
| DELETE | `/api/threads/:id` | Delete thread |
| POST | `/api/threads/:id/messages` | Send message (returns 202) |
| POST | `/api/threads/:id/abort` | Abort current run |
| GET | `/api/threads/:id/events` | SSE stream |

## bun

Default to using Bun instead of Node.js. Bun automatically loads .env.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/montyanderson)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/montyanderson)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
