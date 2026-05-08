## sxzz-dev

> Personal portfolio and blog for Kevin Deng (sxzz) at https://sxzz.dev. Built with Astro, Vue 3, and UnoCSS, deployed on Cloudflare Workers/Pages.

# CLAUDE.md

## Project Overview

Personal portfolio and blog for Kevin Deng (sxzz) at https://sxzz.dev. Built with Astro, Vue 3, and UnoCSS, deployed on Cloudflare Workers/Pages.

## Tech Stack

- **Framework**: Astro 5 with Vue 3 integration
- **Styling**: UnoCSS (Wind3 preset, Tailwind-compatible utilities)
- **Icons**: Iconify (Simple Icons)
- **Markdown**: Astro Content Collections with `remark-github-blockquote-alert`
- **Deployment**: Cloudflare Workers via `@astrojs/cloudflare`
- **Package Manager**: pnpm 10
- **Language**: TypeScript (strict mode)

## Commands

```bash
pnpm dev       # Start dev server
pnpm build     # Build for production
pnpm preview   # Preview production build
```

No test or lint commands are configured.

## Project Structure

```
src/
├── content/           # Markdown content collections
│   ├── posts/en/      # English blog posts
│   ├── posts/zh/      # Chinese blog posts
│   └── musings/       # Chinese-only informal posts
├── i18n/              # Internationalization (EN/ZH)
│   ├── ui.ts          # Translation strings
│   └── utils.ts       # i18n helpers
├── layouts/           # Layout components (Layout.astro)
├── pages/             # File-based routing
│   ├── zh/            # Chinese routes (mirror of English)
│   └── posts/         # Blog post routes
├── scripts/           # Client-side scripts
├── consts.ts          # Navigation and social link constants
├── content.config.ts  # Content collection definitions
└── middleware.ts      # Language redirect middleware
```

## Conventions

- **Astro components**: PascalCase (e.g., `Layout.astro`)
- **TS/utility files**: camelCase (e.g., `typewriter.ts`)
- **Content/blog posts**: kebab-case (e.g., `my-post-title.md`)
- **Imports**: ES modules, relative paths for local files, package specifiers for deps
- **Styling**: Utility-first UnoCSS classes; avoid custom CSS unless necessary
- **i18n**: URL-based routing (`/` for English, `/zh/` for Chinese). Default language is English. Add translations to `src/i18n/ui.ts`.
- **Fonts**: Serif (Newsreader / Noto Serif SC) for headings, Sans (Outfit) for body, Mono (JetBrains Mono) for code

## Content

Blog posts go in `src/content/posts/{en,zh}/` as `.md` files with required frontmatter:

```yaml
---
title: Post Title
date: 2025-01-01
description: Short description
---
```

Musings (Chinese-only informal posts) go in `src/content/musings/`.

---
> Source: [sxzz/sxzz.dev](https://github.com/sxzz/sxzz.dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
