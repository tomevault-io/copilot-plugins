## asist-front

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev        # Start dev server (Vite, port 5173)
npm run build      # Type-check + production build
npm run lint       # ESLint checks
npm run preview    # Preview production build locally
```

**Environment:** Set `VITE_API_BASE_URL` in `.env` to point at the backend (defaults to `https://cozum.net/api/v1`).

## Architecture

**Stack:** React 19 + TypeScript (strict) + Vite 7 + TailwindCSS + Radix UI

### Provider hierarchy (App.tsx)
```
ThemeProvider → AuthProvider → SocketProvider → UserCustomerProvider → BrowserRouter
```

### Contexts (`src/contexts/`)
- **AuthContext** — auth state, tokens (localStorage), contract acceptance status
- **ThemeContext** — light/dark/system; public routes force light mode
- **SocketContext** — Socket.io connection; auto-connects on login, disconnects on logout
- **UserCustomerContext** — shared customer-related state

### Routing
Two protected layout groups:
- **MainLayout** — all authenticated roles except SUPPORT
- **SupportLayout** — SUPPORT role only (`/dashboard/support/*`)

**ProtectedRoute** enforces: authentication, allowed roles, and contract acceptance (skippable with `skipContractCheck`).

### User roles (least → most privileged)
`USER` → `BRANCH_USER` → `BRANCH_ADMIN` → `AGENCY_ADMIN` → `SUPER_AGENCY_ADMIN` → `SUPER_ADMIN`

`SUPPORT` is a parallel role with its own layout and limited routes.

### API layer
- **`src/api/config.ts`** — Axios instance; request interceptor injects `Authorization` header + `X-Selected-Agency-Id` for `AGENCY_ADMIN`; response interceptor handles 401 with token refresh (falls back to `/login` on failure)
- **`src/services/apiService.ts`** — `createCRUDService<T>(endpoint)` factory generating `.getAll()`, `.getById()`, `.create()`, `.update()`, `.delete()`; entity-specific services extend this
- API responses follow `{ success: boolean; data: T; message?: string }`

### Path alias
`@/` maps to `src/` — use it for all internal imports.

### Styling
TailwindCSS with HSL CSS variables for theming. Dark mode via `.dark` class on `<html>`. `cn()` utility in `src/lib/utils.ts` for conditional class merging.

### Key integrations
- **PayTR** — payment gateway iframe (`src/components/payment/`, `src/services/paytrService.ts`)
- **Tesseract.js** — OCR for vehicle document scanning (`src/services/ocrService.ts`)
- **Turkey map** — `ts-turkey-map` / `turkey-map-react` for regional sales visualization
- **Recharts + Chart.js** — dashboard analytics
- **React Quill** — rich text editor for CMS content

---
> Source: [alperendoniglobal/asist-front](https://github.com/alperendoniglobal/asist-front) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
