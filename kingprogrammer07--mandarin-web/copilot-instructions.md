## mandarin-web

> This file provides guidance to AI coding agents when working with code in this repository.

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Project Overview

**Mandarin Cargo** is a Telegram Mini App and Progressive Web App (PWA) for cargo/shipping logistics management. It serves multiple user roles — end clients tracking their parcels, warehouse workers managing flights and cargo, accountants verifying transactions, and administrators managing users and roles.

The app is primarily designed to run inside Telegram via the Telegram WebApp SDK, but several internal routes (`/admin/*`, `/pos`, `/flights`, `/statistics`) are accessible directly from a browser for back-office staff.

## Technology Stack

| Layer | Technology |
|-------|-----------|
| Framework | React 19 (with StrictMode) |
| Language | TypeScript 5.9 (strict mode, no `any`) |
| Build Tool | Vite 7 with `@vitejs/plugin-react-swc` |
| Styling | Tailwind CSS 4 + `tw-animate-css` |
| UI Primitives | Radix UI (shadcn/ui-style wrappers in `src/components/ui/`) |
| Icons | Lucide React |
| State (Server) | TanStack Query (React Query) v5 |
| State (Client) | Zustand v5 |
| Forms | React Hook Form + Zod v4 |
| Animations | Framer Motion |
| Charts | Recharts |
| Toasts | Sonner |
| Maps | Leaflet + React-Leaflet |
| QR Scanning | html5-qrcode |
| i18n | i18next + react-i18next |
| Offline DB | IndexedDB via `idb` |
| HTTP Client | Axios (two instances: JSON and multipart/form-data) |

## Build & Development Commands

```bash
npm run dev        # Start Vite dev server (host: true, port: 5173)
npm run build      # TypeScript compile (tsc -b) + Vite production build
npm run lint       # ESLint check across the project
npm run preview    # Preview production build locally
```

No test suite is configured.

## Project Structure

```
src/
  api/               # Axios clients and domain API services
    client.ts        # apiClient (JSON) & apiClientFormData (multipart)
    services/        # Per-domain async functions (auth, cargo, flights, admin, etc.)
    hooks/           # TanStack Query custom hooks (useAdminClients, etc.)
  components/        # React components
    ui/              # Shadcn/ui-style reusable primitives (button, dialog, input, etc.)
    admin/           # Admin-layout shell components
    carousel/        # Home page carousel management
    delivery/        # UzPost delivery request flow
    expectedCargo/   # Warehouse expected-cargo UI
    history/         # Client cargo history
    manager/         # Manager client data table
    modals/          # Shared modal components
    navigation/      # Nav bars (UserNav, VerificationNav, FloatingNavbar)
    notifications/   # Notification center
    pages/           # Page-level components (DeliveryHistoryPage, etc.)
    profile/         # User profile sections
    statistics/      # Charts and stat cards
    user_page/       # User home action buttons
    verification/    # Accountant verification UI
    warehouse/       # Warehouse transaction tables & filters
    wallet/          # Wallet & card management modals
  config/
    config.ts        # Runtime env var validation (VITE_API_BASE_URL, etc.)
  constants/         # Static JSON data (districts in uz/ru)
  hooks/             # Custom React hooks (useConfirm, useToast, useProfile, etc.)
  i18n/              # i18next configuration and locale files (uz.json, ru.json)
  lib/               # Utility libraries (cn, telegram helpers, validation, formatting)
  pages/             # Top-level page components
    admin/           # Admin pages (accounts, roles, audit, carousel, warehouse, etc.)
    dashboard/       # Client dashboard & track-code search
  schemas/           # Zod validation schemas
  store/             # Zustand stores (warehouse filters, expected cargo, manager state)
  types/             # Global TypeScript type definitions
    telegram-web-app.d.ts  # Custom Telegram WebApp SDK types
  utils/             # Helper modules (offlineStorage, audioUtils, numberFormat, etc.)
  App.tsx            # Root component: auth, routing, layout orchestration
  main.tsx           # React root render, QueryClient, ThemeProvider, service worker registration
  index.css          # Tailwind imports, CSS variables (light/dark), custom scrollbar, focus styles
```

## Architecture Deep Dive

### Entry & Auth Flow

1. `index.html` loads the Telegram WebApp SDK (`telegram-web-app.js`) and PWA manifest.
2. `src/main.tsx` creates the React root, wraps the tree in `QueryClientProvider` and `ThemeProvider`, and registers the service worker in production only.
3. `TelegramWebAppGuard` wraps the entire app. It:
   - Skips Telegram validation for browser routes (`/admin/*`, `/pos`, `/flights`, `/statistics`).
   - Validates `initData` with the backend for all other routes.
   - Calls `window.Telegram.WebApp.ready()` and `.expand()` on success.
   - Attempts auto-login via `/auth/telegram-login` if no token exists.
