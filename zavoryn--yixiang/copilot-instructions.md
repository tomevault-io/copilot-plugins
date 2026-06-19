## yixiang

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

颐享 (YiXiang) — a knowledge-sharing community platform. Two sub-projects: `yixiang_be` (Spring Boot backend) and `yixiang_fe` (React frontend).

---

## yixiang_be — Backend

**Stack:** Java 21, Spring Boot 3.2.4, Spring Security + OAuth2 Resource Server (JWT RS256), MyBatis, MySQL 8.0, Redis (Redisson), Elasticsearch 9.x, Kafka, Caffeine, Canal, Spring AI 1.0.3 (DeepSeek + OpenAI), Alibaba Cloud OSS, Lombok

### Build & Run

```bash
# Build (uses system mvn — no Maven wrapper)
cd yixiang_be && mvn clean package -DskipTests

# Run tests
mvn test                     # all tests
mvn test -Dtest=ClassName    # single test class

# Run app (needs MySQL, Redis, ES, Kafka running)
mvn spring-boot:run
```

**Setup:** Copy `src/main/resources/application.yml.example` to `application.yml` and fill in credentials. JWT requires a key pair in `src/main/resources/keys/` (public.pem checked in, private.pem gitignored).

Spring Boot entry point: `com.tongji.ZhiGuangApplication` (no custom annotations beyond `@SpringBootApplication`).

### Architecture (module overview)

The codebase is organized by domain module under `com.tongji`. Each module follows a layered pattern: `api/` (controllers + DTOs) → `service/` (interfaces + `impl/`) → `mapper/` (MyBatis) → `model/`.

| Module | Package | Responsibility |
|--------|---------|----------------|
| **auth** | `com.tongji.auth` | JWT dual-token auth (RS256), 15-min access + 7-day refresh tokens, Redis refresh whitelist, instant revocation, phone/email verification codes |
| **counter** | `com.tongji.counter` | Like/fav counters stored in Redis as compact binary SDS, Lua atomic updates, bitmaps for dedup/idempotency. Kafka async write aggregation for high-concurrency writes. Sampling consistency checks with self-healing rebuild. |
| **knowpost** | `com.tongji.knowpost` | CRUD for knowledge posts ("知文"), progressive publish workflow (draft → review → published), Snowflake ID generation, OSS pre-signed URL direct upload for images/video/markdown content |
| **relation** | `com.tongji.relation` | Follow/unfollow with **Outbox pattern**: writes to `following`/`follower` tables + `outbox` table in same DB transaction, then Canal subscribes to outbox binlog → Kafka → async updates to counters, caches, lists |
| **search** | `com.tongji.search` | Elasticsearch content indexing, `search_after` cursor pagination, `function_score` mixing BM25 + business weights, `completion` suggester for prefix search. Separate Canal→Kafka consumer for outbox-driven index updates. |
| **llm** | `com.tongji.llm` | Spring AI integration (DeepSeek for chat, OpenAI for embeddings), RAG pipeline: check index → vector retrieval → prompt construction → SSE streaming generation. Per-document Q&A with chunked indexing and idempotent deletion. |
| **profile** | `com.tongji.profile` | User profile CRUD, avatar upload (OSS), tag management. Has its own `CorsConfig` scoped to `/api/v1/profile/**`. |
| **storage** | `com.tongji.storage` | OSS pre-signed URL generation for direct frontend uploads |
| **user** | `com.tongji.user` | User domain model, mapper |
| **cache** | `com.tongji.cache` | Three-tier feed cache (Caffeine local → Redis page → Redis fragment), hotkey detector with sliding window, single-flight lock against thundering herd on cache misses |
| **common** | `com.tongji.common` | `BusinessException`, `ErrorCode`, `GlobalExceptionHandler` (@RestControllerAdvice), `OutboxMessageUtil` |
| **config** | `com.tongji.config` | Cross-cutting infrastructure: `ElasticsearchConfig`, `RedissonConfig`, `ThreadPoolConfig` |

### Key architecture patterns

