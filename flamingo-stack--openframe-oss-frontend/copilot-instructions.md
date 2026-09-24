## openframe-oss-frontend

> **Next.js 16 + React 19 + TypeScript 5.9 + @flamingo-stack/openframe-frontend-core (0.0.653)**

# OpenFrame Frontend - Claude Development Guide

**Next.js 16 + React 19 + TypeScript 5.9 + @flamingo-stack/openframe-frontend-core (0.0.653)**

> Comprehensive instructions for Claude when working with the OpenFrame Frontend service.

## Core Principles

**MANDATORY REQUIREMENTS:**
1. ALL UI components MUST use `@flamingo-stack/openframe-frontend-core` — never create custom UI primitives
2. ALL styling MUST use ODS design tokens (no hardcoded colors)
3. Follow WCAG 2.1 AA accessibility standards
4. Use `react-hook-form` + `zod` for forms; use `useToast` for all API feedback
5. Use `react-relay` for GraphQL data fetching wherever possible — the codebase is gradually migrating to Relay. Use `@tanstack/react-query` for REST APIs and for legacy GraphQL code that has not been migrated yet. Do NOT introduce new raw-POST GraphQL calls.

## Setup & Commands

### Quick Setup
```bash
npm install
cp .env.local.example .env.local   # or create manually:
echo "NEXT_PUBLIC_TENANT_HOST_URL=http://localhost" >> .env.local
echo "NEXT_PUBLIC_APP_MODE=oss-tenant" >> .env.local
npm run dev
```
Access: http://localhost:3000

### All Commands
| Command | Purpose |
|---------|----------|
| `npm run dev` | Dev server (port 3000, `PORT` env to override) |
| `npm run build` | Production build (`generate-enums` + `relay-compiler` + `next build`; standalone output in `dist/`) |
| `npm run build:export` | Static-export build (`OPENFRAME_BUILD_TARGET=export`) — SPA bundle for Capacitor/Tauri native shells |
| `npm run build:local` | Production build with webpack |
| `npm run start` | Start production server |
| `npm run start:standalone` | Serve the standalone build (`dist/standalone/server.js`) |
| `npm run type-check` | TypeScript validation (`tsc --noEmit`) |
| `npm run relay` | Relay compiler — regenerates `src/__generated__/` artifacts |
| `npm run relay:watch` | Relay compiler in watch mode |
| `npm run fetch-schema` | Pull `schema.graphql` from a backend via introspection (`-- --endpoint <url> --token <JWT>`) |
| `npm run generate-enums` | Regenerate `src/generated/schema-enums.ts` (enum const+type) from `schema.graphql` |
| `npm run lint` | ESLint — the fast pass (`eslint.config.mjs`), cached |
| `npm run lint:ci` | What CI blocks on: the fast pass minus the `relay/unused-fields` backlog (`eslint.ci.mjs`) |
| `npm run lint:fix` | ESLint autofix (import order, unused imports, type imports) |
| `npm run lint:types` | ESLint type-aware pass (`eslint.types.mjs`; slow, needs an 8 GB heap) |
| `npm run lint:cycles` | ESLint `import/no-cycle` pass (`eslint.cycles.mjs`; walks the whole import graph) |
| `npm run format` | Prettier check |
| `npm run format:fix` | Prettier write |
| `npm run core:link` / `core:unlink` | yalc-link/unlink the core library for local lib development |

### Pre-commit Hooks
Husky (`.husky/pre-commit`) is **staged-file-scoped**: it runs ESLint (via `eslint.ci.mjs`, the same set CI blocks on) and `prettier --check` on the staged frontend files, plus `tsc --noEmit` with errors filtered to staged files only. A clean commit does not require the whole repo to pass, but today it very nearly does: `npm run lint:ci` — the fast pass minus the `relay/unused-fields` backlog — is green, and CI blocks on it. Keep `npm run type-check` and `npm run format` green too.

### Environment Variables

**Required:**
```bash
NEXT_PUBLIC_TENANT_HOST_URL=http://localhost   # Backend API host
NEXT_PUBLIC_APP_MODE=oss-tenant                # App mode (see below)
```

**SaaS deployment:**
```bash
NEXT_PUBLIC_SHARED_HOST_URL=https://auth.openframe.ai   # Shared auth host
NEXT_PUBLIC_GTM_CONTAINER_ID=GTM-XXXXXXX                # Google Tag Manager
```

**Dev auth:**
```bash
NEXT_PUBLIC_ENABLE_DEV_TICKET_OBSERVER=true   # Dev ticket auth mode (Bearer tokens instead of cookies)
```

Feature flags are **not** env vars — they are server-loaded via GraphQL (see Feature Flags below). Native-shell env split is documented in `.env.export.example`.

### Payment UI Visibility (native app builds)

`src/lib/billing-visibility.ts` is the single switch for every payment surface, in three tiers keyed
on the shell: **web** shows and changes everything; **desktop** (`isBillingReadOnly()` =
`isDesktopShell()`) shows everything and changes nothing; **mobile** (`isBillingHidden()` =
`isMobileShell()`) shows none of it. Billing runs through Stripe on the web; App Store Guideline 3.1.1
forbids showing plans, prices, invoices, or any CTA leading to a non-IAP purchase, and Google Play
treats an in-app Stripe Checkout for a digital subscription as bypassing Play Billing. Desktop ships
outside any store, so neither rule reaches it — but payment still belongs in the browser (bank
verification steps, autofill, saved cards), hence read-only plus one exit.

No env var backs this: the shell injects `window.Capacitor` / the Tauri globals itself, so a native
build can't forget to declare what it is, and the web bundle can't be misconfigured into hiding its
billing.

### Which shell am I in? (`src/lib/platform.ts`)

The same export runs in three places, so ask the axis that owns the feature — never "is this native?":

- `isAppShell()` — either shell. Shell-custodied tokens, no Next server behind the origin (so
  `/content` goes absolute), in-app auth pages, no external navigation, and the `authMobile=true`
  login completing on the custom scheme (`APP_SCHEME`).
- `isMobileShell()` — the phone. FCM push, biometrics, status bar/splash/safe-area insets, Android
  back, and the billing ban above.
- `isDesktopShell()` — Tauri. Shell-side token rotation + OS-notification click event transports,
  read-only billing.

`platform.ts` is the only module that reads the injected globals, and it probes **Tauri first**: the
desktop shell injects a Capacitor-shaped bridge, so a Capacitor check alone reports every desktop
install as mobile. `native-shell.ts` owns typed bridge access and gates each plugin on its own axis.
CSS scopes on `html[data-shell="mobile"|"desktop"]`.

- `isBillingHidden()` — mobile: no payment surface at all
- `isBillingReadOnly()` — desktop: billing is displayed, never changed
- `isPaymentUiEnabled()` — `billings` server flag **and** a build that may change billing (web only)
- `openBillingInBrowser()` — the read-only build's one exit: the web app's `/settings/billing-usage`
  in the system browser. The web page, not the Stripe portal — the portal exposes neither plan
  changes nor cancellation. Never `openDeferredTab()` on desktop: the shell denies its blank
  placeholder tab and the fallback navigates the app window itself.

