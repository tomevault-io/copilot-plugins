## casaconstrucao

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Single-page marketing landing page for **Casa Construção** (a construction fair/event). Built with React 19, TypeScript, Vite 8, Chakra UI v3, TanStack Router, and Framer Motion. The site is entirely **dark-mode** and content is in **Brazilian Portuguese**.

## Commands

- `npm run dev` — start dev server with HMR at `http://localhost:5173`
- `npm run build` — type-check (`tsc -b`) then produce a production build via Vite
- `npm run preview` — serve the production build locally
- `npm run lint` — run ESLint over the project
- `npm run typegen` — regenerate Chakra UI theme typings from `src/theme/index.ts`

There is no test suite. After editing the theme (`src/theme/index.ts`), run `npm run typegen` so Chakra's typed token autocomplete stays in sync.

## Architecture

### Routing (file-based, auto-generated)
TanStack Router with `autoCodeSplitting`. The `@tanstack/router-plugin` regenerates `src/routeTree.gen.ts` automatically during dev/build — **never edit that file by hand**.

- `src/routes/__root.tsx` — root layout: `<Navbar />` + `<Outlet />` + devtools
- `src/routes/index.tsx` — the home page; composes all sections in order inside a `<Stack>`, each wrapped in a `<Box id={...}>` for anchor scrolling
- `src/routes/design-system.tsx` — internal `/design-system` route showcasing tokens/components
- `src/routes/-sections/` — the home page sections (the `-` prefix excludes the folder from route generation; these are plain components, not routes)

### Page sections pattern
Each section in `src/routes/-sections/` is a self-contained component that:
1. Reads its copy/data from a matching file in `src/constants/` (e.g. `benefits-section.tsx` ↔ `home-benefits.ts`). **Text content and data live in `src/constants/`, not inline in components.** Edit copy there.
2. Wraps content in `GradientSection` (`src/components/ui/gradient-section.tsx`) for the per-section radial-gradient backdrop, and uses `layerStyle="section.*"` tokens.
3. Uses the motion system for scroll-reveal animations.

To add/remove/reorder a section: edit `src/routes/index.tsx`, add an ID to `HOME_SECTION_IDS` in `src/constants/navigation-sections.ts`, and add a nav entry to `NAVBAR_SECTION_LINKS` there. (Note: the `video` section has been fully removed — its `video-section.tsx` component, its usage in `index.tsx`, the navbar "Vídeo" link, and its `video` id in `HOME_SECTION_IDS` are all gone.)

### Theme (Chakra UI v3)
`src/theme/index.ts` is the single source of truth, exporting `globalSystem` (a `createSystem(...)`) consumed by `src/components/ui/provider.tsx`. It defines:
- `tokens.colors` — base palette
- `semanticTokens` — dark-only semantic mappings (`bg.canvas`, `fg.default`, `brand.green`, etc.)
- `layerStyles` — reusable surfaces and section backdrops (`section.default`, `section.surface`, `section.cta`, `surface.card`, gradients)
- `textStyles` — typography hierarchy (`display`, `h1`–`h3`, `body.*`, `accent`, `label`)
- custom `breakpoints` including non-standard `tablet` (992px), `desktop` (1100px), `wide` (1400px)

The app is forced dark via `ColorModeProvider defaultTheme="dark"`. See `THEME.md` for the full palette, gradient, and typography reference. Fonts (Blauer Nue, Libre Baskerville, Montserrat) are imported in `src/theme/global.css`.

### Motion system (`src/motion/`)
- `chakra-motion.tsx` — `MotionBox` = `motion.create(Box)`, the bridge between Chakra and Framer Motion
- `presets.ts` — variant factories (`fadeUp`, `fade`, `fadeLeft`, `fadeRight`, `staggerContainer`, `pressableTransition`); **each takes the `reduceMotion` flag** and collapses to no-motion when set
- `section-reveal.tsx` — `<SectionReveal>` wraps content with `whileInView` stagger reveal

Always thread `useReducedMotion()` into the preset factories (the established pattern) so prefers-reduced-motion is respected.

### Anchor scrolling
Navbar links and cross-route navigation use `src/lib/scrollToSection.ts`. For navigating to a section from another route, set a pending scroll via `setPendingSectionScroll(id)` then `runPendingSectionScroll()` runs in `index.tsx`'s `useEffect` (it retries across animation frames until the target element mounts). Default scroll offset is 72px (navbar height).

### UI component library (`src/components/ui/`)
Generated Chakra UI v3 snippet components (accordion, dialog, drawer, etc.). Mostly stock — treat as the base component layer rather than authoring from scratch.

## Conventions
- Path alias `@/*` → `src/*` (configured in `tsconfig.app.json` + `vite-tsconfig-paths`)
- TypeScript is strict with `noUnusedLocals`/`noUnusedParameters`/`verbatimModuleSyntax` — use `import type` for type-only imports
- Icons come from `react-icons` (the `Lu*` / Lucide set is used throughout)

---
> Source: [devsetheventos/casaconstrucao](https://github.com/devsetheventos/casaconstrucao) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
