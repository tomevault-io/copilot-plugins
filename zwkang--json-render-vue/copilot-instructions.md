## json-render-vue

> Provides `tdesignRegistry` - a component registry mapping type names to TDesign components (Card, Button, Input). Uses `catalog.ts` for component metadata.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Vue 3 implementation of the json-render framework - a JSON-driven UI rendering system. The core idea: define UI structure as JSON specs, render them with Vue components.

## Commands

```bash
pnpm install          # Install dependencies
pnpm build            # Build all packages
pnpm test             # Run tests across all packages
pnpm lint             # Lint codebase
pnpm lint:fix         # Auto-fix lint issues
pnpm typecheck        # TypeScript type checking

# Per-package commands (run from package directory)
pnpm dev              # Watch mode build
pnpm test             # Run package tests with vitest
```

## Architecture

### Monorepo Structure

- `packages/json-render-vue` - Core Vue 3 rendering framework
- `packages/tdesign-vue-next` - TDesign Vue Next component bindings

### json-render-vue Core

**Components:**
- `Renderer.vue` - Top-level entry point, wraps JSONUIProvider + ElementRenderer
- `JSONUIProvider.vue` - Provides data store and action config via Vue's provide/inject
- `ElementRenderer.vue` - Recursively renders JSON specs using component registry
- `ConfirmDialog.vue` - Dialog utility for actions

**Composables:**
- `useData` / `provideData` - Shared reactive data store (provide/inject pattern)
- `useDataBinding(path)` - Returns `WritableComputedRef` for v-model binding to nested paths
- `useDataValue(path)` - Read-only computed ref for a data path
- `useActions` - Action handling system
- `useValidation` - Form validation
- `useVisibility` - Conditional element visibility
- `useUIStream` - UI streaming support

**Path Syntax:** Data paths support dot notation and bracket access: `user.name`, `items[0].id`, `data["key"]`

### @zwkang-dev/json-render-tdesign-vue-next Package

Provides `tdesignRegistry` - a component registry mapping type names to TDesign components (Card, Button, Input). Uses `catalog.ts` for component metadata.

### Rendering Flow

1. `Renderer` receives a `spec` (JSON UI definition) and `registry` (component map)
2. `JSONUIProvider` establishes data context via `provideData`
3. `ElementRenderer` looks up component by `spec.type` in registry and renders recursively

## Dependencies

- `@json-render/core` - Core rendering engine (external)
- `@vueuse/core` - Vue composition utilities
- `vue >= 3.3.0` - Peer dependency

## save plan

save the plan into current working directory, into `/docs/{YYYY-MM-DD}_{HHMMSS}-{plan-name}.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ZWkang)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ZWkang)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
