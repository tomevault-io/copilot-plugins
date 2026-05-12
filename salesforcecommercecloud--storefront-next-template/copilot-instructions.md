## storefront-next-template

> Storefront template for Salesforce Commerce Cloud built with React Router v7, React, Tailwind CSS, and Vite. Integrates with Commerce Cloud via SCAPI.

# Template Retail App

Storefront template for Salesforce Commerce Cloud built with React Router v7, React, Tailwind CSS, and Vite. Integrates with Commerce Cloud via SCAPI.

This file is the single source of truth for AI coding agents (Claude Code, Cursor, Codex, etc.) working in this package. `CLAUDE.md` is a symlink to this file.

## Project Structure

- `./src/` — Application source code
  - `./src/routes/` — React Router file-based routes
  - `./src/components/` — React components (`./src/components/ui/` for Radix + Tailwind primitives)
  - `./src/lib/` — Shared utilities, hooks, business logic (adapters, adapter registry, decorators)
  - `./src/hooks/`, `./src/providers/` — React hooks and context providers
  - `./src/extensions/` — Optional feature extensions
  - `./src/locales/` — i18n translation files
  - `./src/types/config.ts` — Template-specific config types
- `./config.server.ts` — Configuration defaults
- `./.storybook/` — Storybook configuration
- `./public/` — Static assets
- `./docs/` — Detailed docs (see **Key Documentation** below)

## Common Commands

```bash
# Dev
pnpm dev                         # Dev server at http://localhost:5173
pnpm dev:debug                   # Dev server with Node debugger
pnpm storybook                   # Storybook at http://localhost:6006

# Build / deploy
pnpm build                       # Production build
pnpm preview                     # Preview production build
pnpm push                        # Deploy to Commerce Cloud Managed Runtime
pnpm generate:cartridge          # Extract Page Designer metadata

# Quality
pnpm typecheck
pnpm lint                        # Strict: --max-warnings 0 (CI enforces)
pnpm lint:fix
pnpm bundlesize:test             # Verify bundle size limits
pnpm lighthouse:ci

# Tests — prefer :agent variants for condensed output
pnpm test:agent                  # Unit tests (summary only)
pnpm test                        # Unit tests (verbose, with coverage)
pnpm test:watch
pnpm test src/components/foo     # Single file/dir

pnpm test-storybook:interaction:agent
pnpm test-storybook:a11y:agent
pnpm test-storybook:snapshot:agent

# UITargets
pnpm --filter template-retail-rsc-app dev:ui-targets        # Visual overlay showing targets
pnpm --filter template-retail-rsc-app smoke-test:generate   # Sync target-config.json (additive)
```

### Agent Command Summary

| Command | Purpose | Output |
|---------|---------|--------|
| `pnpm test:agent` | Unit tests | Last 30 lines |
| `pnpm test:agent:coverage` | Unit tests + coverage | Last 40 lines |
| `pnpm test-storybook:snapshot:agent` | Snapshot tests | Last 30 lines |
| `pnpm test-storybook:interaction:agent` | Interaction tests | Last 20 lines (PASS/FAIL only) |
| `pnpm test-storybook:a11y:agent` | A11y tests | Last 20 lines (violations only) |

## Performance & Data Rules

These rules take priority when designing routes, components, and state. Apply them as a checklist for every route module and every component that consumes async data. See [Data Fetching](./docs/README-DATA.md), [Loading States](./docs/README-SUSPENSE.md), [State Management](./docs/README-STATE.md), and [Performance](./docs/README-PERFORMANCE.md) for full context.

### Data Loading

1. **Server-load everything.** All initial data must come from server `loader` functions — never `useEffect`, `fetch`, or other client-side fetching for data needed on first render.
2. **Classify every data field per route.** Critical data (SEO, LCP, CLS, HTTP status) is `await`ed in the loader. Non-critical data is returned as an unresolved Promise. Interaction-driven data is fetched via `useFetcher` on user action.
3. **Never block the loader on non-critical data.** Return the Promise directly — don't `await` recommendations, reviews, or below-the-fold content.
4. **Export `shouldRevalidate` on routes with URL-driven filtering.** Prevent redundant loader re-execution when only search params change and the loader already handles them on the next navigation.
5. **No `clientLoader` or `clientAction`.** Only server `loader` and server `action` exports are permitted in route modules.

### Rendering & Visual Stability

