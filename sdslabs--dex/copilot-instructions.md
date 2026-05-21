## dex

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Dex** — a full-stack TypeScript CMS for managing collections of items (links, notes). Monorepo with Turborepo + pnpm workspaces.

**Workspace packages:**

- `apps/server` (`@repo/server`) — Hono + tRPC backend with PostgreSQL
- `apps/web` (`@repo/web`) — React 19 + Vite frontend
- `apps/extension` (`@repo/extension`) — Browser extension via WXT
- `packages/ui` (`@repo/ui`) — Shared Radix/shadcn component library

## Commands

```bash
# Development (all apps concurrently)
pnpm dev

# Single app dev
pnpm dev --filter @repo/server
pnpm dev --filter @repo/web

# Build / Lint / Type-check (all via Turbo)
pnpm build
pnpm lint
pnpm check-types
pnpm format                    # Prettier

# Database (PostgreSQL 16 in Docker)
pnpm db:up                     # Start container
pnpm db:down                   # Stop container
pnpm db:push                   # Push schema directly
pnpm db:generate               # Generate migration from schema changes
pnpm db:migrate                # Run pending migrations
pnpm db:studio                 # Drizzle Studio UI

# Add shadcn/ui component
pnpm ui-add <component>
```

**Requirements:** Node >=22, pnpm 10.28.1

## Architecture

### Backend (`apps/server`)

- **Hono** HTTP server with **tRPC** for type-safe RPC
- **Drizzle ORM** with PostgreSQL — schema-as-code in `src/db/schema/`
- **better-auth** for authentication (Google OAuth, API keys with rate limiting)
- **MCP server** at `POST /mcp` for Claude integration (`src/mcp/`)
- **Groq AI** for auto-tagging and content summarization (`src/services/utils/agent/`)
- Environment validation via `@t3-oss/env-core` + Zod in `src/lib/env.ts`

**Server entry:** `src/index.ts` — routes: `/api/auth/*`, `/api/trpc/*`, `/mcp`, `/ping`

**Service layer pattern:** Business logic in `src/services/`, routers in `src/trpc/routers/`, RBAC in `src/services/rbac/`

**tRPC routers:** items, collections, collectionAccess, users, ogp, apiKeys, notifications, invitations

### Frontend (`apps/web`)

- **React 19** + **React Router v7** + **Vite**
- **tRPC client** with TanStack React Query for server state
- **Zustand** for client-only state (`src/lib/stores/`)
- **@repo/ui** for all UI components (Radix + shadcn + Tailwind CSS v4)
- **dnd-kit** for drag-and-drop, **motion** for animations

**Routes:** `/` (login), `/dashboard` (collections), `/dashboard/:collectionId`, `/dashboard/api-keys`

**Path alias:** `~/*` maps to `src/*` in both server and web

### Database Schema

Key tables: `user`, `session`, `account`, `collections`, `items` (with tsvector full-text search), `invitations`, `notifications`, `apikey`

Junction tables: `user_collections` (with role: owner/admin/member), `collection_items`, `api_key_collections`

PostgreSQL enums: `item_type`, `collection_access_role`, `UserStatus`, `invitation_status`, `notification_type`, `api_key_mode`

### Browser Extension (`apps/extension`)

Built with **WXT** framework. Uses same tRPC/auth clients as web. Custom PostCSS rem-to-px plugin (extensions can't use CSS rem).

### Shared UI (`packages/ui`)

Exports: `components/*`, `hooks/*`, `lib/*`, `icons`, `globals.css`. Import as `@repo/ui/<export>`.

## Type Safety

The frontend imports the `AppRouter` type from `@repo/server/trpc` to get end-to-end type inference. The server exports types via `trpc-typegen` script (declaration-only emit to `dist/`).

**Props must be inferred from router outputs.** In `apps/web`, never hardcode prop types that can be inferred from tRPC router outputs. Use `RouterOutput['router']['procedure']` (or indexed access on the query return type) to derive component props. This keeps the frontend in sync with backend changes automatically.

**UI-side RBAC uses shared helpers.** When a component needs to check permissions or determine available role actions, import from `@repo/server/rbac/helpers` (`hasPermission`, `canManageMembers`, `canEditMember`, `getAvailableRoleActions`, etc.). Do not duplicate RBAC logic in frontend code.

## CI/CD

- **Pre-commit hook:** Prettier via Husky + lint-staged
- **PR CI:** Build → type-check → test (Node 22, pnpm 10.28.1)
- **Deploy:** Server as Docker image to DigitalOcean registry; web as static Vite build

---
> Source: [sdslabs/dex](https://github.com/sdslabs/dex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
