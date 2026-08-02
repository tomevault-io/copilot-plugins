## ohbug

> Ohbug is a JavaScript error and behavior monitoring SDK. It's a **pnpm monorepo** that provides framework-specific SDKs for Browser, React, Vue, and Angular, all built on a shared core.

# AGENTS.md

## Project Overview

Ohbug is a JavaScript error and behavior monitoring SDK. It's a **pnpm monorepo** that provides framework-specific SDKs for Browser, React, Vue, and Angular, all built on a shared core.

## Repository Structure

```
ohbug/
├── packages/
│   ├── ohbug-core/       # Core client, event system, extension API, action recording
│   ├── ohbug-browser/    # Browser SDK — error capture, network monitoring, event dispatch
│   ├── ohbug-react/      # React ErrorBoundary integration
│   ├── ohbug-vue/        # Vue errorHandler integration
│   ├── ohbug-angular/    # Angular ErrorHandler provider
│   ├── ohbug-types/      # Shared TypeScript type definitions (source-only, no build)
│   └── ohbug-utils/      # Shared utility functions (validators, DOM helpers, logger)
├── playground/           # React dev playground (localhost:4000)
├── test/                 # Shared test setup
├── vite.config.ts        # Root vite-plus config (lint, test, fmt)
├── tsconfig.json         # Root TypeScript config
└── pnpm-workspace.yaml   # Workspace definition
```

## Tech Stack

- **Language:** TypeScript (strict mode, ES2023 target, ESM-only)
- **Package Manager:** pnpm 10+ with workspaces
- **Build:** [vite-plus](https://github.com/nicepkg/vite-plus) (`vp pack` for libraries, `vp build` for apps)
- **Test:** Vitest with jsdom environment, V8 coverage
- **Lint/Format:** vite-plus integrated tooling (`vp check`, `vp lint`, `vp fmt`)
- **Versioning:** [Changesets](https://github.com/changesets/changesets)
- **Module Resolution:** Bundler mode with `@ohbug/*` path aliases → `./packages/ohbug-*/src`

## Key Commands

| Command        | Description                                  |
| -------------- | -------------------------------------------- |
| `pnpm install` | Install all dependencies                     |
| `pnpm dev`     | Watch mode for all packages                  |
| `pnpm build`   | Build all packages under `packages/`         |
| `pnpm test`    | Run tests with Vitest                        |
| `pnpm check`   | Type-check with auto-fix                     |
| `pnpm lint`    | Lint with auto-fix (type-aware)              |
| `pnpm fmt`     | Format code, sort imports, sort package.json |
| `pnpm version` | Create changeset and version packages        |
| `pnpm publish` | Build and publish via changesets             |

## Architecture & Design Patterns

### Core Concepts

- **Client** (`@ohbug/core`): Central orchestrator. Manages config, extensions, actions, user info, metadata, and event lifecycle (create → hook → notify).
- **Event**: Standardized data structure containing error details, device info, user actions, and metadata. Created via `client.createEvent()`.
- **Extension**: Plugin interface with `onSetup` and `onDestroy` lifecycle hooks. Loaded via `client.use(extension)`. Defined with `defineExtension()`.
- **Action**: Breadcrumb-like user action log with a configurable max length (FIFO).
- **Notifier**: Platform-specific transport (Browser uses `sendBeacon` with XHR fallback).

### Event Types

Defined in `packages/ohbug-core/src/types.ts` as `EventTypes`:

- Error types: `uncaughtError`, `resourceError`, `unhandledrejectionError`, `ajaxError`, `fetchError`, `websocketError`, `unknownError`
- Framework types: `react`, `vue`, `angular`, `miniappError`, etc.
- Other: `message` (manual reports), `feedback`, `view` (PV/UV)

### Browser SDK Architecture

`@ohbug/browser` registers a built-in extension that:

1. **Captures** errors via global event listeners (`error`, `unhandledrejection`) and monkey-patches (`XMLHttpRequest`, `fetch`, `WebSocket`)
2. **Handles** errors through type-specific handlers that normalize error details
3. **Dispatches** events via the core `client.notify()` pipeline
4. **Reports** using `navigator.sendBeacon` (with XHR fallback)

### Framework Integrations

- **React**: `OhbugErrorBoundary` — React class component wrapping `componentDidCatch`
- **Vue**: `install()` — Hooks into `Vue.config.errorHandler`
- **Angular**: `createProvider()` — Returns an Angular `ErrorHandler` provider

## Coding Conventions

- **ESM only** — All packages use `"type": "module"` with `.mjs` output
- **Strict TypeScript** — `strict: true`, `noUnusedLocals`, `noUnusedParameters`, `erasableSyntaxOnly`
- **No side effects** — All library packages declare `"sideEffects": false`
- **Internal properties** — Private/internal properties are prefixed with `__` (e.g., `__client`, `__config`, `__logger`)
- **Bilingual comments** — Core code uses English comments followed by Chinese translations
- **Workspace dependencies** — Inter-package deps use `"workspace:*"`
- **Path aliases** — `@ohbug/*` resolves to `./packages/ohbug-*/src` in development

## Testing

- Tests live in `packages/*/__tests__/*.test.ts` or `packages/*/src/**/*.test.ts`
- Environment: `jsdom`
- Coverage: V8 provider, excludes `index.ts` barrel files and `@ohbug/types`
- Shared setup: `test/setupTests.ts`

## Publishing

1. Run `pnpm version` to create a changeset and bump versions
2. Run `pnpm publish` to build all packages and publish to npm
3. All packages use `"access": "public"` in `publishConfig`
4. Only `dist/` directories are published (except `@ohbug/types` which publishes `src/`)

## Important Notes

- The `@ohbug/types` package is **types-only** — it has no build step and publishes raw `.d.ts` source files
- The playground uses `@ohbug/unplugin` for automatic sourcemap upload and `@ohbug/extension-rrweb` for session replay
- Config validation uses a schema-based approach in `packages/ohbug-core/src/config.ts`
- The project uses `@voidzero-dev/vite-plus-core` (aliased as `vite`) and `@voidzero-dev/vite-plus-test` (aliased as `vitest`) via pnpm overrides

---
> Source: [ohbug-org/ohbug](https://github.com/ohbug-org/ohbug) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
