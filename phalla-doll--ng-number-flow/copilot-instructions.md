## ng-number-flow

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A pnpm workspace publishing **`ng-number-flow`** — an Angular 19+ standalone wrapper around the framework-agnostic [`number-flow`](https://github.com/barvian/number-flow) custom element. The library is a thin adapter: it does not reimplement animation, it drives `number-flow/lite`'s web component from Angular signals.

- `packages/ng-number-flow/` — the publishable library (built with ng-packagr).
- `apps/demo/` — a zoneless standalone Angular app used to develop/exercise the library.
- `docs/` — feasibility investigation, as-built notes, and the npm publishing plan.

## Commands

Run from the repo root unless noted. Toolchain: pnpm 10, Node ≥18.19, Angular 22 devdeps (library targets `@angular/core >=19`).

```bash
pnpm install

# Library
pnpm --filter ng-number-flow build       # ng-packagr → packages/ng-number-flow/dist
pnpm --filter ng-number-flow dev          # ng-packagr --watch
pnpm --filter ng-number-flow test         # ng test (Vitest runner, jsdom)
pnpm --filter ng-number-flow typecheck    # tsc --noEmit against tsconfig.lib.json

# Demo app (imports the library from source — no rebuild needed)
pnpm --filter demo start                  # ng serve, http://localhost:4200

# Whole workspace (recursive)
pnpm build | pnpm test | pnpm typecheck   # runs the script in every package
pnpm format                               # prettier --write .
```

Tests use the `@angular/build:unit-test` builder with the **Vitest** runner (not Karma/Jasmine), though specs are written with the Jasmine-style `describe`/`it`/`expect` globals. Run a single spec file with `pnpm --filter ng-number-flow test <path-or-name-pattern>`.

## Source-linking (why demo edits hot-reload the library)

`apps/demo/tsconfig.app.json` maps `"ng-number-flow"` directly to `packages/ng-number-flow/src/public-api.ts` via `paths`, and `include`s the library source. So the demo compiles the library from **source**, not from `dist` — edit a `.ts` under `packages/ng-number-flow/src` and the running dev server picks it up without `pnpm build`. `dist` is only needed for publishing and for `pnpm typecheck`.

The demo's `package.json` still lists `ng-number-flow: workspace:*`, but the tsconfig path mapping is what the build actually resolves during development.

## Architecture

The library is four small pieces plus the wrapper component. Understanding the data flow across them is the key to being productive here.

**`number-flow.element.ts`** — registers the custom element `number-flow-ng` via `define('number-flow-ng', NumberFlowLite)`. This is a **side-effecting import**; the package marks `**/number-flow.element.ts` in `sideEffects` so bundlers don't tree-shake the registration away. The component imports this file for its effect.

**`number-flow.component.ts`** (`<number-flow>`) — the public wrapper. Fully signal-based, `OnPush`, `standalone`, zoneless-compatible, `host: { style: 'display: contents' }`. Its template is a single `<number-flow-ng #flow>`. All configuration comes in as `input()` signals. The bridge to the custom element happens in the constructor via **two client-only render hooks** — critical because the custom element only exists in the browser:
  - `afterNextRender(...)` — one-time wiring: attaches `animationsstart`/`animationsfinish` listeners (re-emitted as the `animationsStart`/`animationsFinish` outputs), registers with a group coordinator if present, and cleans all of it up on `DestroyRef`.
  - `afterRenderEffect({ write })` — reactive sync: on any input change, writes props onto the element (`applyProps`) and pushes formatted data (`applyData`). DOM property assignment must not run during SSR, hence `afterRenderEffect` rather than a plain `effect`.
  - Inputs default via `NumberFlowLite.defaultProps` (`?? d.x`) rather than hardcoded values, so defaults track the upstream library.

**`formatter-cache.ts`** — `getFormatter(locales, format)` memoizes `Intl.NumberFormat` instances keyed by `JSON.stringify` of the args. Constructing `Intl.NumberFormat` is expensive; every `applyData` call goes through this cache. Data is produced by `number-flow`'s `formatToData(value, formatter, prefix, suffix)`.

**Group synchronization** (`number-flow-group.directive.ts` + `number-flow.tokens.ts`) — `[numberFlowGroup]` provides a `Coordinator` (implementing `NumberFlowGroupCoordinator`) through the `NUMBER_FLOW_GROUP` injection token. Each child component injects the token `{ optional: true }`, registers its element, and sets `el.batched = !!group && !isolate()`. On value changes inside a group, the component brackets the write with `group.beginUpdate()` / `group.endUpdate()`, which fan `willUpdate()`/`didUpdate()` across all registered elements so their animations start on the same frame. `endUpdate()` defers the `didUpdate` flush to a `queueMicrotask` so multiple sibling updates in one tick coalesce. A child can opt out of batching with `[isolate]="true"`.

Data flow summary: `input signals → afterRenderEffect → applyProps + applyData(getFormatter → formatToData) → number-flow-ng custom element`, with the group coordinator bracketing writes when batching is on.

## Public API surface

Exports flow through `src/public-api.ts` (and `src/index.ts` re-exports it). Public entry points: `NumberFlowComponent`, `NumberFlowGroupDirective`, `NUMBER_FLOW_GROUP` + `NumberFlowGroupCoordinator`, and re-exported upstream types from `types.ts` (`Value`, `Format`, `Data`, `Trend`, `Digits`, `Props`, `Plugin`, `NumberFlowElement`). When adding a public symbol, export it from `public-api.ts` — that file is the ng-packagr `entryFile`.

## Conventions

- **Signals-only, zoneless.** Use `input()`/`output()`/`viewChild.required()` and the `afterRender*` family. Do not add `@Input`/`@Output` decorators, `ngOnInit`, or anything assuming Zone.js — the component and demo are designed to run zoneless.
- The library uses **tabs** for indentation; the demo app uses **spaces** (see `.editorconfig`/`.prettierrc` under `apps/demo`). Match the file you are editing; `pnpm format` runs Prettier over both.
- Import from `number-flow/lite` (not the full `number-flow`) inside the library to keep the bundle small; `number-flow` is the single `allowedNonPeerDependencies` entry in `ng-package.json`.
- Keep DOM/custom-element writes inside render hooks, never in field initializers or plain effects — SSR safety depends on it.

## Publishing

Published at <https://www.npmjs.com/package/ng-number-flow> (`0.2.0`). The library builds to `packages/ng-number-flow/dist` and publishes that directory as-is (no `files` field — ng-packagr would copy it into `dist/package.json`, where `["dist", ...]` matches nothing and drops the bundle; `publishConfig.access: "public"`). `tsconfig.lib.json` sets `angularCompilerOptions.compilationMode: "partial"` (required — full mode is rejected by the generated `prepublishOnly` guard). Releases run via `.github/workflows/publish.yml` on `v*` tag push, using an `NPM_TOKEN` secret that **must be a Bypass-2FA token** (the account has 2FA disabled). See `docs/publishing-to-npm.md` for the full procedure. The package README (`packages/ng-number-flow/README.md`) is the full API reference; the root README is the overview.

## Reference material

`.agents/skills/angular-developer/` bundles Angular best-practice references (signals, DI, testing, SSR, ARIA, etc.). Consult it for idiomatic modern-Angular patterns when extending the library or demo.

---
> Source: [phalla-doll/ng-number-flow](https://github.com/phalla-doll/ng-number-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
