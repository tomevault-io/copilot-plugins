## infinitum

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.\n

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.\n
## Project Overview

Infinitum is a Next.js 16 App Router-based news aggregation system with the following pipeline:
1. RSS source fetching
2. Full-text content extraction (when RSS lacks content)
3. AI-powered analysis, translation, and summarization
4. Content clustering and deduplication
5. Public feed display with admin management panel

## Architecture

### Directory Structure

- `src/app/` - Next.js App Router pages and API Route Handlers
  - `page.tsx` - Public visitor homepage (feed display)
  - `login/page.tsx` - Admin login
  - `admin/*` - Admin dashboard pages (content, settings, monitor)
  - `api/feed/*` - Public feed API endpoints
  - `api/admin/*` - Admin API endpoints (require session auth)
  - `api/ingest/*` - Ingestion trigger endpoints

- `src/lib/feed/` - Public feed query layer
  - `service.ts` - Cached feed queries (primary interface for feed data)
  - `repository.ts` - Database queries for feed items
  - `cache.ts` - Feed caching utilities
  - `range.ts` - Date range filtering logic
  - `types.ts` - Feed-related type definitions

- `src/lib/ingestion/` - Content ingestion pipeline
  - `service.ts` - Main ingestion orchestration (RSS → DB)
  - `parser.ts` - RSS feed parsing
  - `article.ts` - Full-text article extraction
  - `filtering.ts` - Blacklist keyword filtering
  - `dedupe.ts` - Deduplication logic

- `src/lib/clusters/` - Content clustering
  - `service.ts` - Cluster assignment and management
  - `repository.ts` - Cluster database operations

- `src/lib/tasks/` - Background task system
  - `service.ts` - Task queue management
  - `worker.ts` - Background worker execution
  - `scheduler.ts` - Task scheduling logic
  - `handlers.ts` - Task type handlers

- `src/lib/items/` - Individual item operations
  - `service.ts` - Item regeneration, reanalysis

- `src/lib/settings/` - Configuration management
  - `service.ts` - Settings CRUD operations
  - `ai-config.ts` - AI provider configuration

- `src/lib/ai/` - AI provider abstraction
  - `provider.ts` - OpenAI-compatible API client

- `prisma/schema.prisma` - Single source of truth for database schema (SQLite)

- `scripts/setup-sqlite.mjs` - Database initialization/migration script

- `config/infinitum.config.json` - Initial configuration template (imported to DB on first run)

## Common Commands

```bash
# Development
npm run dev              # Start Next.js dev server (localhost:3000)
npm run worker           # Start background task worker

# Database
npm run db:setup         # Initialize/upgrade SQLite dev database
npm run db:test:setup    # Reset and initialize test database
npm run prisma:generate  # Regenerate Prisma Client
npm run prisma:migrate   # Run Prisma migrations

# Testing
npm test                 # Run all tests with coverage (resets test DB)
npm run test:watch       # Run tests in watch mode
vitest run tests/integration/feed-api.test.ts  # Run single test file

# Linting & Building
npm run lint             # Run ESLint
npm run build            # Production build with webpack
```

## Key Business Constraints

1. **Feed time filtering uses `items.createdAt`, NOT `publishedAt`**
   - This represents "system ingestion time window" semantics
   - Do not change to `publishedAt` for performance optimization

2. **Cache invalidation required on content changes**
   - Any backend write that affects public feed must call `invalidateFeedCache()` from `src/lib/feed/cache.ts`
   - Feed has short TTL caching for performance

3. **Ingestion prioritizes total duration over per-step concurrency**
   - Bottlenecks: full-text fetching, AI enrichment, cluster summarization, frequent DB writes
   - Optimize by reducing serial steps and unnecessary DB writes

4. **Background tasks have dual execution paths**
   - Synchronous execution (admin actions)
   - Worker execution (background queue)
   - Changes must consider both paths

5. **Feed queries affect multiple features**
   - When modifying feed queries, verify impact on:
     - Cluster/single item mixing
     - Group counting
     - Title search
     - Timezone-aware date boundaries

## Development Principles

1. **Layer order**: Modify `src/lib/*` → `src/app/api/*` → Components
2. **Extract shared logic**: Move repeated param parsing/DTO assembly to shared helpers
3. **Database changes**: Only edit `prisma/schema.prisma`, no handwritten SQL
4. **Feed reads**: Use `src/lib/feed/service.ts` instead of direct cache logic
5. **High-frequency DB writes**: Consider batching, throttling, or stage-end flushing

## Testing

- **Feed changes**: `tests/integration/feed-api.test.ts`, `tests/unit/feed-range.test.ts`
- **Ingestion/Task changes**: `tests/integration/ingestion-service.test.ts`, `tests/integration/background-task-service.test.ts`
- **Database changes**: `tests/integration/sqlite-setup-migration.test.ts`

Test environment uses jsdom with Prisma + SQLite. Tests reset database state via `prisma.*.deleteMany()` in `beforeEach`.

## Environment Variables

Required in `.env`:
- `DATABASE_URL` - SQLite DB path (e.g., `file:./prisma/dev.db`)
- `ADMIN_PASSWORD` - Admin login password
- `ADMIN_SESSION_SECRET` - Session cookie signing secret

## Configuration

Runtime configuration is stored in database, not config file. Initial values are imported from `config/infinitum.config.json` on first startup. After import, use admin settings UI or API to modify:
- RSS sources and groups
- Blacklist keywords
- AI model API configs
- Prompt templates
- Task schedules

## Admin Access

- Login: `http://localhost:3000/login`
- Dashboard: `http://localhost:3000/admin`
- Settings: `http://localhost:3000/admin/settings`
- Task Monitor: `http://localhost:3000/admin/monitor`

Admin session uses HTTP-only cookie signed with `ADMIN_SESSION_SECRET`.

---
> Source: [shawnxie94/infinitum](https://github.com/shawnxie94/infinitum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
