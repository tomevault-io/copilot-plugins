## jinx-inventory

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev       # Start dev server (Vite, proxies /api → http://localhost:8000)
npm run build     # Type-check + production build
npm run lint      # ESLint
npx tsc --noEmit  # Type-check only
```

No test runner is configured.

## Architecture

This is a React + TypeScript + Vite frontend that talks to a Django REST Framework backend running at `http://localhost:8000`. The Vite dev server proxies all `/api/*` requests to the backend (configured via `VITE_API_URL` env var, defaults to `http://localhost:8000`) — there is no Express middleware layer.

### Directory structure

```
src/
├── main.tsx                      # React entry point with Redux Provider
├── App.tsx                       # Router setup with ToastContainer, runs checkAuth on mount
├── store.ts                      # Redux store: user reducer + 3 RTK Query API middlewares
├── index.css                     # Tailwind CSS with custom Jinx brand colors
│
├── components/
│   ├── ui/                       # Shadcn-style UI primitives (button, card, dialog, drawer, form, input, label, select, tabs)
│   ├── shared/
│   │   ├── AddForm.tsx           # Generic form wrapper with success/error toast handling
│   │   └── ResourceList.tsx      # Generic list with loading/error states
│   ├── inventorycategories/
│   │   ├── AddInventoryCategoryForm.tsx
│   │   └── InventoryCategoryList.tsx
│   ├── inventoryitems/
│   │   ├── AddInventoryItemForm.tsx
│   │   └── InventoryItemList.tsx
│   ├── restock/
│   │   ├── RestockForm.tsx       # Quantity +/- buttons, date fields, optional note
│   │   ├── RestockModal.tsx      # Dialog (desktop) / Drawer (mobile) via useIsMobile
│   │   ├── ItemCard.tsx
│   │   ├── ItemGrid.tsx
│   │   └── CategorySidebar.tsx   # Horizontal chips (mobile) / vertical list (desktop)
│   ├── routes/
│   │   └── ProtectedRoute.tsx    # Auth guard: null → spinner, false → /login
│   ├── navbar/
│   │   └── Navbar.tsx            # Links to /restock and /settings
│   ├── Layout.tsx                # Navbar + main content wrapper
│   ├── PageLoading.tsx           # Full-page spinner
│   └── Icons.tsx                 # LoadingIcon component
│
├── containers/                   # Page-level components (route targets)
│   ├── LoginPage.tsx
│   ├── HomePage.tsx
│   ├── SettingsPage.tsx          # Tabs: Categories and Items management
│   └── RestockPage.tsx           # Category filter + item grid + restock modal
│
├── services/                     # RTK Query API definitions
│   ├── baseQuery.ts              # baseQueryWithReauth: injects Bearer token, handles 401 refresh
│   ├── inventoryCategories.ts    # GET/POST /api/inventory/categories/
│   ├── inventoryItems.ts         # GET/POST /api/inventory/items/
│   └── inventoryLogs.ts          # GET/POST /api/inventory/logs/
│
├── features/
│   └── user.ts                   # Auth slice: tokens, user profile, loading, isAuthenticated
│
├── types/
│   ├── user.ts                   # MeProps, LoginProps, AuthState
│   ├── inventoryCategories.ts    # InventoryCategory
│   ├── inventoryItems.ts         # InventoryItem, CreateInventoryItem
│   └── inventoryLogs.ts          # InventoryLog, CreateInventoryLog
│
├── lib/
│   ├── axios.ts                  # Axios instance with Bearer token injection + 401 refresh queue
│   └── utils.ts                  # cn() helper (clsx + twMerge)
│
└── hooks/
    └── use-mobile.ts             # useIsMobile hook (768px breakpoint)
```

### State & data fetching

There are two distinct patterns in use:

**RTK Query (`src/services/`)** — used for all resource APIs. Each service file calls `createApi` with `baseQueryWithReauth` from `src/services/baseQuery.ts`, defines endpoints, and exports auto-generated hooks. The base query reads the Bearer token from Redux state via `prepareHeaders` and handles 401s by attempting a token refresh before retrying. New resource APIs belong here.

**Redux slices (`src/features/`)** — used only for global state that components need to `useSelector` from. Currently only `user.ts` lives here, managing auth state (tokens, user profile, loading). Prefer RTK Query for anything that doesn't need to be globally selected.

### Auth flow

- On app load, `checkAuth` thunk runs — reads `accessToken` from `sessionStorage` and `refreshToken` from `localStorage`, dispatches `setTokens`, then calls `GET /api/users/me`
- The axios instance in `src/lib/axios.ts` handles token refresh for non-RTK-Query calls (currently only used internally during auth). RTK Query calls go through `src/services/baseQuery.ts` which has its own refresh logic. Both use a request queue to prevent duplicate refresh calls on concurrent 401s.
- `ProtectedRoute` checks `isAuthenticated` from Redux; null = still loading (shows spinner), false = redirect to `/login`

### API endpoints

All prefixed with `/api`:

| Method | Path | Purpose |
|--------|------|---------|
| POST | `/token/` | Login — `{ email, password }` → `{ access, refresh }` |
| POST | `/token/refresh/` | Refresh — `{ refresh }` → `{ access }` |
| GET | `/users/me` | Current user profile |
| GET/POST | `/inventory/categories/` | List/create categories |
| GET/POST | `/inventory/items/` | List/create items |
| GET/POST | `/inventory/logs/` | List/create restock logs |

### Adding a new resource API

1. Add TypeScript interfaces to `src/types/<resource>.ts`
2. Create `src/services/<resource>.ts` with `createApi`, add `tagTypes` if list+create endpoints need cache invalidation
3. Register the API's reducer and middleware in `src/store.ts`

### UI components

Shadcn-style primitives live in `src/components/ui/`. Generic shared components (`AddForm`, `ResourceList`) are in `src/components/shared/` — use these as wrappers when building new feature forms and lists.

Forms use react-hook-form + zod (`zodResolver`) throughout. Select fields use `Controller` from react-hook-form with Radix UI Select. Toast notifications use `react-toastify`; `<ToastContainer>` is mounted once in `App.tsx`, call `toast.success()` / `toast.error()` anywhere.

Responsive breakpoint: `useIsMobile` (768px) drives layout switches — e.g. `RestockModal` uses Dialog on desktop, Drawer on mobile.

Custom brand colors defined in `index.css`: `jinxBlue` (#2A70E9), `jinxYellow`, `jinxRed`, `jinxGreen`, `jinxLime`.

Path alias `@/` maps to `src/`.

---
> Source: [tylercheney05/jinx-inventory](https://github.com/tylercheney05/jinx-inventory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
