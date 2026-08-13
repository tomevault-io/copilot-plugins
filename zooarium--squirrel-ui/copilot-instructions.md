## squirrel-ui

> This document provides context and guidelines for AI agents working on the Squirrel-UI codebase.

# Squirrel-UI AI Agent Context

This document provides context and guidelines for AI agents working on the Squirrel-UI codebase.

## 🎯 Project Overview

Squirrel-UI is a React-based **Personal Expense Tracker**. Users manage transactions (income/expense), categorize them, and view financial summaries via charts. Currency is **INR** (`en-IN` locale).

## 🛠️ Tech Stack

| Layer | Library |
|-------|---------|
| UI framework | React 19 (functional components + hooks) |
| CSS | **Tabler CSS 1.2** (Bootstrap-based — use Bootstrap utility classes) |
| Icons | `@tabler/icons-react` |
| Dialogs | `@radix-ui/react-dialog`, `@radix-ui/react-alert-dialog` |
| Charts | Recharts |
| Routing | React Router DOM v7 |
| Server state | **TanStack Query v5** (`@tanstack/react-query`) |
| Forms | **React Hook Form v7** + **Zod v4** (`@hookform/resolvers`) |
| Mocking | **MSW v2** (tests + optional dev mode) |
| Build | Vite 7 |
| Testing | Vitest + Testing Library |

> **No Tailwind.** Use Tabler/Bootstrap utility classes (`d-flex`, `gap-2`, `text-secondary`, `min-vh-100`, etc.).

## 📁 Source Structure

```
src/
  api/           auth.js · categories.js · transactions.js
  components/    AppLayout.jsx · ErrorBoundary.jsx · Notification.jsx
  context/       NotificationContext.jsx · ThemeContext.jsx
  hooks/         useCategories.js · useTransactions.js
  infra/
    auth/        guards.jsx · storage.js
    http/        client.js · errors.js
    router/      index.jsx
    config.js
  mocks/         handlers.js · server.js · browser.js
  pages/         LoginPage.jsx · DashboardPage.jsx · CategoriesPage.jsx
  test/          setup.js · *.test.jsx
  ui/            barrel — import everything from here (see below)
```

## 🧩 UI Barrel (`src/ui/`)

**Always import from `@/ui` (absolute) or `../ui` (relative). Never from individual files.**

```js
import {
  Button, Card, CardHeader, CardTitle, CardBody,
  Badge, Spinner, FormField, Input, Select,
  Alert, Modal, ConfirmDialog,
  IconPlus, IconEdit, IconTrash, /* …other Tabler icons */
  CHART_COLORS, TRANSACTION_COLORS,
} from '@/ui';
```

Swapping the underlying library only requires changes inside `src/ui/` — zero app-code changes.

## 📐 Absolute Imports

`@/` maps to `src/`. Use in all new files.

```js
import { useCategories } from '@/hooks/useCategories';
import { storage }       from '@/infra/auth/storage';
import { Button }        from '@/ui';
```

## 🗺️ Routes

| Path | Page | Guard |
|------|------|-------|
| `/login` | `LoginPage` | `PublicRoute` |
| `/dashboard` | `DashboardPage` | `PrivateRoute` |
| `/categories` | `CategoriesPage` | `PrivateRoute` |
| `/` | redirect → `/dashboard` or `/login` | — |

## 🌐 HTTP Layer

`src/infra/http/client.js` — thin `fetch` wrapper.
- `apiRequest(path, opts)` — authenticated API calls (attaches Bearer token).
- `authRequest(path, opts)` — auth-service calls (login, refresh).
- **Refresh token**: on 401, silently calls `config.refreshPath`, retries original request. Concurrent 401s queue — only one refresh fires.
- Falls back to `/login` redirect if refresh fails or no refresh token stored.
- To swap `fetch` → `axios`: edit `client.js` only.

## 🔑 Auth Storage

`src/infra/auth/storage.js` — single point for localStorage keys.
- `storage.getToken()` / `setToken()` — access token.
- `storage.getRefreshToken()` / `setRefreshToken()` — refresh token.
- `storage.getUser()` / `setUser()` — parsed user object (`{ id, email, role, permissions[] }`).
- `storage.clear()` — clears all three.
- Swap localStorage → cookie: edit this file only.

## 🛡️ Auth Guards

```jsx
import { PrivateRoute, PublicRoute, RequireRole, RequirePermission } from '@/infra/auth/guards';

// Route-level
<PrivateRoute><Dashboard /></PrivateRoute>
<PublicRoute><LoginPage /></PublicRoute>

// Component-level role check
<RequireRole role="admin" fallback={<p>No access</p>}>
  <AdminPanel />
</RequireRole>

// Component-level permission check
<RequirePermission permission="transactions:write">
  <CreateButton />
</RequirePermission>
```

`user.role` (string) and `user.permissions` (string[]) come from `storage.getUser()`.

## 🔁 TanStack Query — Data Fetching Pattern

**All server state goes through TanStack Query. No manual `useState` + `useEffect` for fetching.**

```js
// Query (read)
const { data, isLoading, error } = useQuery({
  queryKey: ['resource', filters],
  queryFn: () => fetchResource(filters),
  select: (res) => res.data,        // transform here, not in component
});

// Mutation (write)
const mutation = useMutation({
  mutationFn: (payload) => createResource(payload),
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['resource'] });
    showNotification('Created!', 'success');
  },
  onError: (err) => showNotification(err.message, 'error'),
});
```

