## lost-the-plot

> When an Edit fails due to whitespace mismatch, do NOT waste turns diagnosing it (e.g., no hexdump). Re-read and copy indentation exactly. `bun run format` will fix it later.

# StagePlotter Developer Guide

## Editing Files: Read Tool Indentation
When an Edit fails due to whitespace mismatch, do NOT waste turns diagnosing it (e.g., no hexdump). Re-read and copy indentation exactly. `bun run format` will fix it later.

## Build & Test Commands
- **Install**: `bun install`
- **Dev (PWA)**: `bun run dev`
- **Dev (Marketing)**: `bun run dev:marketing`
- **Dev (Desktop)**: `bun run dev:desktop` (Requires Rust/Tauri)
- **Build (PWA)**: `bun run build`
- **Build (Marketing)**: `bun run build:marketing`
- **Build (Desktop)**: `bun run build:desktop`
- **Check/Lint**: `bun run check` (TS), `bun run lint` (ESLint/Prettier), `bun run format`
- **Test**: `bun run test` (unit + E2E in `app/`)

## Settings & Preferences
- **Preferences Singleton**: `app/src/lib/state/preferences.svelte.ts` (Svelte 5 class).
- **Time Format**: `preferences.timeFormat` ('12h' | '24h'), localStorage key: `stageplotter-time-format`.
- **Dark Mode**: Managed by `mode-watcher`. Use `mode.current`, `toggleMode()`.
- **Time Utility**: `app/src/lib/utils/time.ts` -> `formatTimeMs(ms, use24h?)`.

## Engineering Mandates
- **Svelte 5**: Use Runes ($state, $derived, $effect, $props).
- **Styling**: Tailwind CSS 4.
- **Persistence**: SQLite (OPFS in browser, Tauri plugin on desktop).
- **Whitespaces**: When an Edit fails due to whitespace mismatch, do NOT waste turns diagnosing. Re-read and copy indentation exactly. `bun run format` will fix it later.

## Svelte MCP Tools
- Use `list-sections` first to discover documentation.
- Use `get-documentation` for full content.
- **MANDATORY**: Use `svelte-autofixer` on all Svelte code before completion.
- Ask user before generating `playground-link`.

---
> Source: [cdslipp/lost-the-plot](https://github.com/cdslipp/lost-the-plot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
