## highway-warning-platform

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Fork of [Vue Vben Admin](https://github.com/vbenjs/vue-vben-admin) v5.7.0, customized as a **highway secondary accident intelligent warning platform** (高速公路二次事故智能预警平台). A pnpm monorepo with Turbo orchestration, using Vue 3, Vite, TypeScript, and Ant Design Vue.

## Essential Commands

```bash
pnpm install          # Install dependencies (requires Node ^20.19 || ^22.18 || ^24)
pnpm dev:antd         # Start dev server for the Ant Design Vue app (port 5666)
pnpm build:antd       # Production build for the Ant Design Vue app
pnpm dev              # Run ALL apps in development mode (via turbo-run)
pnpm lint             # Lint all packages (oxlint + eslint via vsh)
pnpm format           # Auto-format all packages
pnpm check:type       # Type-check across the monorepo
pnpm test:unit        # Run Vitest unit tests
pnpm build            # Build the entire monorepo
pnpm check:circular   # Check for circular dependencies
pnpm changeset        # Create a changeset for versioning
```

Run a single test: `pnpm vitest run --dom path/to/test.test.ts`

## Monorepo Structure

```
apps/
  web-antd/          ← **Primary app** (Ant Design Vue 4, the customized application)
  backend-mock/      Nitro-based mock API server

packages/
  @core/
    base/            Framework-agnostic base utilities and types
    composables/     Shared Vue composables
    preferences/     Preferences/store configuration system
    ui-kit/
      form-ui/       Schema-driven form components
      layout-ui/     Layout components (header, sidebar, tabs, etc.)
      menu-ui/       Menu components
      popup-ui/      Modal and drawer components
      shadcn-ui/     shadcn-style reka-ui components
      tabs-ui/       Tab components
  effects/
    access/          Permission/access control
    common-ui/       Higher-level UI components (form, table, etc.)
    hooks/           Shared composables (sortable, etc.)
    layouts/         Layout definitions (basic, auth)
    plugins/         Framework plugins (motion, etc.)
    request/         HTTP client with interceptors, auth, and error handling
  constants/         Shared constants (routes, etc.)
  icons/             Icon components
  locales/           i18n configuration
  preferences/       App preferences store
  stores/            Global Pinia stores (user, access, etc.)
  styles/            Global styles and Tailwind config
  types/             Shared TypeScript types
  utils/             Shared utilities

internal/
  lint-configs/      ESLint, oxlint, stylelint, commitlint configs
  vite-config/       Shared Vite config factory
  tsconfig/          Shared tsconfig presets
  tailwind-config/   Shared Tailwind config
  node-utils/        Node.js build utilities
```

## Custom Application Architecture (`apps/web-antd/src/`)

**Initialization flow:** `main.ts` → init preferences → `bootstrap.ts` → adapter setup → i18n → Pinia → router + guards → app mount

```
src/
  adapter/           Bridges Vben framework components to Ant Design Vue
    component/       Registers all form/UI component mappings (Input, Select, Upload, etc.)
  api/               API layer
    core/            Auth, user, menu API endpoints
    request.ts       HTTP client with token refresh, error handling (expects {code: 0, data: T})
  router/
    guard.ts         Navigation guards: common (progress bar) + access (auth/permission)
    routes/modules/  Route definitions — *.ts files are auto-discovered via glob import
      warning.ts     /warning → 预警监控台
      monitor.ts     /monitor → 系统监控看板
  store/
    auth.ts          Auth store: login, logout, fetchUserInfo
  views/
    warning/         Leaflet map dashboard with MQTT-sourced accident markers, coordinate transform,
                     warning animations, virtual rear vehicles, and AI analysis panel
    monitor/         ECharts dashboards: delay trends, hourly accident distribution, edge node status
    dashboard/       Standard Vben analytics & workspace pages
    _core/           Framework pages (login, profile, fallback, etc.)
```

## Key Customizations from Upstream

- **Views**: Custom `warning/` (map-based monitoring with Leaflet, MQTT, AI streaming analysis) and `monitor/` (ECharts system dashboards) replace/extend upstream pages
- **App title**: Overridden to "高速公路二次事故智能预警平台" in `.env.development`
- **MQTT integration**: `mqtt` package + `VITE_MQTT_BROKER` env var for real-time accident data
- **ECharts**: Full import (`import * as echarts from 'echarts'`) — tree-shaking caused blank pages on partial imports
- **State**: Custom composable pattern (`useMqtt`, `useMockData`, `useMonitorData`, etc.) in per-view `composables/` directories
- **No external/static routes**: Only core routes + dynamically glob-imported module routes are used

## Environment Variables

| Variable              | Description                            |
| --------------------- | -------------------------------------- |
| `VITE_APP_TITLE`      | Application display title              |
| `VITE_APP_NAMESPACE`  | Prefix for cache/store keys            |
| `VITE_PORT`           | Dev server port (5666)                 |
| `VITE_BASE`           | Base URL path                          |
| `VITE_GLOB_API_URL`   | API base URL (proxied to backend-mock) |
| `VITE_NITRO_MOCK`     | Enable Nitro mock server               |
| `VITE_MQTT_BROKER`    | MQTT broker WebSocket URL              |
| `VITE_ROUTER_HISTORY` | `hash` or `history` routing mode       |

There are two `.env.development` files:

- `apps/web-antd/.env.development` — full config (checked into git)
- `.env.development` (repo root) — minimal overrides (untracked, gitignored)

## Commit Convention

Angular-style: `type(scope): subject` where type ∈ `feat|fix|docs|style|refactor|perf|test|workflow|build|ci|chore|types|wip`. `feat`, `fix`, `perf` appear in changelog. Use `czg` for interactive commits.

## API Response Format

All API responses follow `{ code: 0, data: T }` where `code: 0` indicates success. The request client is configured in `apps/web-antd/src/api/request.ts` with automatic Bearer token injection, token refresh, and error notification via Ant Design `message`.

---
> Source: [whao91211-dotcom/highway-warning-platform](https://github.com/whao91211-dotcom/highway-warning-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
