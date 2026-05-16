## lydawei

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal portfolio and blog site for David Ly, built with **Astro 2.x** as a static site generator. Hosted on GitHub Pages at `lydawei.com` with automatic deployment via GitHub Actions on push to `master`.

## Commands

```bash
npm run dev       # Start dev server (localhost:4321)
npm run build     # Production build to ./dist
npm run preview   # Preview production build locally
./setup.sh        # First-time setup (checks Node 18+, installs deps)
```

No test framework is configured — this is a static site.

## Architecture

**Astro SSG** with file-based routing. No client-side JS framework; pages are pure static HTML/CSS.

### Routing & Pages

- `src/pages/` — file-based routing (index, about, blog, projects)
- `src/pages/blog/[...slug].astro` — dynamic route for blog posts
- Blog posts are Markdown files using Astro's **Content Collections** with schema in `src/content/config.ts` (title, description, pubDate, tags, author, optional draft flag)

### Layouts

- `src/layouts/BaseLayout.astro` — master layout: nav header, sticky profile sidebar, particle animations, design token system via CSS custom properties
- `src/layouts/AboutLayout.astro` — about page variant

### Design System

CSS custom properties defined in BaseLayout for colors, spacing, border-radius, shadows, and animation timing. Dark theme with blue accent (`#64b5f6`). Animations use CSS keyframes with staggered delays; reduced on mobile/touch.

### External Integrations

- **Pokedex project**: Separate Svelte app on Vercel, proxied via Cloudflare Worker at `/projects/pokedex`
- **Conductor**: Dev environment orchestration (`conductor.json` runs setup + dev server)

### Deployment

- GitHub Actions (`.github/workflows/astro.yml`) builds with Node 20 and deploys to GitHub Pages
- Custom domain configured via `CNAME` file

---
> Source: [LyDawei/lydawei](https://github.com/LyDawei/lydawei) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
