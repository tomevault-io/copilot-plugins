## portfolio-astro

> Personal portfolio site for Samiul Alim. Migrated from Next.js 14 to Astro 5.x for a leaner, faster static site.

# Portfolio — Astro

Personal portfolio site for Samiul Alim. Migrated from Next.js 14 to Astro 5.x for a leaner, faster static site.

## Commands

```bash
npm run dev       # Start dev server (localhost:4321)
npm run build     # Production build → dist/
npm run preview   # Preview production build locally
```

## Tech Stack

- **Astro 5.x** — static site generator (`output: 'static'`)
- **React 19** — interactive islands only (not full SPA)
- **Tailwind CSS 3.x** — styling via `@astrojs/tailwind`
- **Three.js / React Three Fiber** — WebGL hero image effect
- **Motion** — animations (parallax, curtains, scroll effects)
- **Lenis** — smooth scrolling
- **Radix UI** — accessible primitives (accordion, checkbox, dialog/sheet)
- **Zustand** — state management (PocketBase store)
- **PocketBase** — backend client (retained for future use, not actively called)

## Architecture

### SSG with React Islands

The entire page is **pre-rendered to static HTML at build time**. JavaScript only ships for interactive components via Astro's `client:*` directives.

**Zero-JS static components** (`.astro` files):
- `Layout.astro` — HTML shell, `<head>`, Google Fonts, meta tags
- `NavBar.astro` — logo, desktop nav links (plain `<a>` anchors)
- `Feed.astro` — section heading, column labels, filter layout
- `LandingFooter.astro` — contact info, copyright, inline SVG icons

**React islands** (`.tsx` files hydrated on the client):

| Component | Directive | Reason |
|---|---|---|
| `Curtains` | `client:load` | Must animate immediately on page load |
| `Hero` | `client:only="react"` | Three.js/WebGL cannot SSR |
| `VerticalParallax` | `client:visible` | Below fold, scroll-driven |
| `ScrollSection` | `client:visible` | Composite island, far below fold |
| `Post` (each feed entry) | `client:visible` | Expand/collapse interaction |
| `Type` | `client:visible` | Accordion filter |
| `FooterTitle` | `client:visible` | Scroll-triggered letter animation |
| `BackToTopButton` | `client:visible` | Click handler at page bottom |
| `MobileNav` | `client:idle` | Sheet drawer, deferred until idle |

### ScrollSection Composite Island

Astro cannot pass hydrated React children between separate islands. The original Next.js nesting:

```
SmoothScroll > HorizontalScroll > TextFill
SmoothScroll > VerticalScroll
```

is wrapped into a single React component `ScrollSection.tsx` that renders the full tree internally, used as `<ScrollSection client:visible />` in `index.astro`.

## Project Structure

