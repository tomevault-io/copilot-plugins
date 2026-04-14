## claw-approve

> Remote approval system for Claude Code permissions. Intercepts tool calls via a

# ClawApprove

Remote approval system for Claude Code permissions. Intercepts tool calls via a
PreToolUse hook, classifies risk, and routes medium/high-risk operations to a
server where connected devices (phone, watch, browser) can approve or deny.

## Stack

- **Server**: Node.js + Express + Socket.io + Prisma (SQLite) -- `packages/server/`
- **Web**: Next.js 14 + Tailwind CSS + PWA -- `packages/web/`
- **iOS**: SwiftUI (iOS 17+) -- `ios/`
- **watchOS**: SwiftUI (watchOS 10+) -- `ios/`
- **Hook**: Bash (Claude Code PreToolUse hook) -- `packages/hook/`

## Development

- pnpm monorepo (`pnpm-workspace.yaml` includes `packages/*`)
- `pnpm dev` -- starts server (port 3200) and web (port 3201)
- `pnpm build` -- builds server and web
- `pnpm db:push` -- sync Prisma database schema
- `pnpm db:studio` -- open Prisma Studio
- `pnpm hook:install` -- install the Claude Code hook

## Conventions

- TypeScript strict mode
- Functional components with hooks (React)
- Zod for request validation
- nanoid for IDs
- ISO 8601 timestamps
- JWT for auth (device tokens expire 365d, session tokens expire 30d)

## API Endpoints

### Health

| Method | Path      | Auth | Description          |
|--------|-----------|------|----------------------|
| GET    | `/health` | None | Server health status |

Returns: `{ status: "ok", uptime: number, timestamp: string }`

### Hook Routes (`/api/hook/`)

These endpoints are called by the bash hook script running on the developer machine.

| Method | Path                  | Auth           | Description                              |
|--------|-----------------------|----------------|------------------------------------------|
| POST   | `/api/hook/register`  | None           | Register a new Claude Code session       |
| POST   | `/api/hook/approve`   | Bearer session | Submit approval request (long-polling)   |
| POST   | `/api/hook/heartbeat` | Bearer session | Keep session marked as active            |
| POST   | `/api/hook/disconnect`| Bearer session | Mark session as inactive                 |

#### POST /api/hook/register

Body: `{ name: string, hostname: string, platform: string, pid: number }`
Returns: `{ sessionId: string, token: string }`

#### POST /api/hook/approve

Body: `{ tool: string, input: unknown }`
Headers: `Authorization: Bearer <session-token>`
Returns (long-polls until decision or timeout):
- `{ decision: "approve" }` -- approved by user or auto-approved
- `{ decision: "deny", reason: string }` -- denied by user or timed out

#### POST /api/hook/heartbeat

Headers: `Authorization: Bearer <session-token>`
Returns: `{ ok: true }`

#### POST /api/hook/disconnect

Headers: `Authorization: Bearer <session-token>`
Returns: `{ ok: true }`

### Device/API Routes (`/api/`)

These endpoints are used by the web dashboard and mobile apps. All require device
auth (`Authorization: Bearer <device-token>`) unless noted.

| Method | Path                        | Auth           | Description                          |
|--------|-----------------------------|----------------|--------------------------------------|
| POST   | `/api/devices/pair`         | None           | Start device pairing (returns QR)    |
| POST   | `/api/devices/confirm`      | None           | Confirm pairing with 6-digit code    |
| GET    | `/api/devices`              | Bearer device  | List paired devices                  |
| DELETE | `/api/devices/:id`          | Bearer device  | Unpair a device                      |
| GET    | `/api/sessions`             | Bearer device  | List Claude Code sessions            |
| GET    | `/api/sessions/:id`         | Bearer device  | Session detail with recent requests  |
| GET    | `/api/requests`             | Bearer device  | List approval requests (filterable)  |
| GET    | `/api/requests/pending`     | Bearer device  | Current pending requests             |
| POST   | `/api/requests/:id/decide`  | Bearer device  | Approve/deny via REST                |
| POST   | `/api/push/subscribe`       | Bearer device  | Save push subscription for device    |
| GET    | `/api/stats`                | Bearer device  | Dashboard statistics                 |

#### POST /api/devices/pair

Body: `{ name: string, platform: "ios"|"web"|"watch"|..., serverUrl?: string }`
Returns: `{ deviceId, code, qrCode, qrPayload, expiresIn }`

#### POST /api/devices/confirm

Body: `{ code: string (6 digits), name: string, platform: string }`
Returns: `{ deviceId, token, name, platform }`

#### POST /api/requests/:id/decide

Body: `{ decision: "approve"|"deny", reason?: string }`
Returns: `{ ok: true, status: "APPROVED"|"DENIED", hookConnected: boolean }`

#### GET /api/requests

Query params: `?status=PENDING|APPROVED|DENIED|TIMEOUT|AUTO&sessionId=...&limit=50&offset=0`

## WebSocket Events

WebSocket connections require device auth via `socket.handshake.auth.token`.

### Server -> Client Events

