## website

> - **Product:** CWK. Experience — PLOS Product Website

# AGENTS.md — CWK PLOS Website

## Project Identity

- **Product:** CWK. Experience — PLOS Product Website
- **Type:** Static marketing/product site
- **Tagline:** "The sports agent for entrepreneurs."
- **Philosophy:** Build. Learn. Earn. Play.
- **Target URL:** cwkexperience.com (Cloudflare Pages — project: houseofcwk)
- **Reference site:** cwkexperience.com (old site on page builder — used as content/design source only, not the deploy target)

## Tech Stack

| Layer         | Technology                                |
|---------------|-------------------------------------------|
| Framework     | Astro (latest stable)                     |
| Language      | TypeScript (strict)                       |
| Styling       | Scoped CSS + global CSS variables         |
| Content       | Astro Content Collections (Markdown/MDX)  |
| Islands       | React (for interactive components only)   |
| Deployment    | Cloudflare Pages                          |
| Analytics     | Cloudflare Web Analytics                  |
| Fonts         | DM Sans (primary, all elements), Agrandir (display accents only) |

## Architecture Rules

- **Static-first** — every page should be statically generated at build time. No SSR unless explicitly required.
- **Islands architecture** — interactive components (quiz, forms) use Astro's `client:*` directives. Default to `client:visible` for lazy hydration.
- **Content Collections** — all case studies, portfolio items, and articles live in `src/content/` as Markdown/MDX with typed frontmatter schemas.
- **No JavaScript by default** — pages ship zero JS unless an island component is present. This is Astro's default; preserve it.
- **Scoped styles** — use `<style>` blocks in `.astro` files for component-scoped CSS. Global styles live in `src/styles/global.css`.
- **SEO-first** — every page must have: `<title>`, `<meta name="description">`, Open Graph tags, canonical URL. Use a shared `<SEO>` component.
- **Responsive** — mobile-first design. Breakpoints: 640px (sm), 768px (md), 1024px (lg), 1280px (xl).

## File Conventions

```
cwk-plos-site/
├── astro.config.mjs           # Astro config (Cloudflare adapter, integrations)
├── package.json
├── tsconfig.json
├── public/                    # Static assets (favicons, robots.txt, images)
│   ├── favicon.svg
│   ├── robots.txt
│   ├── og-image.png           # Default Open Graph image
│   └── images/                # Optimized images
├── src/
│   ├── layouts/               # Page layouts
│   │   ├── Base.astro         # HTML shell: <head>, fonts, analytics, footer
│   │   └── Article.astro      # Layout for case study / article pages
│   ├── components/            # Reusable UI components
│   │   ├── Header.astro       # Global navigation
│   │   ├── Footer.astro       # Global footer
│   │   ├── Hero.astro         # Homepage hero section
│   │   ├── ProductHero.astro  # Product page hero
│   │   ├── FeatureShowcase.astro # Alternating text + mockup layout
│   │   ├── PillarCard.astro   # Mind/Body/Soul/Pocket card
│   │   ├── CaseStudyCard.astro
│   │   ├── SEO.astro          # Meta tags component
│   │   ├── BrandMirror.tsx    # Interactive quiz (React island)
│   │   ├── WaitlistForm.tsx   # Email capture form (React island)
│   │   └── mocks/             # Static product mockup components
│   │       ├── MockDashboard.astro
│   │       ├── MockRelationships.astro
│   │       ├── MockPipeline.astro
│   │       ├── MockActions.astro
│   │       ├── MockKaia.astro
│   │       └── MockDestination.astro
│   ├── pages/                 # File-based routing
│   │   ├── index.astro        # Homepage (with waitlist + product preview)
│   │   ├── about.astro        # About Kris
│   │   ├── work.astro         # Portfolio grid
│   │   ├── work/[slug].astro  # Individual case study pages
│   │   ├── product.astro      # Agent+ prototype mockup intro page
│   │   ├── brand-mirror.astro # Brand Mirror quiz page
│   │   ├── side-quests.astro  # Side quests page
│   │   └── 404.astro          # Custom 404
│   ├── content/               # Content Collections
│   │   ├── config.ts          # Collection schemas
│   │   └── work/              # Case study markdown files
│   │       ├── the-lab-miami.md
│   │       ├── rob-dial.md
│   │       ├── spraycation.md
│   │       ├── dawa.md
│   │       ├── stephy-lee.md
│   │       ├── raasin-in-the-sun.md
│   │       ├── pay-the-creators.md
│   │       └── lifes-tapestry.md
│   └── styles/
│       └── global.css         # CSS variables, resets, typography, brand system
└── docs/                      # Project documentation (not deployed)
```

## Brand Design System

### CSS Variables (Official Brand Palette)

> See `docs/DESIGN.md` for full design system reference.

