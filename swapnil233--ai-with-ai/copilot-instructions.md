## ai-with-ai

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is this app?

This is an **AI-powered no-code application builder**. Users create applications by describing what they want in natural language (chat interface) rather than traditional drag-and-drop.

**How it works:**

- Users chat with an LLM to describe the app they want to build
- Each project gets its own isolated sandbox environment powered by **Modal**
- The AI generates and deploys the application within that sandbox
- Authentication handled via **better-auth** (same-origin on Next.js)

**Key integrations:**

- **Modal** - Sandbox environments for running user-created applications
- **OpenAI API** - LLM provider for understanding user intent and generating code
- **better-auth** - User authentication and authorization (runs in Next.js)
- **MinIO (S3-compatible)** - Local object storage for generated assets/artifacts

## Build & Development Commands

```bash
# Development (starts Docker, FastAPI, and Next.js — single command)
pnpm dev

# Build all packages
pnpm build

# Lint and format
pnpm lint
pnpm format

# Testing
pnpm test                                    # Run all JS/TS tests once
pnpm test:watch                              # Watch mode
pnpm --filter @ai-app-builder/web test       # Web tests only

# Python API tests
cd services/api && source .venv/bin/activate && pytest tests/ -v

# Run a single test file
pnpm --filter @ai-app-builder/web test src/lib/utils.test.ts

# Adding UI components (shadcn)
cd apps/web && npx shadcn@latest add [component-name]

# Database (requires DATABASE_URL in .env)
pnpm db:start                                        # Start PostgreSQL + MinIO
pnpm db:generate                                     # Generate Prisma client
pnpm db:push                                         # Push schema to database
pnpm db:migrate                                      # Run migrations
pnpm --filter @ai-app-builder/database db:studio     # Open Prisma Studio
```

## Architecture

```
Browser → Next.js (:3000)
            ├─ /api/auth/*       → better-auth (same-origin, Prisma via packages/database)
            ├─ /api/chat         → Vercel AI SDK → OpenAI
            │                       └─ tool calls → FastAPI (:4000) /sandbox/*
            ├─ /api/projects/*   → rewrite proxy → FastAPI (:4000)
            ├─ /api/user/*       → rewrite proxy → FastAPI (:4000)
            ├─ /api/security/*   → rewrite proxy → FastAPI (:4000)
            └─ /sandbox/*        → rewrite proxy → FastAPI (:4000)
                                      ├─ /api/projects, /api/user, /api/security
                                      ├─ /sandbox/* (create, write-files, run-command, tunnel-url, terminate)
                                      └─ Socket.io (python-socketio)
```

**Monorepo Structure:**

- `apps/web` - Next.js 16 frontend + better-auth (port 3000)
- `services/api` - FastAPI backend with Socket.io (port 4000)
- `packages/database` - Shared Prisma schema, migrations, and client singleton
- `packages/shared` - Shared TypeScript types for API contracts

**Key Technologies:**

- **Frontend**: Next.js App Router, React 19, Tailwind CSS v4, shadcn/ui components, Vercel AI SDK
- **Backend**: FastAPI (Python 3.12), python-socketio for real-time, SQLAlchemy (async) for DB access
- **Database**: PostgreSQL — Prisma owns schema/migrations, SQLAlchemy reads/writes the same tables
- **Storage**: MinIO (S3-compatible) for object storage
- **AI/Sandbox**: OpenAI API (gpt-5.2), Modal for isolated sandbox environments
- **Auth**: better-auth in Next.js (same-origin cookies), FastAPI validates session cookie directly
- **Build**: pnpm workspaces + Turborepo for task orchestration and caching

**Path Aliases:**

- Web uses `@/*` → `./src/*` (e.g., `import { cn } from "@/lib/utils"`)

**API Proxying:** Next.js rewrites in `next.config.ts` proxy `/api/projects`, `/api/user`, `/api/security`, `/sandbox`, and `/health` from `:3000` to FastAPI at `:4000`. This keeps all browser requests same-origin, avoiding cross-origin cookie issues. Frontend fetch calls use relative URLs (no explicit host/port).

**FastAPI Structure (`services/api/`):**

- `main.py` - FastAPI app + middleware stack + Socket.io ASGI mount
- `config.py` - Pydantic Settings (reads `../../.env`, converts `DATABASE_URL` to async format)
- `models/` - SQLAlchemy models (mirror Prisma schema exactly — same table names, camelCase columns)
- `routes/` - Route handlers (health, user, projects, security, sandbox)
- `middleware/` - Security headers, rate limiting, CSRF, request ID
- `dependencies/` - Auth (session cookie validation), DB session
- `services/` - Sandbox manager (Modal), audit logging
- `ws/` - python-socketio server with session-based auth
- `tests/` - pytest + httpx AsyncClient + aiosqlite (in-memory SQLite)

**Database Sharing:** Prisma owns DDL (schema/migrations) in `packages/database`. SQLAlchemy in FastAPI reads/writes the same PostgreSQL tables. Prisma uses `postgresql://` from `DATABASE_URL`; FastAPI's `config.py` auto-converts to `postgresql+asyncpg://` and strips `?schema=public`. No Alembic needed.

