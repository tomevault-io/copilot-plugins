## 3dsvg

> Project context for Claude Code.

# CLAUDE.md

Project context for Claude Code.

## Overview

3dsvg is a monorepo with two packages:

- **`packages/engine`** — `<SVG3D>` React component published to npm as `3dsvg`
- **`packages/web`** — Next.js visual editor at 3dsvg.design

The web editor renders the engine's `<SVG3D>` component directly — what you see in the editor is exactly what you get with the embed.

## Commands

```bash
npm install              # install all dependencies
npm run build:engine     # build the engine (must run before dev:web)
npm run dev:web          # start the web editor at localhost:3000
```

## Architecture

- Engine is a tsup-bundled React component library using React Three Fiber + Three.js
- Web editor is a Next.js 16 app with Tailwind CSS v4 and shadcn/ui
- Web imports engine as a workspace dependency (`"3dsvg": "file:../engine"`)
- Changes to the engine require a rebuild to be picked up by the web editor

## Conventions

- All source files have `/** === TITLE === */` block headers with `@packageDocumentation`
- Engine files use `@packageDocumentation` tag; web components do not
- TypeScript strict mode throughout
- Tailwind CSS for styling (no CSS modules)
- shadcn/ui for UI components (in `packages/web/src/components/ui/`)
- Conventional commit messages: `feat:`, `fix:`, `chore:`, `docs:`

## LLM context

- `llms.txt` — root-level project overview for AI assistants
- `packages/engine/llms.txt` — full `<SVG3D>` API reference (props, defaults, examples). Included in the npm package.

## Key files

- `packages/engine/src/types.ts` — all props and defaults for `<SVG3D>`
- `packages/engine/src/index.tsx` — public API entry point
- `packages/engine/src/scene.tsx` — Three.js rendering pipeline
- `packages/web/src/app/page.tsx` — main editor page with all state
- `packages/web/src/components/svg-to-3d-canvas.tsx` — editor wrapper around `<SVG3D>`

## Important notes

- The `svg` prop accepts raw SVG markup or URLs (fetched automatically)
- Default color is `#ffffff` (white) — neutral base for materials
- `scrollZoom` defaults to `false` so embeds don't hijack page scroll
- Width and height default to `"100%"` — the component fills its parent

---
> Source: [renatoworks/3dsvg](https://github.com/renatoworks/3dsvg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
