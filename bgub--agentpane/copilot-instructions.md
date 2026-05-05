## agentpane

> Web UI for AI coding agents. `npx agentpane` starts the API server (port 3456) and a Next.js frontend (port 6767). No separate deployment — everything runs locally.

# AgentPane

Web UI for AI coding agents. `npx agentpane` starts the API server (port 3456) and a Next.js frontend (port 6767). No separate deployment — everything runs locally.

Communication with agents uses ACP (Agent Client Protocol) — JSON-RPC 2.0 over stdio.

## Quick Start

```sh
npx agentpane          # starts API (:3456) + frontend (:6767)
```

For development: `npm run dev` starts both Next.js dev server (port 6767) and Hono API server (port 3456) via Turbo. Next.js rewrites `/api` requests to localhost:3456.

## Tech Stack

- **Monorepo:** Turborepo + npm workspaces (`apps/*`)
- **Backend (`apps/server`):** Hono + Effect.ts services/layers, Node.js server on port 3456. Published to npm as `agentpane`.
- **Frontend (`apps/web`):** Next.js 16 (App Router, standalone output), React 19, Tailwind CSS 4, TanStack React Query.
- **Docs (`apps/docs`):** Astro + Starlight.
- **Database:** SQLite via `@effect/sql-sqlite-node` + `better-sqlite3` at `~/.agentpane/agentpane.db`
- **ACP:** `@agentclientprotocol/sdk` + `@zed-industries/claude-agent-acp` + `@zed-industries/codex-acp`

## Architecture

### Two-Process Production Server

`bin/agentpane.js` starts two processes: the Hono API server on :3456 (imported directly) and a forked Next.js standalone server on :6767. In development, the Next.js dev server on :6767 rewrites `/api/*` to :3456. No auth or CORS — everything is same-origin.

### Backend (`apps/server/`)

Hono HTTP server with Effect.ts service layers composed into `ManagedRuntime`:

- `src/index.ts` — Entry point, Hono app + `serve()` on port 3456
- `src/lib/schema.ts` — Effect Schema classes for `Session`, `Turn`, `MessageBlock`, and error types
- `src/lib/db.ts` — `SqliteLive` layer (SQLite connection, migrations, WAL mode)
- `src/lib/runtime.ts` — `AppRuntime = ManagedRuntime.make(AppLayer)`
- `src/lib/session-repo.ts` — `SessionRepo` service (CRUD for sessions, turns, message blocks, settings)
- `src/lib/acp-client.ts` — `AcpClient` facade composing ConnectionManager, EventHub, PromptEngine, WriteQueue
- `src/lib/connection-manager.ts` — Agent subprocess lifecycle, ACP connection setup, session load/resume
- `src/lib/prompt-engine.ts` — Prompt execution orchestration, turn persistence, token tracking
- `src/lib/event-hub.ts` — Session-level `EventBroadcaster` management with idle cleanup
- `src/lib/event-broadcaster.ts` — Ring buffer SSE implementation (replay on reconnect via `Last-Event-ID`)
- `src/lib/write-queue.ts` — Batched DB writes (50ms debounce per session, crash recovery)
- `src/lib/write-ops.ts` — Write operation types for the queue
- `src/lib/acp-client-callbacks.ts` — Agent→client callbacks (sessionUpdate, file I/O, terminal management)
- `src/lib/acp-types.ts` — ACP protocol interfaces
- `src/lib/providers.ts` — Agent provider config + binary resolution (walks up `node_modules/.bin/` for npm hoisting compatibility)
- `src/routes/sessions.ts` — All Hono route handlers
- `src/routes/validation.ts` — Request validation helpers

### Layer Composition

```
AppLayer =
  AcpClient.layer
  → PromptEngine.layer
  → ConnectionManager.layer
  → EventHub.layer
  → WriteQueue.layer
  → SessionRepo.layer
  → SqliteLive
```

### API Routes (all in `apps/server/src/routes/sessions.ts`)

```
GET    /api/health                        → health check
GET    /api/metrics                       → uptime, memory, ACP stats
GET    /api/git-branch?cwd=...            → current git branch
GET    /api/settings/:key                 → get setting
PUT    /api/settings/:key                 → save setting

GET    /api/sessions                      → list sessions
POST   /api/sessions                      → create (optionally auto-connect)
GET    /api/sessions/status               → { connected: id[], prompting: id[] }
GET    /api/sessions/:id                  → get session
PATCH  /api/sessions/:id                  → rename session
DELETE /api/sessions/:id                  → disconnect + delete

GET    /api/sessions/:id/conversation     → turns with message blocks
GET    /api/sessions/:id/token-usage      → aggregated token counts
POST   /api/sessions/:id/prompt           → send prompt (text, images, resource links)
POST   /api/sessions/:id/cancel           → cancel in-progress prompt
POST   /api/sessions/:id/permission       → respond to permission request

GET    /api/sessions/:id/commands         → available slash commands
GET    /api/sessions/:id/config           → config options
POST   /api/sessions/:id/config           → set config option
GET    /api/sessions/:id/mode             → current mode
POST   /api/sessions/:id/mode             → set mode
GET    /api/sessions/:id/agent-sessions   → agent's session history

POST   /api/sessions/:id/connect          → connect agent
DELETE /api/sessions/:id/connect          → disconnect agent
GET    /api/sessions/:id/events           → SSE stream (reconnect-safe via Last-Event-ID)
```

### Frontend (`apps/web/`)

Next.js 16 App Router — pure UI, no backend dependencies. Same-origin with the API via rewrites.

