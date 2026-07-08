## chinabox-admin

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

chinaexpress-Admin is a Vue 3 + TypeScript admin panel for an e-commerce platform (Silk Shop). The application is bilingual (Turkmen/Russian) and manages products, orders, customers, analytics, and settings. It uses Pinia for state management, Vue Router for routing, and Axios for API communication with automatic token refresh.

## Tech Stack

- Vue 3 (Composition API with `<script setup>`)
- TypeScript
- Vite (dev server on port 1010)
- Pinia (state management)
- Vue Router 5 (with route guards)
- Axios (HTTP client with interceptors)
- ESLint + Oxlint + Prettier

## Development Commands

```bash
# Install dependencies
pnpm install

# Start development server (port 1010)
pnpm dev

# Build for production
pnpm build

# Preview production build
pnpm preview

# Type checking
pnpm type-check

# Linting (runs both oxlint and eslint with auto-fix)
pnpm lint

# Format code
pnpm format
```

## Architecture & Structure

### High-Level Patterns

- **Feature-based organization**: Views are organized by domain in `src/views/` (products, orders, customers, settings, requests)
- **API layer**: Each domain has its own module in `src/api/` exporting typed interfaces and API functions
- **State management**: Pinia stores (e.g., `auth`, `ui`) using the Composition API `defineStore` pattern
- **Routing**: Vue Router with lazy-loaded route components and global `beforeEach` guard for authentication
- **Layout**: Shell layout with sidebar + topbar for authenticated routes; login page is standalone

### Key Conventions

1. **Axios client** (`src/api/client.ts`):
   - Auto-attaches `Authorization: Bearer` token from localStorage
   - Auto-refreshes tokens on 401 with request queuing during refresh
   - Forces logout if refresh fails

2. **API modules** (`src/api/*.ts`):
   - Export typed `interface`s for requests/responses
   - Export `xxxApi` object with CRUD functions
   - Separate `uploadApi` for file uploads when needed

3. **Pinia stores** (`src/stores/*.ts`):
   - Use `defineStore('name', () => { ... })` Composition API style
   - Return refs/computed directly
   - Auth store syncs to localStorage; has `restore()` session logic

4. **Views** (`src/views/*.vue`):
   - Typically import from `@/api/` and `@/stores/`
   - Use Composition API with refs/computed
   - Inline styles with scoped CSS
   - Bilingual: use `ui.lang` to switch between Turkmen/Russian

5. **Components** (`src/components/`):
   - Reusable UI components
   - `ProductOptionsEditor.vue` for product options editing (used in ProductsView)

6. **Router** (`src/router/index.ts`):
   - Lazy-loaded components with `() => import('@/views/...')`
   - `meta: { public: true }` for public routes (login)
   - Global guard redirects unauthenticated users to login

## Important Files

- `src/main.ts` - App entry point (creates Pinia, router, mounts App.vue)
- `src/App.vue` - Root component with layout shell logic
- `src/router/index.ts` - Route definitions and auth guard
- `src/api/client.ts` - Configured Axios instance with interceptors
- `src/stores/auth.ts` - Authentication state and methods
- `src/stores/ui.ts` - UI state (language, sidebar collapse)
- `src/views/ProductsView.vue` - Example of full-featured CRUD view (reference implementation)
- `src/components/layout/TheSidebar.vue` - Navigation sidebar with badges and collapse
- `src/types/index.ts` - Shared TypeScript types

## Environment

- Requires Node.js `^20.19.0 || >=22.12.0`
- Uses pnpm (lockfile present)
- `.env` file for environment variables (e.g., `VITE_API_BASE`)

## Styling & Design

- Scoped CSS in Vue components
- CSS variables for theming (defined in `src/assets/main.css`)
- Color scheme: Gold accent (`--gold`), dark sidebar, white content area
- Components use rounded corners, shadows, and transitions
- Stock indicators: low stock (<10) in orange, out of stock (0) in red

## API Integration

All API calls go through the `client` in `src/api/client.ts`. The base URL is `import.meta.env.VITE_API_BASE` with fallback to `http://localhost:3001/api/v1`. The API likely returns standard REST responses with `data` payload.

## Testing & Type Safety

- Type checking via `vue-tsc` (run with `pnpm type-check`)
- No test framework configured yet
- ESLint configured with Vue and TypeScript rules
- Oxlint for additional linting

## Common Development Tasks

**Add a new view**: Create component in `src/views/`, add route in `src/router/index.ts` with lazy import.

**Add API calls**: Define interfaces in `src/api/your-module.ts`, export functions using `client.get/post/patch/delete`.

**Update authentication**: Modify `src/stores/auth.ts`; note that token refresh is handled automatically by the client interceptor.

**Change layout**: Edit `src/App.vue` (shell structure) and `src/components/layout/` components.

**Add i18n**: Use `ui.lang` ('tk' or 'ru') and provide bilingual text objects/lookups; see existing patterns in views.

---
> Source: [babon55/chinabox-admin](https://github.com/babon55/chinabox-admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
