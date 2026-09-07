## taksopark-client

> Vite + React 19 + Redux Toolkit + TanStack Query + shadcn/ui + Tailwind. **Based on FSD**, the `owner/` panel is split internally into `features/`.

# Frontend - Taksopark (client/)

Vite + React 19 + Redux Toolkit + TanStack Query + shadcn/ui + Tailwind. **Based on FSD**, the `owner/` panel is split internally into `features/`.

## Folder structure

```
client/src/
├─ main.jsx
├─ app/                     # app level (routes, store, query-client)
├─ shared/                  # Everything used GLOBALLY
│  ├─ api/                  # http.js (axios + interceptors), endpoints.js
│  ├─ components/
│  │  ├─ shadcn/            # Used ONLY inside shared
│  │  ├─ ui/                # ModalWrapper, Pagination, ... (wrappers over shadcn)
│  │  ├─ guards/            # AuthGuard, GuestGuard, RoleGuard, PermissionGuard
│  │  ├─ layout/            # AppHeader, AppSidebar, ...
│  │  └─ bg/
│  ├─ hooks/                # useModal, useMediaQuery, useAuth, usePermissions, ...
│  ├─ helpers/              # role.helpers
│  ├─ utils/                # cn, date, formatPhone, sounds
│  ├─ data/                 # static data
│  ├─ layouts/              # DashboardLayout, AuthLayout
│  ├─ lib/query/            # TanStack helpers (keys, ...)
│  ├─ store/                # global redux slices (modal)
│  └─ constants/            # roles, permissions, modals
├─ features/                # role-independent global features
│  └─ auth/
└─ owner/                   # OWNER panel (only role)
   ├─ features/<feature>/   # api, hooks, components, pages, store, utils, index.js
   ├─ pages/                # role-level pages (e.g. DashboardPage)
   ├─ routes/index.jsx
   ├─ navigation/sidebar.config.js
   └─ index.js
```

## Feature rules

Each feature has its own **"public API"** (`<feature>/index.js`). External code imports only from this file. Internal working files stay inside.

```js
// owner/features/users/index.js
export { default as UsersListPage } from "./pages/UsersListPage";
export { useUsersQuery } from "./hooks/useUsersQuery";
```

```
<feature>/
├─ api/<feature>.api.js     # pure axios request functions
├─ hooks/use*Query.js       # TanStack Query
├─ hooks/use*Mutation.js    # TanStack Mutation
├─ components/              # list, table, card, modals
│  └─ modals/               # modals using ModalWrapper
├─ pages/                   # page files bound to routes
├─ layouts/                 # (if needed) shared shell with <Outlet/> for nested routes
├─ store/                   # (if needed) redux slice
├─ utils/                   # feature-specific helpers
└─ index.js                 # public API
```

## Layout rules (strict)

A **layout** is a route component that renders a shared shell (header, tabs, breadcrumbs, ...) and an `<Outlet />` for nested route content. **Never put layouts in `pages/`** - they live in `layouts/`.

Trigger to create a layout:
- A route has nested children rendered via `<Outlet />`.
- Multiple sibling pages share tabs, a title bar, or any chrome.

Rules:
- File name: `<Feature>Layout.jsx` (e.g. `PaymentsLayout.jsx`, `PenaltiesLayout.jsx`) - never `*HubPage.jsx`, never `*ContainerPage.jsx`.
- Folder: `<feature>/layouts/`.
- Export: from `<feature>/index.js` as `{ default as <Feature>Layout }`.
- Use as the parent `element` in `routes/index.jsx`:
  ```jsx
  <Route path="payments" element={<PaymentsLayout />}>
    <Route index element={<PaymentsListPage />} />
    <Route path="reports/finance" element={<FinanceReportPage />} />
  </Route>
  ```
- Pages rendered inside a layout must NOT repeat the shared chrome (no duplicate `<h1>`, no duplicate tabs).

## Roles and protection

- `shared/constants/roles.js` - `ROLES.OWNER | TEACHER | STUDENT`.
- `shared/constants/permissions.js` - all permission keys (e.g. `"users.read"`).
- `<RoleGuard role="owner">` - if `me.role` does not match, redirects to `/`.
- `<PermissionGuard required="users.read">` - owner always passes.
- `useAuth()` - returns `{ user, role, isOwner, permissions }`.
- `usePermissions()` - `has(key) -> boolean`.

## Modal management

1. Add a constant to `shared/constants/modals.js`:
   ```js
   export const MODAL = Object.freeze({
     USER_CREATE: "user:create",
     // ...
   });
   ```
2. Create the modal component: `feature/components/modals/UserCreateModal.jsx` - write only the main form inside it (not `ModalWrapper`); `ModalWrapper` wraps it at the page level.
3. Render it on the page:
   ```jsx
   <ModalWrapper name={MODAL.USER_CREATE} title="Foydalanuvchi qo'shish">
     <UserCreateModal />
   </ModalWrapper>
   ```
4. Open it:
   ```js
   const { openModal } = useModal();
   openModal(MODAL.USER_CREATE, { someData });
   ```

## API rules

- Pure axios calls live in `feature/api/<name>.api.js` and **return a Promise only**:
  ```js
  // owner/features/users/api/users.api.js
  import http from "@/shared/api/http";
  export const usersAPI = {
    list: (params) => http.get("/users", { params }),
    create: (body) => http.post("/users", body),
  };
  ```
- Use it via a hook:
  ```js
  // owner/features/users/hooks/useUsersQuery.js
  import { useQuery } from "@tanstack/react-query";
  import { qk } from "@/shared/lib/query/keys";
  import { usersAPI } from "../api/users.api";

  export const useUsersQuery = (params) =>
    useQuery({
      queryKey: qk.users.list(params),
      queryFn: () => usersAPI.list(params).then((r) => r.data.data),
    });
  ```
- `qk` - global query key registry (`shared/lib/query/keys.js`). **Do not invent keys out of thin air**, always go through this registry.

## State management (strict)

If a component holds **more than 1 state** - instead of multiplying `useState` calls, **use `useObjectState`**:

```js
// ❌
const [phone, setPhone] = useState("");
const [name, setName] = useState("");

// ✅
const { phone, name, setField } = useObjectState({ phone: "", name: "" });
```

`useObjectState` returns: `...state`, `state`, `setField(key, value)`, `setFields({ ... })`, `resetState()`.

Exceptions (only these three cases):
1. A single primitive state, nothing else (`useState(false)`).
2. Inside another hook implementation (`useObjectState` itself lives there).
3. When lazy init is needed (rare).

Details - `.claude/skills/useobjectstate-bilan-state-boshqarish/SKILL.md`.

## Language rules

- UI text - Uzbek (`"Saqlash"`, `"Bekor qilish"`, `"Foydalanuvchilar ro'yxati"`).
- Code values - English (`role: "owner"`, `MODAL.USER_CREATE`, route `/users`, query key `["users", "list"]`).

## Aliases (jsconfig.json)

- `@/` -> `client/src/`
- `@/components/*` -> `client/src/shared/components/*` (additional alias)

## Commands

```bash
npm run dev      # on port 5173
npm run build
npm run lint
```

---
> Source: [yaxyobekuz/taksopark-client](https://github.com/yaxyobekuz/taksopark-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
