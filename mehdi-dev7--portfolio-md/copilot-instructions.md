## portfolio-md

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # Start dev server (Vite HMR)
npm run build    # Production build → dist/
npm run preview  # Preview production build
npm run lint     # ESLint check
```

## Architecture

Single-page portfolio built with **React + Vite** (JavaScript, not TypeScript). No routing — navigation is anchor-link based.

```
src/
├── App.jsx              # Root: assembles Navbar + all sections in order
├── main.jsx             # React entry point
├── index.css            # Tailwind v4 import + @theme tokens + .glass utility
├── layout/
│   └── Navbar.jsx       # Fixed top navbar with glassmorphism
├── components/
│   └── Button.jsx       # Reusable pill button
└── sections/            # One file per page section
    ├── Hero.jsx
    ├── About.jsx
    ├── Experience.jsx
    ├── Projects.jsx
    ├── Testimonials.jsx
    └── Contact.jsx
```

**Path alias**: `@/` resolves to `src/` (configured in `vite.config.js`).

## Tailwind v4 — Important Differences

This project uses **Tailwind CSS v4**, which has a different API than v3:

- Config is CSS-first via `@theme` in `index.css`, not `tailwind.config.js`
- Import syntax: `@import "tailwindcss"` (not `@tailwind base/components/utilities`)
- Custom tokens are defined as CSS variables under `@theme` and become Tailwind utilities automatically

```css
/* index.css */
@import "tailwindcss";

@theme {
  --color-primary: #20b2a6;   /* → bg-primary, text-primary, etc. */
  --color-surface: #1a2329;   /* → bg-surface */
  /* ... */
}
```

## Design Tokens

| Token | Value | Usage |
|---|---|---|
| `background` | `#0f1418` | Page background |
| `foreground` | `#f0f2f5` | Default text |
| `primary` | `#20b2a6` | Teal accent |
| `surface` | `#1a2329` | Card/glass backgrounds |
| `muted-foreground` | `#7a8491` | Secondary text |
| `highlight` | `#f5a623` | Orange accent |

**`.glass` utility** (defined in `@layer components`): frosted glass effect — `background-color` with 80% surface + `backdrop-filter: blur(16px)` + semi-transparent border.

## Fonts

Loaded externally (not installed as npm packages):
- `Inter` (body)
- `Playfair Display` (serif accent, via `.font-serif` class)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Mehdi-dev7) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
