## link-sync-action

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

GitHub Action that declaratively syncs short links from a `shortio.yaml` config file to Short.io. The YAML file is the source of truth — the action creates, updates, and deletes links to match the config. Only links tagged `github-action-managed` are eligible for deletion, protecting manually-created links.

## Commands

```bash
npm test                # Run tests (Vitest, single run)
npm run test:watch      # Run tests in watch mode
npm run typecheck       # TypeScript type checking (tsc --noEmit)
npm run build           # Bundle with esbuild → dist/index.js
```

To run a single test file: `npx vitest run src/config.test.ts`

The `dist/index.js` bundle is checked into the repo — run `npm run build` after changes and commit the output.

## Architecture

**Dataflow:** `shortio.yaml` → `parseConfig()` → `computeDiff()` → `executeSync()` → GitHub Action outputs

Three core modules in `src/`:

- **`config.ts`** — Parses and validates YAML config. Supports multi-domain via YAML document streams (`---` separator). Throws `ConfigError` on invalid input.
- **`sync.ts`** — Core logic. Calls `@short.io/client-node` SDK directly with module-level helpers for domain ID resolution (cached) and paginated link fetching. `computeDiff()` compares YAML state vs API state to produce `LinkDiff` (toCreate/toUpdate/toDelete). `executeSync()` applies the diff with dry-run support. Errors are collected without halting execution.
- **`types.ts`** — Shared type definitions. `MANAGED_TAG` constant. Helper functions `getLinkKey()` (domain+path composite key) and `getLinksArray()`.
- **`index.ts`** — Entry point. Reads GitHub Action inputs via `@actions/core`, orchestrates the pipeline, sets outputs.

## Testing

Tests are colocated with source files (`*.test.ts`). Each module has its own test file. External dependencies (`@short.io/client-node`, `@actions/core`) are mocked using `vi.mock()`. SDK functions are mocked directly in sync tests.

## Key Design Decisions

- Links are keyed by `domain + path` (see `getLinkKey()` in `types.ts`)
- The action only deletes links that have the `github-action-managed` tag — all created/updated links get this tag automatically
- `executeSync()` continues processing after individual operation failures, collecting errors in the result
- Multi-domain configs use YAML document streams (multiple `---`-separated documents), not nested YAML
- Build target is Node.js 20 (ES2022), strict TypeScript

---
> Source: [Short-io/link-sync-action](https://github.com/Short-io/link-sync-action) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
