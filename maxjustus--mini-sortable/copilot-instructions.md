## mini-sortable

> mini-sortable: vanilla JS drag-to-reorder library with Alpine.js bindings. Placeholder + FLIP animation approach. Supports lists, grids, variable heights, cross-container transfer.

# CLAUDE.md

## Project

mini-sortable: vanilla JS drag-to-reorder library with Alpine.js bindings. Placeholder + FLIP animation approach. Supports lists, grids, variable heights, cross-container transfer.

## Files

- `sortable.js` -- vanilla drag engine (sortable class)
- `alpine-sortable.js` -- thin Alpine.js directive wrapper
- `test.html` -- test page with 7 scenarios (serve with `make serve`, open http://localhost:3813/test.html)

## Commands

- `make check` -- run TypeScript type checking (strict mode, JSDoc types, no emit)
- `make serve` -- start local dev server on port 3813

## Type checking

Uses JSDoc annotations with `tsconfig.json` (`strict: true`, `checkJs: true`). Run `make check` before committing. Fix all type errors -- do not use `@ts-ignore` or weaken the tsconfig.

## Architecture

- `sortable` class manages the full drag lifecycle: pointer tracking, placeholder insertion, FLIP animation, auto-scroll, cross-container transfer
- Alpine wrapper maps `x-sortable`, `x-sortable`, `x-sortable-handle` directives to data attributes consumed by the sortable class
- `meta` property on sortable instances carries framework-specific data (e.g. Alpine splice helpers)
- Groups (`opts.group`) enable cross-container drag via a module-level `Map<string, Set<sortable>>`

## Conventions

- No build step -- plain ES modules, served directly
- No `_` prefix on methods -- just descriptive names
- Safari workaround: `repaintContainer` toggles `will-change` to fix hit-test desync after scroll + transform

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/maxjustus)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/maxjustus)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
