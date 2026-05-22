## applog

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AppLog is a full-stack blog application built as a **pnpm monorepo** with three packages:

- `packages/frontend` — Vue 3 SPA (`@applog/frontend`)
- `packages/backend` — NestJS REST API (`@applog/backend`)
- `packages/common` — Shared types and utilities (`@applog/common`)

## Commands

All commands are run from the **repo root** unless specified.

### Development

```bash
pnpm fe          # Start frontend only (Vite dev server)
pnpm be          # Start backend only (NestJS watch mode)
pnpm dev         # Start all packages in dev mode
```

### Build

```bash
pnpm build:fe    # Build frontend (turbo handles common dependency automatically)
pnpm build:be    # Build backend (turbo handles common dependency automatically)
pnpm build       # Build all packages
```

### Per-package commands (run from package directory or use `--filter`)

```bash
# Frontend
pnpm --filter @applog/frontend run lint        # Run oxlint + eslint with auto-fix
pnpm --filter @applog/frontend run format      # Prettier format
pnpm --filter @applog/frontend run type-check  # vue-tsc type checking

# Backend
pnpm --filter @applog/backend run lint         # ESLint with auto-fix
pnpm --filter @applog/backend run format       # Prettier format

# Common
pnpm --filter @applog/common run build         # Compile TypeScript
pnpm --filter @applog/common run dev           # Watch mode
```

### Commits

```bash
pnpm c   # Commitizen interactive commit (uses cz-customizable)
```

## Architecture

### `packages/common`

Pure TypeScript library compiled to `dist/`. Exports shared types, constants, and utilities for system config (`ISystemBaseConfig`, `SYSTEM_CONFIG_KEYS`, etc.). Both frontend and backend depend on this as a workspace reference. Turbo's `"dependsOn": ["^build"]` ensures it is built before dependent packages automatically.

### `packages/backend`

NestJS application using:
- **Fastify** as the HTTP adapter (not Express)
- **TypeORM** + **MySQL** for data persistence (schema `synchronize: true`)
- **URI versioning** (default version `1` and `VERSION_NEUTRAL`)
- **Global `AuthGuard`** from `@reus-able/nestjs` on all routes
- **Global `ValidationPipe`** with implicit type conversion
- **Global interceptors/filters**: `TransformInterceptor`, `AllExceptionsFilter`, `HttpExceptionFilter`

All responses are wrapped as `{ data, code, msg }` by `TransformInterceptor`.

Modules: `UserModule`, `PostModule`, `CommentModule`, `PageModule`, `SystemConfigModule`. Each module lives in `src/module/<name>/` with its own controller, service, and `dto/` folder.

Backend listens on port **4000**.

Backend env file loading order (highest to lowest priority): `.env.production.local` → `.env.development.local` → `.env.production` → `.env.development` → `.env`.

### `packages/frontend`

Vue 3 SPA using:
- **Vite** (using `rolldown-vite` build)
- **Vue Router 4** with `createWebHistory`
- **Pinia** for state management
- **Alova** (not Axios/fetch directly) for all HTTP requests — configured in `src/utils/alova.ts`
- **TailwindCSS v4** + `konsta` UI components
- **@vueuse/motion** for animations

#### HTTP Client Pattern

`src/utils/alova.ts` creates the singleton `alovaInstance`. All API modules in `src/api/` use this instance. The alova response interceptor automatically unwraps `{ data, code, msg }` responses — API functions return the inner `data` directly. JWT token is auto-injected from `localforage` via `src/utils/token.ts`.

#### Auth / Routing

`useUserStore.initializeAuth()` runs at app startup and returns a Promise. `setupPermissionGuard(authInitPromise)` wraps this promise into a `beforeEach` guard that waits for auth before allowing protected navigation. Route `meta` uses `{ requiresAuth: true, roles: ['admin'] }` pattern.

#### State Stores (Pinia)

- `useUserStore` — current user info and auth
- `useSystemStore` — system-wide config fetched from backend
- `useAdminStore` — admin-specific state
- `useLayoutStore` — layout/UI state
- `usePostListStore` — post list pagination state

#### Markdown Rendering System

`src/utils/markdown/` contains a custom pipeline built on `remark`/`rehype` (unified) with additional custom plugins:
- `remark-bbcode-plugin.ts` — parses `[tag]...[/tag]` BBCode syntax into AST nodes
- `remark-meme-plugin.ts` — meme/sticker support
- `component-registry.ts` — maps BBCode tag names to Vue components for rendering
- `bbcode-handler-registry.ts` — maps BBCode tag names to hast handler functions

Registered BBCode components: `[art]` → ArticleCard, `[bili]` → BiliVideo, `[collapse]` → Collapse, `[photos]` → Photos, `[dplayer]` → VideoPlayer. Registered handler tags: `[tag]`, `[warn]`, `[notice]`.

The `MarkdownRenderer` component (`src/components/ui/markdown-renderer/`) uses `IntersectionObserver` for lazy image loading.

#### Frontend Path Alias

`@/` maps to `src/` in frontend (configured in `tsconfig.app.json` and Vite).

## Environment Setup

**Backend** (`packages/backend/.env`):
```
SSO_URL, SSO_ID, SSO_SECRET, SSO_REDIRECT  # SSO OAuth integration
FRONT_URL                                   # Frontend origin
MYSQL_SERVER, MYSQL_PORT, MYSQL_USER, MYSQL_PASSWORD, MYSQL_DATABASE
TOKEN_SECRET, SALTROUND
MODE, PORT
SYSTEM_CONFIG_PREFIX                        # Prefix for system-level config keys (default: SYSTEM_)
SYSTEM_ADMIN_ROLE_VALUE                     # Numeric role value for admin users
```

**Frontend** (`packages/frontend/.env`):
```
VITE_SSO_LOGIN_URL
VITE_SSO_CALLBACK_URL
VITE_SSO_CLIENT_ID
VITE_API_BASE_URL    # Backend URL, defaults to http://localhost:4000
```

## Key Conventions

- **SSO authentication**: The backend integrates with an external SSO service (`@reus-able/sso-utils`). Users authenticate via SSO redirect flow (`/user/login` → `/user/callback`).
- **Admin role**: Role-based access uses a numeric `role` value compared against `SYSTEM_ADMIN_ROLE_VALUE`. In the frontend, `roles: ['admin']` in route meta triggers the permission guard.
- **System config**: Key-value store in DB. Keys prefixed with `SYSTEM_` (configurable) are read-only for non-admins.
- **Backend path alias**: `@/` maps to `src/` in backend TypeScript (via `tsconfig-paths`).

---
> Source: [youranreus/Applog](https://github.com/youranreus/Applog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
