## vectra-icons-figma-plugin

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run build       # one-shot build
npm run watch       # rebuild on file changes
npm run format      # run prettier
```

After any build, reload the plugin in Figma to pick up changes.

## Architecture

This is a Figma plugin with two sandboxed environments that communicate only via `postMessage`:

**Plugin main thread** (`src/code.ts` → `code.js`)
- Runs in Figma's sandboxed JS VM — no DOM, no `window`, no `fetch`
- Has full access to the Figma API (`figma.*`)
- Handles `create-icon` messages from the UI to insert SVG nodes
- Handles drag-and-drop via `figma.on("drop", ...)`
- Built with esbuild as `platform: 'browser'`, `format: 'iife'` — must NOT use `platform: 'node'` (breaks in Figma's sandbox)

**Plugin UI** (`src/ui.ts` + `ui.template.html` → `ui.html`)
- Runs in a sandboxed iframe — has DOM and `fetch`, but no Figma API
- Fetches icon metadata and SVGs from `raw.githubusercontent.com`
- Uses Fuse.js for fuzzy search
- Communicates with the main thread via `window.parent.postMessage`

**Build system** (`build.mjs`)
- Builds `src/code.ts` → `code.js` (IIFE, browser platform)
- Builds `src/ui.ts` → inlines the bundle into `ui.template.html` at the `<!-- BUILD:SCRIPT -->` marker → outputs `ui.html`
- Never edit `ui.html` directly; edit `ui.template.html` for markup/styles and `src/ui.ts` for logic

## Key data flow

1. On load, UI fetches `icons-metadata.json` from GitHub, then fetches all SVGs for the selected style (Stroke/Fill) in parallel
2. SVG responses are cached in `styleCache: Map<style, SearchableIcon[]>` — switching styles doesn't re-fetch
3. Icons are recolored at render time by setting `fill="var(--figma-color-text)"` on all SVG shape elements (selector: `path, circle, rect, ellipse, line, polyline, polygon`)
4. Clicking an icon posts `{ type: 'create-icon', svg, name }` to the main thread
5. Main thread inserts the node, calls `lockAspectRatio()`, resizes to 64×64 on FigJam, and positions it relative to the current selection or viewport center

## Icon placement logic (`src/code.ts`)

- If the last appended icon is still selected → place next to it (inside the same frame if applicable)
- If something else is selected → place next to that node (inside its parent frame if applicable)
- If nothing is selected → place at viewport center on the page canvas

`appendToParentOrPage` handles frame-aware placement (checks if reference node's parent is a FRAME). `appendToPage` places directly on the canvas with no reference.

## Network access

Allowed domains are declared in `manifest.json` under `networkAccess.allowedDomains`. Any new fetch target must be added there or it will be blocked by Figma's CSP.

## Icon version

The UI version string (`v1.6.2`) is hardcoded in `src/ui.ts` in the `DOMContentLoaded` handler. Update it manually when releasing a new version.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/abhishekshankr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
