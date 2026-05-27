## flexplorer

> **Flexplorer** is an Obsidian plugin that enhances the native file explorer with custom sorting, pinning, and hiding features.

# AGENTS.md

**Flexplorer** is an Obsidian plugin that enhances the native file explorer with custom sorting, pinning, and hiding features.

## Stack
- TypeScript v6
- React v19
- Voicss (CSS-in-TS)
- tsdown (build)
- Bun (package management)

## Architecture
- core layer (`src/core/`): DnD engine, order management, explorer observation, and patching logic
- UI layer (`src/ui/`): React components and Obsidian views

## Scripts
- `bun dev` — development build
- `bun run build` — production build
- `bun typecheck` — type checking
- `bun eslint` — linting

## Guidelines
- Run `bun typecheck` and `bun eslint` after making changes

---
> Source: [kh4f/flexplorer](https://github.com/kh4f/flexplorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