```
src/
├── layouts/Layout.astro              # HTML shell, fonts, NavBar
├── pages/index.astro                 # Homepage with island hydration
├── styles/
│   ├── globals.css                   # Tailwind, CSS vars, @font-face, layout classes
│   └── curtains.css                  # Curtain animation block positions
├── components/
│   ├── NavBar.astro                  # Static — desktop links, embeds MobileNav island
│   ├── ScrollSection.tsx             # Composite island (SmoothScroll + HorizontalScroll + TextFill + VerticalScroll)
│   ├── HorizontalScroll.tsx          # Scroll-driven horizontal translate (desktop)
│   ├── sections/
│   │   ├── Hero.tsx                  # Scrambler + Zoop text + Three.js canvas
│   │   ├── Curtains.tsx              # Loading animation blocks
│   │   ├── VerticalParallax.tsx      # 4-col (desktop) / 2-col (mobile) parallax images
│   │   ├── Feed.astro               # Static feed layout, embeds Post + Type islands
│   │   ├── LandingFooter.astro       # Static footer, embeds FooterTitle + BackToTopButton
│   │   └── MobileNav.tsx            # Sheet drawer nav
│   ├── text/
│   │   ├── TextFill.tsx             # Horizontal scroll text + SVG arrow animations
│   │   ├── Scrambler.tsx            # Text scrambling reveal effect
│   │   └── Zoop.tsx                 # Hover character slide animation
│   ├── three/
│   │   ├── SmoothScroll.tsx         # Lenis smooth scroll wrapper
│   │   ├── CanvasComponent.tsx      # R3F Canvas wrapper
│   │   ├── ImageAbberation.tsx      # Chromatic aberration shader (configurable)
│   │   └── ProfileImage.tsx         # Chromatic aberration shader (fixed params)
│   ├── blocks/
│   │   ├── VerticalScroll.tsx       # Mobile alternative to HorizontalScroll
│   │   ├── FooterTitle.tsx          # "SAMIUL ALIM" letter-by-letter reveal
│   │   └── Type.tsx                 # Filter accordions (Type + Topic)
│   ├── atoms/
│   │   ├── Post.tsx                 # Expandable feed entry card
│   │   └── BackToTopButton.tsx      # Scroll-to-top button
│   └── ui/                          # shadcn/ui primitives
│       ├── button.tsx
│       ├── accordion.tsx
│       ├── sheet.tsx
│       └── checkbox.tsx
├── lib/utils.ts                      # cn() helper, deviceType()
├── stores/PocketStore.ts             # Zustand store for PocketBase
├── constants/
│   ├── feeds.ts                      # Static feed data (23 entries)
│   ├── blogTypes.ts                  # Filter types: Blog, Videos
│   └── blogTopics.ts                 # Filter topics: React, TypeScript, etc.
└── assets/icons/
    ├── ReactIcon.tsx
    └── TSIcon.tsx
```

```
public/
├── fonts/reross_quad.otf             # Local custom font
└── assets/
    ├── images/
    │   ├── cat_portrait.png          # Hero image (Three.js shader)
    │   ├── hero_portrait.jpg
    │   ├── landing_img_2.png
    │   ├── placeholder.jpg
    │   ├── potrait.png
    │   └── parallax/1.jpg–12.jpg     # Vertical parallax gallery
    └── svg/pointer_1.svg
```

## Styling

- **Tailwind CSS 3.x** with `tailwindcss-animate` and `tailwind-scrollbar-hide` plugins
- **CSS custom properties** for colors (HSL-based shadcn theme) and font families
- **Fonts:** Inter, JetBrains Mono, Bebas Neue, Nunito (Google Fonts via `<link>`), Reross Quad (local `@font-face` from `public/fonts/`)
- **Font Tailwind classes:** `font-mono`, `font-inter`, `font-bebas`, `font-nunito`, `font-reross`
- Dark mode configured via `darkMode: ["class"]` but not actively toggled

## Environment Variables

```bash
PUBLIC_POCKETBASE_URL=   # Astro uses PUBLIC_ prefix for client-exposed vars
```

Set in `.env` at project root. Accessed via `import.meta.env.PUBLIC_POCKETBASE_URL`.

## Vite Config Notes

In `astro.config.mjs`:
```js
vite: {
  ssr: { noExternal: ['three', 'lenis'] }
}
```
Required because `three` and `lenis` need to be bundled during SSR to avoid Node ESM resolution issues.

## Key Patterns

- **No `"use client"` directives** — Astro handles client/server split via `client:*` attributes
- **No `next/link`** — all navigation uses plain `<a>` tags with anchor hrefs (`#feed`, `#contact`)
- **Path alias** `@/*` maps to `./src/*` (configured in `tsconfig.json`)
- **Static data** — feed entries are hardcoded in `constants/feeds.ts`, not fetched at runtime
- **Inline SVGs** in `LandingFooter.astro` for Copyright and MapPin icons (avoids shipping lucide-react JS for static icons)

## Known Build Warnings

- Hero chunk is ~887KB (Three.js). Expected — could be code-split with dynamic `import()` if needed.
- `"MotionValue" is imported but never used` in VerticalParallax — harmless type import from framer-motion.

## What Was Removed (vs. original Next.js project)

- All `@udecode/plate` editor packages (28+)
- Blog pages, portfolio pages, dashboard routes
- `Logout` component and `AuthStore`
- `sharp` (image processing)
- `next`, `next/font`, `next/link`, `next/image`
- ~75 unused dependencies

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/XooTB)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/XooTB)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
