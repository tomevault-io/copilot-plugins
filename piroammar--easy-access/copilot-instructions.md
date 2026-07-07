## easy-access

> - Keep responses concise and to the point - unless the user asks otherwise

# CRITICAL RULES - MUST FOLLOW

## RESPONSES

- Keep responses concise and to the point - unless the user asks otherwise

## DATABASE

- NEVER use Prisma or any ORM — raw `pg` (node-postgres) only
- NEVER interpolate user input into SQL strings — always use `$1, $2, ...` placeholders
- Always use the shared pool from `hub/lib/db.ts` — never create a new Pool
- Use snake_case in SQL, camelCase in TypeScript. Bridge via `AS "camelCase"` aliases in SELECT

## SECURITY (NON-NEGOTIABLE)

- NEVER allow `../` or path traversal in file operations
- ALL file paths must be resolved against the server's `allowedDirs` via `validatePath()` (`agent/src/security.ts`)
- Agent tokens are unique per server, required for WebSocket auth, NEVER logged
- File content is limited to 50 MB — always check size before reading
- NEVER use shell commands (exec/spawn) for file operations — Node.js `fs` only
- NEVER recursive delete (`rmSync({ recursive: true })`) — only files and empty dirs
- The agent's local UI server must bind to 127.0.0.1 only

## TESTING

- Never assume changes simply work — verify end-to-end (hub + agent + Postgres locally)

## UI DESIGN

- When creating new components, first check if there is an existing component that can be used
- CSS: Vanilla CSS only. No Tailwind. All tokens are in `hub/styles/globals.css`

---

# Easy Access — Agent Instructions

## Project Overview

Easy Access is a **self-hosted remote server/PC data management platform**.
- Single admin, 2–5 connected machines
- The admin browses, uploads, downloads, and previews files on remote machines via a web dashboard
- Step-by-step setup: `DEPLOYMENT.md`

## Components

1. **Hub** (`hub/`) — Next.js 15 App Router full-stack app, deployed to Railway (Root Directory = `hub`)
2. **Agent** (`agent/`) — Node.js TypeScript service on each remote machine, with a local setup UI at http://localhost:4400
3. **Shared code** — duplicated in `hub/shared/` (aliased `@easy-access/shared`) and `agent/src/shared/` (relative imports); keep both copies in sync when the protocol changes

## Architecture

```
Browser → Next.js Hub → WebSocket Server (/ws) → Agent → filesystem
```

- Two standalone npm projects — no workspace/monorepo tooling
- Agents connect **OUTBOUND** to the hub via WebSocket (wss://)
- Database: PostgreSQL via node-postgres (raw SQL, parameterized queries)
- Auth: NextAuth.js v5 with Credentials provider (single admin, seeded from ADMIN_USERNAME/ADMIN_PASSWORD env on boot)
- Migrations + admin seeding run automatically on hub startup (`hub/lib/bootstrap.ts`)
- The ConnectionManager and pg Pool are cached on `globalThis` unconditionally — required because the custom server (tsx) and Next-bundled API routes are separate module graphs. Do not remove.

## Code Standards

- **TypeScript strict mode** everywhere
- **API responses**: `{ success: boolean, data?: T, error?: string }`
- **WebSocket messages**: follow the `WSMessage<T>` schema in `hub/shared/protocol.ts`
- **Next.js**: Server components by default, client only when interactive
- **Imports**: `@/` alias for `hub/`, `@easy-access/shared` for shared code (hub only)

## Project Structure

```
easy-access/
├── hub/                          # Next.js central hub (deploys to Railway)
│   ├── app/                      # App Router pages + API routes
│   ├── components/               # React components
│   ├── lib/
│   │   ├── auth.ts               # NextAuth v5 config
│   │   ├── db.ts                 # PostgreSQL pool (single pool, global cache)
│   │   ├── bootstrap.ts          # startup migrations + admin upsert
│   │   ├── ws-server.ts          # WebSocket server (import ONLY from server.ts)
│   │   └── connection-manager.ts # Track connected agents, pair requests
│   ├── db/
│   │   ├── queries.ts            # All typed query functions
│   │   └── migrations/           # Numbered SQL migration files
│   ├── shared/                   # Protocol/types/utils (copy A)
│   ├── styles/globals.css        # Design tokens + reset
│   ├── server.ts                 # Custom Next.js server (WebSocket upgrade)
│   ├── railway.toml              # Railway build/deploy config
│   └── .env.local.example        # Documented env vars (local + Railway)
│
└── agent/                        # Remote agent (runs on user machines)
    └── src/
        ├── index.ts              # Entry point (starts UI + connection)
        ├── manager.ts            # Connection lifecycle (config changes → reconnect)
        ├── ui-server.ts          # Local setup/status UI on 127.0.0.1:4400
        ├── connection.ts         # WebSocket client → hub
        ├── file-ops.ts           # Secure file system operations
        ├── security.ts           # validatePath() — MANDATORY
        ├── config.ts             # ~/.easy-access-agent/config.json
        └── shared/               # Protocol/types/utils (copy B)
```

## File Operation Safety Pattern

```typescript
// ALWAYS use validatePath before any fs operation in the agent
import { validatePath, SecurityError } from './security';

const safePath = validatePath(userProvidedPath, config.allowedDirs);
// Now it's safe to use safePath with fs
```

## WebSocket Auth Flow

```
Agent                          Hub
  |--- agent:auth (token) ------->|
  |<-- hub:auth-ok (serverId) ----|   ← connection registered
  |--- agent:heartbeat (30s) ---->|   ← loop forever
  |<-- hub:list-dir (requestId) --|
  |--- agent:file-list ---------->|
```

Error responses (`agent:error` with a `requestId`) REJECT the hub's pending request — never resolve it.

## Running Commands

```bash
# Hub (from hub/)
npm install
docker compose up -d      # local Postgres on port 5434
npm run dev               # http://localhost:3000
npm run build && npm run start   # production
npm run type-check

# Agent (from agent/)
npm install
npm start                 # UI at http://localhost:4400
```

---
> Source: [PiroAmmar/easy-access](https://github.com/PiroAmmar/easy-access) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