6. **One `<Suspense>` boundary per async operation.** Never place multiple `use()` calls or `<Await>` components inside a single `<Suspense>` boundary — each deferred Promise gets its own boundary and its own skeleton. See [Suspense Boundary Granularity](./docs/README-SUSPENSE.md#suspense-boundary-granularity) for examples and anti-patterns.
7. **Skeleton screens for known layouts, spinners for indeterminate operations.** If the shape of the resolved content is known, use a skeleton. Spinners are only for global or unknown-layout loading states.
8. **Above the fold: avoid `fallback={null}` without reserving space.** Rendering nothing and then injecting content causes CLS. If no visual fallback is desired, the container must maintain explicit dimensions (`minHeight`, aspect ratio).
9. **Below the fold: prefer `fallback={null}` or a simple placeholder.** Users don't perceive layout shift for content they can't see, and complex skeletons add hydration cost without visible benefit.

### Mutations & Interactions

10. **Navigating mutations: `action` + `<Form>`.** Non-navigating mutations: `useFetcher`. Never mix these — the choice determines whether React Router triggers a route transition.
11. **Prefer optimistic UI when failure is unlikely and reversible.** Use `fetcher.formData` for simple optimistic reads, `useOptimistic` for complex state transformations (e.g., list insertions).

### State Management

12. **URL-worthy state goes in `useSearchParams`, not `useState`.** Filters, pagination, sort order, and modal visibility belong in the URL — they must survive refresh and be shareable.
13. **Never store derived state in `useState`.** Compute inline or use `useMemo` for expensive derivations. A second source of truth is a bug waiting to happen.
14. **Split React Contexts by concern.** One context per domain (theme, locale, user) — never a single large `AppContext`. Every value change re-renders all consumers of that context.
15. **Persistent cross-request state via cookies/sessions, not `localStorage`.** Cookies are SSR-compatible, avoid hydration mismatches, and work before scripts load.

### Images

16. **Use `<DynamicImage>` with `widths` or `heights` for all product and content images.** Without either, the component renders a plain `<img>` with no responsive sources and no DIS resizing. Set `priority="high"` on LCP-candidate images (hero, first product image) to trigger React 19 SSR preloading. See [Images](./docs/README-IMAGES.md).
17. **Use `DynamicImageProvider` for image grids.** Wrap product grids in a provider to control priority and responsive widths centrally rather than prop-drilling through every tile.

### Best Practices

18. **Lazy-load overlays and heavy below-the-fold content.** Use `React.lazy()` with deferred mounting — only mount the `<Suspense>` subtree after the first user interaction. See [Lazy Loading for Overlays](./docs/README-SUSPENSE.md#lazy-loading-for-overlays-modals-drawers-dialogs).
19. **Self-host web fonts.** Use WOFF2 variable fonts, preload in `<head>`, inline the `@font-face` declaration, and set `font-display: swap` or `optional`. Never load fonts from third-party CDNs (cache partitioning, GDPR).
20. **Never load third-party scripts synchronously.** Always use `async` or `defer`. Lazy-load interaction-driven widgets (chat, social) on scroll or click, not on page load.
21. **Monitor bundle size.** Run `pnpm bundlesize:test` to verify against configured size limits — CI enforces these on every PR. Check bundle impact with `pnpm bundlesize:analyze` before adding large dependencies.
22. **Configure resource hints via `config.server.ts`.** Use `preconnect` for origins contacted on every page (e.g., image CDN), `dns-prefetch` for optional origins. Don't preconnect to origins that aren't used on every page.

## Code Conventions

### Copyright Header (required)

All TypeScript/JavaScript files must include this Apache 2.0 header. Enforced by ESLint via `eslint-plugin-header`.

```typescript
/**
 * Copyright 2026 Salesforce, Inc.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
```

### Site-context-aware navigation — use project wrappers, not React Router originals

This project provides `Link`, `NavLink` (from `@/components/link`) and `useNavigate` (from `@/hooks/use-navigate`) that automatically apply site/locale URL prefixes via `buildUrl`. Using the React Router originals silently produces unprefixed URLs that break multi-site routing.

```typescript
// Correct — site-context-aware
import { Link, NavLink } from '@/components/link';
import { useNavigate } from '@/hooks/use-navigate';

// Wrong — bypasses site context, produces unprefixed URLs
import { Link, NavLink, useNavigate } from 'react-router';
```

Use React Router's `href()` for type-safe param interpolation; combine it with the project's wrappers — `href()` interpolates params, the wrapper adds the site prefix:

```typescript
import { href } from 'react-router';
import { Link } from '@/components/link';

<Link to={href('/product/:id', { id: product.id })}>Product</Link>
```

### Lazy loading for overlays (modals, drawers, dialogs)

Overlay components hidden on initial render **must** use `React.lazy()` with deferred mounting — only mount the `<Suspense>` subtree after the first user interaction. See [Lazy Loading for Overlays](./docs/README-SUSPENSE.md#lazy-loading-for-overlays-modals-drawers-dialogs) for the pattern, anti-patterns, and rationale.

### Styling

- Use Tailwind utility classes
- Use design tokens (`bg-foreground`, `text-muted-foreground`), not hard-coded colors
- Use `cn()` from `@/lib/utils` to merge class names
- Responsive breakpoints: `sm`, `md`, `lg`, `xl`, `2xl`

See [docs/README-UI-STYLING.md](./docs/README-UI-STYLING.md) for the full guide.

## Testing

Three strategies — see [docs/README-TESTS.md](./docs/README-TESTS.md) for patterns.

- **Unit tests (Vitest):** `*.test.ts`, `*.test.tsx` — React Testing Library, jsdom, MSW for API mocking.
- **Storybook snapshot / interaction tests:** Visual regression and user-flow testing via `play` functions.
- **Storybook a11y tests:** Accessibility violation detection via axe-core.

## Key Documentation

The docs below are where architectural detail lives — consult them for tasks in the relevant area.

**Architecture & patterns:**
- [docs/README-DATA.md](./docs/README-DATA.md) — Data fetching: loaders, actions, fetchers, middlewares, cookies/sessions
- [docs/README-SUSPENSE.md](./docs/README-SUSPENSE.md) — Loading states and Suspense patterns
- [docs/README-STATE.md](./docs/README-STATE.md) — State management: server state, URL state, optimistic UI
- [docs/README-ADAPTER-PATTERN-GUIDE.md](./docs/README-ADAPTER-PATTERN-GUIDE.md) — Adapter pattern for data fetching (Einstein, Active Data, custom)
- [docs/README-CUSTOM-APIS.md](./docs/README-CUSTOM-APIS.md) — Custom SCAPI clients
- [docs/README-CONFIG.md](./docs/README-CONFIG.md) — Configuration system (including `PUBLIC__` prefix behavior)
- [docs/README-CONFIG-OPTIONS.md](./docs/README-CONFIG-OPTIONS.md) — Configuration options reference
- [docs/README-AUTH.md](./docs/README-AUTH.md) — Authentication patterns
- [docs/README-I18N.md](./docs/README-I18N.md) — Internationalization
- [docs/README-MULTI-SITE.md](./docs/README-MULTI-SITE.md) — Site context and locale URL routing
- [docs/README-ENGAGEMENT-ANALYTICS.md](./docs/README-ENGAGEMENT-ANALYTICS.md) — Einstein / Active Data event mapping
- [docs/README-PAGE-DESIGNER.md](./docs/README-PAGE-DESIGNER.md) — Page Designer component development (decorators, metadata)

**UI & frontend:**
- [docs/README-UI-STYLING.md](./docs/README-UI-STYLING.md) — Tailwind, shadcn, design tokens
- [docs/README-IMAGES.md](./docs/README-IMAGES.md) — DIS integration, `<DynamicImage>`, alt text
- [docs/README-SEO.md](./docs/README-SEO.md) — Page titles, meta tags, canonical URLs
- [docs/README-PERFORMANCE.md](./docs/README-PERFORMANCE.md) — Web fonts, third-party scripts, bundles
- [docs/README-PERFORMANCE-METRICS.md](./docs/README-PERFORMANCE-METRICS.md) — Performance monitoring

**Testing & quality:**
- [docs/README-TESTS.md](./docs/README-TESTS.md) — Testing strategy and patterns
- [docs/README-ESLINT.md](./docs/README-ESLINT.md) — ESLint configuration
- [docs/README-STORY-COVERAGE.md](./docs/README-STORY-COVERAGE.md) — Story coverage enforcement
- [.storybook/README-STORYBOOK.md](./.storybook/README-STORYBOOK.md) — Storybook setup

**Development:**
- [docs/README-HYBRID-PROXY.md](./docs/README-HYBRID-PROXY.md) — Hybrid proxy for local development
- [src/extensions/README.md](./src/extensions/README.md) — Extensions system (including `@sfdc-extension-*` markers)

---
> Source: [SalesforceCommerceCloud/storefront-next-template](https://github.com/SalesforceCommerceCloud/storefront-next-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