4. `App.tsx` performs the second auth gate:
   - Admin tokens live in `localStorage` (`access_token`, `admin_role`).
   - Regular user tokens live in `sessionStorage` (`access_token`).
   - Calls `/auth/me` to validate user tokens.
   - On 401, clears storage and dispatches `auth:logout`.

**Important**: Admin auth and user auth are mutually exclusive. Admin tokens are sent via `X-Admin-Authorization`; user tokens via `Authorization`. Sending both causes the backend to reject the request.

### Routing

Custom history-based routing — **not** React Router components. `App.tsx` maintains a `currentPage` state of type `Page` (a large string union).

Key functions:
- `resolvePageFromPath(path)` — maps a URL path to a `RouteInfo` object.
- `checkAccess(targetPage, role)` — enforces role-based whitelists.
- `applyRoute(routeInfo, role, method)` — syncs React state, `window.history`, and the URL bar atomically.
- `popstate` listener handles browser back/forward.

The `ROLE_CONFIG` object defines default landing pages and allowed page arrays for each role.

### API Layer (`src/api/`)

All HTTP goes through Axios instances defined in `src/api/client.ts`:

- `apiClient` — JSON requests, 30s timeout.
- `apiClientFormData` — multipart uploads, 60s timeout.

Request interceptors attach:
- `X-Telegram-Init-Data` from `window.Telegram.WebApp.initData`.
- `X-Admin-Authorization: Bearer <token>` (if admin token in `localStorage`).
- `Authorization: Bearer <token>` (if user token in `sessionStorage`).
- `Accept-Language` from i18next (`uz` or `ru`).

Response interceptors:
- On 401 (except whitelisted public endpoints), clear all tokens and dispatch `auth:logout`.
- Error messages are resolved with hardcoded Uzbek text for infrastructure errors (401, 403, 5xx) because the backend returns English for those. Business-logic errors (400, 409, 422) use the backend's `detail` field, which is already in Uzbek.

Domain services live in `src/api/services/` (auth, cargo, flights, payments, stats, admin, etc.) and are plain async functions that call the Axios clients.

### State Management

- **TanStack Query** — server/API state, caching, background refetch. Default config: `retry: 1`, `staleTime: 5 * 60 * 1000`.
- **Zustand** — client UI state (warehouse filters, expected-cargo bulk edits, manager search).
- **React Hook Form + Zod** — form state and validation.

### UI & Styling

- Tailwind CSS 4 with CSS variables for theming (`@theme inline`).
- Dark mode supported via `next-themes` (`class` attribute strategy).
- Orange/amber is a useful brand accent, but it is not the only allowed palette. Use other colors when they improve hierarchy, meaning, accessibility, or role-specific workflows. Choose palettes deliberately, keep contrast strong, and avoid making every new surface look like a variation of the same orange/amber theme.
- Custom focus rings may use the established orange accent (`oklch(0.646 0.222 41.116)`) or another accessible token when the surrounding UI requires a different semantic color.
- Thin custom scrollbars are styled globally for WebKit and Firefox.
- `font-size: 16px` is forced on inputs to prevent iOS zoom.

### Premium Mandarin User Theme

The approved user-side dark-mode direction is **Premium Mandarin**:

- Keep Mandarin Cargo's orange/amber brand color, but never use it as a full-screen muddy brown/amber wash.
- Dark surfaces should start from a clean obsidian base (`#06080d`, `#0a0e15`, `#0f151f`) with subtle top warm light.
- Orange/amber should appear as controlled premium accents: logo, active nav pill, key CTA, icon badge, status chip, thin rim light.
- Avoid amber diagonal grids, large blurred amber orbs, amber rings, decorative watermarks, and repeated animated blobs on user pages.
- Mobile-first matters more than hover. Use `active:` or `whileTap` feedback, clear tap targets, strong text contrast, and compact scannable cards.
- Flight/report list cards should not show "new", "paid", or other status labels unless the API provides real status/freshness data. Do not invent state from a flight name string.
- The current reference preview is `frontend/design-previews/premium-mandarin-flightcard-preview.html`.

### Senior Frontend & UX Expectations

- Think like a senior frontend engineer and UI/UX designer before changing UI: understand the user role, workflow frequency, data density, failure states, and mobile/desktop constraints.
- Prefer existing local patterns, shared primitives, strict TypeScript types, accessible Radix/shadcn-style components, and predictable state boundaries.
- Build complete states for real usage: loading, empty, error, disabled, optimistic or pending states where appropriate, validation, and recovery paths.
- Keep operational screens efficient and scannable. Do not turn admin, warehouse, accounting, or logistics workflows into marketing-style pages.
- Use semantic color intentionally: status, risk, success, warnings, ownership, grouping, and action priority should be visually distinct and accessible.
- Verify responsive behavior, text overflow, tap targets, keyboard focus, and dark mode impact when a UI change can affect them.
- Avoid `any`, fragile string parsing, duplicated business rules, hidden coupling, and cosmetic changes that make future maintenance harder.

### Internationalization

