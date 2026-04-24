## kohout

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Development (runs server + client concurrently)
npm run dev

# Build (sequential: shared → client → server)
npm run build

# Production
npm start                    # node server/dist/index.js

# Workspace-specific dev
npm run dev -w server        # tsx watch src/index.ts (port 3001)
npm run dev -w client        # vite dev server (port 5173, proxies /api → :3001)
npm run build -w shared      # must build first if shared types changed
```

No test runner is configured.

## Architecture

**Monorepo** with npm workspaces: `shared/`, `server/`, `client/`.

**Kohout** is a personal ticket aggregation tool that imports tickets from Jira and Azure DevOps into a local SQLite database. It provides unified search, folders, kanban boards, categories, tags, and personal notes — all local-only, never writing back to source systems (one-way sync).

### Shared (`@kohout/shared`)

Single `types.ts` file exporting all domain types, API wrappers, and string literal union types (ProviderType, TicketStatus, TicketPriority, SwimlaneGroupBy). Both server and client depend on this package.

### Server (`@kohout/server`)

Express 5 + sql.js-fts5 (WASM SQLite) + Zod validation.

- **Routes** (`src/routes/`) — 10 route files mounted at `/api/*` in `app.ts`
- **Services** (`src/services/`) — `ticket-service.ts` (import, list, FTS index), `sync-service.ts` (periodic refresh, bulk import)
- **Integrations** (`src/integrations/`) — Jira and Azure DevOps each have `client.ts` (REST API wrapper) + `mapper.ts` (normalize to ExternalTicket)
- **Database** (`src/db/`) — sql.js-fts5 (pure WASM, no native binaries), FTS5 full-text search, migration system (`_migrations` table, SQL files in `migrations/`). Compatibility wrapper (`SqlJsWrapper`) mimics better-sqlite3 API.
- **Middleware** — `error-handler.ts` (AppError class, global catch), `validation.ts` (Zod schema middleware attaching to `req.validated`)

DB location: `~/.kohout/kohout.db` (override with `DATABASE_PATH` env var).

Every POST/PUT/PATCH route uses Zod schema validation middleware. All route handlers use try/catch → `next(err)` → global error handler.

### Client (`@kohout/client`)

React 19 + Vite 6 + Tailwind CSS v4 + TanStack Query + Zustand.

**Routing** (react-router-dom v7, wrapped in `AppShell` layout):
- `/` — Dashboard (stats, pinned tickets, recent activity, category chart)
- `/tickets` — Three-panel mail view (folder sidebar + ticket list + detail panel)
- `/kanban` — Kanban board with drag-and-drop
- `/settings` — Provider configs, sync settings, categories, tags

**State management**:
- TanStack Query for server state (30s stale time, 1 retry). Query keys: `['tickets', params]`, `['ticket', id]`, `['dashboard-stats']`, etc.
- Zustand stores (`src/stores/`) for UI state: filters, theme (persisted to localStorage), UI toggles

**API layer** (`src/api/`): `client.ts` provides `apiFetch<T>()` wrapper. Domain modules export custom hooks (`useTickets()`, `useImportTicket()`, etc.) that wrap useQuery/useMutation with automatic query invalidation.

**Path alias**: `@` → `client/src` (configured in vite.config.ts)

**Drag-and-drop** (@dnd-kit): tickets → folders, folders → folders, kanban cards → columns/swimlanes.

## Design System

- **Fonts**: Outfit (headings), Plus Jakarta Sans (body), JetBrains Mono (mono) — via `@theme` CSS variables and Tailwind classes `font-heading`, `font-body`, `font-mono`
- **Colors**: OKLCH color space. Amber/copper primary on cool slate neutrals. All defined as CSS custom properties in `client/src/index.css`
- **Cards**: Use `ring-1 ring-border/40` instead of `border`. Use `shadow-[var(--shadow-card)]` for elevation
- **Dividers**: Use `shadow-[0_1px_0_0_var(--color-border)]` instead of `border-b` for subtle lines
- **UI components** (`src/components/ui/`): Hand-rolled Shadcn-like primitives (button with CVA variants, card, dialog, select, input, tabs, badge, etc.) — NOT @shadcn/ui
- **Noise texture**: SVG fractal noise overlay applied globally
- **Dark mode**: Supported via theme store, CSS custom properties swap lightness values

## Conventions

- All packages use ES Modules (`"type": "module"`). Server TypeScript imports use `.js` extensions
- SQLite FTS5 for search with LIKE fallback if FTS query fails
- Folder hierarchy uses recursive descent for folder IDs and ticket counts
- Chunk size warning (~985KB JS bundle) is expected due to recharts + dnd-kit

## Important: No Native Binaries

App is distributed as portable zip via GitHub Releases. **Nikdy nepřidávat závislosti s nativními C/C++ binárkami** (např. better-sqlite3, sharp, bcrypt). Firemní antiviry na Windows blokují `.node` soubory extrahované ze stažených zipů (Mark of the Web). Proto se používá sql.js-fts5 (čistý WASM) místo better-sqlite3. Při volbě nových závislostí vždy preferovat pure JS/WASM alternativy.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tomkolovratnik) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
