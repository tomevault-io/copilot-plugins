## temar

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Commands

```sh
# Install
pnpm install

# Dev servers
pnpm nx serve @temar/web                      # Next.js frontend (port 5173)
pnpm nx serve @temar/fsrs-service              # FSRS service (port 3334)
pnpm nx serve @temar/question-gen-service      # Question gen (port 3335)
pnpm nx serve @temar/answer-analysis-service   # Answer analysis (port 3336)
pnpm nx serve admin                              # Admin panel (port 3000)

# Build
pnpm nx build @temar/web                       # single project
pnpm nx run-many -t build                      # all projects
pnpm nx affected -t build                      # changed only

# Test
pnpm nx test @temar/fsrs-service               # single project
pnpm nx affected -t test                       # changed only

# Lint
pnpm nx lint @temar/web                        # single project
pnpm nx affected -t lint                       # changed only

# DB schema changes & migrations
# 1. Edit the schema in libs/db-client/src/schema/*.ts
# 2. Generate migration SQL (NEVER create SQL files manually):
npx drizzle-kit generate --config libs/db-client/drizzle.config.ts
# 3. Apply migration:
pnpm migrate                                   # runs drizzle-kit migrate from libs/db-client

# Docker
docker compose -f docker-compose.dev.yml up    # local dev
docker build --target web-app-service -t temar-web .
docker build --target fsrs-service -t temar-fsrs .
```

---

## Architecture

**Temar** is a spaced-repetition learning platform. Users organize study material as **Topics -> Notes -> Chunks**, create content via a **Lexical rich-text editor**, and review it through AI-powered question generation and FSRS-based scheduling.

### Core data flow

```
User <-> Next.js (web) <-> NestJS microservices <-> PostgreSQL
              |                    |
         Lexical editor       LLM providers (via question-gen / answer-analysis)
         (content CRUD)
```

- **Monorepo:** Nx 22.x + pnpm workspaces
- **Frontend:** Next.js 15 (App Router) + React 19 + Tailwind 4 + shadcn/ui (shared via `@temar/ui`)
- **Backend:** NestJS microservices, each with Swagger docs at `/api/docs`
- **Database:** PostgreSQL 16 + Drizzle ORM (schemas in `libs/db-client/src/schema/`)
- **AI:** Vercel AI SDK (`ai` package), multi-provider (OpenAI, Anthropic, Google), BYOK support
- **Editor:** Lexical (migrated from Plate.js) for both chunk content authoring and review answers
- **Payments:** Paddle v2 (migrated from Stripe), Pass-based billing
- **Deploy:** Docker multi-stage -> GHCR -> VPS + Caddy

### Applications (`apps/`)

| App                       | Port | Purpose                                                                    | Status                    |
| ------------------------- | ---- | -------------------------------------------------------------------------- | ------------------------- |
| `web`                     | 5173 | Next.js frontend -- dashboard, materials browser, review sessions, billing | **Active**                |
| `fsrs-service`            | 3334 | NestJS -- ts-fsrs engine, tracking cascades, review lifecycle              | **Active**                |
| `question-gen-service`    | 3335 | NestJS -- LLM question + rubric generation per chunk                       | **Active**                |
| `answer-analysis-service` | 3336 | NestJS -- LLM semantic answer evaluation, mapped to FSRS ratings           | **Active**                |
| `admin`                   | 3000 | Next.js admin panel -- analytics, AI model management                      | **Active**                |
| `notion_sync-service`     | 3333 | NestJS -- Notion sync (OAuth, webhooks, markdown conversion)               | **Retired, not deployed** |
| `api`                     | --   | NestJS API scaffold                                                        | **Vestigial, ignore**     |

### Libraries (`libs/`)

| Library            | Purpose                                                                                      |
| ------------------ | -------------------------------------------------------------------------------------------- |
| `db-client`        | Drizzle ORM client, all schema definitions, crypto utilities, re-exported Drizzle operators  |
| `shared-types`     | Cross-app TypeScript interfaces and DTOs (`ModelConfig`, `OperationType`, etc.)              |
| `pricing-service`  | In-memory cached pricing engine -- computes Pass costs, records usage with balance deduction |
| `payment-provider` | Strategy-pattern billing abstraction (Paddle adapter)                                        |
| `ui`               | Shared shadcn/ui component library -- 24 components used by both web and admin apps          |

### Inter-service communication

All HTTP REST with `x-api-key` headers. The web app calls services via server-side fetch wrappers (`fsrsServiceFetch`, `questionGenServiceFetch`, etc.) -- never from the browser. All NestJS services use `app.setGlobalPrefix('api')`, so endpoint env vars **must include `/api`** (e.g., `http://fsrs-service:3334/api`).

Authentication lives in the Next.js app via `better-auth`. Backend services have **no user session awareness** -- they authenticate callers via `x-api-key` and receive user identity via `x-user-id` headers.

### Dual data paths in the web app

