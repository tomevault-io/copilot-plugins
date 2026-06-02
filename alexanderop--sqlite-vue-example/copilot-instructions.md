## sqlite-vue-example

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Vue 3 SQLite Playground - A demonstration application that runs SQLite entirely in the browser using WebAssembly (WASM). The database persists using the Origin Private File System (OPFS) and runs in a Web Worker for non-blocking operations.

## Development Commands

### Essential Commands

- `npm run dev` - Start development server (localhost:5173)
- `npm run build` - Full production build (runs type-check + build-only)
- `npm run preview` - Preview production build locally

### Code Quality

- `npm run lint` - Run ESLint with auto-fix enabled (uses @antfu/eslint-config)
- `npm run type-check` - Run TypeScript compiler without emitting files

### Testing

- `npm run test:unit` - Run Vitest unit tests in watch mode
- `npm run test:e2e` - Run Playwright end-to-end tests across browsers (chromium, firefox, webkit)

## Architecture

### SQLite Integration

The core SQLite integration uses the official `@sqlite.org/sqlite-wasm` package. Key architectural points:

1. **Web Worker Pattern**: SQLite runs via `sqlite3Worker1Promiser`, which provides a promise-based API to a Web Worker
2. **OPFS Storage**: Database persists using the filename format `file:mydb.sqlite3?vfs=opfs`
3. **Singleton Pattern**: `useSQLite.ts:21` uses a module-level `isInitialized` ref to ensure single database instance across all component uses
4. **Lazy Initialization**: Database initializes on first query if not already initialized (`useSQLite.ts:84`)

### Critical Vite Configuration

The Vite config at `vite.config.ts:20-23` sets required headers for SharedArrayBuffer support:

- `Cross-Origin-Opener-Policy: same-origin`
- `Cross-Origin-Embedder-Policy: require-corp`

These headers are **required** for SQLite WASM to function. Without them, SharedArrayBuffer will be unavailable and the database will fail to initialize.

Also note: `@sqlite.org/sqlite-wasm` is excluded from Vite's dependency optimization (`vite.config.ts:26`) to prevent bundling issues.

### Router Configuration

Uses `unplugin-vue-router` for file-based routing. Routes are auto-generated from files in `src/pages/`. The router imports from `vue-router/auto-routes` (`router/index.ts:2`).

### Project Structure

```
src/
├── composables/     # Reusable composition functions
│   └── useSQLite.ts # Main SQLite integration composable
├── config/          # Configuration files
│   └── database.ts  # Database schema and config
├── components/      # Vue components
├── pages/           # File-based routes (auto-generated)
├── router/          # Router setup
├── types/           # TypeScript type definitions
└── utils/           # Utility functions
```

### State Management

Pinia is installed and configured (`main.ts:12`) but not currently used for SQLite state. Database state is managed through the `useSQLite` composable which maintains refs for `isLoading`, `error`, and `isInitialized`.

## Working with the Database

### Adding New Tables

1. Update `src/config/database.ts` with new table schema
2. Add schema to initialization in `useSQLite.ts:64-67`
3. Create table-specific composables or extend `useSQLite` for CRUD operations

### Query Execution

The `executeQuery` function (`useSQLite.ts:83`) accepts SQL and optional bind parameters. It returns the raw worker response which includes `result` and `type` fields. Always check for `type === 'error'` when handling responses.

## Browser Requirements

This application requires modern browser features:

- WebAssembly support
- Web Workers
- Origin Private File System (OPFS)
- SharedArrayBuffer (enabled via COOP/COEP headers)

Safari, Chrome, and Firefox are supported (see Playwright config for tested browsers).

## Build Configuration

The base path changes depending on build mode (`vite.config.ts:18`):

- Development: `/`
- Production: `/starter-vue/`

Adjust this if deploying to a different base path.

---
> Source: [alexanderop/sqlite-vue-example](https://github.com/alexanderop/sqlite-vue-example) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
