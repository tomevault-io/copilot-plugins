## websiteverion

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Marketing/landing page for Verion EDA (verion-eda.com) — a GPU-accelerated RTL simulation tool. Single-page Astro site with Tailwind CSS. Deployed to GitHub Pages via GitHub Actions on push to `main`.

## Commands

- `npm run dev` — Start dev server at http://localhost:4321
- `npm run build` — Production build to `dist/`
- `npm run preview` — Preview production build locally
- `npm run favicon` — Regenerate `public/favicon.ico` from `public/chipicon.svg` (requires ImageMagick)
- `nix develop` — Enter Nix dev shell (Node.js 22 + npm)

No test suite or linter is configured.

## Architecture

**Stack:** Astro 4 + Tailwind CSS 3, static output (no SSR).

**Page structure:** Single page (`src/pages/index.astro`) assembles section components in order:
`Nav → Hero → Problem → Product → CodeExample → Team → Waitlist → Footer`

**Layout:** `src/layouts/Base.astro` provides the HTML shell, font loading (Inter, JetBrains Mono via Google Fonts), global CSS (restrained dark theme with subtle grid background and muted cyan accent), and GSAP ScrollTrigger for `.reveal` scroll animations.

**Styling:** Dark theme with custom `verion-*` color palette, neon glow effects, and custom animations defined in `tailwind.config.mjs`. Global utility classes (`card-glow`, `text-glow`, `btn-neon`, `neon-line`, `reveal`) are in `Base.astro`'s `<style>` block, not in separate CSS files.

**Deployment:** GitHub Actions workflow (`.github/workflows/deploy.yml`) runs `npm ci && npm run build` and deploys `dist/` to GitHub Pages. Custom domain `verion-eda.com` via `public/CNAME`.

**Static assets:** `public/` contains `chipicon.svg` (SVG favicon source), `favicon.ico` (generated, Safari fallback), `apple-touch-icon.png`, `hero-chip.jpg`, and `CNAME`. The favicon link order in `Base.astro` is `.ico` → SVG → apple-touch-icon ("golden favicon" pattern) — Safari needs the `.ico` at the root to render the icon reliably.

**Waitlist form:** `Waitlist.astro` uses Web3Forms.

---
> Source: [verion-eda/WebsiteVerion](https://github.com/verion-eda/WebsiteVerion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
