## joshillichmann-com

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- `npm run dev` - Start Astro dev server (http://localhost:4321)
- `npm run build` - Build the static site to `dist/`
- `npm run preview` - Serve the built site locally

## Project Architecture

Portfolio site built with Astro 5 + React islands, deployed as static files to Cloudflare Pages.

### Key Technology Stack

- **Framework**: Astro 5 (static output, no SSR adapter)
- **Interactive islands**: React 19 via `@astrojs/react`
- **Styling**: Tailwind CSS v4 via `@tailwindcss/vite` (CSS-based config in `src/styles/global.css`)
- **UI components**: Radix UI primitives (Slot) + shadcn-style components
- **Animations**: Framer Motion inside React islands
- **Carousel**: `embla-carousel-react` + `embla-carousel-autoplay`
- **Typography**: Inter via `@fontsource/inter`
- **Icons**: `lucide-react`

### Project Structure

- `src/pages/index.astro` - Homepage; static shell + two React islands.
- `src/layouts/BaseLayout.astro` - `<html>`/`<head>` with metadata, font imports, global CSS.
- `src/components/ProjectsCarousel.tsx` - React island, hydrated with `client:visible`.
- `src/components/TestimonialsRotator.tsx` - React island, hydrated with `client:visible`.
- `src/components/ui/{button,carousel}.tsx` - Shared React primitives used by islands.
- `src/lib/utils.ts` - `cn()` helper (clsx + tailwind-merge).
- `src/styles/global.css` - Tailwind entrypoint, theme tokens, custom utilities.
- `public/` - Static assets served as-is at the site root (images, favicon, og-image).

### Islands architecture

The page renders as static HTML with zero JS by default. React is only hydrated for the two interactive sections (carousel, testimonials), both using `client:visible` so their JS loads lazily when they scroll into view. The profile section's entrance animation is plain CSS (`.animate-fade-in-up` in `global.css`) — no React runtime needed.

When adding new interactivity, prefer: (1) static HTML, (2) CSS animation, (3) small `<script>` block in `.astro`, (4) React island only when state + lifecycle are needed.

### Deployment

Cloudflare Pages with Git integration; framework preset = Astro, build command `npm run build`, output `dist`. Analytics via Cloudflare Web Analytics (injected at the edge, no code change). No `wrangler.toml`, no adapter — pure static output.

---
> Source: [studioprisoner/joshillichmann.com](https://github.com/studioprisoner/joshillichmann.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
