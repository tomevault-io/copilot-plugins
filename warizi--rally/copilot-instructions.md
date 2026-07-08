## rally

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

**Rally** is an Electron desktop application built with React, TypeScript, and SQLite. It follows **Feature-Sliced Design (FSD)** architecture with a clean separation between the Electron main process and the React renderer.

## Commands

```bash
# Development
npm run dev              # Start Electron app with hot reload

# Build
npm run build            # Typecheck + build all (main, preload, renderer)
npm run build:mac        # macOS DMG
npm run build:win        # Windows NSIS installer
npm run build:linux      # Linux AppImage/snap/deb

# Type Checking
npm run typecheck        # Check both main/preload and renderer
npm run typecheck:node   # Main + preload only
npm run typecheck:web    # Renderer only

# Code Quality
npm run lint             # ESLint
npm run format           # Prettier

# Database
npm run db:generate      # Generate Drizzle migrations from schema changes
npm run db:migrate       # Apply pending migrations
npm run db:studio        # Open Drizzle Studio (DB GUI)

# Testing
npm run test             # Run all tests
npm run test:watch       # Watch mode (Node tests)
npm run test:web         # Renderer tests only
npm run test:web:watch   # Renderer watch mode
```

## Architecture

### Electron 3-Process Model

```
src/main/        → Electron main process (Node.js, DB access, IPC handlers)
src/preload/     → Security bridge (exposes limited API to renderer via contextBridge)
src/renderer/    → React application (browser-like environment, no direct Node access)
```

The renderer communicates with the main process **only through the preload bridge** (`window.api.*`). Types for this bridge are in `src/preload/index.d.ts`.

### Renderer: Feature-Sliced Design

```
src/renderer/src/
├── app/         → Root providers, router, layouts, global styles
├── pages/       → Full-page route components
├── widgets/     → Complex composite UI modules
├── features/    → User interaction logic (forms, actions)
├── entities/    → Domain models and their UI
└── shared/      → Reusable utilities, hooks, UI components
```

**Import rules are enforced by ESLint:** layers can only import from layers below them (`app → pages → widgets → features → entities → shared`). Importing upward is a lint error.

### Path Aliases

Configured in `electron.vite.config.ts` and `tsconfig.web.json`:

```ts
@/          → src/renderer/src/
@app/       → src/renderer/src/app/
@pages/     → src/renderer/src/pages/
@widgets/   → src/renderer/src/widgets/
@features/  → src/renderer/src/features/
@entities/  → src/renderer/src/entities/
@shared/    → src/renderer/src/shared/
```

### Database

- **Engine**: SQLite via `better-sqlite3`
- **ORM**: Drizzle ORM
- **Schema**: `src/main/db/schema/index.ts`
- **Migrations**: auto-generated to `src/main/db/migrations/`
- **Dev DB file**: `rally-dev.db` (project root); production uses Electron's `userData` path
- After changing the schema, always run `npm run db:generate` then `npm run db:migrate`

### Routing

React Router v7 using **hash-based routing** (required for Electron file protocol). Configured in `src/renderer/src/app/routes/router.tsx`.

### Styling

- **Tailwind CSS v4** — uses `@import 'tailwindcss'` syntax (not the v3 `@tailwind` directives)
- Theme CSS variables (including sidebar tokens) defined in `src/renderer/src/app/styles/global.css`
- Sidebar color tokens: `--sidebar`, `--sidebar-foreground`, `--sidebar-border`, etc. — must be referenced with `border-sidebar-border` (not bare `border-r`) or the border inherits `currentColor`
- **shadcn/ui** components live in `src/shared/ui/` — style: `new-york`, icons: Lucide

### Tab Page Responsive Layout

All page elements rendered inside a tab **must use container queries**, not viewport queries. The `@container` context is provided by `TabContainer` (`src/renderer/src/shared/ui/tab-container.tsx`), which applies `@container` to the outermost div.

**Breakpoints (container-based):**

| Breakpoint | Container width | Tailwind class prefix |
| ---------- | --------------- | --------------------- |
| Small      | < 400px         | (default, no prefix)  |
| Medium     | ≥ 400px         | `@[400px]:`           |
| Large      | ≥ 800px         | `@[800px]:`           |

**Rules:**

- Use `@[400px]:` and `@[800px]:` container query variants instead of `sm:` / `md:` / `lg:` viewport breakpoints for tab content
- Every tab page component must be a descendant of `<TabContainer>` to inherit the `@container` context
- Example: `className="grid grid-cols-1 @[400px]:grid-cols-2 @[800px]:grid-cols-3"`

**Documented exception — `FolderPage`:** `FolderPage` (`src/renderer/src/pages/folder/ui/FolderPage.tsx`) does **not** use `TabContainer`. `FolderTree` owns a full-height virtualized scroll container (`ScrollArea viewportRef` + `useVirtualizer` `scrollElement`); wrapping it in `TabContainer` (even `scrollable={false}`) inserts padding/wrapper divs that change the scroll target and break virtualization and drag-and-drop. The folder feature also uses no container queries, so it does not need the `@container` context. This exception is locked by `FolderPage.test.tsx` (fails if `TabContainer` is added). Any new tab page that genuinely owns its own virtualized scroll should follow the same pattern and document the exception here.

### State Management

- **Zustand v5** — client/UI state
- **TanStack React Query v5** — server/async state (IPC calls to main process)
- Query client provider: `src/renderer/src/app/providers/query-client-provider.tsx`

## Code Style

Enforced by Prettier (`.prettierrc.yaml`):

- Single quotes
- No semicolons
- Print width: 100
- No trailing commas

---
> Source: [warizi/Rally](https://github.com/warizi/Rally) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