| Event                | Payload                                                        | Description                          |
|----------------------|----------------------------------------------------------------|--------------------------------------|
| `approval:pending`   | `{ id, sessionId, sessionName, hostname, tool, input, createdAt }` | New approval request awaiting decision |
| `approval:decided`   | `{ requestId, decision, decidedBy, decidedAt, reason }`       | Request was decided                  |
| `approval:error`     | `{ requestId, error }`                                        | Error processing decision            |
| `device:connected`   | `{ deviceId, online: boolean }`                                | Device connected/disconnected        |

### Client -> Server Events

| Event                | Payload                                                  | Description                  |
|----------------------|----------------------------------------------------------|------------------------------|
| `approval:decide`    | `{ requestId: string, decision: "approve"|"deny", reason?: string }` | Submit approval decision |

## Hook Configuration

### Installation

```bash
bash packages/hook/install.sh
```

The installer:
1. Checks prerequisites (curl, bash)
2. Prompts for server URL (or uses `CLAW_APPROVE_URL` env var)
3. Registers a session with the server, gets a session token
4. Saves config to `~/.claw-approve/config`
5. Adds the hook to `~/.claude/settings.local.json` under `hooks.PreToolUse`

### Uninstallation

```bash
bash packages/hook/uninstall.sh
```

### Config File (`~/.claw-approve/config`)

```
CLAW_APPROVE_URL=https://your-server.example.com
CLAW_APPROVE_TOKEN=<jwt-session-token>
```

### Risk Classification

The hook classifies each tool call into a risk level:

| Risk   | Tools                                        | Behavior                         |
|--------|----------------------------------------------|----------------------------------|
| Low    | Read, Glob, Grep, WebSearch, WebFetch, etc.  | Auto-approved, no server call    |
| Medium | Edit, Write, NotebookEdit                    | Requires server approval         |
| High   | Bash (non-readonly), unknown tools           | Requires server approval         |

For Bash tools, the hook analyzes the command to determine if it is read-only
(e.g., `ls`, `cat`, `git status`) or potentially destructive (e.g., `rm`, `git push`).

### Fail-Open Policy

If the server is unreachable or returns an error, the hook auto-approves the
request with a warning logged to stderr and the audit log. This prevents the
server from blocking Claude Code work if it goes down.

### Audit Log

All decisions are logged to `~/.claw-approve/audit.log` with timestamps, tool
names, risk levels, and decision details.

## Environment Variables

### Server

| Variable             | Required | Default                                    | Description                        |
|----------------------|----------|--------------------------------------------|------------------------------------|
| `PORT`               | No       | `3200`                                     | HTTP server port                   |
| `HOST`               | No       | `0.0.0.0`                                  | Bind address                       |
| `NODE_ENV`           | No       | `development`                              | Node environment                   |
| `DATABASE_URL`       | No       | `file:./dev.db`                            | Prisma database URL (SQLite)       |
| `JWT_SECRET`         | **Yes**  | --                                         | Secret for signing JWTs            |
| `CORS_ORIGINS`       | No       | `http://localhost:5173,http://localhost:3000` | Comma-separated allowed origins |
| `VAPID_PUBLIC_KEY`   | No       | (empty)                                    | Web Push VAPID public key          |
| `VAPID_PRIVATE_KEY`  | No       | (empty)                                    | Web Push VAPID private key         |
| `VAPID_SUBJECT`      | No       | `mailto:admin@example.com`                 | Web Push VAPID subject             |
| `APPROVAL_TIMEOUT_MS`| No      | `120000`                                   | Timeout for approval requests (ms) |
| `TIMEOUT_ACTION`     | No       | `timeout`                                  | Action on timeout: `timeout` or `auto_approve` |

### Hook (environment overrides)

| Variable             | Description                          |
|----------------------|--------------------------------------|
| `CLAW_APPROVE_URL`   | Server URL (overrides config file)   |
| `CLAW_APPROVE_TOKEN` | Session token (overrides config file)|

### Docker Compose

| Variable             | Default                      | Description                  |
|----------------------|------------------------------|------------------------------|
| `SERVER_PORT`        | `3200`                       | Host port for server         |
| `WEB_PORT`           | `3201`                       | Host port for web dashboard  |
| `JWT_SECRET`         | (required)                   | Secret for signing JWTs      |
| `CORS_ORIGINS`       | `http://localhost:3201`      | Allowed CORS origins         |
| `NEXT_PUBLIC_API_URL`| `http://localhost:3200`      | API URL for web client       |
| `NEXT_PUBLIC_WS_URL` | `ws://localhost:3200`        | WebSocket URL for web client |

## Docker

```bash
# Build and start
JWT_SECRET=your-secret docker compose up -d

# Rebuild after changes
docker compose build && docker compose up -d
```

The server uses a named volume (`claw-data`) for SQLite persistence at `/data/`.

## Deploy
- **Auto-deploy**: Push to `main` → GitHub Actions → SSH → pnpm install + build + systemctl restart
- **Infra repo**: `pabesfu/infra` — owns nginx config
- **Server path**: `/opt/claw-approve/` (git clone of this repo)
- **Services**: `claw-server.service` (API, port 3200), `claw-web.service` (Next.js, port 3201)
- **Nginx config**: `pabesfu/infra` → `nginx/sites/claw`
- **Domain**: claw.pabesfu.com
- To change nginx: edit in `pabesfu/infra` repo
- To change app code: edit here, push to main → auto-deploys

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pabesfu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pabesfu)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