- `src/app/layout.tsx` — Root layout with providers (theme, React Query)
- `src/app/page.tsx` — Main page with SSR prefetching of sessions + conversations
- `src/App.tsx` — Client root: SessionProvider + LayoutProvider → Sidebar + PaneContainer
- `src/lib/api.ts` — Backend API client (relative URLs, plain fetch)
- `src/lib/queries.ts` — React Query hooks (`useSessionsQuery`, `useConversationQuery`, mutations)
- `src/lib/types.ts` — API response types
- `src/lib/layout-types.ts` / `layout-utils.ts` — Multi-pane layout types and state helpers
- `src/components/session-provider.tsx` — Session state (active session, health polling, setup screen)
- `src/components/layout-provider.tsx` — Multi-pane layout state (up to 4 panes, tabs, resize, persisted to backend)
- `src/components/sidebar.tsx` — Session list with Active/History sections, drag-and-drop
- `src/components/pane-container.tsx` — Resizable panel grid
- `src/components/pane-view.tsx` — Single pane with chat
- `src/components/tab-bar.tsx` — Tabs within a pane
- `src/components/chat-header.tsx` — Cwd, git branch, connect button, config dropdowns
- `src/components/chat-footer.tsx` — Prompt input with slash command autocomplete
- `src/components/use-pane-session.ts` — Hook for per-pane session state (connection, config, commands, modes)
- `src/components/chat-view/index.tsx` — Chat display with always-on SSE
- `src/components/chat-view/streaming.ts` — SSE event reducer (optimistic turns, streaming blocks)
- `src/components/chat-view/tool-call-box.tsx` — Tool execution display with permission UI
- `src/components/chat-view/code-block.tsx` — Syntax highlighting (Shiki via Streamdown)
- `src/components/chat-view/plan-view.tsx` — Plan visualization
- `src/components/chat-view/markdown.ts` — Streamdown markdown plugins
- `src/components/session-setup-screen.tsx` — Agent selection + MCP config

## Key Patterns

### Backend

- Effect services use `Context.Tag` class pattern with static `layer` properties
- Errors use `Schema.TaggedError` with `httpStatus`/`httpMessage` getters for self-describing HTTP responses
- Service methods use `Effect.fn` for call-site tracing
- Hono routes bridge Effect via `runEffect(c, effect, status?)` — centralizes exit matching and error-to-HTTP mapping
- For routes without typed errors (DELETE, cancel), use `AppRuntime.runPromise` directly
- WriteQueue batches rapid writes (50ms debounce, max 5000 ops/2MB per session), recovers from crashes via DB-persisted ops
- EventHub manages session-level broadcasters that survive agent disconnects, with 5min idle cleanup
- EventBroadcaster supports SSE replay (512KB/1000 events ring buffer) for `Last-Event-ID` reconnection
- ConnectionManager spawns agent binaries, negotiates ACP capabilities, attempts `loadSession` for resumption
- PromptEngine orchestrates user→agent turns: creates turns, enqueues blocks via WriteQueue, streams responses
- No auth or CORS middleware — everything is same-origin

### Frontend

- **No manual memoization** — React Compiler (`babel-plugin-react-compiler`) handles all memoization automatically. Never use `useMemo`, `useCallback`, or `React.memo`.
- React Query for all server state — SSR prefetch with dehydration, query invalidation on SSE events
- EventSource is always-on (no `if (!connected) return` guard)
- Multi-pane layout with drag-and-drop (sidebar→pane copies session, pane→pane moves tab)
- Layout state auto-persisted to backend via `/api/settings/layout`
- Setup screen lives in `PaneView`, not `ChatView` — no DB session until user clicks Start
- Sidebar splits sessions into Active (connected, with status dots) and History (disconnected, muted)
- Streaming markdown via Streamdown with Shiki syntax highlighting
- No auth tokens — API calls use plain `fetch` with relative URLs

### Distribution

- Agent binaries (`claude-agent-acp`, `codex-acp`) are resolved by walking up from the package root checking each `node_modules/.bin/` directory. This handles both dev mode (binary in `apps/server/node_modules/.bin/`) and npm-installed mode (binary hoisted to consumer's top-level `node_modules/.bin/`). Never use hardcoded `node_modules` paths for binary resolution.

### Build Pipeline

- `npm run build` — Turbo builds `@agentpane/web` first (`next build` → `.next/standalone`), then server (`tsc` + copies standalone to `apps/server/web/`)
- `apps/server/web/` is a build artifact (gitignored), included in npm publish via `files` field
- `bin/agentpane.js` — imports API server + forks Next.js standalone server
- **Testing distribution:** `cd apps/server && npm pack`, then `mkdir /tmp/test && cd /tmp/test && npm install <path-to-tgz> && npx agentpane`

### Environment Variables

| Variable | Default | Purpose |
|---|---|---|
| `AGENTPANE_DATA_DIR` | `~/.agentpane` | SQLite + config directory |
| `AGENTPANE_DEBUG_STREAM` | (unset) | When `"1"`, logs stdout chunks to console |

<!-- effect-solutions:start -->
## Effect Best Practices

**IMPORTANT:** Always consult effect-solutions before writing Effect code.

1. Run `effect-solutions list` to see available guides
2. Run `effect-solutions show <topic>...` for relevant patterns (supports multiple topics)
3. Search `.reference/effect/` for real implementations (run `effect-solutions setup` first)

Topics: quick-start, project-setup, tsconfig, basics, services-and-layers, data-modeling, error-handling, config, testing, cli.

Never guess at Effect patterns - check the guide first.
<!-- effect-solutions:end -->

---
> Source: [bgub/agentpane](https://github.com/bgub/agentpane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
