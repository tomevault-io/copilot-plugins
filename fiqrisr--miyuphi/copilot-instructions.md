## miyuphi

> Single-page portfolio/pricing site for a freelance video editor. Astro + Tailwind CSS v4 + TypeScript, package-managed with pnpm.

# AGENTS.md — Yuphi Portfolio Site

## Project

Single-page portfolio/pricing site for a freelance video editor. Astro + Tailwind CSS v4 + TypeScript, package-managed with pnpm.

## Commands

- `pnpm install` — install dependencies
- `pnpm run dev` — start dev server
- `pnpm run build` — static production build
- `pnpm run preview` — preview production build
- `pnpm astro check` — typecheck + diagnostics (Astro CLI)
- `pnpm run format` — format all files with Prettier

## Structure & Conventions

- **Naming Style:** ALL files and directories must use `kebab-case` strictly (e.g., `order-flow.astro`, `pricing-tier.astro`). _Agent Note: When importing these kebab-case Astro components in the frontmatter, you must still alias or import them as PascalCase variables (e.g., `import OrderFlow from './order-flow.astro';`)._
- `src/pages/index.astro` — main landing page with smooth-scroll sections
- `src/pages/404.astro` — custom not-found page
- `src/components/` — reusable UI components (`phone-frame.astro`, `video-placeholder.astro`, etc.)
- `src/partials/` — page section / layout partials composed of smaller components (`navbar.astro`, `hero.astro`, `about.astro`, `order-flow.astro`, `style-showcase.astro`, `kitchen-section.astro`, `contact-section.astro`, `footer.astro`, `parallax-background.astro`)
- `src/layouts/layout.astro` — base HTML shell, SEO metadata, JSON-LD, global fonts
- `src/styles/global.css` — Tailwind v4 entry point with `@theme`, custom colors, fonts, keyframes, and utility classes (`glass`, `glass-card`, `float-slow`, `float-mid`, `drift`, `fade-up`)
- `src/data/content.ts` — all website copy, order flow, pricing, portfolio links, and section content
- `public/assets/` — static images, profile picture, portfolio thumbnails/gifs, cloud/star SVGs, OG image
- `scripts/generate-favicon.py` — Python helper for generating favicon assets

## Toolchain gotchas

- Tailwind v4 is Vite-first. Wire it via `@tailwindcss/vite` in `astro.config.mjs`, not the Astro Tailwind v3 integration pattern.
- Tailwind v4 theme tokens are defined in `src/styles/global.css` using the `@theme` directive. Reference them as CSS variables (e.g., `var(--color-sky-soft)` or use the mapped semantic names like `bg-background`).
- Navbar uses anchor links (`#about`, `#how-to-order`, etc.); `scroll-behavior: smooth` is applied to `html` in `global.css`.
- Copy is bilingual (Indonesian/English) with K-vibe / Japanese aesthetic.
- Standard fonts: **Nunito Sans** for body text and **Varela Round** for headings, loaded from Google Fonts in `src/layouts/layout.astro`.

## Design system

- Color palette: soft sky blue (`#e5f1fb`), cloud white, navy deep (`#1e2a4a`), pink pastel (`#fad2e1`), yellow soft (`#fff4bd`), blue pale (`#d6ecff`).
- Surface treatments: `glass` (72% white) and `glass-card` (35% white) with `backdrop-filter: blur(16px)` and reduced-motion fallbacks.
- Decorative motifs: floating clouds, stars, and Cinnamoroll-style cinnamon roll illustrations in `public/assets/`.
- Rounded corners: `--radius-lg: 1.5rem`.

## Content source

Full website copy, order flow, pricing details, and portfolio references should live in `src/data/content.ts` and be imported by partials or reusable components. Do not hardcode long copy blocks directly in `.astro` files.

---
> Source: [fiqrisr/miyuphi](https://github.com/fiqrisr/miyuphi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