## Key Patterns

### Auth: better-auth Session Cookies

better-auth runs in Next.js and sets a signed cookie `better-auth.session_token`. The cookie value format is `TOKEN.HMAC_SIGNATURE` (HMAC-SHA256 signed with `BETTER_AUTH_SECRET`). The database stores the **plain token** (not hashed).

When FastAPI validates a session:

1. Read the `better-auth.session_token` cookie from the request
2. URL-decode the value, then split on the **last** `.` to extract the plain token (strip the HMAC signature)
3. Query the `Session` table with the plain token
4. Check `expiresAt` is in the future

See `dependencies/auth.py` → `_extract_token()`. The same logic applies in `ws/server.py` for Socket.io auth.

**Do NOT hash the token** before querying — the DB stores plain tokens. Do NOT use the full signed cookie value — strip the `.SIGNATURE` suffix first.

### Datetime Columns (Prisma ↔ asyncpg)

Prisma uses `TIMESTAMP WITHOUT TIME ZONE` by default. asyncpg is strict: it rejects timezone-aware Python datetimes for these columns. Always strip tzinfo before writing:

```python
now = datetime.now(timezone.utc).replace(tzinfo=None)  # naive UTC
```

When **comparing** datetimes read from the DB (e.g., `session.expiresAt`), add tzinfo back: `session.expiresAt.replace(tzinfo=timezone.utc)`.

### CSRF Middleware Exemptions

The CSRF middleware in `middleware/csrf.py` skips:

- Safe methods (`GET`, `HEAD`, `OPTIONS`)
- Auth paths (`/api/auth/*`) — handled by better-auth
- Sandbox paths (`/sandbox/*`) — server-to-server calls from Next.js chat tools (no browser cookies/CSRF tokens)

### Sandbox Tool Calls

Chat tools (`apps/web/src/app/api/chat/tools.ts`) call FastAPI sandbox endpoints server-to-server using `API_URL` (defaults to `http://localhost:4000`). These are **not** proxied through Next.js rewrites — they go directly from the Next.js server process to FastAPI.

### FastAPI Route Registration

Routes use empty-string paths (`@router.get("")`, `@router.post("")`) — **not** `"/"`. FastAPI's default `redirect_slashes=True` would 307-redirect `/api/projects` to `/api/projects/`, which breaks cookie forwarding through the Next.js proxy.

### SQLAlchemy Models

SQLAlchemy models must exactly mirror Prisma's schema:

- `__tablename__` matches Prisma model name (PascalCase, e.g., `"User"`, `"Session"`, `"Project"`)
- Column names are camelCase (e.g., `userId`, `createdAt`, `emailVerified`) matching Prisma field names
- IDs use `cuid2` package (matching Prisma's `@default(cuid())`)
- Prisma owns all DDL — never use Alembic or create tables from SQLAlchemy

### Error Response Format

FastAPI returns errors as `{"error": "..."}` (matching the original Express API contract). Custom exception handlers in `main.py` handle `HTTPException` and `RequestValidationError` to produce this format.

### Shared Types

The `@ai-app-builder/shared` package exports User, Project, Chat, Message types, API response wrappers, and WebSocket event types. Build shared package first when types change (`pnpm --filter @ai-app-builder/shared build`).

## Testing

- **Web**: Tests co-located with source files (`*.test.ts`, `*.test.tsx`). Uses Vitest with jsdom and React Testing Library.
- **FastAPI**: Tests in `services/api/tests/`. Uses pytest + httpx AsyncClient + aiosqlite (in-memory SQLite for speed). Test fixtures simulate signed cookies by using `TOKEN.fakesignature` format, with the plain token stored in the test DB.

## Git Workflow

- Husky runs lint-staged on pre-commit (Prettier + ESLint)
- Pre-push runs type checks and full test suite
- Commits must follow conventional format: `feat:`, `fix:`, `docs:`, etc.

## Environment Variables

Required in `.env` (copy from `.env.example`):

- `DATABASE_URL` - PostgreSQL connection string (`postgresql://...`; FastAPI auto-converts to `postgresql+asyncpg://`)
- `BETTER_AUTH_SECRET`, `BETTER_AUTH_URL` - Authentication (better-auth runs in Next.js)
- `OPENAI_API_KEY` - LLM provider key (OpenAI)
- `MODAL_TOKEN_ID`, `MODAL_TOKEN_SECRET` - Modal API for sandbox environments
- `API_URL` - FastAPI URL for Next.js rewrites (defaults to `http://localhost:4000`)
- `S3_ENDPOINT`, `S3_REGION`, `S3_BUCKET`, `S3_ACCESS_KEY_ID`, `S3_SECRET_ACCESS_KEY`, `S3_FORCE_PATH_STYLE` - S3-compatible object storage config
- `MINIO_ROOT_USER`, `MINIO_ROOT_PASSWORD` - Local MinIO bootstrap credentials

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/swapnil233)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/swapnil233)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