When read-only: `billing-usage-content.tsx` renders every figure (plan, rates, next payment, usage,
invoices) with a single **Manage Billing** header action in place of the plan/AI-limit/cancel/pay
actions; `/checkout/*` 404s; the AI spend bar stays off (its only action is the limit control); and
the lock screen is `read-only-lock-content.tsx` — `WorkspaceInactiveScreen` with Manage Billing for
owners/admins, the "contact the owner" copy (`no-access-copy.ts`) for everyone else.

When hidden: Settings shows a **Usage** card (`billing-usage/components/usage-view.tsx` — device/AI
counters and workspace limits over its own price-free query), `/checkout/*` 404s, and the subscription lock screen renders `WorkspaceInactiveScreen`
instead of the plan picker. `isRouteAllowedInCurrentMode()` deliberately does NOT gate these: it
answers from app mode alone, and consulting a server-loaded flag there once threw "Access restricted"
over billing routes for as long as the flags query was in flight.

There is no `/settings/billing-usage/subscription` route. The plan is changed in the **Upgrade Plan
modal** on the billing page (`billing-usage/components/upgrade-plan-modal.tsx`), and the same picker
(`subscription/components/device-plan-picker.tsx`) is what the subscription lock screen shows. The
`subscription/` folder under `billing-usage/` holds those components/hooks and has no page of its own.

**Any new payment-adjacent UI must pick its tier: figures (price, plan, invoice) are gated on
`isBillingHidden()`; anything that changes billing (upgrade/pay/cancel CTA, limit control) also on
`isBillingReadOnly()`.**

## Architecture & Structure

### Technology Stack
| Category | Technology | Version |
|----------|-----------|---------|
| Framework | Next.js | 16 (16.3.5) |
| UI Library | React | 19 (19.2.4) |
| Auto-memoization | React Compiler (`reactCompiler: true` + babel-plugin-react-compiler) | 1.0 |
| Type System | TypeScript | 5.9 (5.9.3) |
| Component Library | @flamingo-stack/openframe-frontend-core | 0.0.653 (npm registry) |
| GraphQL Data Fetching | react-relay + relay-runtime + relay-compiler | 20.1 |
| REST / Legacy Data Fetching | @tanstack/react-query | 5.90 |
| Forms | react-hook-form + @hookform/resolvers | 7.71 + 5.2 |
| Validation | zod | 4.3 |
| State Management | Zustand + immer | 5.0.8 + 10.1 |
| Styling | Tailwind CSS + tailwindcss-animate | 3.4 |
| Terminal | @xterm/xterm + @xterm/addon-fit | 6.0 + 0.11 |
| Code Editor | @monaco-editor/react | 4.7 |
| GraphQL | graphql | 16.12 |
| Date Utils | date-fns | 4.1 |
| Icons | `@flamingo-stack/openframe-frontend-core/components/icons-v2` (no `lucide-react` — not a dependency; `import/no-extraneous-dependencies` rejects it) | — |
| Runtime Env | next-runtime-env | 3.3 |
| Linting | ESLint + `@flamingo-stack/openframe-frontend-core/eslint-config` | 9.39 |
| Formatting | Prettier + the shared preset (Tailwind class sorting) | 3.9 |
| Git Hooks | Husky | 9.1 |

### Dependency Versions Are Pinned

`package.json` carries **exact versions — no `^` or `~`** — and `.npmrc` sets `save-exact=true`, so
`npm install <pkg>` keeps it that way. `package-lock.json` already froze what CI and the Docker build
install (`npm ci`); the exact pins make `package.json` say the same thing, so an `npm install` or
`npm update` on a laptop cannot move a version no PR asked for. The Dockerfile base image is pinned
to a full version tag for the same reason.

- **To bump:** change the version, run `npm install`, and check the `Scan Code` job — Trivy over
  `package-lock.json` and the Dockerfile's base images, failing on any HIGH/CRITICAL that has a fix.
- **A transitive finding** is fixed with `npm update <pkg>` when the parent's range allows the
  patched version, and with `overrides` only when it does not.
- **`overrides` → `next-runtime-env`:** the package declares `next@^14` and `react@^18` as hard
  dependencies (still true in 3.3.0), which installed a second Next 14 + React 18 tree — 16 packages,
  and the source of most scanner findings. The override points it at this app's own `next` and `react`.
- **An override does not reach a subtree the lockfile already holds.** Delete that package's entries
  from `package-lock.json` (not the whole file — that re-resolves everything), then `npm install`.

### Core Library is External

`@flamingo-stack/openframe-frontend-core` is **NOT part of OpenFrame** — it is a **separate, external library** shared across the Flamingo Stack.

**Key Facts:**
- **Source repo**: `openframe-oss-lib/openframe-frontend-core/`
- **Ownership**: Shared across Flamingo Stack projects (OpenFrame, OpenMSP, Flamingo, TMCG, hubs, openframe-chat)
- **Normal state**: installed from the **npm registry** (`"@flamingo-stack/openframe-frontend-core": "0.0.653"`); the lib repo's own `package.json` version lags the registry (CI bumps at publish)
- **Local lib development**: link via **yalc** — `npm run core:link` here, and in the lib repo `npm run build && yalc push` after every change (consumers see `dist/`, not `src/`)
- **Updates**: Changes affect ALL Flamingo Stack projects

**yalc Workflow:**
```bash
# In openframe-frontend-core repo (after each change):
npm run build && yalc push

# In openframe-frontend (once, to link):
npm run core:link   # = yalc add @flamingo-stack/openframe-frontend-core
npm install
npm run core:unlink # when done — restore the registry version
```

**NEVER:**
- Treat core library as part of the OpenFrame codebase
- Make breaking changes without coordinating across projects
- Import from `@flamingo/ui-kit` (old name — does not exist)

### App Modes

Controlled by `NEXT_PUBLIC_APP_MODE` (see `src/lib/app-mode.ts`):

| Mode | Auth Pages | App Pages | Mingo | Description |
|------|-----------|-----------|-------|-------------|
| `oss-tenant` (default) | Yes | Yes | No | Self-hosted, full-featured |
| `saas-tenant` | No | Yes | Yes | SaaS customer tenant |
| `saas-shared` | Yes | No | No | SaaS shared auth service |

Helper functions: `isOssTenantMode()`, `isSaasTenantMode()`, `isSaasSharedMode()`, `isAuthEnabled()`, `isAppEnabled()`

### Feature Flags

Flags are **server-loaded**, not env-based. Names defined in `src/lib/feature-flags.ts` (e.g. `billings`, `help-center`, `notifications`, `time-tracker`, `script-schedules`, `mingo-sidebar`, `cancel-subscription`); fetched via the `feFeatureFlags(names:)` GraphQL query (`src/app/hooks/use-feature-flags-query.ts`) into `src/stores/feature-flags-store.ts`. `src/components/feature-flags-loader.tsx` runs that query but does NOT gate render: read a flag through `useFeatureFlagGate` (tri-state `loading | on | off`) wherever a wrong value would be visible or would redirect, and render the loading branch — see `src/app/hooks/use-feature-flag.ts`.

