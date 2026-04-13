## xpl

> This is a React-based replica of the [xplore.ca](https://www.xplore.ca/) website — a Canadian rural Internet service provider. The site is a marketing/informational website showcasing 5G, fibre, and satellite Internet products.

# Claude — Project Context

## Project Overview

This is a React-based replica of the [xplore.ca](https://www.xplore.ca/) website — a Canadian rural Internet service provider. The site is a marketing/informational website showcasing 5G, fibre, and satellite Internet products.

## Tech Stack

- **React 19** with JSX (no TypeScript)
- **Vite 7** for dev server and builds
- **Tailwind CSS 4** (v4 uses `@import "tailwindcss"` syntax, not the v3 `@tailwind` directives)
- **React Router DOM 7** for client-side routing
- **Lucide React** for SVG icons
- **ESLint 9** with flat config (`eslint.config.js`)

## Brand Colors (xplore.ca aligned)

Defined in `src/index.css` under `@theme`:

| Token                  | Hex       | Usage                              |
| ---------------------- | --------- | ---------------------------------- |
| `xplore-green`         | `#00FF57` | Primary brand green (Spring Green) |
| `xplore-green-dark`    | `#00D448` | Darker green for hovers/accents    |
| `xplore-green-light`   | `#E5FFF0` | Light green backgrounds            |
| `xplore-dark`          | `#283238` | Dark backgrounds (Outer Space)     |
| `xplore-dark-deep`     | `#1C2428` | Deeper dark for gradients          |
| `xplore-gray`          | `#6b7280` | Secondary text                     |
| `xplore-light`         | `#f8f9fa` | Light section backgrounds          |

## Key Conventions

- Components use **function declarations** (`export default function ComponentName()`) rather than arrow functions.
- **No TypeScript** — all files are `.jsx` or `.js`.
- Styling is done entirely with **Tailwind utility classes** inline — no separate CSS modules or styled-components.
- Custom brand colors are defined in `src/index.css` under `@theme` (e.g., `xplore-green`, `xplore-dark`).
- Static data (nav links, blog posts, FAQs, testimonials) lives in `src/data/` as exported JS arrays/objects.
- Pages are in `src/pages/`, reusable components in `src/components/ui/`, layout shell in `src/components/layout/`.
- Hero sections use a `bgImage` prop pointing to SVG banners in `public/images/` with a dark overlay on top.

## Common Commands

```bash
npm run dev       # Start Vite dev server (http://localhost:5173)
npm run build     # Production build → dist/
npm run preview   # Preview production build
npm run lint      # ESLint check
```

## Known Patterns & Gotchas

- The `Infinity` icon from `lucide-react` must be imported as `Infinity as InfinityIcon` to avoid shadowing the global JavaScript `Infinity` value (ESLint `no-shadow-restricted-names` rule).
- The `<Layout>` component wraps all routes and provides Header + Footer via React Router `<Outlet>`.
- `ScrollToTop` (inside Layout) scrolls to the top on every route change.
- There is no backend or API — all data is static/mocked.
- The address lookup in the footer is a demo placeholder.
- No test framework is currently set up.
- `HeroSection` and `HeroCarousel` both accept a `bgImage` prop (path to SVG in `public/images/`) and render it as a CSS `background-image` with a `bg-xplore-dark/60` overlay.
- Banner SVGs in `public/images/` contain **no text** — all text is rendered by React components on top of the background.

## File Structure

```
public/
├── images/            # SVG banner images for hero sections
│   ├── hero-home.svg, hero-5g-ultra.svg, hero-cottage.svg, ...
│   └── (13 themed banner SVGs total)
└── xplore-logo.svg    # Brand logo

src/
├── components/
│   ├── layout/        # Header, Footer, Layout (with Outlet)
│   ├── sections/      # BlogPreview, CTABanner
│   └── ui/            # Button, Card, HeroSection, FAQAccordion, etc.
├── data/              # navigation.js, blogPosts.js, testimonials.js, faq/
├── pages/             # One component per route (Home, About, Shop, etc.)
├── App.jsx            # All <Route> definitions
├── main.jsx           # createRoot + BrowserRouter
└── index.css          # Tailwind @import + custom @theme colors
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/harshalsoni)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/harshalsoni)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
