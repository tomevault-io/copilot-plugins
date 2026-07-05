## discord-clone

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Session Management

At the start of each session, read `HANDOFF.md` in the repo root to understand
current progress and the next task.

At the end of each session, overwrite `HANDOFF.md` with the current state
(completed work, next task, deferred items). Do not append — replace entirely.

## Project Overview

A Discord clone built as a pnpm monorepo with two packages:
- **`server/`** — NestJS 11 backend (TypeORM + PostgreSQL + Socket.IO)
- **`client/`** — React 19 + Vite frontend (Tailwind CSS v4)

## Commands

### Root (run from repo root)
```bash
pnpm dev:server       # Start NestJS in watch mode
pnpm dev:client       # Start Vite dev server
```

### Server (run from `server/`)
```bash
pnpm build            # Compile TypeScript via nest build
pnpm start:dev        # Watch mode
pnpm lint             # ESLint with auto-fix
pnpm test             # Run all unit tests
pnpm test:watch       # Jest watch mode
pnpm test:cov         # Coverage report
pnpm test:e2e         # E2E tests (jest-e2e.json config)
```

Run a single test file:
```bash
pnpm test -- --testPathPattern=auth.service
```

### Client (run from `client/`)
```bash
pnpm dev              # Vite dev server
pnpm build            # tsc + vite build
pnpm lint             # ESLint
```

### Infrastructure
```bash
docker compose up -d  # Start PostgreSQL 17 (port 5432)
```

## Environment Setup

Create `server/.env` with:
```
DB_HOST=localhost
DB_PORT=5432
DB_USERNAME=...
DB_PASSWORD=...
DB_DATABASE=...
JWT_SECRET=...
PORT=3000
```

The `docker-compose.yaml` reads this file for the PostgreSQL container.

## Server Architecture

### Global Guards & Pipes (applied in `main.ts`)
- **JWT guard** is global — all routes require authentication by default
- Use `@Public()` decorator on controller methods to opt out
- `ValidationPipe` with `whitelist: true`, `forbidNonWhitelisted: true`, `transform: true`
- `ClassSerializerInterceptor` for response DTO serialization (use `@Exclude()` on entity fields)
- Swagger docs at `/api`

### Module Structure
```
auth/         JWT auth — register, login, guards, strategies, decorators
users/        User entity and CRUD
servers/      Server (guild) creation, membership, invite codes
channels/     Channels within servers
messages/     Message HTTP CRUD
chat/         WebSocket gateway (Socket.IO) — real-time messaging
server-members/  Join/leave server membership
common/       BaseEntity, exception filters, interceptors, shared interfaces
config/       SnakeNamingStrategy for TypeORM
```

### Conventions
- **Entities** extend `BaseEntity` from `common/base.entity.ts` which provides `id`, `createdAt`, `updatedAt`, `deletedAt` (all `timestamptz`)
- **Custom exceptions** extend `BaseException`; domain-specific errors live in `<module>/exceptions/` with error codes in `<module>/constants/`
- **Response DTOs** use `plainToInstance` + `@Expose()` on DTO fields (not entities); applied via custom `SerializeInterceptor` with `@Serialize(DtoClass)` decorator
- **Error codes** use `as const` objects (not enums) — avoids runtime code generation, supports holding multiple values per constant
- **Exports** prefer named exports (`export const`) over `export default` to prevent renaming confusion on import
- TypeORM uses `synchronize: true` (dev only); `SnakeNamingStrategy` maps camelCase entities to snake_case DB columns
- TypeORM transactions use `DataSource.transaction()` callback pattern (`@Transaction()` decorator is deprecated)

### Real-time Chat
`chat/chat.gateway.ts` is a `@WebSocketGateway` that handles Socket.IO connections. It imports `AuthModule` and `MessagesModule`. CORS is configured for `http://localhost:5173`.

## Client Architecture

React Router v7 with nested layouts:
- `AuthLayout` — unauthenticated pages (Login, Register)
- `MainLayout / HomeLayout` — authenticated shell
- `ServerLayout` — server-specific view with sidebars

`ProtectedRoute` component guards authenticated routes. Form handling uses `react-hook-form`.

## Roadmap

### Phase 1 — Core (Active)
- [x] Email/password auth + JWT (without Passport.js)
- [x] Server CRUD + invite code join
- [x] Channel CRUD (nested routing: `/servers/:serverId/channels`)
- [x] WebSocket text chat (Socket.IO)
- [x] Custom exception architecture (`BaseException` → `HttpExceptionFilter` + `AllExceptionFilter`)
- [x] Unit tests — ServersService
- [x] Swagger (`@nestjs/swagger`)
- [ ] Response DTO (`SerializeInterceptor` + `plainToInstance` + `@Expose()`)
- [ ] Unit tests — ChannelsService, AuthService
- [ ] Integration tests — Auth
- [ ] Selective E2E tests

### Phase 2 — Messaging
- DMs (1:1 and group)
- File upload (S3 presigned URL)
- Emoji reactions

### Phase 3 — Voice/Video
- WebRTC voice/video channels
- SFU architecture

### Phase 4 — Mobile
- React Native app
- QR code login (mobile → web authentication)

## Code Style

Prettier config (`.prettierrc`): single quotes, trailing commas, 80-char print width, Tailwind class sorting via `prettier-plugin-tailwindcss`.

---
> Source: [alsry922/discord-clone](https://github.com/alsry922/discord-clone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-05 -->