### Route Registry (MANDATORY)

All internal navigation URLs are built through the typed registry `src/lib/routes.ts` — never
hand-write an internal path string in `router.push`/`<Link href>`/`useSafeBack`/`redirect`:

```ts
import { routes } from '@/lib/routes';

router.push(routes.monitoring.root({ tab: 'policies' }));   // /monitoring?tab=policies
router.push(routes.customers.details(id, { tab: 'tickets' }));
<Link href={routes.devices.details(deviceId)} />
```

**When adding a new page, tab, or component that links anywhere, update `routes.ts` first,
following its typing:**
- New page/route → add an entry to `routes` (static string, or a builder function when it takes
  an id / query params), then use `routes.*` at every call site.
- New `?tab=` view → add the tab id to `TAB_IDS` and reference the derived union / `TAB_IDS`
  from the page's `TabItem[]` definition instead of re-typing string literals.
- New query param on an existing route → extend that builder's typed options object.
- Builders take `string | number` ids on purpose — guard nullable ids at the call site instead
  of widening the type.

Full rules, rationale, and the list of intentional exceptions (`not-found.tsx` legacy-redirect
table, `pathname.startsWith()` active-state checks, external/API URLs):

@./src/lib/ROUTES.md

### Application Modules

Routes live under the `(app)` / `(auth)` route groups. **Detail pages use query params** (`/x/details?id=`), not dynamic segments (static-export constraint; read via `useRequiredIdParam`). URL strings themselves come from the route registry above.

- **Authentication** (`/auth`) — Multi-provider SSO, signup, login, password reset, invite
- **Dashboard** (`/dashboard`) — Overview stats + the tenant Initial Setup card; standalone `/onboarding` (user Get Started tour)
- **Devices** (`/devices`) — Fleet MDM, detail pages, MeshCentral remote shell/desktop/file manager
- **Logs** (`/logs-page`, `/log-details`) — Streaming, search, filtering
- **Scripts** (`/scripts`) — fully Relay; thin route wrappers over the implementation in `src/app/(app)/scripts/{script,schedule,shared}/`. Schedules (`/scripts/schedules/*`) are gated by flag `script-schedules`
- **Customers** (`/customers`) — Customer/organization CRM (route renamed from `/organizations`; sidebar item id is still `organizations`)
- **Monitoring** (`/monitoring`) — Fleet osquery queries + policies (not feature-flagged)
- **Tickets** (`/tickets`) — Ticket board + AI chat dialogs (saas-tenant only; talks to `/chat/graphql`)
- **Mingo** (`/mingo`) — Admin AI assistant chat (saas-tenant only; legacy page, superseded by the in-layout drawer when flag `mingo-sidebar` is on)
- **Knowledge Base** (`/knowledge-base`) — Articles/folders (fully Relay)
- **Help Center** (`/help-center/*`) — Content pages via core-lib `help-center-pages` (flag `help-center`)
- **Worktime** (`/worktime`) — Time entries (flag `time-tracker`)
- **Notifications** (`/notifications`) — Relay reference implementation (flag `notifications`)
- **Settings** (`/settings/*`) — ai-settings, api-keys, architecture (OSS-only), billing-usage (flag `billings`), employees, sso
- **Checkout** (`/checkout/success|cancel`) — Stripe checkout result pages

### Project Structure
```
src/
├── proxy.ts               # Next 16 middleware — server-side app-mode route blocking
├── app/                   # Next.js App Router (route groups)
│   ├── (auth)/auth/       # Authentication (login, signup, invite, password-reset, stores/auth-store.ts)
│   ├── (app)/             # All app pages, wrapped by AppLayout in (app)/layout.tsx
│   │   ├── dashboard/  onboarding/  devices/  logs-page/  log-details/
│   │   ├── scripts/       # route wrappers + {script,schedule,shared}/{components,hooks,types,utils}
│   │   ├── customers/  monitoring/  tickets/  mingo/  knowledge-base/
│   │   ├── help-center/  worktime/  notifications/  settings/  checkout/
│   ├── hooks/             # Shared hooks (use-feature-flags-query, use-required-id-param, …)
│   └── components/        # Shared components (notifications provider, subscription-lock, shared tables)
├── components/            # Root-level shared (route-guard, feature-flags-loader, assignments/)
├── stores/                # Zustand stores (feature-flags-store, devices-store [mostly unused])
├── graphql/               # Relay operations by domain (notifications/, scripts/, time-tracker/)
├── __generated__/         # Relay artifacts (owned by relay-compiler — never import enums from here)
├── generated/             # schema-enums.ts (from npm run generate-enums)
├── lib/                   # Utilities & config
│   ├── api-client.ts          # Centralized REST API client (singleton, 401 refresh queue)
│   ├── auth-api-client.ts     # Auth endpoints against NEXT_PUBLIC_SHARED_HOST_URL
│   ├── fleet-api-client.ts    # Fleet MDM via /tools/fleetmdm-server
│   ├── relay/                 # Relay environment + provider (singleton, 401 refresh)
│   ├── relay-id.ts            # toGlobalId / global-id normalization
│   ├── token-store.ts  token-refresh-manager.ts  force-logout.ts  # auth token plumbing
│   ├── app-mode.ts  runtime-config.ts  feature-flags.ts
│   ├── nats/                  # NatsAppProvider + WS URL config
│   ├── platform.ts            # web | mobile | desktop shell detection (SSOT)
│   ├── native-shell.ts  native-login.ts  # Capacitor/Tauri shell bridge
│   ├── register-embed-shims.ts  navigation-config.tsx  navigation-sidebar-state.ts
│   ├── subscription-lock-signal.ts  analytics.ts  openframe-core-ui.tsx
│   ├── query-client-provider.tsx  fonts.ts  handle-api-error.ts
│   ├── meshcentral/           # MeshCentral control/tunnel/desktop/file-manager protocol
│   └── platform-configs/      # Platform-specific config
```

## Core Library Integration

### Import Patterns

```typescript
// Styles (import in root layout only)
import '@flamingo-stack/openframe-frontend-core/styles';

// UI components — direct import
import {
  Button, Card, CardHeader, CardContent, CardFooter,
  Input, Label, Badge, Skeleton,
  Dialog, DialogContent, DialogHeader, DialogTitle, DialogFooter,
  Tabs, TabsList, TabsTrigger, TabsContent,
  ContentPageContainer, DetailPageContainer,
  DeviceCard, StatusTag, CardLoader, CompactPageLoader,
  DashboardInfoCard, OrganizationCard, BenefitCard,
} from '@flamingo-stack/openframe-frontend-core/components/ui';

// Feature components
import {
  AuthProvidersList,
} from '@flamingo-stack/openframe-frontend-core/components/features';

// Navigation
import { AppLayout } from '@flamingo-stack/openframe-frontend-core/components/navigation';

// Icons
import { MingoIcon } from '@flamingo-stack/openframe-frontend-core/components/icons';
import {
  DashboardIcon, DevicesIcon, LogsIcon, ScriptsIcon,
} from '@flamingo-stack/openframe-frontend-core/components/icons-v2';

// Hooks — CRITICAL: useToast is MANDATORY for all API operations
import {
  useToast,           // REQUIRED for all API feedback
  useApiParams,       // URL state management
  useDebounce,
  useLocalStorage,
  useTablePagination,
} from '@flamingo-stack/openframe-frontend-core/hooks';

// Utilities
import {
  cn,                             // Tailwind class merging
  getPlatformAccentColor,
  getProxiedImageUrl,
  normalizeToolTypeWithFallback,
  getSlackCommunityJoinUrl,
  DEFAULT_OS_PLATFORM,
} from '@flamingo-stack/openframe-frontend-core/utils';

// Types
import type { NavigationSidebarItem, NavigationSidebarConfig } from '@flamingo-stack/openframe-frontend-core/types/navigation';
import type { OSPlatformId } from '@flamingo-stack/openframe-frontend-core/utils';
```

