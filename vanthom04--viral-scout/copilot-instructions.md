## viral-scout

> Đây là context file cho AI tools (Claude Code, Cursor, Copilot). Đọc file này trước khi làm bất kỳ task nào trong project.

# Viral Scout — CLAUDE.md

Đây là context file cho AI tools (Claude Code, Cursor, Copilot). Đọc file này trước khi làm bất kỳ task nào trong project.

## Tổng quan

Viral Scout là một **content intelligence platform** tự động cào dữ liệu từ các mạng xã hội (Reddit, YouTube, Facebook, TikTok, X, LinkedIn), phân tích virality bằng AI, và giúp content creator Việt Nam tạo ý tưởng content nhanh hơn.

**Monorepo** — pnpm + Turborepo. Gồm 3 layer:

1. `packages/` — shared code (types, database schema/queries, config)
2. `workers/` — Cloudflare Workers độc lập (scraper queue consumer, scheduler cron)
3. `apps/web/` — Next.js 16 app chạy trên Cloudflare via OpenNext

## Tech Stack

| Layer           | Stack                                                     |
| --------------- | --------------------------------------------------------- |
| Frontend        | Next.js 16 + React 19 + Tailwind CSS v4 + shadcn/ui       |
| Auth            | Better Auth 1.5 + Drizzle adapter + D1                    |
| Database        | Cloudflare D1 (SQLite) + Drizzle ORM                      |
| AI              | Cloudflare Workers AI (`llama-3.3-70b-instruct-fp8-fast`) |
| Scraping        | Cloudflare Workers + Queues (fan-out pattern)             |
| Deploy          | OpenNext Cloudflare (`@opennextjs/cloudflare`)            |
| Package manager | pnpm + Turborepo                                          |
| Language        | TypeScript strict mode, `verbatimModuleSyntax: true`      |

## Monorepo Structure

```
viral-scout/
├── packages/
│   ├── config/          # tsconfig.base.json shared
│   ├── types/           # @viral-scout/types — tất cả TypeScript types
│   └── database/        # @viral-scout/database — Drizzle schema + queries
├── workers/
│   ├── scraper/         # Queue consumer — cào + analyze posts
│   └── scheduler/       # Cron trigger — fan-out jobs vào Queue
└── apps/
    └── web/             # Next.js 16 app — dashboard UI
```

## Quy tắc code bắt buộc

### Naming conventions

- `camelCase` — variables, functions
- `PascalCase` — classes, types, interfaces, React components
- `kebab-case` — file names, directory names
- `UPPER_SNAKE_CASE` — constants

### TypeScript

- **Strict mode** — `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`
- **Không dùng `any`** — dùng `unknown` + type guard hoặc Zod parse
- **`verbatimModuleSyntax: true`** — luôn dùng `import type` cho pure types
- **Không dùng `as` assertion tuỳ tiện** — parse bằng Zod thay thế

### Next.js 16 specific

- Route guard dùng `src/proxy.ts` (export `proxy`) — **không phải `middleware.ts`**
- `params` và `searchParams` là Promise — phải `await` trước khi dùng
- Caching explicit bằng `"use cache"` + `cacheLife()` + `cacheTag()`
- **Không** dùng implicit fetch caching kiểu Next.js 15

### shadcn/ui

- **`Form` component cũ đã bị xoá** — dùng `Field`, `FieldLabel`, `FieldError`, `FieldGroup`, `FieldSet` thay thế
- `Toast` → dùng `Sonner` (`@/components/ui/sonner`)
- Import trực tiếp từng component, **không** import từ barrel

### Server vs Client

- Default: **Server Component**
- Client Component chỉ khi cần: event handler, useState, browser API, streaming
- Data fetching trong Server Component, **không** dùng `useEffect` để fetch

### Error handling

- Server Actions dùng Result pattern:
  ```typescript
  type ActionResult<T> = { success: true; data: T } | { success: false; error: string }
  ```
- Không throw trong Server Actions — return error thay vì throw

## Cloudflare Bindings

Tất cả bindings được typed trong `packages/types/src/env.types.ts`:

```typescript
interface CloudflareEnv {
  DB: D1Database // Drizzle D1
  AI: Ai // Workers AI
  SCRAPER_QUEUE: Queue // Scraper jobs queue
  // ... API keys
}
```

Trong `apps/web` — lấy env qua:

```typescript
import { getCloudflareContext } from "@opennextjs/cloudflare"
const { env } = await getCloudflareContext({ async: true })
```

## Data Flow

```
Cron (mỗi 2h)
  → workers/scheduler: đọc active sources từ D1 → enqueue ScraperJob
    → workers/scraper: consume queue → scrape platform API
      → classifier (llama-3.1-8b): lọc off-topic
        → analyzer (llama-3.3-70b): virality score + hooks
          → D1: lưu posts + analyzed_posts + post_tags
            → apps/web: Server Component đọc D1 → render feed
```

## Database Schema (D1)

7 bảng chính — xem `packages/database/src/schema.ts`:

- `sources` — cấu hình platform/target cào
- `posts` — raw scraped posts
- `analyzed_posts` — kết quả AI phân tích (1-1 với posts)
- `tags` — taxonomy 15 tags cố định (seed data)
- `post_tags` — junction many-to-many
- `saved_ideas` — bài user lưu lại
- `cron_logs` — health monitor cho mỗi scraper run

Bảng auth (Better Auth quản lý): `user`, `session`, `account`, `verification`

## Workspace Dependencies

```
apps/web → @viral-scout/database, @viral-scout/types
workers/scraper → @viral-scout/database, @viral-scout/types
workers/scheduler → @viral-scout/database, @viral-scout/types
packages/database → @viral-scout/types
```

## Commands

```bash
pnpm dev              # Chạy tất cả (turbo)
pnpm build            # Build tất cả packages
pnpm type-check       # TypeScript check toàn monorepo

# Trong apps/web
pnpm preview          # Build + wrangler dev (test CF environment)
pnpm deploy           # Deploy lên Cloudflare

# Database
wrangler d1 execute viral-scout-db --local --file=packages/database/migrations/0001_init_seed.sql
```

## Anti-patterns — Không làm

- ❌ `import { X } from "@/components/ui"` — dùng direct import
- ❌ `useEffect` để fetch data — dùng Server Component
- ❌ Sequential `await` cho independent operations — dùng `Promise.all`
- ❌ `middleware.ts` — đã đổi thành `proxy.ts` trong Next.js 16
- ❌ Sync `params` access — phải `await params`
- ❌ Hardcode colors với oklch — dùng CSS token (`bg-primary`, `text-foreground`)
- ❌ `any` type — dùng `unknown` hoặc Zod
- ❌ Import Form cũ của shadcn — dùng `Field` component thay thế

---
> Source: [vanthom04/viral-scout](https://github.com/vanthom04/viral-scout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
