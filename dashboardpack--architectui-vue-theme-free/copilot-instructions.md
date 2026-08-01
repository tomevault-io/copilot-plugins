## architectui-vue-theme-free

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ArchitectUI Vue Free - a Vue 3 admin dashboard template built with Bootstrap 5 and BootstrapVueNext. Uses Pinia for state management, Vite for builds, and Chart.js for visualizations.

## Development Commands

```bash
npm run serve          # Start dev server (http://localhost:8087) — alias for `npm run dev`, both run vite
npm run build          # Production build: vue-tsc --noEmit (type-check) THEN vite build
npm run preview        # Preview production build
npm run lint           # Run ESLint with auto-fix
npm run format         # Format code with Prettier
npm run format:check   # Check formatting without changes
npm run type-check     # Run TypeScript type checking (vue-tsc --noEmit)
npm run test           # Run tests in watch mode
npm run test:run       # Run tests once
npm run test:ui        # Vitest browser UI
npm run test:coverage  # Run tests with coverage report (v8 provider)

# Run a single test file or by name
npx vitest run tests/utils/helpers.test.ts
npx vitest run -t "partial test name"
```

A **husky pre-commit hook** runs `lint-staged` (Prettier + ESLint on staged files), so commits auto-format/lint. `npm run prepare` installs the hook.

## Language: JavaScript app code, TypeScript tooling

Despite the full TS toolchain (vue-tsc, `type-check`, tsconfig), the **application source is plain JavaScript** — `main.js`, all stores (`*.js`), `router/index.js`, and `<script>` blocks in `.vue` files. The only `.ts`/`.d.ts` files are `src/types/*.d.ts`, tests, and config. Match the surrounding style: add new stores/router entries as `.js`, not `.ts`. Components mix Composition API and Options API (e.g. [App.vue](src/App.vue) uses Options API).

## Architecture

### Layout System

The app uses dynamic layouts determined by route `meta.layout` property:

- **`default-layout`** ([baseLayout.vue](src/Layout/Wrappers/baseLayout.vue)) - Main dashboard with header, sidebar, footer
- **`userpages-layout`** ([pagesLayout.vue](src/Layout/Wrappers/pagesLayout.vue)) - Minimal layout for auth pages

Routes without `meta.layout` default to `default-layout`. Example:

```javascript
// Uses default-layout (dashboard)
{ path: '/', component: () => import('../DemoPages/Dashboards/Analytics.vue') }

// Uses userpages-layout (auth pages)
{ path: '/pages/login-boxed', meta: { layout: 'userpages' }, component: ... }
```

The layout switching happens in [App.vue](src/App.vue): a computed property appends `-layout` to `route.meta.layout` (defaulting to `default`), so `meta.layout: 'userpages'` resolves to the `<userpages-layout>` component. Both layout wrappers are **registered globally in [main.js](src/main.js)** (`app.component('default-layout', ...)`); a new layout must be registered there before any route's `meta.layout` can reference it.

### State Management (Pinia)

Stores in `/src/stores/`:

- **`dashboard.js`** - Analytics data, statistics, todos, timeline
- **`ui.js`** - Sidebar state, theme, mobile responsiveness, page title
- **`notifications.js`** - System messages and alerts
- **`sidebar.js`** - Sidebar-specific state

Usage:

```javascript
import { useDashboardStore } from '@/stores/dashboard'
import { useUIStore } from '@/stores/ui'

const dashboardStore = useDashboardStore()
const uiStore = useUIStore()

// Access state
dashboardStore.stats
uiStore.sidebarCollapsed

// Actions
uiStore.toggleSidebar()
uiStore.setTheme('dark')
```

### Component Registration

BootstrapVueNext components are imported and registered **individually** in [main.js](src/main.js) with kebab-case names (e.g., `<b-button>`, `<b-modal>`) — the full plugin is not installed. **To use a `<b-*>` component not already in the list, import it and add an `app.component(...)` line in main.js**, otherwise it silently renders as an unknown element.

FontAwesome is likewise tree-shaken: each icon must be **both imported and passed to `library.add(...)`** in main.js before `<font-awesome-icon :icon="['fas', 'icon-name']" />` will resolve it.

### Build Configuration

- **Vite config**: [vite.config.ts](vite.config.ts) - Includes manual chunk splitting for vendor, bootstrap, charts, and icons
- **Vitest config**: [vitest.config.ts](vitest.config.ts) - Test setup with jsdom environment
- **Production base path**: `/architectui-vue-free/` — set in **both** [vite.config.ts](vite.config.ts) `base` and the router's `createWebHistory` ([src/router/index.js](src/router/index.js)), applied only when `NODE_ENV === 'production'` / `import.meta.env.PROD`. Keep the two in sync (GitHub Pages deployment).

### Key Directories

- `/src/DemoPages/` - All demo pages organized by category (Dashboards, Elements, Components, Forms, Tables, Charts, Widgets, UserPages)
- `/src/Layout/Components/` - Header, Sidebar, Footer components
- `/src/assets/` - SCSS styles and themes
- `/tests/` - Test files with setup in `tests/setup.ts`

### Icons

Two icon systems coexist:

- **FontAwesome** - `<font-awesome-icon :icon="['fas', 'icon-name']" />` (import + `library.add` required — see Component Registration)
- **Pe7-icon** - CSS class-based, no registration (e.g., `<i class="pe-7s-rocket"></i>`)

### Styling

SCSS with Bootstrap 5. Entry point is `/src/assets/base.scss`. Theme colors use primary green `#69aa8a`.

---
> Source: [DashboardPack/architectui-vue-theme-free](https://github.com/DashboardPack/architectui-vue-theme-free) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