### Client Boundary Pattern

Server Components cannot import client-side UI barrel exports directly. Use the re-export wrapper:

**File:** `src/lib/openframe-core-ui.tsx`
```typescript
'use client';
export * from '@flamingo-stack/openframe-frontend-core/components/ui';
```

**Usage:**
```typescript
// In server-adjacent code (layout.tsx, etc.)
import { Toaster } from '@/lib/openframe-core-ui';
```

For regular client components, import directly from the core library.

### Component Categories
- **Core UI** — Button, Card, Input, Dialog, Tabs, Badge, Skeleton, etc.
- **Page Containers** — ContentPageContainer, DetailPageContainer
- **Data Display** — DeviceCard, StatusTag, DashboardInfoCard, OrganizationCard
- **Feature Components** — AuthProvidersList, Terminal, ToolBadge
- **Navigation** — AppLayout, sidebar config types

### Embedded Page Components (standalone + tab reuse)

Some page components render their own `PageLayout` and are used **both** as a standalone route
**and** embedded inside another page's tab (e.g. `LogsTable`, `DevicesPanel` inside customer/device
details). Core `PageLayout`/`TitleBlock` are **FROZEN** — the `TitleBlock` has a hardcoded leading
`pt-[var(--spacing-system-l)]` that is correct standalone but leaves a redundant gap under the tab bar
when embedded.

**Convention:** such a component accepts an `embedded?: boolean` prop. When `embedded`, forward
`EMBEDDED_PAGE_OFFSET` (a `-mt-[var(--spacing-system-l)]` from `@/app/components/shared`) into its
`PageLayout` `className` to cancel that top padding. The header stays; only the top gap is removed.
Callers in tabs just pass `embedded`:

```tsx
<LogsTable organizationId={organizationId} embedded />
<DevicesPanel embedded /* ... */ />
```

Do **not** modify `PageLayout`/`TitleBlock` to fix this — they are frozen.

## Development Patterns

### CRITICAL: React Hooks Rules

**React Hooks MUST be called unconditionally:**
```typescript
// BAD: Hooks called conditionally
export function MyComponent() {
  if (someCondition) {
    return null;  // Early return BEFORE hooks
  }
  const data = useSomeHook();  // ERROR: Hook called after conditional
}

// GOOD: All hooks at the top, unconditionally
export function MyComponent() {
  const data = useSomeHook();
  const router = useRouter();
  const searchParams = useSearchParams();

  // THEN check conditions
  if (someCondition) {
    return null;
  }

  return <div>{data}</div>;
}
```

**Rules:**
1. Move all hooks to the top of the component
2. Use conditional logic INSIDE hooks (useEffect, useMemo), not around them
3. Never wrap hooks in try-catch — handle errors inside the hook instead

### React Compiler (on)

`reactCompiler: true` in `next.config.mjs` — the compiler memoizes components and hooks
automatically, client bundles only (Next passes `isServer` and skips the server compile). It runs
in `dev` as well as in both production targets (`build`, `build:export`).

What it changes about how you write code here:

- **Stop adding `useMemo`/`useCallback`/`React.memo` for render performance.** The compiler
  produces the same memoization from the plain code, and it does it per-value instead of
  per-hook. Keep a manual memo only when it is *semantically* required — a value used as a
  `useEffect` dependency that must stay referentially stable, an object handed to a third-party
  library that compares by identity. The existing manual memos stay: `preserve-manual-memoization`
  (already at `error`) makes the compiler honour them rather than fight them.
- **The lint rules ARE the compiler's diagnostics.** The shared config runs react-hooks v7
  (`set-state-in-effect`, `purity`, `refs`, `immutability`, `preserve-manual-memoization`,
  `static-components`) at `error`, and they were cleared to zero before this was switched on. A
  new violation is not a style nit: it is the compiler telling you it will bail out of that
  component, so the file silently loses the optimization.
  The one exception is `react-hooks/incompatible-library`, which the shared config turns off — it
  reports a *missed* optimization (a third-party hook like `useReactTable` returning functions
  the compiler cannot prove stable), not a bug.
- **`panicThreshold` is the default `'none'`**: a function the compiler cannot compile is skipped,
  never a build error. So enabling this cannot break the build — but it also means a bail-out is
  invisible unless the lint rules catch it.
- **Escape hatch:** the `'use no memo'` directive opts a function — or, at the top of a file,
  the whole module (`hasModuleScopeOptOut`) — out of compilation. Adding one needs a stated
  reason, because it is a silent, permanent de-optimization otherwise. The only ones in `src/`
  today are the react-hook-form opt-outs below.

