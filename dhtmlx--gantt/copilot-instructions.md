## gantt

> Guidance for AI coding agents working in this repository.

# AGENTS.md

Guidance for AI coding agents working in this repository.

## What this is

DHTMLX Gantt, MIT edition — an open source JavaScript Gantt chart library. The repository ships library sources, samples, and a smoke test suite. It is generated from an internal source tree on each release (see CONTRIBUTING.md), so structural refactors should stay minimal and focused.

## Layout

| Path | Content |
|---|---|
| `src/dhtmlxgantt.ts` | library entry point |
| `src/core/` | core: datastores, task data layer, UI (grid, timeline, lightbox), worktime |
| `src/core/plugins/` | always-on plugins (batch update, task types, formatters…) |
| `src/ext/` | opt-in extensions activated via `gantt.plugins({...})` (tooltip, quick_info, keyboard_navigation…) |
| `src/css/` | LESS styles, fonts, skins |
| `src/locale/` | locales |
| `samples/` | runnable demos; served by `npm run start`, and the corpus the smoke test loads |
| `scripts/` | build/dev/test node scripts, sample-data backend |
| `codebase/` | build output (generated; never edit by hand) |

## Commands

```bash
npm install
npm run build          # vite build -> codebase/ + d.ts
npm run start          # watch build + backend (:9200) + vite devserver
npm run test           # build + smoke test: every sample loads without console errors
npm run test 07_grid   # smoke just one samples folder
npm run lint           # eslint over src/ (flat config, intentionally lax)
```

## Working on the code

- The build must stay green: `npm run build` after any `src/` change, and `npm run lint` should pass.
- `npm run test` is a smoke pass: it loads every sample headless and fails on a page error or `console.error`. It is the regression net — keep it green after `src/` changes.
- Sample pages reference `../../codebase/dhtmlxgantt.js` — run a build before opening them.
- `src/types.d.ts` declares the compile-time globals (`VERSION`, `LICENSE`, …) injected by vite `define`.
- Some modules are intentional basic implementations of Pro features (e.g. `src/core/ui/render/dummy_layer.js`, re-export shims like `src/core/ui/grid/resource_grid.js`, and `src/core/worktime/strategy/calendar_strategy.js` which re-exports the disabled `no_work_time` strategy — working-time calendars are out of scope). Do not "complete" them — the Pro feature set is out of scope for this edition (see README).

## Conventions

- Match the style of the file you edit (the codebase mixes older ES5-style JS and newer TS — both are fine in their places).
- Tabs for indentation.
- No new runtime dependencies — the library bundle must stay dependency-free.

---
> Source: [DHTMLX/gantt](https://github.com/DHTMLX/gantt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
