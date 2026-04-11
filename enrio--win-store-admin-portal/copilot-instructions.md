## win-store-admin-portal

> > **See also:** [Copilot Instructions](.github/copilot-instructions.md) for the full project context shared with GitHub Copilot.

# WinStore Admin Portal — Gemini Project Context

> **See also:** [Copilot Instructions](.github/copilot-instructions.md) for the full project context shared with GitHub Copilot.

---

## Overview

**WinStore Admin Portal** is a SaaS omnichannel order management dashboard. Built with:

| Layer       | Technology                                        |
| ----------- | ------------------------------------------------- |
| Framework   | **Refine v5** (`@refinedev/core`, `@refinedev/mui`, `@refinedev/react-hook-form`, `@refinedev/react-table`, `@refinedev/kbar`) |
| UI Library  | **MUI v7** (`@mui/material`, `@mui/icons-material`, `@mui/lab`, `@mui/x-data-grid`) |
| CSS         | **Tailwind CSS** (planned integration alongside MUI) |
| React       | **React 19** with **TypeScript 5** (strict mode)  |
| Bundler     | **Vite 7** with `@vitejs/plugin-react`            |
| Routing     | **React Router v7** (`react-router` + `@refinedev/react-router`) |
| i18n        | **i18next** with `en` / `de` locales (loaded from `/public/locales/`) |
| Charts      | **Recharts**                                       |
| Maps        | **Google Maps** via `@googlemaps/react-wrapper`    |
| Forms       | **react-hook-form** via `@refinedev/react-hook-form` |
| Data        | `@refinedev/simple-rest` → demo API (`api.finefoods.refine.dev`). Migrating to **Supabase**. |
| Deployment  | **Cloudflare Pages** (`wrangler.toml`) + **Netlify** fallback (`netlify.toml`) |
| Package Mgr | **pnpm** (with `legacy-peer-deps=true`)           |
| Node        | `>=20`                                              |

---

## Build & Run

```bash
pnpm install         # install dependencies
pnpm run dev         # Vite dev server via Refine CLI
pnpm run build       # tsc && refine build (production)
```

---

## Project Structure

```
src/
├── App.tsx              # Refine <Refine> provider tree, resource definitions, routing
├── index.tsx            # React root, dayjs setup, i18n import
├── authProvider.ts      # AuthProvider (demo; migrating to Supabase Auth)
├── theme.ts             # MUI Light/Dark themes (RefineThemes.Orange base)
├── i18n.ts              # i18next config: en/de, HTTP backend from /locales/
├── vite-env.d.ts        # Vite client type declarations
│
├── contexts/            # ColorModeContextProvider (light/dark toggle via React Context)
├── hooks/               # useAutoLoginForDemo, useOrderCustomKbarActions
├── interfaces/          # Shared TypeScript interfaces: IOrder, IProduct, IUser, IStore, ICourier, ICategory, etc.
├── utils/               # geocoding, use-image-upload, unique-list-with-count
│
├── components/          # Reusable UI (barrel-exported via index.ts)
│   ├── card/            # Generic card wrapper
│   ├── order/           # Order status, details, map, products, tableColumnProducts
│   ├── product/         # Product drawer-form, image-upload, list-card, list-table, status
│   ├── store/           # Store form, info-card, map, status, table, courier-table
│   ├── courier/         # Courier image-upload, rating, status, table-reviews
│   ├── customer/        # Customer status
│   ├── category/        # Category status
│   ├── dashboard/       # Chart widgets (dailyOrders, dailyRevenue, deliveryMap, etc.)
│   ├── drawer/          # Shared drawer + header
│   ├── header/          # App header with locale/theme toggles
│   ├── icons/           # Custom SVG icon components
│   ├── map/             # Google Maps (advanced-marker, marker, map wrapper)
│   ├── customTooltip/   # Chart custom tooltip
│   ├── refine-list-view/# Refine list view wrapper
│   └── title/           # App title/logo component
│
├── pages/               # Route-level page components
│   ├── auth/            # Login / Register / Forgot / Update password (AuthPage wrapper)
│   ├── dashboard/       # Dashboard with charts, stats, maps
│   ├── orders/          # Order list + show (read-only)
│   ├── customers/       # Customer list + show
│   ├── products/        # Product list + create + edit (drawer-based CRUD)
│   ├── stores/          # Store list + create + edit
│   ├── couriers/        # Courier list + create + edit
│   └── categories/      # Category list
```

---

## Refine Resources (defined in `App.tsx`)

| Resource     | Routes                                        | CRUD         |
| ------------ | --------------------------------------------- | ------------ |
| `dashboard`  | `/`                                            | list         |
| `orders`     | `/orders`, `/orders/:id`                       | list, show   |
| `users`      | `/customers`, `/customers/:id`                 | list, show   |
| `products`   | `/products`, `/products/new`, `/products/:id/edit` | list, create, edit |
| `categories` | `/categories`                                  | list         |
| `stores`     | `/stores`, `/stores/new`, `/stores/:id/edit`   | list, create, edit |
| `couriers`   | `/couriers`, `/couriers/new`, `/couriers/:id/edit` | list, create, edit |

