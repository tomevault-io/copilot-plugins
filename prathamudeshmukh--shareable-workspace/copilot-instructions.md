## shareable-workspace

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

**Dropzone** — an ephemeral file-sharing workspace. Users create a workspace, upload files, share the link; everything auto-expires. Deployed on Cloudflare Workers + Next.js (App Router).

## Commands

```bash
# Local development (initializes D1 + starts Next.js + PartyKit concurrently)
npm run dev:local

# Run only Next.js dev server (assumes D1 is already set up)
npm run dev

# Set up local D1 database
npm run dev:setup

# Build
npm run build

# Lint
npm run lint

# Run all tests (Vitest)
npm test

# Run tests in watch mode
npm run test:watch

# Deploy to Cloudflare
npm run cf:deploy

# Deploy cron worker separately
npm run cf:deploy-cron
```

To run a single test file:
```bash
npx vitest run lib/__tests__/file-utils.test.ts
```

## Architecture

**Request path:**
1. Browser hits `app.prathamesh.cloud/workspace*`
2. Cloudflare Worker routes to OpenNext.js handler (Next.js App Router)
3. API routes in `app/api/` handle CRUD, using `lib/get-env.ts` to load Cloudflare bindings (D1, R2, KV) in both dev and prod
4. Real-time updates go through PartyKit: API routes call `lib/partykit.ts` to broadcast, browser subscribes via PartySocket in `WorkspacePage.tsx`
5. Expired files are cleaned up by the cron worker (`worker/cleanup-cron.ts`), which also broadcasts expiry events

**Cloudflare bindings (wrangler.toml):**
- `DB` → D1 (SQLite) — workspace + file metadata
- `FILES` → R2 — actual file blobs, keyed as `workspaces/[workspaceId]/[fileId]/[filename]`
- `RATE_LIMIT` → KV — fixed-window rate limiting per IP

**Environment loading (`lib/get-env.ts`):**
In dev, uses `wrangler getPlatformProxy()` (Miniflare); in prod, uses `@opennextjs/cloudflare`'s `getCloudflareContext()`. All API route handlers call `getEnv()` to get access to bindings.

**Real-time (PartyKit):**
- Server-side room: `party/workspace.ts` — a simple broadcast relay
- Client-side: `WorkspacePage.tsx` connects via `PartySocket` and listens for `files_added` / `file_expired` events to update React state without re-fetching

**Database schema** (canonical: `schema.sql`):
- `workspaces(id, created_at)`
- `files(id, workspace_id, name, mime_type, size, r2_key, uploaded_at, expires_at)` — expiry is per-file

**Key constants** (`lib/constants.ts`): file TTL, max file size, max file count, rate limit windows.

## Key Patterns

**`getEnv()` in every API route** — never access `process.env` directly for Cloudflare bindings; always call `lib/get-env.ts`.

**Zod validation at API boundaries** — schemas live in `types/workspace.ts`; validate before touching D1/R2.

**Filename sanitization** — always run `sanitizeFilename()` from `lib/file-utils.ts` before writing to R2 or D1 to prevent path traversal.

**PartyKit broadcast is fire-and-forget** — don't let broadcast failures block API responses.

**basePath is `/workspace`** — configured in `next.config.ts`; all internal links must account for this.

## Testing

Tests use Vitest with jsdom environment. Path alias `@` maps to the project root.

- `lib/__tests__/` — unit tests for db, r2, file-utils
- `components/__tests__/` — React component tests (Testing Library)
- `app/api/workspace/__tests__/` — API route integration tests

Cloudflare bindings are mocked in tests; real D1/R2 is not used in the test suite.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/prathamudeshmukh)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/prathamudeshmukh)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
