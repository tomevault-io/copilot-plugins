## project-overview

> understand layout and tech used in this codebase

# Project: whatdidyougetdonetoday-ai

Monorepo SaaS template: auth, payments, UI.

## Core Technologies

-   **Runtime/PM**: Bun
-   **Language**: TypeScript
-   **API**: tRPC (type-safe client-server)
-   **Styling**: Tailwind CSS
-   **UI Components**: Shadcn UI (Radix UI + Tailwind)

## Monorepo Packages

### 1. `client/` (Web Frontend)

-   **Framework**: React (CRA + Craco)
-   **Styling**: Tailwind CSS, Shadcn UI
-   **API**: tRPC client (to `server/`)
-   **Key Libs**: `react-router-dom`, `@tanstack/react-query` (via tRPC), `zod`

### 2. `server/` (Backend)

-   **Runtime**: Bun
-   **API**: tRPC (type-safe endpoints)
-   **Database**: MongoDB (Mongoose ODM)
    -   **Schema Location**: Mongoose schemas are defined in `server/src/db/schemas/` (e.g., `categorySchema.ts`, `userSchema.ts`).
-   **Payments**: Stripe
-   **Key Libs**: `express` (tRPC adapter), `mongoose`, `zod`, `jsonwebtoken`

### 3. `electron-app/` (Desktop App)

-   **Framework**: Electron
-   **Renderer UI**: React, TypeScript
-   **Builder**: `electron-vite`
-   **API**: tRPC client (to `server/` or local main process)

### 4. `shared/` (Shared Code)

-   **Purpose**: Shared TypeScript types, interfaces, Zod schemas, utils.
-   **Key File**: `shared/types.ts` (central for shared type definitions).
-   **Import Style**: Direct file imports (e.g., `import { User } from 'shared/types';`), no `index.ts` main entry.
-   **Benefit**: Cross-package type safety (critical for tRPC).

## Key Dev Guidelines

-   **Type Safety**: Use `shared/` types for data exchange.
-   **UI**: Build with Shadcn UI + Tailwind CSS.
-   **API**: Follow tRPC patterns (server routers/procedures, client calls).
-   **Env Vars**: Manage as per main `README.md`.
-   **Bun**: For scripts, deps, server runtime.

---
> Source: [moritzWa/cronus](https://github.com/moritzWa/cronus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