---

## Key Conventions

### Refine.dev Patterns
- **Data hooks**: `useList`, `useOne`, `useCreate`, `useUpdate`, `useDelete`, `useCustom` from `@refinedev/core`. **Never call APIs directly.**
- **Auth hooks**: `useIsAuthenticated`, `useGetIdentity`, `useLogout` from `@refinedev/core`.
- **Tables**: `useDataGrid` from `@refinedev/mui` with MUI `<DataGrid>`, or `useTable` from `@refinedev/react-table`.
- **Forms**: `useForm` from `@refinedev/react-hook-form`. Fields via `register()` or `Controller`.
- **i18n**: `useTranslate` from `@refinedev/core`. Keys in `/public/locales/{en,de}.json`.
- **Notifications**: `RefineSnackbarProvider` + `useNotificationProvider`.
- **KBar**: Command palette via `@refinedev/kbar`. Custom actions in `useOrderCustomKbarActions`.

### MUI Conventions
- **MUI v7** — import components individually: `import Button from "@mui/material/Button"`.
- **Theming**: `theme.ts` extends `RefineThemes.Orange` / `RefineThemes.OrangeDark`. Toggled via `ColorModeContext`.
- **Styling**: MUI `sx` prop for component styles. `styled()` or `GlobalStyles` for broader overrides. **No separate CSS files.**
- **Grid**: `Grid` from `@mui/material/Grid` (v2-style). Vite alias: `@mui/material/Grid2` → `@mui/material/Grid`.
- **DataGrid**: `@mui/x-data-grid` for all tabular data. Columns via `GridColDef[]`.

### Tailwind CSS (Planned Integration)
- Use Tailwind for **layout utilities** (`flex`, `grid`, `gap`, `p-*`, `m-*`) and spacing.
- Keep **MUI components** for interactive UI (buttons, dialogs, data grids, form controls).
- Configure `preflight: false` to avoid conflicting with MUI resets.
- Prefer MUI `sx` for component-specific styles; Tailwind classes for page layout.

### TypeScript & Code Style
- **Strict mode** enabled. No `any` unless absolutely necessary.
- Interfaces in `src/interfaces/index.d.ts`, prefixed with `I` (e.g., `IOrder`, `IProduct`).
- **Barrel exports**: Every folder has `index.ts` re-exporting its public API.
- **Named exports** preferred (no default exports except `App.tsx` and `i18n.ts`).

---

## Key Patterns

| Pattern                  | Description                                                    |
| ------------------------ | -------------------------------------------------------------- |
| **Drawer-based CRUD**    | Products & couriers use `<Drawer>` for create/edit, not separate pages. List renders `<Outlet />` for nested drawer routes. |
| **Status components**    | Each entity has a dedicated `status/` component rendering status chips with colors. |
| **Dashboard**            | Stat cards + Recharts charts + Google Maps delivery map + order timeline. Uses `useCustom` for aggregated data. |
| **Google Maps**          | Custom `<Map>` wrapper with `@googlemaps/react-wrapper`. Advanced markers for stores/couriers. |
| **Image upload**         | Shared `useImageUpload` hook in `src/utils/use-image-upload/`. |
| **Color mode**           | Light/Dark toggle via React Context (`ColorModeContextProvider`), persisted to `localStorage`. |

---

## File Placement Rules

- **New pages** → `src/pages/<resource>/` + register resource in `App.tsx`.
- **Reusable components** → `src/components/<domain>/` with barrel export.
- **Translation keys** → add to both `en.json` and `de.json`.
- **Dates** → use `dayjs` (already extended with `relativeTime`).
- **Utilities** → use `lodash` sparingly (already a dependency).

---

## Planned Migrations

### Supabase Integration
- Replace `@refinedev/simple-rest` with `@refinedev/supabase` data provider.
- Replace demo `authProvider` with Supabase Auth.
- Create shared `src/supabaseClient.ts`.
- Use Supabase Storage for image uploads.
- Use Supabase Realtime for live order updates.
- Configure Row Level Security (RLS) on all tables.

---

## Related Configuration Files

| File              | Purpose                                      |
| ----------------- | -------------------------------------------- |
| `vite.config.ts`  | Vite config with React plugin + Grid2 alias  |
| `tsconfig.json`   | Strict TS, ESNext target, bundler resolution |
| `wrangler.toml`   | Cloudflare Pages SPA config                  |
| `netlify.toml`    | Netlify SPA fallback redirects               |
| `.npmrc`          | `legacy-peer-deps=true`                      |
| `.agent/`         | AI agent configs, skills, scripts, workflows |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/enrio)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/enrio)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