**react-hook-form files are opted out, and `openframe/react-hook-form-needs-no-memo`
(`eslint-rules/`, at `error`, autofixable) enforces it.** The library mutates `control` and proxies
`formState`, so memoization on top of it does not re-read what it cannot see change — stale
`watch()`, dead `reset()`
([discussion](https://github.com/orgs/react-hook-form/discussions/12524)) — and the compiler's own
diagnostics cannot see it. The rule requires a module-level `'use no memo'` in any file importing
react-hook-form at runtime, or importing one of its live-form handle types (`UseFormReturn`,
`Control`, …). 34 files, 41 functions' worth of memoization. The compiler knows only about
`useForm().watch` itself, plus `@tanstack/react-table`, which only the core lib calls (its
`react-virtual` entry matches nothing — neither repo depends on it). Delete the rule and its
directives on react-hook-form 7.75 + React 19.2.5.

**The core library IS compiled.** Next skips node_modules for the compiler loader, but
`transpilePackages` packages are exempted from that skip (`exclude()` in
`next/dist/build/webpack-config.js`), so every `@flamingo-stack/openframe-frontend-core` `dist`
file goes through it. A `'use no memo'` there is load-bearing in this app, not decoration.

**A hook that WRAPS an incompatible library hides it from the compiler.** Calling
`useReactTable` directly makes the compiler skip that component (`IncompatibleLibrary`); calling
it through a wrapper like the core lib's `useDataTable` does not — the caller compiles, and
whatever the wrapper returns is cached on its identity. TanStack's table instance is one mutated
object whose identity never changes, so every `DataTable` froze on its first page and the
infinite-scroll footer re-requested forever. Fixed in `useDataTable` (it publishes a fresh handle
per render); the same shape of bug is waiting in any other wrapper over a mutated instance.

Cost measured on this repo when it was turned on: `next build` 18.0s → 20.6s, client chunks
17.8 MB → 18.4 MB raw (+3.3%) — the compiler emits memo-cache bookkeeping into every component it
touches.

### Data Fetching Strategy

The app is **gradually migrating GraphQL data fetching to react-relay**. The rules:

1. **New GraphQL code against `/api/graphql` → react-relay.** Queries, fragments, mutations, pagination — all through Relay.
2. **REST APIs → `@tanstack/react-query`** with `apiClient` (this is not changing).
3. **Legacy GraphQL** (raw POST through `apiClient` or react-query wrappers) still exists — leave it working, but migrate it to Relay when touching it substantially. Do not add new code in that style.
4. **Exception — the `/chat/graphql` domain (tickets, mingo, AI settings)**: it talks to the saas-ai-agent service whose schema is NOT in `schema.graphql`, so it stays on raw-POST permanently. Extending raw-POST there is correct, not a violation.
5. No Apollo Client anywhere.

**Every request goes out BELOW `SubscriptionGuard`.** The guard (`src/app/components/subscription-lock/subscription-guard.tsx`) wraps the whole app tree in `app-layout.tsx`, and the network gate it feeds (`src/lib/subscription-gate.ts`) holds app *queries* until the subscription answers and while it locks. **Mutations bypass that gate by design** — they are user actions, and the paywall's own are what a locked workspace needs (`useMutation` takes no `cacheConfig`, so there is no per-call opt-out either). So a mutation fired by a timer/effect rather than by a click goes straight out on a locked workspace and fails on every interval — which `recordPresence` did, once every ten seconds behind the lock screen.

Rules for anything automatic (heartbeats, registrations, telemetry, hydrators):
- Mount it **under** `SubscriptionGuard` — do not add siblings beside it in `AppLayoutInner`.
- Gate it on `useSubscriptionOpen()` (from `subscription-guard.tsx`): `false` until the answer lands, `false` while locked. It fails closed and `console.error`s in dev when there is no guard above it, so a component mounted in the wrong place says so instead of silently spamming the API.
- `useSubscriptionLock()` is the *other* hook — `{ status, isLocked, isResolved }` for UI that renders differently when locked. It falls back to unlocked with no guard above; do not use it to decide whether to send traffic.

### GraphQL with react-relay (preferred)

**Setup:**
- Schema: `schema.graphql` (repo root of the frontend service)
- Config: `relay.config.json`; generated artifacts in `src/__generated__/`
- Environment/provider: `src/lib/relay/` (singleton, cookie auth + 401 refresh, mounted in root layout above all other providers)
- Run `npm run relay` after adding/changing any `graphql\`...\`` tag (`npm run build` also runs it)
- Operation names MUST be prefixed with the camelCased file name (e.g. `unread-counts-relay.ts` → `unreadCountsRelayQuery`)
- **Enum / scalar types come from `@/generated/schema-enums`, NEVER from a query's Relay artifact.** `src/generated/schema-enums.ts` is generated from `schema.graphql` by `npm run generate-enums` (Prisma-style: the same name is both a `const` value and a `type`, so `ScriptShell.CMD` and `const s: ScriptShell` both work). Do NOT `import type { ScriptShell } from '@/__generated__/<someQuery>.graphql'` — relay-compiler owns `src/__generated__/`, re-emits per-operation copies, and prunes them, so those imports are unstable. Refresh the SDL with `npm run fetch-schema`, then `npm run generate-enums`.

**Reference implementations** (notifications domain, fully on Relay):
- `src/graphql/notifications/` — query/fragment/mutation definitions, connection updaters via `ConnectionHandler`
- `src/app/components/notifications/notifications-data-provider.tsx` — `useLazyLoadQuery`, `usePaginationFragment`, `commitLocalUpdate` for NATS live updates
- `src/graphql/notifications/unread-counts-relay.ts` — small query + `fetchQuery` store refresh pattern

**Patterns:**
```typescript
import { graphql, useLazyLoadQuery, useMutation } from 'react-relay';
import type { myFileNameQuery as MyFileNameQueryType } from '@/__generated__/myFileNameQuery.graphql';

export const myFileNameQuery = graphql`
  query myFileNameQuery($first: Int!) {
    notifications(first: $first) { ... }
  }
`;

// In a component (wrap in <Suspense> — useLazyLoadQuery suspends):
const data = useLazyLoadQuery<MyFileNameQueryType>(myFileNameQuery, { first: 30 }, { fetchPolicy: 'store-and-network' });
```
- Prefer fragments + `usePaginationFragment` for connections; use `@connection` + `ConnectionHandler` updaters to keep lists consistent after mutations
- Mutations: `useMutation` with `optimisticUpdater`/`updater`; toast feedback via `useToast` in `onError` stays mandatory
- To refresh store data imperatively: `fetchQuery(environment, query, vars, { fetchPolicy: 'network-only' }).subscribe({})` — all subscribed components re-render from the store

**Shared row shapes → `@inline` fragments, NEVER a hand-written node type.** When several
operations feed the same mapper/table, do not describe the node with a structural interface
(`interface MachineLike { readonly id: string; readonly hostname?: string | null; … }`). Such a
type has to make every field optional to fit all callers, so an operation that forgets a field
renders an empty column instead of failing to compile. Declare the selection once as an
`@inline` fragment, spread it in every operation, and read it in the plain function:

```typescript
// src/graphql/<domain>/<thing>-fields.ts
export const executionFieldsFragment = graphql`
  fragment executionFields_execution on ScriptExecution @inline { id status source … }
`;

// the mapper — a function, not a component, which is exactly what @inline is for
export function toUiExecution(ref: executionFields_execution$key): UiExecution {
  const node = readInlineData(executionFieldsFragment, ref);
  …
}
```

- **Lists that can afford different selections → compose a ladder**, each fragment spreading
  the one below (`deviceRowFields` ⊂ `deviceSelectorFields` ⊂ `deviceFields`), with one mapper
  per step building on the step below it. A list spreads the step it can pay for — e.g.
  `assignedDevices` resolves per machine and has timed out on test-dev, so the schedule tab
  stops at the row step — and gets a mapper typed to exactly that step. `@inline` does accept
  `@argumentDefinitions` + `@include(if:)`, but a conditional field lands in `$data` as
  `field?:`, i.e. back to "everything optional" — that is why the ladder is separate fragments.
- **A field only some lists want** (e.g. `scriptName` on the schedule execution lists) stays
  OUT of the shared fragment: those operations select it beside the spread and pass it to the
  mapper explicitly, so "not selected" stays distinguishable from "null".
- **Never split one field across two steps.** `tags { id key }` low and `tags { color }` high
  land in two different `$data` types with nothing to join the halves on. Select a field whole,
  at one step. (Different *sub-fields of a linked record* are fine — the row step reads
  `organization { name }`, the step above reads `organization { contactInformation { … } }`.)
- **Non-Relay callers** (legacy raw-POST paths) have no fragment reference: give them
  `Omit<someFragment$data, ' $fragmentType'>` plus a mapper taking that data, so they are
  type-checked against the generated shape instead of casting through `unknown`.
- Reference: `src/graphql/devices/device-{row,selector,}-fields.ts` +
  `devices/utils/device-transform.ts`; `src/graphql/scripts/execution-fields.ts`;
  `src/graphql/notifications/notification-fields.ts`.

**Backticks end a `graphql` template literal.** Never write a backticked word inside a GraphQL
`#` comment — the template closes there and relay-compiler reports a bogus syntax error at 1:1.

### REST Fetching with TanStack React Query

REST (non-GraphQL) server state uses `@tanstack/react-query` with `apiClient`.

**Query pattern:**
```typescript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { useToast } from '@flamingo-stack/openframe-frontend-core/hooks';
import { apiClient } from '@/lib/api-client';

// Define query keys
export const devicesQueryKeys = {
  all: ['devices'] as const,
  detail: (id: string) => ['devices', id] as const,
};

// Query hook
export function useDevices() {
  return useQuery({
    queryKey: devicesQueryKeys.all,
    queryFn: async () => {
      const response = await apiClient.get('/api/devices');
      if (!response.ok) throw new Error(response.error || 'Failed to fetch devices');
      return response.data;
    },
  });
}

// Mutation hook with toast feedback
export function useDeleteDevice() {
  const { toast } = useToast();
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async (deviceId: string) => {
      const response = await apiClient.delete(`/api/devices/${deviceId}`);
      if (!response.ok) throw new Error(response.error || 'Failed to delete device');
      return response.data;
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: devicesQueryKeys.all });
      toast({ title: 'Success', description: 'Device deleted', variant: 'success' });
    },
    onError: (err) => {
      toast({
        title: 'Error',
        description: err instanceof Error ? err.message : 'Failed to delete device',
        variant: 'destructive',
      });
    },
  });
}
```

**QueryClient configuration** (in `src/lib/query-client-provider.tsx`):
```typescript
new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60 * 1000,          // 1 minute
      refetchOnWindowFocus: false,
    },
  },
});
```

### Legacy GraphQL Usage (do not extend)

Older code sends GraphQL queries as raw POST requests through `apiClient`:
```typescript
const response = await apiClient.post('/api/graphql', {
  query: GET_DEVICES_QUERY,
  variables: { limit: 20, cursor: null },
});
```
This style is being migrated to react-relay. Don't write new code like this; when substantially reworking a feature that uses it, migrate it to Relay. The exception is the `/chat/graphql` domain (tickets/mingo/AI settings) — permanently raw-POST, see Data Fetching Strategy.

The GraphQL endpoint is determined at runtime: `${NEXT_PUBLIC_TENANT_HOST_URL || window.location.origin}/api/graphql` (the Relay environment resolves the same endpoint).

### API Error Handling with useToast

**MANDATORY:** All API operations must provide user feedback via `useToast`:

```typescript
import { useToast } from '@flamingo-stack/openframe-frontend-core/hooks';

// In mutations — use onSuccess/onError callbacks
const mutation = useMutation({
  mutationFn: someApiCall,
  onSuccess: () => {
    toast({ title: 'Success', description: 'Operation completed', variant: 'success' });
  },
  onError: (err) => {
    toast({
      title: 'Error',
      description: err instanceof Error ? err.message : 'Operation failed',
      variant: 'destructive',
    });
  },
});
```

Use `src/lib/handle-api-error.ts` for reusable error extraction:
```typescript
import { handleApiError, getErrorMessage } from '@/lib/handle-api-error';
```

### Forms with react-hook-form + zod

Use `react-hook-form` with `zod` schemas for all forms:

```typescript
import { z } from 'zod';
import { zodResolver } from '@hookform/resolvers/zod';
import { useForm, Controller } from 'react-hook-form';
import { useToast } from '@flamingo-stack/openframe-frontend-core/hooks';
import { useMutation } from '@tanstack/react-query';

// 1. Define schema
const formSchema = z.object({
  name: z.string().min(1, 'Name is required'),
  timeout: z.number().min(1).max(86400),
  platforms: z.array(z.string()).min(1, 'Select at least one platform'),
});

type FormData = z.infer<typeof formSchema>;

// 2. Use in component
export function MyForm() {
  const { toast } = useToast();
  const form = useForm<FormData>({
    resolver: zodResolver(formSchema),
    defaultValues: { name: '', timeout: 90, platforms: ['windows'] },
  });

  const mutation = useMutation({
    mutationFn: async (data: FormData) => { /* API call */ },
    onSuccess: () => toast({ title: 'Saved', description: 'Form submitted', variant: 'success' }),
    onError: (err) => toast({ title: 'Error', description: getErrorMessage(err), variant: 'destructive' }),
  });

  const onSubmit = form.handleSubmit(
    (data) => mutation.mutate(data),
    (errors) => {
      const messages = Object.values(errors).map(e => e?.message).filter(Boolean);
      toast({ title: 'Validation Error', description: messages.join(', '), variant: 'destructive' });
    },
  );

  return (
    <form onSubmit={onSubmit}>
      <Controller name="name" control={form.control} render={({ field }) => <Input {...field} />} />
      <Button type="submit" disabled={mutation.isPending}>
        {mutation.isPending ? 'Saving...' : 'Save'}
      </Button>
    </form>
  );
}
```

**Real example:** See `src/app/(app)/scripts/hooks/use-edit-script-form.ts` and `src/app/(app)/scripts/types/edit-script.types.ts`.

### State Management with Zustand

```typescript
import { create } from 'zustand';
import { devtools } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

interface MyState {
  items: Item[];
  setItems: (items: Item[]) => void;
}

export const useMyStore = create<MyState>()(
  devtools(
    immer((set) => ({
      items: [],
      setItems: (items) => set(state => { state.items = items }),
    })),
    { name: 'my-store' },
  ),
);
```

**Existing stores:**
- `useAuthStore` — authentication state (`src/app/(auth)/auth/stores/auth-store.ts`; persist key `auth-storage`)
- `useFeatureFlagsStore` — server-loaded feature flags (`src/stores/feature-flags-store.ts`)
- `useDevicesStore` — `src/stores/devices-store.ts` (persist key `devices-store`; mostly unused — devices flow through react-query)
- Domain stores live in their modules (tickets, mingo, scripts); central re-exports from `src/stores/index.ts`

### Code Quality: ESLint + Prettier

Biome was removed on 2026-09-01. **ESLint owns the rules, Prettier owns the formatting**, and
neither rule set lives in this repo: both come from the shared config shipped inside the core
library, the same one the library and every other Flamingo frontend loads.

```
eslint.config.mjs   next + relay + tests + prettier-compat  ← the fast pass, what the editor loads
eslint-rules/       repo-local rules, registered under the `openframe/` prefix
eslint.ci.mjs       − relay/unused-fields                   ← npm run lint:ci, the PR gate
eslint.types.mjs    + type-checked                          ← npm run lint:types
eslint.cycles.mjs   + cycles (import/no-cycle)              ← npm run lint:cycles
prettier.config.mjs the shared preset, re-exported unchanged
```

Rules are documented in `node_modules/@flamingo-stack/openframe-frontend-core/eslint-config/README.md`.
The parts that change how you write code here:

- **No inline suppressions.** `noInlineConfig` is on: an `// eslint-disable-next-line` comment does
  nothing and is itself reported as an error. A finding is fixed, or it is carried by a **named,
  `files:`-scoped block in `eslint.config.mjs` that states its reason** — reviewable, unlike a
  comment buried in a diff.
- **Severity means autofixable.** `error` is what `eslint --fix` and the editor's
  `source.fixAll.eslint` clear. `warn` is what a human has to decide, so `npm run lint` runs with
  `--max-warnings 0`.
- **Import order is an ESLint rule** (`perfectionist/sort-imports`), not a formatter concern — one
  save-time actor, no fight with Prettier. Do not add VS Code's `source.organizeImports`.
- **Prettier settings reproduce the old Biome formatter** (2-space, 120 cols, single quotes,
  trailing commas, semicolons, avoided arrow parens) and add Tailwind class sorting on top.
- **Type-aware rules are a separate pass.** `npm run lint:types` (floating promises, the unsafe-`any`
  family, misused await) needs a TypeScript program and an 8 GB heap, so the editor does not run it.

Two rules the shared config deliberately omits, and this repo does not add back: `no-console` (the
frontends use it as a logging channel) and `@typescript-eslint/naming-convention` (Biome's
`useNamingConvention` equivalent, never actually enforced).

**The one remaining backlog: `relay/unused-fields` (543).** Everything else is at zero.

The migration surfaced ~1 130 errors, because the old `eslint.config.mjs` declared no `files:`
patterns, matched no `.ts`/`.tsx` file at all, and `npm run lint` therefore linted **nothing**. All
of them are now fixed except this rule, which cannot be cleared mechanically — each finding is a
decision about whether a query should stop selecting a field or a consumer should start reading it
through a fragment.

**CI and the pre-commit hook both run `eslint.ci.mjs`** (`npm run lint:ci`;
`.github/workflows/test.yml`, job `Lint`) — the fast pass with `relay/unused-fields` turned off, so
neither gate refuses a change over a field somebody else over-fetched. The rule stays ON in
`eslint.config.mjs`, which is what the editor loads, so you still see it in the file you are in.
Delete `eslint.ci.mjs` and point both at `npm run lint` once the count reaches zero.

There is no suppressions file anywhere, so no count can drift back up: what is not fixed is carried
by a named `files:`-scoped block that states its reason.

**Run manually:**
```bash
npm run lint         # fast pass — everything, including the backlog rule
npm run lint:ci      # what CI blocks on
npm run lint:fix     # autofix
npm run format:fix   # Prettier
```

## URL State Management (useApiParams)

URL state (pagination, filters, search) uses the core library's `useApiParams` with a manual schema — for GraphQL-backed and REST-backed tables alike. The old runtime-introspection `useQueryParams` system is no longer used in this app.

```typescript
import { useApiParams } from '@flamingo-stack/openframe-frontend-core/hooks';

const { params, setParam, setParams } = useApiParams({
  search: { type: 'string', default: '' },
  page: { type: 'number', default: 1 },
  status: { type: 'string', default: 'all' },
});
```

**Used in:** LogsTable, DevicesView, ScriptsTable, customers/monitoring/tickets tables (~22 files).

## Root Layout & Provider Stack

The root layout (`src/app/layout.tsx`) establishes the global provider hierarchy:

```
<html> (dark mode, font variables)
  <head>
    <PublicEnvScript />              <!-- next-runtime-env (skipped in export builds) -->
    (sidebar-width FOUC script)
  </head>
  <body>
    <GoogleTagManager />             <!-- Analytics (if GTM ID set) -->
    <EmbedShimRegistration />        <!-- registers Next router/Link/Image into core-lib embed-shims -->
    <DeploymentInitializer />        <!-- Runtime detection -->
    <NativeShellInitializer />       <!-- Capacitor/Tauri shell bridge -->
    <RelayProvider>                  <!-- react-relay environment (singleton) -->
      <QueryClientProvider>          <!-- TanStack React Query -->
        <DevTicketObserver />        <!-- Dev auth (if auth enabled) -->
        <NatsAppProvider>            <!-- NATS live updates -->
          <BiometricLockBoundary>    <!-- Native-shell biometric cold-start lock -->
            <FeatureFlagsLoader>     <!-- Runs the flags query; does NOT gate render -->
              <NotificationsDataProvider>  <!-- Notifications drawer/popups (Relay) -->
                <RouteGuard>         <!-- App mode route filtering -->
                  {children}         <!-- Page content -->
                </RouteGuard>
              </NotificationsDataProvider>
            </FeatureFlagsLoader>
          </BiometricLockBoundary>
        </NatsAppProvider>
      </QueryClientProvider>
    </RelayProvider>
    <Toaster />                      <!-- Toast notifications -->
  </body>
</html>
```

**Fonts:** DM Sans (body) + Azeret Mono (code) — loaded via `next/font/google`

**Rendering:** dual output — `standalone` (default) or full static `export` (`OPENFRAME_BUILD_TARGET=export`); `trailingSlash: true` everywhere, `skipTrailingSlashRedirect` only in standalone (export builds break without trailing slashes). This is why detail pages use `?id=` query params instead of dynamic segments.

## Fleet MDM Integration

OpenFrame integrates device monitoring data from multiple sources with normalization.

### Multi-Source Data Architecture

**Data Sources:**
1. **GraphQL** — Primary device registry and agent information
2. **Fleet MDM** — Accurate hardware specs, battery health, users
3. **MeshCentral** — Live online/offline status + last-seen

> **Tactical RMM has been fully removed** from the frontend — no `tactical-api-client.ts`,
> no Tactical types, and no Tactical fields in the device merge. Remaining references are
> legacy `/scripts` stubs (see `src/app/(app)/scripts/lib/scripts-migration.ts`, all marked
> `TODO(openframe-rmm)`) that return empty / throw a "migration pending" error, plus mention-chip
> id-shape handling in Mingo. `runScript` now means a GraphQL mutation via the Scripts module
> (`src/graphql/scripts/run-script-mutation.ts`), not Tactical REST.

**Merge logic locations** (there is no `normalize-device.ts`):
- Detail page: `createDevice()` in `src/app/(app)/devices/hooks/use-device-details.ts` — raw-POST GraphQL node + fan-out to Fleet host and MeshCentral deviceStatus (no Tactical)
- List page: `createDeviceListItem()` in `src/app/(app)/devices/utils/device-transform.ts` — GraphQL node only, no external fan-out

**Priority rules** (in `createDevice()`):
```
Hardware (CPU/RAM/storage/battery/software/users):  Fleet only
Serial/manufacturer/model/OS:  Fleet -> GraphQL node
Status:                        GraphQL node -> Fleet
Last seen:                     Fleet -> GraphQL node
Agent version:                 GraphQL node -> Fleet osquery_version
Public IP:                     filtered by isPrivateIp (10/172.16-31/192.168/127/169.254/fe80/fc00/fd00/::1)
Local IPs:                     dedup [fleet.primary_ip, fleet.public_ip-if-public, node.ip]
```

### Key Types

**Fleet types** — `src/app/devices/types/fleet.types.ts`:
```typescript
export interface FleetHost {
  cpu_brand: string;
  cpu_physical_cores: number;
  cpu_logical_cores: number;
  memory: number;           // bytes
  primary_ip: string;
  public_ip: string;        // May be private — filter it!
  users: FleetUser[];
  batteries: FleetBattery[];
  software: FleetSoftware[];
  mdm: FleetMDMInfo;
}
```

**Unified types** — `src/app/(app)/devices/types/device.types.ts`: flat `Device` with all fields at
root (no nesting). Fleet is the only external source that populates hardware/users/software, so
there is no multi-source `source` discriminator on the user/hardware types anymore.

### Key Files
- `src/app/(app)/devices/types/fleet.types.ts` — Complete Fleet MDM types
- `src/app/(app)/devices/types/device.types.ts` — Unified device types (flat `Device`, all fields at root)
- `src/app/(app)/devices/hooks/use-device-details.ts` — Multi-source merge (`createDevice()`)
- `src/app/(app)/devices/utils/device-transform.ts` — List-item transform
- `src/app/(app)/devices/components/tabs/` — hardware/network/users/os/software/… tabs
- `src/lib/fleet-api-client.ts` — Fleet API integration
- `src/lib/meshcentral/meshcentral-api.ts` — MeshCentral live status / last-seen

## Accessibility Standards

### Required Practices
1. **Semantic HTML** — Use proper HTML elements and core library components
2. **Keyboard Navigation** — Core library provides automatic support
3. **Screen Reader Support** — Add aria-labels and descriptions
4. **Color/Contrast** — Use ODS design tokens only
5. **Focus Management** — Handle focus in modals and dynamic content

### ODS Design Tokens (MANDATORY)

ALL styling must use ODS design tokens — never hardcode colors, font families, font sizes, or spacing.

The full canonical ODS token rules (colors, spacing, typography, Figma workflow) are the **single
source of truth** maintained in `@flamingo-stack/openframe-frontend-core` and imported here straight
from the installed package. Edit the rules in the core lib, not here:

@./node_modules/@flamingo-stack/openframe-frontend-core/src/ODS_TOKEN_RULES.md

**Tailwind preset:** ODS colors/utilities are provided via the core library's Tailwind preset (see `tailwind.config.ts`).

### Inverted Progress Bar
```typescript
// Disk usage: high = bad (red)
<ProgressBar progress={diskUsage} inverted={false} />

// Battery health: high = good (green)
<ProgressBar progress={batteryHealth} inverted={true} />
```

## Testing & Deployment

### Development Testing
| Command | Purpose |
|---------|---------|
| `npm run type-check` | TypeScript validation |
| `npm run lint` | ESLint (fast pass) |
| `npm run format` | Prettier formatting check |
| `npm run build` | Production build verification |

### Build & Deployment
```bash
npm run build       # Output: dist/ directory
npm run start       # Serve production build
```

**Deployment Targets:**
- Container deployment with nginx
- Static hosting (Vercel, Netlify, AWS S3)
- CDN distribution

## Troubleshooting

### Common Issues

**Port Conflicts:**
```bash
lsof -i:3000                    # Check port usage
PORT=3001 npm run dev           # Use different port
```

**Core Library Issues (when yalc-linked):**
```bash
# In the lib repo — rebuild + push into linked consumers
cd ~/flamingo/openframe-oss-lib/openframe-frontend-core
npm run build && yalc push

# In this repo — (re)link / unlink
npm run core:link && npm install
npm run core:unlink   # back to the registry version
```

**Lint / format errors:**
```bash
npm run lint:fix          # Auto-fix most issues
npm run format:fix        # Fix formatting
```
An `// eslint-disable` comment will not silence anything — `noInlineConfig` is on. See
Code Quality above for what to do instead.

**API Connection:**
- Verify `NEXT_PUBLIC_TENANT_HOST_URL` matches backend
- Check CORS configuration
- For dev ticket mode, ensure `NEXT_PUBLIC_ENABLE_DEV_TICKET_OBSERVER=true`

**State Management:**
```javascript
// Clear corrupted localStorage
localStorage.removeItem('devices-store');
localStorage.removeItem('auth-storage');
```

## Key Integration Points

### Backend Services (all via the gateway)
- **REST** — `/api/*` — openframe-api
- **GraphQL** — `/api/graphql` — openframe-api (Relay + legacy); `/chat/graphql` — saas-ai-agent (tickets/mingo, raw-POST, SaaS only)
- **Live updates** — NATS over WebSocket at `/ws/nats-api` (notifications, chat chunks); tool WS at `/ws/tools/{toolId}`
- **Authentication** — `/oauth/*` (gateway BFF: login/callback/refresh/logout/dev-exchange); registration via `/sas/oauth/*`
- **Tool proxies** — `/tools/{toolId}/*` (Fleet; API keys injected by the gateway)

### API Client Architecture

The `ApiClient` singleton (`src/lib/api-client.ts`) handles:
- Cookie-based auth (production) + header-based auth (dev ticket mode)
- Automatic 401 detection and token refresh
- Request queuing during refresh
- Force logout on auth failure

```typescript
import { apiClient } from '@/lib/api-client';

const response = await apiClient.get<Device[]>('/api/devices');
if (response.ok) {
  console.log(response.data);
} else {
  console.error(response.error);
}
```

### External Dependencies
- **Core Library** — `@flamingo-stack/openframe-frontend-core` (npm registry; yalc for local lib dev)
- **Terminal** — @xterm/xterm 6.0 integration
- **Code Editor** — Monaco Editor for script editing
- **Fleet MDM** — Device monitoring integration
- **MeshCentral** — Remote desktop/shell/file management (via `src/lib/meshcentral/`)

---

**Final Reminders:**
1. **Core library is EXTERNAL** — separate repo, not part of OpenFrame
2. **Always use core library components** — no custom UI primitives
3. **Always use `useToast`** for all API operation feedback
4. **Use ODS design tokens** — never hardcode colors or styles
5. **Use react-relay for GraphQL** (gradual migration — prefer it wherever possible); **TanStack React Query for REST** — no Apollo Client, no new raw-POST GraphQL
6. **Use react-hook-form + zod** for forms
7. **ESLint + Prettier, rules from the core library's shared config** — no inline
   `eslint-disable`; keep the files you touch clean
8. **Normalize multi-source device data** — Fleet-first priority; merge logic in `use-device-details.ts` `createDevice()`
9. **Build internal URLs via `routes.*` from `src/lib/routes.ts`** — no raw path strings; new pages/tabs must be added to the registry (see `src/lib/ROUTES.md`)

---
> Source: [flamingo-stack/openframe-oss-frontend](https://github.com/flamingo-stack/openframe-oss-frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
