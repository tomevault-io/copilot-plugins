## shaveen-balasooriya-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Portfolio site for Shaveen Balasooriya ("The Illustrated Architect"). Built with SvelteKit + TypeScript. The design system is documented in `DESIGN.md` — all visual decisions must conform to it. Use `/design` to load the full design system rules when working on UI.

## Commands

```bash
npm run dev        # start dev server
npm run build      # production build
npm run preview    # preview production build
npm run check      # svelte-check type checking
npm run check:watch
```

## Architecture

Standard SvelteKit minimal scaffold:
- `src/routes/` — file-based routing; `+page.svelte` files are pages, `+layout.svelte` for shared layout
- `src/lib/` — shared components and utilities (import via `$lib/`)
- `static/` — served as-is at root
- `svelte.config.js` — adapter config (currently `adapter-auto`)
- `vite.config.ts` — Vite config + `@tailwindcss/vite` plugin

The reference prototype lives in `code.html` (a standalone Tailwind CDN file). Use it as the visual/structural reference when building out Svelte components, but do not replicate its CDN-based approach — install packages properly.

## Tailwind CSS v4

This project uses **Tailwind CSS v4** with the Vite plugin (`@tailwindcss/vite`). Key differences from v3:
- **No `tailwind.config.ts` or `postcss.config.js`** — all configuration lives in `src/app.css`
- Theme tokens are defined in a `@theme { }` block using CSS custom properties (e.g. `--color-ink: #1C1409`)
- Custom utilities use `@utility name { ... }` instead of `@layer utilities`
- Content scanning is automatic; external packages use `@source "path"` directives
- Plugins are loaded via `@plugin "package-name"` in CSS

## Flowbite Svelte

[flowbite-svelte](https://flowbite-svelte.com/) is installed as a UI component library. Use it for common interactive components (modals, drawers, dropdowns, tabs, tooltips, etc.) instead of building from scratch. Import components from `flowbite-svelte`:

```svelte
<script>
  import { Modal, Button, Drawer } from 'flowbite-svelte';
</script>
```

**Important:** Flowbite-svelte components come with their own Tailwind classes. Always override or wrap them with our design system tokens (ink borders, lithograph shadows, design colors) to maintain visual consistency. Do not use flowbite's default blue/gray theme as-is.

## Design System: "The Digital Lithograph"

Key rules from `DESIGN.md` (run `/design` for the full reference):

**Colors** — use only the named design tokens (e.g. `ink`, `sand`, `parchment`, `terracotta`, `sky-blue`). Never use `#000000` or standard Tailwind grays. `ink` (`#1C1409`) is for all outlines, hard shadows, and body text.

**Shadows** — hard offset only: `6px 6px 0px 0px #1C1409`. No blur, no soft shadows.

**Borders** — 2–3px solid `ink` on every distinct container. Max border-radius 4px for containers (`rounded-lg`). Pills/tags may use `rounded-full`.

**Buttons (CTAs)** — `terracotta` fill, 2px `ink` border, hard shadow. Hover: shift `-2px/-2px` to simulate a press. Text: Space Grotesk Bold, all-caps.

**Sectioning** — no `<hr>` or 1px dividers. Use hard background color shifts or 24px spacing gaps.

**Typography**:
- Headlines: `Newsreader` Bold (`font-headline`)
- Body/UI: `Space Grotesk` (`font-body`)
- Labels/metadata: `Space Grotesk` all-caps (`font-label`)
- Code: `IBM Plex Mono` (`font-mono`)

**No** smooth gradients, soft shadows, or standard icon sets (use Material Symbols Outlined).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Shaveen-Balasooriya) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
