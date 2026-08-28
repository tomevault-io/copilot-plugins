## faouzi-wi-4444-fi

> This document provides an overview of the project structure for developers and AI agents working on this codebase.

# AGENTS.md

This document provides an overview of the project structure for developers and AI agents working on this codebase.

## Project Overview

A single-page marketing/landing site for **wifite2**, a Python tool for auditing wireless networks. The page presents wifite2's attack methods, required tooling, what's new in the rewrite, and install instructions. Built with TanStack Start and deployed on Netlify.

The content is derived from the wifite2 project README (github.com/derv82/wifite2).

### Tech Stack

| Layer | Technology |
|-------|------------|
| Framework | TanStack Start |
| Frontend | React 19, TanStack Router v1 |
| Build | Vite 7 |
| Styling | Tailwind CSS 4 (via `@tailwindcss/vite`) |
| Language | TypeScript 5.9 (strict mode) |
| Deployment | Netlify |

### Design Direction

A dark **terminal / phosphor** aesthetic that reflects wifite2's identity as a CLI auditing tool:
- Off-black background (`#0a0e0a`) with phosphor-green and amber accents.
- `JetBrains Mono` for all UI/code; `Syne` as the display heading face.
- Atmospheric layers: scanlines, film grain, a subtle grid, and a slow vertical scan sweep.
- Animated terminal mock in the hero that simulates a wifite session (Pixie-Dust crack).

## Directory Structure

```
├── public
│   ├── favicon.ico
│   └── placeholder.png
├── src
│   ├── data
│   │   └── content.ts        # Typed content: attackMethods, tools, improvements.
│   ├── routes
│   │   ├── __root.tsx        # Root layout: metadata, scanlines/grain body wrapper.
│   │   └── index.tsx         # The single landing page (all sections).
│   ├── router.tsx            # TanStack Router setup with scroll restoration.
│   └── styles.css            # Tailwind import + theme tokens + animations.
├── AGENTS.md                 # This file.
├── README.md                 # Project readme for humans.
├── netlify.toml              # Build command (vite build), publish (dist/client), dev port.
├── package.json
├── tsconfig.json             # Strict TS, `@/*` path alias for `src/*`.
└── vite.config.ts            # Vite plugins: tsconfig-paths, tailwind, netlify, tanstack-start, react.
```

## Key Concepts

### Single-Route Architecture

The site is one route (`src/routes/index.tsx`) composed of section components rendered in order:
`SweepOverlay → Nav → Hero → Ticker → AttackMethods → Requirements → Improvements → Install → Footer`.

There are no other routes. The original template's `products/$productId` route and `data/products.ts` were removed since this is not an e-commerce site.

### Content Model

All page content lives in `src/data/content.ts` as typed, exported arrays:
- `attackMethods` — the five wifite2 attack vectors (Pixie-Dust, PIN, PMKID, Handshake, WEP).
- `tools` — required and recommended binaries wifite orchestrates.
- `improvements` — what changed in the rewrite vs. original wifite.

Edit these arrays to update the page; the UI maps over them.

### Styling

- Tailwind CSS 4 utility classes plus CSS custom properties defined in `src/styles.css` (`:root`).
- Theme tokens: `--bg`, `--bg-elev`, `--bg-card`, `--phosphor`, `--phosphor-dim`, `--amber`, `--ink`, `--line`, etc.
- Reusable CSS classes: `panel`, `panel-hover`, `corner-tick`, `btn-primary`, `btn-ghost`, `section-tag`, `terminal-cursor`, `grid-bg`, `radial-fade`, `scanlines`, `grain`.
- Animations defined as keyframes: `blink`, `flicker`, `marquee`, `fade-up`, `scan-sweep`, `pulse-dot`.
- Fonts loaded via Google Fonts `@import` in `styles.css`.

### File-Based Routing (TanStack Router)

- `__root.tsx` — root layout; sets `<title>` and meta description, applies the `scanlines grain` body classes.
- `index.tsx` — the `/` route.

## Configuration Files

| File | Purpose |
|------|---------|
| `vite.config.ts` | Vite plugins: tsconfig-paths, tailwind, netlify, tanstack-start, react |
| `tsconfig.json` | Strict TS, `noUnusedLocals`/`noUnusedParameters` on, `@/*` alias |
| `netlify.toml` | `vite build`, publish `dist/client`, dev target port 3000 |
| `styles.css` | Tailwind import + theme tokens + font imports + animations |

## Development Commands

```bash
npm run dev      # Start dev server on port 3000
npm run build    # Production build
```

## Conventions

### Naming
- Components: PascalCase (`AttackMethods`, `InstallCard`, `FlagRow`).
- Content arrays / utilities: camelCase (`attackMethods`, `tools`).
- CSS custom properties: kebab-case (`--phosphor-dim`).

### TypeScript
- Strict mode enabled; `noUnusedLocals` and `noUnusedParameters` are on — do not leave unused imports or variables.
- Import paths use the `@/` alias for `src/*`.
- Type-only imports use the `type` keyword where appropriate.

### Styling Rules
- Use Tailwind utility classes for layout; use the CSS custom properties (`var(--phosphor)`, etc.) inside arbitrary values for theme colors, e.g. `text-[var(--phosphor)]`.
- Do not introduce a `cn()` helper or Radix UI — this project does not use them.
- Avoid generic fonts (Inter, Roboto, system-ui). Stick to JetBrains Mono / Syne.

### Unicode in JSX
- Use literal Unicode characters (✓, ●, ◆, →, —) in JSX text and attribute string values. Do **not** use `\uXXXX` escapes in JSX text content or double-quoted JSX attributes — they render literally there.
- `\uXXXX` escapes are only valid inside JS string literals (single quotes / template literals) and in CSS use `\XXXX` (four hex digits, no `u`).

## Notes
- There is no backend, database, or AI integration in this project — it is a static marketing page.
- The hero terminal mock is illustrative; it does not execute anything.

---
> Source: [faouzisoufiane10-blip/FAOUZI-WI-4444-FI](https://github.com/faouzisoufiane10-blip/FAOUZI-WI-4444-FI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
