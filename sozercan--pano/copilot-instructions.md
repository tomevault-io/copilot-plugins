## pano

> Guidance for AI coding assistants working on this repository.

# CLAUDE.md

Guidance for AI coding assistants working on this repository.

## What is Pano?

A modern **TestGrid viewer** for Kubernetes CI/CD dashboards built with **React** and **TypeScript**.

- **Dashboard Navigation** — Accordion sidebar with groups, dashboards, and tabs
- **Virtual Scrolling Grid** — Efficiently renders thousands of test results
- **Subscriptions** — Track dashboards, tabs, or individual tests
- **Global Search** — Quick search via `Cmd/Ctrl+K`
- **Desktop App** — Optional Tauri wrapper with native notifications

## Tech Stack

| Layer | Technology |
|-------|------------|
| Runtime | Bun 1.2+ |
| Frontend | React 19, TypeScript |
| Build | Vite 6 |
| Styling | Tailwind CSS 4 |
| State | Zustand 5 (localStorage persistence) |
| Data Fetching | TanStack Query 5 |
| Routing | TanStack Router |
| Validation | Zod schemas |
| Desktop | Tauri 2 |

## Project Structure

```
src/
├── frontend/
│   ├── components/
│   │   ├── dashboard/      # StatusBadge, dashboard views
│   │   ├── grid/           # TestGrid, VirtualRow, GridCell, FilterToolbar
│   │   ├── layout/         # AppShell, Header, Sidebar, AccordionNav
│   │   ├── search/         # SearchModal
│   │   └── subscriptions/  # SubscribeButton, SubscriptionManager
│   ├── hooks/              # useTestGridApi, useSearch, useVirtualScroll
│   ├── stores/             # Zustand: ui, theme, subscription, filter
│   ├── routes/             # TanStack Router file-based routes
│   └── lib/                # api-client, storage, tauri utils
└── shared/
    └── schemas/            # Zod schemas for API validation
apps/
└── desktop/                # Tauri desktop wrapper
tests/                      # Bun test files
```

## Documentation

See `docs/` for details:
- **[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)** — Component hierarchy, state management, data flow
- **[docs/TESTGRID_API_REPORT.md](docs/TESTGRID_API_REPORT.md)** — API endpoints, response formats

## Commands

```bash
bun install              # Install dependencies
bun run dev              # Start dev server (localhost:5173)
bun test                 # Run tests
bun run build            # Production build
bun run tauri:dev        # Start Tauri desktop app
```

## Critical Rules

> **Validate all API responses** — Use Zod schemas from `src/shared/schemas/`. Never trust raw API data.

> **Use TanStack Query for data fetching** — All API calls go through hooks in `useTestGridApi.ts`. Don't fetch directly in components.

> **Persist client state with Zustand** — Stores in `stores/` use persist middleware. Don't use raw localStorage.

> **Virtual scroll for large lists** — TestGrid uses `useVirtualScroll`. Never render all rows.

## Key Patterns

### API Data Flow

```
Component → useTestGridApi hook → TanStack Query → api-client.ts → Zod validation
```

### State Management

| Store | Purpose |
|-------|---------|
| `uiStore` | Sidebar state, expanded groups |
| `themeStore` | Dark/light mode |
| `subscriptionStore` | User subscriptions |
| `filterStore` | Grid filters per tab |

### TestGrid API

Base URL: `https://testgrid-api.prow.k8s.io/api/v1`

| Endpoint | Purpose |
|----------|---------|
| `GET /dashboard-groups` | List all groups |
| `GET /dashboard-groups/:group` | Dashboards in group |
| `GET /dashboards/:name/tab-summaries` | Tab status overview |
| `GET /dashboards/:name/tabs/:tab/rows` | Test result grid |

## Key Files

| File | Purpose |
|------|---------|
| `src/frontend/lib/api-client.ts` | TestGrid API client |
| `src/frontend/hooks/useTestGridApi.ts` | TanStack Query hooks |
| `src/frontend/components/grid/TestGrid.tsx` | Virtual scrolling grid |
| `src/frontend/stores/subscriptionStore.ts` | Subscription persistence |
| `src/shared/schemas/` | Zod schemas for all API responses |

---
> Source: [sozercan/pano](https://github.com/sozercan/pano) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
