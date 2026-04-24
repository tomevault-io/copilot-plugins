## hasegawa

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal website/blog for Hayato Hasegawa (長谷川 駿) built with Astro, focusing on web accessibility. The site uses static site generation with TypeScript, Tailwind CSS, and MDX for content.

## Essential Commands

```bash
# Development
bun run dev          # Start development server (or bun start)

# Build
bun run build        # Type check and build for production

# Preview
bun run preview      # Preview production build locally

# Code Quality
bun run check        # Run Biome linter/formatter with auto-fix on ./src
bun run format       # Run Biome + Prettier on all files
bun run format:astro # Run Prettier on Astro files only
```

## Architecture Overview

### Technology Stack
- **Astro 5.x** - Static site generator with MDX support
- **TypeScript** - Strict mode enabled
- **Tailwind CSS v4** - Configured via CSS @theme, no separate config file
- **Biome** - Linting and formatting (tabs, double quotes, import organization)
- **Prettier** - For Astro file formatting only
- **Bun** - Package manager and runtime (version managed via `.bun-version`)

### Project Structure
- **Content System**: Uses Astro's content collections for blog posts
  - Blog posts in `/src/content/blog/` (supports .md and .mdx)
  - Schema: title (string), description (string), date (Date), ogImage (optional string)
  
- **Routing**: 
  - Static pages in `/src/pages/`
  - Dynamic blog routes via `[...slug].astro`
  - RSS feed generation at `/rss.xml`

- **Styling Approach**:
  - Tailwind v4 configured entirely via CSS in `/src/styles/style.css`
  - Custom logical properties plugin in `/src/styles/logical-properties-plugin.css` provides:
    - Padding/margin utilities: `pbl-*`, `pbe-*`, `pis-*`, `pie-*`, etc.
    - Space utilities: `space-b-*`, `space-i-*` (block/inline directions)
    - Border utilities: `border-bs`, `border-be`, `border-is-*`, etc.
    - Inset utilities: `block-start-*`, `block-end-*`
    - Grid utilities: `grid-cols-auto-fill-*`, `grid-cols-auto-fit-*`
  - Custom fonts: MFW-PIshiiGothicStdN (regular and bold variants)
  - Typography plugin for blog content
  - All hover states respect user preferences via `@media (hover: hover)`

### Key Configuration
- Site URL: `https://hayatohasegawa.com`
- Path alias: `@/*` → `src/*`
- TypeScript: Extends `astro/tsconfigs/strict`
- Biome ignores: `/src/styles/style.css` and `/src/styles/logical-properties-plugin.css`

## Development Patterns

### Code Style
- **Formatting**: Use tabs for indentation, double quotes for strings
- **Import organization**: Biome automatically organizes imports
- **Astro files**: Use Prettier for formatting (Biome handles TS/JS/HTML within)

### CSS Guidelines
- **Always use logical properties** instead of physical directions:
  - Use `pbs-4` (padding-block-start) instead of `pt-4`
  - Use `mbe-2` (margin-block-end) instead of `mb-2`
  - Use `pis-6` (padding-inline-start) instead of `pl-6`
- This approach supports internationalization for RTL/LTR and different writing modes

### Content Guidelines
- Blog posts must include all required frontmatter: `title`, `description`, `date`
- Optional: `ogImage` (uses Astro's image() helper)
- Place blog posts in `/src/content/blog/` as `.md` or `.mdx` files

### CI/CD
- **CI checks** run on every PR: type check, lint/format, build
- **Auto-deploy** to GitHub Pages on push to `main`
- **Dependabot**: Automated dependency updates with smart grouping and auto-merge for patch/minor updates

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hasegawa-101) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
