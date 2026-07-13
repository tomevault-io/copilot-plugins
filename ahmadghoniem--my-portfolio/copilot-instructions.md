## my-portfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Project Commands

All commands are run from the root of the project using **pnpm**:

| Command               | Action                                              |
| :-------------------- | :-------------------------------------------------- |
| `pnpm install`        | Install dependencies                                |
| `pnpm dev`            | Start local dev server at `localhost:4321`          |
| `pnpm build`          | Build production site to `./dist/`                  |
| `pnpm preview`        | Preview the production build locally                |
| `pnpm astro ...`      | Run CLI commands (e.g., `astro add`, `astro check`) |
| `pnpm generate-types` | Generate TypeScript types for Cloudflare Workers    |

---

## Code Architecture

### Framework & Stack

- **Astro** v6 with **server-side rendering** (`output: "server"`) for Islands Architecture support
- **Tailwind CSS** v4 via `@tailwindcss/vite` plugin (no config file needed)
- **Cloudflare Pages** deployment using `@astrojs/cloudflare` adapter
- **TypeScript** with strict type checking via `@ts-check` in `.mjs` files

### Directory Structure

```
/
├── src/
│   ├── components/          # Reusable UI components
│   │   ├── home/           # Homepage-specific components (Hero, Projects, ProjectCard)
│   │   └── ui/             # Shared UI components (AuraControls, Icons)
│   ├── content.config.ts   # Content collections schema definition (Zod)
│   ├── data/projects/      # Markdown project data files with frontmatter
│   ├── layouts/            # Root layout (Layout.astro) with global chrome
│   ├── lib/utils/          # Utility functions (color extraction)
│   ├── pages/              # Route-based pages & API endpoints
│   │   ├── index.astro     # Homepage
│   │   ├── about.astro     # About page
│   │   ├── projects/[slug].astro  # Dynamic project detail pages
│   │   └── api/            # API routes (Spotify now-playing, artwork)
│   ├── shared/
│   │   ├── layout/         # Header, Footer, Outro shared layout components
│   │   └── ui/             # Reusable UI (Icons, ScrollToTop, Tooltip, SpotifyWidget)
│   └── styles/
│       └── global.css      # Tailwind imports + Radix colors + CSS theme variables + custom utilities
├── public/                 # Static assets (images, fonts, favicon)
├── astro.config.mjs        # Astro configuration (Cloudflare adapter, Vite plugins)
└── wrangler.toml           # Cloudflare Workers configuration (KV namespaces for Spotify)
```

### Frontmatter Schema (Content Collections)

Projects use Zod schema validation in `src/content.config.ts`:

- `title`, `description`, `tags` (string[])
- `image` (optional)
- `quote`, `citation` (optional)
- `ctas` (array of `{label, href}`) (optional)
- `link`, `github` (optional)
- `order` (number, default 0)

### Theme System (global.css)

CSS custom properties define the design system in `@theme`:

- **Fonts**: `--font-display` (Acorn), `--font-ui` (Bricolage Grotesque), `--font-body` (Geist)
- **Colors**: Semantic colors (`--color-background`, `--color-foreground`, `--color-accent`, etc.) mapped to Radix UI color scales
- **Shadows**: `--shadow-bento-*` for bento-card style shadows using gray-alpha

### Animated Aura Background

- Canvas-based gradient animation defined in `Layout.astro` (inline script)
- Configurable via `AuraControls.astro` (speed, blur, scale, opacity, color presets)
- Settings persisted to `localStorage` under key `aura-settings`
- Colors configurable via CSS variables: `--aura-color-1`, `--aura-color-2`, `--aura-color-3`

### API Routes

- `src/pages/api/spotify/now-playing.ts` - fetches current track
- `src/pages/api/spotify/artwork.ts` - proxy for album artwork
- Uses Cloudflare KV namespaces configured in `wrangler.toml`

### Image Processing

- `sharp` for image optimization
- `node-vibrant` (browser build) for extracting color palettes from project images in `src/lib/utils/extract-color.ts`

### Prettier Configuration

Prettier is configured in `package.json` with plugins:

- `prettier-plugin-astro` for `.astro` files
- `prettier-plugin-organize-imports` to auto-sort imports
- `prettier-plugin-tailwindcss` to sort Tailwind classes

Prettier settings: no semicolons, no trailing commas, print width 100.

---

## Development Notes

- Make sure to run `pnpm install` after cloning the repository.
- The site uses **pnpm** as the package manager (not npm or yarn).
- Environment variables for Spotify API are handled via Cloudflare KV (see `wrangler.toml`).
- The build outputs to `dist/server` (SSR mode for Cloudflare).
- For local Cloudflare development, use `wrangler dev` (KV namespaces need to be configured with real IDs in `wrangler.toml`).

---

## Important Patterns

- **Component Imports**: Use relative paths; components are organized in `src/components/` (feature-based) and `src/shared/` (reusable).
- **Layout Hierarchy**: `Layout.astro` wraps all pages and includes global scripts (aura canvas), header, footer, and scroll-to-top.
- **Styling**: All styles use Tailwind utility classes; semantic tokens from `@theme` should be preferred (e.g., `bg-background`, `text-foreground`, `text-accent`).
- **Icons**: SVG icons are stored as `.astro` components in `src/shared/ui/` (e.g., `IconArrowRight.astro`).
- **Content**: Projects are markdown files in `src/data/projects/` with frontmatter matching the Zod schema; use `Astro.glob()` or content collections to query them.

---

---
> Source: [ahmadghoniem/my-portfolio](https://github.com/ahmadghoniem/my-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-13 -->