i18next is configured in `src/i18n/config.ts` with two locales:
- `uz` (Uzbek) — default and fallback language.
- `ru` (Russian).

All user-facing API error messages for infrastructure codes are in Uzbek. The `Accept-Language` header is sent on every request.

### Offline Support

- IndexedDB via the `idb` library caches cargo data for offline warehouse use.
- `src/utils/offlineStorage.ts` and `src/utils/warehouseOfflineStorage.ts` handle persistence.
- A minimal service worker (`public/sw.js`) provides a network-first strategy for the app shell, satisfying PWA install criteria without interfering with API calls.

### PWA / Installability

- `manifest.json` defines the app as a standalone PWA with theme color `#f59e0b`.
- `start_url` is `/admin/login`.
- The service worker is only registered in production (`import.meta.env.PROD`).

## User Roles & Access Control

| Role | Default Page | Allowed Pages |
|------|-------------|---------------|
| `user` | `user-home` | user-home, user-profile, user-history, user-reports |
| `worker` | `flights` | flights, cargo-list, cargo-add, passkey-page, expected-cargo |
| `accountant` | `verification-search` | verification-*, pos-dashboard, admin-profile, passkey-page |
| `admin` | `admin-accounts` | Full access except manager-page |
| `super-admin` | `admin-accounts` | Full access including manager-page |
| `manager` | `manager-page` | manager-page, admin-carousel, admin-profile, passkey-page, flight-schedule-admin |
| `warehouse_worker` | `warehouse-page` | warehouse-page, expected-cargo, admin-profile, passkey-page |
| `warehouse` | `warehouse-page` | Same as warehouse_worker |

The backend JWT may contain a `home_page` claim that overrides the static default.
Admin pages (`admin-accounts`, `admin-roles`, etc.) are rendered inside `AdminLayout` only for roles that have `admin-accounts` in their allowed list. Other roles accessing `admin-profile` or `admin-carousel` get a standalone view without the sidebar.

## Code Style & Conventions

- **TypeScript strict mode is enforced.** `noUnusedLocals`, `noUnusedParameters`, `verbatimModuleSyntax`, and `noUncheckedSideEffectImports` are enabled. Never use `any`.
- **Path alias**: `@/` maps to `src/` (configured in `vite.config.ts` and `tsconfig.app.json`).
- **Comments**: Technical comments are in English. UI text and validation messages are in Uzbek.
- **Imports**: Use type-only imports where possible (`import type { … }`).
- **File naming**: PascalCase for components, camelCase for utilities/hooks, kebab-case for JSON locale files.
- **Component structure**: Prefer functional components with hooks. Keep components modular and focused.
- **Error handling**: Always handle `null`/`undefined` gracefully (optional chaining `?.`, explicit checks). API errors are surfaced via Sonner toasts or inline form errors.
- **Environment variables**: All env vars must be prefixed with `VITE_` and are strictly validated at runtime in `src/config/config.ts`. Missing vars throw hard errors on startup.

## Environment Variables

Required variables (must be present in `.env`):

```
VITE_API_BASE_URL        # Backend API origin (e.g. https://api.example.com)
VITE_API_INIT_DATA_URL   # Path segment for init-data validation endpoint
VITE_API_LOGIN_URL       # Path segment for login endpoint
VITE_API_REGISTER_URL    # Path segment for register endpoint
```

## Security Considerations

- **Telegram initData validation**: Every non-browser route requires valid Telegram `initData`. The backend validates the HMAC signature.
- **Dual auth tokens**: Admin and user tokens are stored separately and sent in different headers to prevent collisions.
- **Automatic logout**: 401 responses (except whitelisted endpoints) immediately purge all tokens and reload the app into the login flow.
- **CSP / External scripts**: `index.html` loads `telegram-web-app.js` from `https://telegram.org`.
- **No secrets in source**: All API URLs come from environment variables.

## Deployment

The project is configured for **Vercel** via `vercel.json`:
- Uses `@vercel/static-build` with `distDir: "dist"`.
- SPA fallback: all unmatched routes serve `index.html`.

## Task Planning & Execution (TASK.md)

Whenever you receive a new feature request or a complex task, you MUST act as a Tech Lead and manage the workflow using a file named `TASK.md` in the root directory.

Before writing or modifying any source code, you must:
1. Create or overwrite `TASK.md`.
2. Write a clear **Objective** (what we are trying to achieve).
3. Draft a strict **Implementation Plan** using markdown checkboxes (e.g., `- [ ] Step 1`, `- [x] Step 2`).
4. Write a brief **Walkthrough/Architecture** section explaining how the components will interact.

As you complete each step of the plan, you MUST update `TASK.md` (checking off the boxes). If the plan changes due to errors or new requirements, update the document to reflect the reality.

This file will serve as our shared "living memory" and progress tracker.

---
> Source: [Kingprogrammer07/mandarin_web](https://github.com/Kingprogrammer07/mandarin_web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
