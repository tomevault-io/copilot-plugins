## randjourney

> **Think in English, output in Japanese.**

# Agent Instructions

**Think in English, output in Japanese.**

## OVERVIEW
RandJourney: React 19 + TypeScript + Vite project for random point generation using Leaflet.
Package Manager: Bun v1.3.6.

## STRUCTURE
- `src/components/`: React components (15 total) with CSS Modules.
- `src/hooks/`: Custom hooks (useDartThrow, useHistory, useRadius, useUrlSharing).
- `src/lib/`: Utility modules (geo, geocoding, muni, random-point, share, urls).
- `src/test/`: Vitest + jsdom setup.
- `src/styles/`: CSS tokens and global styles.
- `src/data/`: `muni.json` (municipality data).

## WHERE TO LOOK
- Entry: `index.html` -> `src/main.tsx` -> `src/App.tsx`.
- Logic: `src/lib/` for core geo/random algorithms.
- State: `src/hooks/` for domain-specific state management.

## CONVENTIONS
- Styles: `ComponentName.module.css` paired with `ComponentName.tsx`.
- CSS: Use CSS custom properties from `src/styles/tokens.css`.
- Exports: Named exports for hooks and utilities.
- Tests: Co-located as `*.test.ts` alongside source files.

## ANTI-PATTERNS
- None documented.

## COMMANDS
- Dev: `bun run dev`
- Build: `bun run build` (Type check + Vite build)
- Test: `bun run test` (Vitest)
- Test CI: `bun run test:ci`

## TOOLS SELECTION (PRESERVE)
When you need to call tools from the shell, use this guide:

- Exclude bulky folders: `.git`, `node_modules`, `coverage`, `out`, `dist`, `.venv`.
- Find files by name: `fd`.
- Find text: `rg` (ripgrep).
- Find Code Structure: `ast-grep`.
  - `.ts` -> `ast-grep --lang ts -p '<pattern>'`
  - `.tsx` -> `ast-grep --lang tsx -p '<pattern>'`
  - TypeScript actions:
    - Prefer `ast-grep` over `rg` for structured code search.
    - Exported interfaces: `ast-grep --lang ts -p 'export interface $I { ... }'`.
    - Function calls: `ast-grep --lang ts -p 'axios.get($URL, $$REST)'`.
- JSON: `jq`.

## ENVIRONMENT
- bun (always use `bun install --frozen-lockfile`)
- uv (python packages management)

## NOTICE
- Do not modify `package.json`/lockfiles without explicit user approval.
- Do not chain `cd` commands with `&&`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yuzolabs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