1. **Server Actions** (`createChunk`, `updateChunkContent`, `deleteTopic`, etc.) -- write directly to DB via `dbClient`
2. **Next.js API routes** (`/api/topics/[topicId]/notes`, `/api/notes/[noteId]/chunks`) -- tree sidebar fetches children lazily via client-side `fetch()`

Schema changes can break both microservices AND the web app's direct DB access.

### Lexical dual-storage model

Chunks store content in two columns: `content_json` (Lexical `SerializedEditorState` as JSONB, source of truth) and `content_markdown` (derived via `lexicalToMarkdown()` at save time, used for AI prompts, review display, and search).

### Database schema (key relationships)

```
user (better-auth)
 +-- pass_balance (1:1)
 +-- pass_transaction (1:N)
 +-- ai_usage_log (1:N)
 +-- topic (1:N)
      +-- note (1:N)
           +-- chunk (1:N)
                +-- recall_item (1:1 per user, FSRS card state)
                     +-- review_log (1:N, append-only)

ai_models -> ai_model_pricing (versioned, append-only)
          -> ai_markup_config (versioned, append-only)
```

Key constraints: `recall_item` unique on `(chunk_id, user_id)`. All Notion-sourced tables use Notion page UUIDs as PKs. `ai_model_pricing` and `ai_markup_config` are append-only with `effective_from`/`effective_to`.

The `dbClient` is a singleton import from `@temar/db-client`, not NestJS DI-injected. Testing requires mocking the module export.

---

## Critical Build Gotchas

### 1. Library bundling (most common build breakage)

pnpm workspace symlinks (`node_modules/@temar/*`) conflict with lib tsconfigs during Docker/webpack builds. The fix has multiple parts:

- **`tsconfig.base.json` path aliases** resolve `@temar/*` imports directly to `libs/*/src/index.ts`
- **`rm -rf node_modules/@temar`** in every Dockerfile builder stage forces webpack to use path aliases
- **`rm -rf libs/*/tsconfig*.json`** in NestJS builder stages prevents `composite: true` interference
- **`extensionAlias`** in `next.config.js` maps `.js` imports to `.ts`/`.tsx` (required by `module: "nodenext"`)
- **`safeWithNx`** wrapper in `next.config.js` handles CI/Docker Nx project graph failures
- **Service `tsconfig.app.json`** must explicitly `include` lib source paths and add `references`

**When adding a new `@temar/*` library or service**, update ALL of:

1. `tsconfig.base.json` -- path alias
2. App `tsconfig.json` files -- add path alias (apps duplicate base paths due to relative path differences)
3. `Dockerfile` -- COPY + rm -rf commands in relevant stages
4. `docker-compose.prod.yml` -- service entry
5. `.github/workflows/deploy.yml` -- build matrix + `docker compose up -d`
6. Service `tsconfig.app.json` -- include + references
7. Deps stage -- COPY the new `package.json`
8. For UI libraries: add `@source` directive in consuming apps' `globals.css` (Tailwind v4)
9. For UI libraries: add content path to consuming apps' `tailwind.config.js`

### 2. Drizzle `$dynamic()` `.where()` overwrites (not appends)

Each `.where()` call on a `$dynamic()` query **replaces** the previous filter. Accumulate conditions in an array, then apply a single `.where(and(...conditions))`.

### 3. `NEXT_PUBLIC_*` env vars in Docker

Statically inlined at build time. Use bracket notation `process.env['NEXT_PUBLIC_X']` to prevent inlining, or pass values from server-side at runtime.

### 4. `onConflictDoNothing` masks insert feedback

Returns no error on duplicates, so code returning `{ success: true }` unconditionally gives false positives. Use `.onConflictDoUpdate()`, check row count, or use `.returning()`.

### 5. Chunk `userId` filtering in cascade queries

`chunk.userId` is nullable (from Notion sync era) and may not match current user. In cascade operations (trackNote, trackTopic), filter by noteId/topicId only -- user ownership is enforced at the `recallItem` level.

### 6. Verify changes before presenting to user

After making changes to any app or library, **always** run lint, test, and build for affected projects before telling the user the work is done:

```sh
pnpm nx affected -t lint                       # lint changed projects
pnpm nx affected -t test                       # test changed projects
pnpm nx affected -t build                      # build changed projects
```

Or for a specific project:
```sh
pnpm nx lint @temar/web
pnpm nx test @temar/fsrs-service
pnpm nx build @temar/web --prod                # matches CI/CD Docker build
```

Fix all errors before presenting work as complete. The CI/CD pipeline runs `pnpm nx build <project> --prod` inside Docker for each service — if it doesn't build locally, it won't deploy.

### 7. Database migrations must use `drizzle-kit generate`

**NEVER** create migration SQL files manually. Always:
1. Edit the schema in `libs/db-client/src/schema/*.ts`
2. Run `npx drizzle-kit generate --config libs/db-client/drizzle.config.ts` from the project root
3. The generated SQL file and journal entry are the source of truth
4. Apply with `pnpm migrate`

### 7. Soft-retired recall items

`recall_item.retiredAt` marks items replaced by regeneration. All active review/due queries **must** include `isNull(recallItem.retiredAt)`. Analytics and performance summary queries should include retired items for complete history.