```css
:root {
  /* Primary Palette */
  --bg:     #07090F;  /* Page background */
  --bg2:    #0B0E18;  /* Cards, panels, elevated surfaces */
  --bg3:    #101422;  /* Tab panels, deep nested elements */
  --text:   #EEF0FF;  /* ALL headings, body, labels */
  --cyan:   #00E5FF;  /* Primary accent: CTAs, mission bars */
  --purple: #7B61FF;  /* Secondary accent: gradients, PLOS */
  --pink:   #FB3079;  /* Tertiary accent: eyebrow labels, warnings */

  /* Transparency */
  --glass:  rgba(255, 255, 255, 0.03);  /* Card fills */
  --gb:     rgba(255, 255, 255, 0.08);  /* Default borders */
  --gb2:    rgba(255, 255, 255, 0.14);  /* Hover borders */
  --muted:  rgba(238, 240, 255, 0.62);  /* Secondary text */
  --dim:    rgba(238, 240, 255, 0.32);  /* Captions, placeholders */

  /* Stage Colors (status only, not general brand) */
  --stage-1: #FF4444;  /* Error */
  --stage-2: #FF7A30;  /* Warning */
  --stage-3: #FFB830;  /* Caution */
  --stage-4: #00E5FF;  /* Refinement */
  --stage-5: #7B61FF;  /* Growth-Ready */
  --stage-6: #00E396;  /* Sovereign / Success */

  /* Typography */
  --font-primary: 'DM Sans', sans-serif;

  /* Spacing */
  --gap-tile: 2px;
}
```

### Typography

- **Primary font:** DM Sans (all weights 300-900). No other typeface.
- **Accent font:** Agrandir (PP Agrandir) for display headlines only. Max one element per screen.
- **Hero:** 48-56px / 800 weight / 1.1-1.15 line-height
- **H1:** 32-36px / 700-800 weight / 1.2
- **H2:** 22-26px / 700 weight / 1.3
- **H3:** 16-18px / 600-700 weight / 1.4
- **Body:** 14-16px / 400-500 weight / 1.5-1.6
- **Eyebrow:** 9-11px / 700 weight / uppercase / 0.35-0.5em letter-spacing / `--pink` color

### Button Styles

- **Primary:** `bg: --cyan`, `color: --bg`, `font-weight: 700`, `border-radius: 8px`, `padding: 12px 24px`
- **Secondary:** `bg: transparent`, `border: 1px solid --cyan`, `color: --cyan`
- **Ghost:** `bg: rgba(255,255,255,0.06)`, `color: --text`, `border: 1px solid --gb`
- **Hover:** `brightness(1.1)` on primary, bg fill on secondary, `opacity: 0.9` on ghost
- **Hero text gradient:** `linear-gradient(135deg, #00E5FF, #7B61FF)` with `background-clip: text`

## Coding Patterns

- **TypeScript** — use `.ts` and `.tsx` for all scripts and React components.
- **Astro components** — use `.astro` for all static components. Only use React (`.tsx`) for interactive islands.
- **Props** — type all Astro component props with `interface Props {}` at the top.
- **Image optimization** — use Astro's `<Image>` component from `astro:assets` for all images. Provide `width`, `height`, and `alt`.
- **Semantic HTML** — use `<header>`, `<main>`, `<section>`, `<article>`, `<nav>`, `<footer>`. No `<div>` soup.
- **Accessibility** — all interactive elements need focus states. All images need alt text. Maintain logical heading hierarchy.
- **Content Collections** — define schemas in `src/content/config.ts` using Zod. Access via `getCollection()` and `getEntry()`.

## Environment Variables

| Variable | Purpose |
|----------|---------|
| `PUBLIC_SITE_URL` | Canonical site URL (for SEO/OG tags) |
| `PUBLIC_CF_ANALYTICS_TOKEN` | Cloudflare Web Analytics beacon token (omit to disable) |
| `PUBLIC_GTM_ID` | Google Tag Manager container ID (omit to disable tracking) |

## Domain Vocabulary

| Term | Meaning |
|------|---------|
| PLOS | Personal Lifestyle Operating System — CWK's proprietary framework |
| Four Pillars | Mind, Body, Soul, Pocket |
| Brand Mirror | Free 2-min diagnostic quiz identifying business bottlenecks |
| X-Ray | Proprietary diagnostic audit of a client's business |
| Brand Destination | A 60–120 day goal sprint with milestones |
| Player | A CWK client/entrepreneur |
| Brand Guide | Admin coach (Kris San / team) |
| Sovereignty Score | Composite metric across 6 leverage dimensions |
| System Tags | codify, create, connect, convert (4 action categories) |
| Action Gradient | The brand's signature blue→purple gradient |

## SEO Requirements

- Every page needs unique `<title>` and `<meta name="description">`
- Open Graph: `og:title`, `og:description`, `og:image`, `og:url`, `og:type`
- Twitter Card: `twitter:card`, `twitter:title`, `twitter:description`, `twitter:image`
- Canonical URLs on every page
- Structured data (JSON-LD) on homepage and case studies
- `sitemap.xml` generated by `@astrojs/sitemap`
- `robots.txt` in `public/`

---
> Source: [houseofcwk/website](https://github.com/houseofcwk/website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