- **Dual-token auth flow:** Login returns access token (15min, RS256 signed JWT) + refresh token (7 days, opaque, stored in Redis whitelist). `/auth/token/refresh` validates refresh against Redis and issues new pair. `/auth/logout` removes refresh from whitelist for instant revocation.
- **Outbox + Canal + Kafka:** Used by both `relation` and `search` modules. Services write to `outbox` table in the same DB transaction as the business write. Canal monitors MySQL binlog, picks up outbox inserts, pushes to Kafka topics. Downstream consumers (relation processor, search indexer) consume from Kafka with manual ack. This decouples the write path from async side effects.
- **Counter system:** Like/fav counts stored as Redis SDS binary (compact integers). Bitmaps track per-user state for idempotency. Writes are fired async via Kafka aggregation (`CounterAggregationConsumer`). On read, if count vs bitmap mismatch detected, a rebuild is triggered. Kafka-based "disaster replay" as last-resort recovery.
- **Feed caching:** Three-tier: Caffeine (local, shortest TTL) → Redis page cache → Redis fragment cache. `HotKeyDetector` uses sliding window counts to classify keys as NONE/LOW/MEDIUM/HIGH and dynamically extend TTL for hotter entries. Single-flight lock prevents concurrent cache-backfill storms for the same key.
- **Search indexing:** `SearchIndexService.upsertKnowPost()` pulls post details, fetches content from OSS URL, enriches with counter values, and indexes into ES. `CanalOutboxConsumerSearch` listens to outbox events to trigger re-index on post updates/deletes.
- **Security:** Stateless JWT auth via `oauth2ResourceServer`. Public endpoints include `/actuator/health`, auth endpoints, GET feed/detail/QA-stream. Everything else requires authentication. Method-level security available via `@PreAuthorize`.
- **Global error handling:** `GlobalExceptionHandler` catches `BusinessException` and returns structured JSON with `ErrorCode`.

### Data layer

- MyBatis with XML mappers in `src/main/resources/mapper/`
- Primary DB tables: `users`, `know_posts`, `outbox`, `following`, `follower`, `login_logs`
- Snowflake IDs generated at the application layer (not auto-increment) for `know_posts`, `following`, `follower`, `outbox`
- JSON columns used for `tags`, `img_urls`, `tags_json`, `payload`

---

## yixiang-web — Frontend (new)

**Stack:** React 18, TypeScript 5, Vite 5, Tailwind CSS v4, shadcn/ui (Radix), TanStack Query v5, React Router v6, lucide-react, sonner, date-fns, react-hook-form + zod, Vitest

### Commands

```bash
cd yixiang-web
npm run dev       # Vite dev server on :5173, proxies /api → localhost:8080
npm run build     # tsc --noEmit + vite build
npm run lint      # tsc --noEmit (type checking only)
npm run preview   # preview production build
npm run test      # vitest --run
```

### Architecture

- **`src/app/`** — App entry, routes (`routes.tsx`), global providers (`providers.tsx`), `ProtectedRoute`
- **`src/pages/`** — 12 route-level pages: HomePage, SearchPage, CreatePage, ProfilePage, EditProfilePage, PostDetailPage, LoginPage, RegisterPage, CircleSquarePage, CircleDetailPage, FollowListPage, NotificationPage, CollectionsPage
- **`src/components/`** — `layout/` (AppLayout, Header, Sidebar, PageShell), `ui/` (shadcn primitives: Button, Dialog, Tabs, Tooltip, Popover, Select, DropdownMenu, Avatar, Badge, Input, Textarea, Skeleton), `common/` (EmptyState, LoadingSkeleton, ErrorBoundary, InfiniteList), `user/` (FollowButton)
- **`src/features/`** — Business hooks (useQuery/useMutation + optimistic update): `profile/useProfile`, `relation/useFollow`/`useUnfollow`, `notification/useNotifications`/`useUnreadCount`
- **`src/services/`** — Thin API client layer: authService, knowpostService, profileService, relationService, searchService, notificationService, favoriteService, circleService, commentService
- **`src/types/`** — Backend DTO types
- **`src/lib/`** — `apiClient` (JWT auto-attach + 401 refresh), `queryClient` (staleTime 30s), `formatters`, `utils`, `sse`
- **`src/config/`** — Feature flags (`features.ts`), env config (`env.ts`)
- **`src/hooks/`** — Generic hooks: `useDebounce`
- **`src/styles/`** — `globals.css` (Tailwind + @theme tokens), `scrollbar.css`
- **Path alias:** `@/` maps to `src/`

### Key flows

- **Auth guard:** `ProtectedRoute` wraps routes that need login; public routes (feed, detail, search) render regardless.
- **API calls:** All go through `services/` → `apiFetch<T>()` → JWT Bearer auto-attach → 401 auto-refresh → typed error throw.
- **State:** TanStack Query for server state (useQuery/useMutation + optimistic update), React Context for auth (AuthContext), local useState for UI state.
- **Optimistic updates:** Follow/unfollow, like/fav use `onMutate` → `setQueryData` → `onError` rollback → `onSettled` invalidateQueries pattern.

---

## yixiang_fe — Frontend (legacy)

**This project is superseded by `yixiang-web/` and will be deleted.**

Stack: React 18, TypeScript 5, Vite 5, React Router v6, CSS Modules

---
> Source: [zavoryn/yixiang](https://github.com/zavoryn/yixiang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
