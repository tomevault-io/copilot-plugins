## reference-next-js-project

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

| Task | Command |
|------|---------|
| Dev server | `bun dev` (Turbopack) |
| Build | `bun build` |
| Lint | `bun lint` (Biome) |
| Format | `bun run format` |
| All tests | `bun test` |
| Single test | `bunx vitest run tests/lib/services/product-service.test.ts` |
| Watch tests | `bun run test:watch` |
| DB studio | `bun run db:studio` |
| Type + lint | `bun run verify:all` |

## Architecture

Next.js 16 App Router with a layered REST API over SQLite.

### Request flow

```
Route Handler (src/app/api/v1/*)
  → Zod validation (src/lib/types/)
  → Service layer (src/lib/services/)
  → Repository layer (src/lib/repositories/)
  → Drizzle ORM → SQLite (better-sqlite3, WAL mode)
```

### Key patterns

- **Error handling**: Custom `NotFoundError`/`ValidationError` classes caught by centralized `handleError()` in route handlers. Error names (not `instanceof`) drive HTTP status mapping.
- **Auth**: Cookie-based sessions via `getSession()` from `src/lib/auth/index.ts`. httpOnly cookie stores `{id, username, role}` as JSON. Test users: `user/user` (USER), `admin/admin` (ADMIN).
- **DB init**: Tables auto-created via raw SQL in `src/lib/db/index.ts`. Seed runs on startup via `src/instrumentation.ts` (Node.js runtime only).
- **Types**: Zod schemas in `src/lib/types/` define DTOs (`CreateProductDto`, `UpdateProductDto`) and response types. Drizzle infers DB types from schema (`$inferSelect`/`$inferInsert`).
- **AI chat**: POST `/api/chat` uses Vercel AI SDK `streamText()` with SSE streaming via `toUIMessageStreamResponse()`. Client uses `useChat()` + `DefaultChatTransport`.
- **Pages**: Server components fetch via internal API (`/api/v1/*`) with cookie forwarding, not direct service calls.

### Linting

Biome extends `ultracite/next`. Notable rules: `noConsole: error` (except seed, and the centralized `error-handler.ts` via an inline `biome-ignore`), `noExplicitAny: warn`. Line width 120, 2-space indent. Config in `biome.jsonc`.

### Tests

Vitest tests live in `tests/` mirroring `src/lib/` structure. Tests cover schema, repositories, services, and Zod type validation.

## Environment

- `OPENAI_API_KEY` — required for AI chat widget
- `CHAT_MODEL` — optional, defaults to `gpt-4.1-nano`
- `DATABASE_URL` — optional, defaults to `sqlite.db`
- `NEXT_PUBLIC_BASE_URL` — optional, defaults to `http://localhost:3000`

---
> Source: [bogdansolga/reference-next-js-project](https://github.com/bogdansolga/reference-next-js-project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