---

## Dead Code

- **`apps/notion_sync-service/`** -- retired, not deployed. Caddyfile still has dead `/api/webhook/*` route.
- **`apps/api/`** and **`apps/api-e2e/`** -- vestigial scaffolds, not deployed.
- **`apps/web/src/app/api/stripe/`** -- replaced by Paddle, dead route handlers.
- **`NOTION_*` env vars** -- unused since sync retirement.

## Shared UI Library (`@temar/ui`)

The `libs/ui/` library contains 24 shadcn/ui components shared between web and admin apps. Import from `@temar/ui` instead of local `@/components/ui/` for shared components.

```tsx
import { Button, Card, CardContent, Input, Select, Table } from '@temar/ui';
```

**Components in shared lib:** button, card, badge, dialog, alert-dialog, input, label, separator, select, table, tabs, textarea, dropdown-menu, popover, tooltip, toggle, toggle-group, command, checkbox, scroll-area, switch, sheet, hover-card, pagination.

**Web-only components** (stay in `apps/web/src/components/ui/`): fab, card-enhanced, chart, calendar, drawer, resizable, sidebar, spinner, skeleton, breadcrumb, avatar, sonner, progress, pagination (re-export).

**Admin-only local:** sonner (custom theme-aware wrapper).

### Tailwind v4 `@source` directive (critical)

Both apps **must** have `@source "../../../../libs/ui/src";` in their `globals.css` for Tailwind v4 to scan the shared library for class usage. Without this, components render with missing/broken styles. The `content` array in `tailwind.config.js` is NOT sufficient for Tailwind v4.

### Card component API

The shared Card uses `gap-6 py-6` with `px-6` on children (NOT the old `p-6`/`p-6 pt-0` pattern). For compact stat cards, use `Card className="gap-2"`. Do NOT use `CardHeader className="pb-2"`.

### Design system

- **Primary color:** Vibrant orange `oklch(0.70 0.19 50)` (matches favicon `#f97316`)
- **Borderless design:** No `border` on cards/containers/overlays -- use tinted backgrounds
- **Corner radius:** `rounded-2xl` for containers/overlays, `rounded-xl` for interactive elements
- **Shadow elevation:** `shadow-md` on cards, `shadow-lg` on overlays
- **Fonts:** Manrope (primary), Lora (serif), plus Merriweather, Literata, Source Serif 4, Crimson Text, Inter (Lexical editor)
- **Colors:** All CSS variables use oklch(). Never wrap in `hsl()`
- **Dark mode active states:** Use `bg-background` (not `bg-card` or `bg-input/30`) for tab/pill active states

### Adding a new shared component

1. Create in `libs/ui/src/components/` with `../lib/utils` for `cn()` import
2. Export from `libs/ui/src/index.ts`
3. No additional Tailwind config needed -- `@source` handles scanning

## Environment Variables

Full list in `.env.template`. Critical note: service endpoint vars (e.g., `FSRS_SERVICE_API_ENDPOINT`) **must include `/api`** suffix. Paddle sandbox uses different CDN (`sandbox-cdn.paddle.com`) and `test_` prefixed tokens.

<!-- nx configuration start-->
<!-- Leave the start & end comments to automatically receive updates. -->

## General Guidelines for working with Nx

- For navigating/exploring the workspace, invoke the `nx-workspace` skill first - it has patterns for querying projects, targets, and dependencies
- When running tasks (for example build, lint, test, e2e, etc.), always prefer running the task through `nx` (i.e. `nx run`, `nx run-many`, `nx affected`) instead of using the underlying tooling directly
- Prefix nx commands with the workspace's package manager (e.g., `pnpm nx build`, `npm exec nx test`) - avoids using globally installed CLI
- You have access to the Nx MCP server and its tools, use them to help the user
- For Nx plugin best practices, check `node_modules/@nx/<plugin>/PLUGIN.md`. Not all plugins have this file - proceed without it if unavailable.
- NEVER guess CLI flags - always check nx_docs or `--help` first when unsure

## Scaffolding & Generators

- For scaffolding tasks (creating apps, libs, project structure, setup), ALWAYS invoke the `nx-generate` skill FIRST before exploring or calling MCP tools

## When to use nx_docs

- USE for: advanced config options, unfamiliar flags, migration guides, plugin configuration, edge cases
- DON'T USE for: basic generator syntax (`nx g @nx/react:app`), standard commands, things you already know
- The `nx-generate` skill handles generator discovery internally - don't call nx_docs just to look up generator syntax

<!-- nx configuration end-->

## Architecture Documentation

After implementing a major feature or architectural change, **update `docs/architecture/`** to reflect the new state:

1. Read the relevant architecture doc(s) and check if diagrams, source file tables, and prose still match the code
2. Update outdated sections (changed data flows, new sync triggers, new services, etc.)
3. Create a new numbered doc if the feature is entirely new (e.g., `13-google-calendar-sync.md`)
4. Update `docs/architecture/README.md` index if new docs are added

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/leykun-gizaw) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
