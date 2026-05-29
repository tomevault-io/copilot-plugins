## personal-website

> Manav Trivedi's personal portfolio. Single page, minimal, content-first.

# Personal Website — Project Context

## What this is
Manav Trivedi's personal portfolio. Single page, minimal, content-first.
Live at `/` — no other routes.

## Design philosophy
**Super minimalist, soft, light — but with personality. Not generic.**
The goal is restrained and considered, not bare and boring. Every element
should feel intentional. Avoid the "AI slop" aesthetic: no cards everywhere,
no gradients on everything, no heavy shadows, no hero sections with big bold
taglines. Think quiet confidence.

References in the right direction: paco.me, rauno.me, leerob.io

## Tech stack
- React 19 + Vite (`@tailwindcss/vite` plugin)
- Tailwind CSS v4 — config is CSS-only (no tailwind.config.js)
- `@custom-variant dark (&:where(.dark, .dark *))` — dark mode uses `.dark`
  class on `<html>`, NOT `prefers-color-scheme`
- Lucide React for icons
- Font: Radio Canada (Google Fonts, loaded in `Home.css`)

## Accent color
Fixed indigo: `rgb(165, 180, 252)` — defined as `--color-accent` in `@theme`.
Use as `text-accent`, `decoration-accent`, `hover:text-accent` etc.
Do not add more colours. The accent is used sparingly — section titles,
underline decorations, hover states only.

## Text colour scale (stone palette)
| Role        | Light                  | Dark                   | Tailwind                          |
|-------------|------------------------|------------------------|-----------------------------------|
| Primary     | rgb(41, 37, 36)        | rgb(231, 229, 228)     | `text-stone-800 dark:text-stone-200` |
| Secondary   | rgb(87, 83, 78)        | rgb(214, 211, 209)     | `text-stone-600 dark:text-stone-300` |
| Muted       | rgb(120, 113, 108)     | rgb(168, 162, 158)     | `text-stone-500 dark:text-stone-400` |

Body background: white (light) / `rgb(28, 25, 23)` (dark) — set in `index.css`.

## Dark mode toggle
Toggled via `toggleTheme` from `useColor()` (ColorContext). Adds/removes
`.dark` class on `<html>`. The button lives in `Overview.jsx`.

## Component structure
```
App
└── Home
    ├── Overview      — name, dark/light toggle, nav links (linkedin/github/email/resume)
    ├── About         — short bio paragraph
    ├── Education     — two degrees, UQ
    ├── Experience    — 4 roles (ANZ grad, ANZ intern, QCC, CMC Global)
    ├── Projects      — 4 projects with github/site links
    └── Footer        — "made with..." line
```

All sections except Overview use `CollapsibleSection` (open by default, chevron to collapse).

## Key rules
- All text is **lowercase** — including section titles (`experience`, `projects` etc.)
  and all body content.
- Font class: `font-secondary` (Radio Canada) on everything.
- `tracking-wide` on all text.
- No inline `onMouseEnter/Leave` handlers — use Tailwind hover classes.
- No inline style objects for colours that can be Tailwind classes.
- `color` (accent) is an exported constant from `ColorContext.jsx`, not state.
  Only `theme` and `toggleTheme` live in context.
- Keep components simple. Most content components are pure functions with no
  hooks (About, Education, Experience, Projects, Footer).

## What to avoid
- Adding complexity "just in case"
- Modals or overlays unless they genuinely fit the aesthetic
- Heavy animations or transitions — subtle only
- More colours beyond the stone scale + indigo accent
- Cards, borders, dividers, box shadows on content
- Generic SaaS/landing page patterns

---
> Source: [ManavTriv/personal-website](https://github.com/ManavTriv/personal-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
