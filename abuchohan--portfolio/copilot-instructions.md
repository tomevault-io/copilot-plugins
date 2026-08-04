## portfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pnpm dev        # Start dev server
pnpm build      # Type-check + build for production (tsc -b && vite build)
pnpm lint       # Run ESLint
pnpm preview    # Preview production build
```

## Adding shadcn/ui components

```bash
pnpm dlx shadcn@latest add <component>
```

## Architecture

React 19 + Vite + TypeScript + Tailwind CSS v4 portfolio site.

- `@` path alias resolves to `src/`
- `src/lib/utils.ts` — exports `cn()` (clsx + tailwind-merge)
- `src/components/ui/` — shadcn/ui components
- `src/components/FloatingLines.tsx` — Three.js WebGL canvas component used as a full-screen animated background in the hero section

### FloatingLines component

Renders animated wave lines via a GLSL fragment shader using Three.js with an orthographic camera over a full-screen quad. Key props:

- `enabledWaves` — which of `top`, `middle`, `bottom` wave groups to render
- `lineCount` / `lineDistance` — number per group and spacing (scalar or per-group array)
- `interactive` — enables mouse/pointer bend effect via `iMouse` uniform
- `parallax` — subtle UV shift based on pointer position
- `linesGradient` — array of hex colors (up to 8) for gradient line coloring
- `bendRadius` / `bendStrength` — control the cursor influence shape

The component mounts a `WebGLRenderer` into a container div, uses `ResizeObserver` for responsive sizing, and fully cleans up (disposes geometry, material, renderer, removes event listeners) on unmount.

## Tailwind CSS v4

This project uses Tailwind v4 (via `@tailwindcss/vite` plugin), not the traditional `tailwind.config.js`. Configuration is done in CSS using `@theme` directives inside `src/index.css`.

---
> Source: [abuchohan/portfolio](https://github.com/abuchohan/portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
