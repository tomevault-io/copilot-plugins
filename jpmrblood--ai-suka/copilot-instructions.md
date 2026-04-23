## ai-suka

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Astro-based blog site deployed to GitHub Pages. It uses Astro 6 with Tailwind CSS 4, DaisyUI component library, and MDX support.

## Commands

```bash
bun dev        # Start dev server at localhost:4321
bun build      # Build production site to ./dist/
bun preview    # Preview build locally
```

## Architecture

### Content Collections

Blog posts are managed via Astro Content Collections:
- **Location**: `src/content/blog/` (`.md` or `.mdx` files)
- **Schema**: Defined in `src/content.config.ts` with frontmatter validation
- **Fields**: `title`, `description`, `pubDate`, `heroImage`, `categories` (max 3), `tags`

### Routing

- **Homepage**: `src/pages/index.astro` - Featured post hero + article grid
- **Blog posts**: `src/pages/[...slug].astro` - Dynamic rendering using `BlogPost.astro` layout
- **Categories**: `src/pages/categories/[category].astro` - Category filtering

### Key Layouts & Components

- `src/layouts/BlogPost.astro` - Main blog post layout with TOC sidebar
- `src/components/Timeline.astro` - Chronological content display with GSAP animations
- `src/utils/readingTime.ts` - Word count-based reading time calculation
- `src/utils/relatedPosts.ts` - Category/tag weighted scoring for related posts

### Styling

- Tailwind CSS 4 via `@tailwindcss/vite` plugin
- DaisyUI 5 for semantic component classes (btn, card, badge, etc.)
- Typography plugin (`prose` classes for article content)
- Theme toggle supported via DaisyUI themes

## Content Creation Guidelines

### Blog Post Frontmatter

```yaml
---
title: "Descriptive Title Without Sensational Language"
description: "Brief summary for meta tags and previews"
pubDate: "2025-10-15"
tags: ["relevant", "tags"]
categories: ["category-name"]  # Max 3 categories
heroImage: "../../assets/YYYY/MM/image-name.png"
---
```

### Asset Organization

Images go in `src/assets/YYYY/MM/` with kebab-case naming.

### YouTube-Derived Content

When articles are generated from YouTube transcripts:
- Include `"youtube"` in categories
- Add attribution as last line:
  `*This article was written by [AI Name] (Model), based on content from: [YouTube URL]*`

## Deployment

- Platform: GitHub Pages at `https://jpmrblood.github.io/ai-suka/`
- CI/CD: `.github/workflows/deploy.yml` triggers on push to main
- Base URL: `/ai-suka` (configured in `astro.config.mjs`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jpmrblood) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