Query keys: `[RESOURCE_KEY]` (all) or `[RESOURCE_KEY, filters]` (filtered).
Export the key constant from the hook file: `export const CATEGORIES_KEY = 'categories'`.

Global config in `App.jsx`: `staleTime: 30_000` (30s fresh window), `retry: 1`.

## 📋 Forms — React Hook Form + Zod

**All forms use RHF + Zod. No per-field `useState`. No manual validation.**

```js
import { useForm } from 'react-hook-form';
import { z } from 'zod';
import { zodResolver } from '@hookform/resolvers/zod';

const schema = z.object({
  email: z.string().min(1, 'Required').email('Invalid email'),
  amount: z.coerce.number().positive('Must be positive'),
});

const { register, handleSubmit, formState: { errors, isSubmitting } } = useForm({
  resolver: zodResolver(schema),
  defaultValues: { email: '', amount: '' },
});
```

Show field errors via `<FormField error={errors.field?.message}>`.
RHF uses **uncontrolled inputs** — zero re-renders while typing.

## 🚧 Error Boundary

`src/components/ErrorBoundary.jsx` — wraps entire app in `App.jsx`.
Catches render crashes; shows fallback UI instead of white screen.

```jsx
// Default fallback (built-in UI):
<ErrorBoundary><App /></ErrorBoundary>

// Custom fallback:
<ErrorBoundary fallback={(error, reset) => <MyError error={error} onRetry={reset} />}>
  <SubTree />
</ErrorBoundary>
```

Wire Sentry by uncommenting the `Sentry.captureException` line in `componentDidCatch`.

## 🧪 Testing

**Setup**: MSW intercepts all API calls in tests — no real network, no backend needed.

```js
// Override a handler for one test:
import { http, HttpResponse } from 'msw';
import { server } from '@/mocks/server';

server.use(
  http.get('*/categories', () => HttpResponse.json({ data: { categories: [] } }))
);
```

**Hook tests** — wrap with QueryClient + NotificationProvider:
```jsx
function makeWrapper() {
  const qc = new QueryClient({ defaultOptions: { queries: { retry: false } } });
  return ({ children }) => (
    <QueryClientProvider client={qc}>
      <NotificationProvider>{children}</NotificationProvider>
    </QueryClientProvider>
  );
}
const { result } = renderHook(() => useCategories(), { wrapper: makeWrapper() });
await waitFor(() => expect(result.current.isLoading).toBe(false));
```

**Mock dev mode** (no backend): set `VITE_MOCK_API=true` in `.env.development`,
then uncomment the worker block in `src/main.jsx` (see `src/mocks/browser.js` for instructions).

## 🎨 Theme

Light/dark toggle via `ThemeContext`. `data-bs-theme` attribute on `<html>` drives Tabler's theme. Standard Tabler defaults — no custom Matrix/cyberpunk styling.

## 📝 Coding Standards

- **Naming**: PascalCase for components, camelCase for variables/functions.
- **Imports**: use `@/` absolute imports in all new files.
- **Formatting**: `npm run format` (Prettier) and `npm run lint` (ESLint) before committing.
- **Modals**: toggle `document.body.style.overflow` (`hidden`/`auto`) when opening/closing if background scroll bleeds through.
- **Forms**: use RHF + Zod; show field-level errors via `FormField error={…}` prop.
- **Notifications**: use `useNotification()` → `showNotification(message, 'success'|'error')`.
- **Data fetching**: use TanStack Query hooks; never raw `useState` + `useEffect` for server state.

## 🚀 Common Tasks

### Add a new page
1. Create `src/pages/NewPage.jsx`, wrap with `<AppLayout>`.
2. Lazy-import in `src/infra/router/index.jsx`.
3. Add `<Route>` (wrap in `<PrivateRoute>` if auth-required).
4. Add nav link in `src/config/nav.jsx` `NAV_ITEMS` array (see below for icons).

### Add a sidebar menu icon
Icons come from `@aviary-ui/ui`, re-exported from `@tabler/icons-react` in
`aviary-ui/packages/ui/src/ui/icons.js` (single swap point for the icon library).
1. If the icon isn't already re-exported, add it to
   `aviary-ui/packages/ui/src/ui/icons.js`.
2. In `src/config/nav.jsx`, import the icon from `@aviary-ui/ui` and set it on
   the entry: `{ path, label, Icon }` in `NAV_ITEMS`.
3. `AppLayout` (`@aviary-ui/ui`) renders the sidebar from `navItems`
   automatically — no other file needs editing.

### Add a new API module
1. Create `src/api/thing.js` using `apiRequest` from `@/infra/http/client`.
2. Create `src/hooks/useThing.js` using `useQuery` / `useMutation`.
3. Export a `THING_KEY` constant from the hook file.
4. Add mock handlers to `src/mocks/handlers.js`.

### Add a new UI primitive
Add file inside `src/ui/`, export from `src/ui/index.js`. Never import UI primitives directly from their file path outside `src/ui/`.

### Add role-gated UI
```jsx
<RequireRole role="admin"><AdminButton /></RequireRole>
<RequirePermission permission="reports:export"><ExportButton /></RequirePermission>
```

### Add a new form
Use RHF + Zod. Define schema → `useForm({ resolver: zodResolver(schema) })` → `register` inputs → `handleSubmit`. See `LoginPage.jsx` as reference.

---
> Source: [zooarium/squirrel-ui](https://github.com/zooarium/squirrel-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
