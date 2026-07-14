## engine

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

NextReport Engine is a schema-driven, headless reporting engine with a visual designer and runtime viewer. It's a pnpm monorepo orchestrated by Turborepo. The JSON schema DSL is the contract between all systems — designer, engine, and renderers never directly depend on each other.

## Commands

```bash
pnpm install              # Install all dependencies
pnpm run dev              # Start all packages in dev/watch mode (Next.js on :3000)
pnpm run build            # Production build (all packages)
pnpm test                 # Run all tests across packages
pnpm run lint             # ESLint with boundary enforcement rules
pnpm run typecheck        # TypeScript type checking
pnpm run format           # Prettier format all files
pnpm run format:check     # Check formatting without writing

# Single-package commands
pnpm --filter @nextreport/engine-core test          # Test one package
pnpm --filter @nextreport/schema test:watch         # Watch mode for one package
pnpm --filter @nextreport/renderer-html test:coverage  # Coverage for one package

# Run a single test file
pnpm --filter @nextreport/engine-core exec vitest run src/__tests__/expression.test.ts
```

## Architecture

**Monorepo layout:** `packages/*` (7 internal + 1 publishable) and `apps/web` (Next.js).

**Dependency graph (strict, ESLint-enforced):**

```
schema (Zod, zero deps) ← engine-core ← renderer-html ← renderer-pdf
                          ↑                               ↑
schema ← canvas (ports + Konva adapter)                   │
                          ↑                               │
         ui-designer (React + Zustand) ──────────────────┘
         ui-viewer   (React) ← renderer-html
```

**Publishable package:** `@nextreport/engine` (packages/nextreport-engine) bundles schema + engine-core + renderers via tsup into ESM/CJS. All other packages are `"private": true`.

### Core Pipeline (engine-core)

The render pipeline is 4 internal modules that are **not public API**. Only `renderReport` and `validateReport` are exported.

1. **pipeline/** — Orchestrator. Calls band-processor then layout-calculator.
2. **expression/** — Tokenizer → parser → AST → evaluator. Function registry pattern with side-effect imports. Handles type coercion for cross-language data (strings → numbers).
3. **band-processor/** — Iterates detail bands over data arrays, resolves expressions per component. Derives iterator name from dataBinding (e.g., `"items"` → `item`).
4. **layout/** — Absolute positioning, page dimensions (A4/A3/Letter/Legal), margin application, pagination with page breaks.

Output is an IR (`RenderResult` with `Page[]` of `ResolvedElement[]`) consumed by renderers.

### Canvas Abstraction (canvas)

Port/adapter pattern: `ports/` defines framework-agnostic interfaces (`CanvasEnginePort`, `CanvasObjectPort`, `CanvasEventsPort`), `adapters/konva/` implements them. Konva is imported **only** inside `packages/canvas/src/adapters/konva/`.

### Designer State (ui-designer)

Zustand + Immer store with 5 slices: Document, Data, Selection, Viewport, History.

## Boundary Rules (ESLint-enforced)

These are enforced via per-package `eslint.config.js` restricted-imports rules:

- **schema** — imports nothing from `@nextreport/*`
- **engine-core, renderer-html, renderer-pdf** — no React, no Next.js, no Konva, no `@nextreport/canvas`, no `@nextreport/ui-*`. These must remain framework-agnostic.
- **canvas** — depends only on schema (for types)
- Konva imports only in `packages/canvas/src/adapters/konva/`
- No package imports from `apps/web`

## Conventions

- **ESM everywhere** — use `.js` extensions in all TypeScript import paths
- **`type` keyword** — `@typescript-eslint/consistent-type-imports` is enforced (`import type { ... }`)
- **TypeScript target** — ES2025, strict mode, `verbatimModuleSyntax`
- **Prettier** — no semicolons, single quotes, trailing commas, 100 char width
- **Pre-commit hook** — Husky runs lint-staged (eslint --fix + prettier) on `*.{ts,tsx}` files
- **Node >= 24, pnpm 10.33**

## Testing

Vitest with workspace config. Tests live in `src/__tests__/` within each package. Coverage via V8 provider, reports in `./coverage/` per package.

Packages with tests: schema, engine-core, renderer-html, renderer-pdf, ui-designer.

## API Routes (apps/web)

- `POST /api/render` — Schema + data → HTML or PDF (set `"format": "pdf"`)
- `POST /api/validate` — Schema validation only
- `POST /api/preview` — First-page-only HTML render
- `GET /api/templates/:id` — Load template + sample data

## Docker

Multi-stage Dockerfile (deps → build → runner) using `node:24-slim`. Runner stage installs system Chromium for Puppeteer PDF rendering. Runs as non-root `nextreport` user on port 3000.

```bash
docker build -t nextreport-engine .
docker run -p 3000:3000 nextreport-engine
```

---
> Source: [nextreport/engine](https://github.com/nextreport/engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
