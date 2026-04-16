## medstaffblog

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Astro-based medical staff newsletter website for Hannibal Health (HH Med Staff News). The site displays medical staff news articles and updates for healthcare professionals. It's built using Astro v1.0.8 with React integration and is deployed to GitHub Pages at `https://hemepositive.github.io`.

## Development Commands

| Command | Action |
|---------|--------|
| `npm install` | Installs dependencies |
| `npm run dev` | Starts local dev server at `localhost:3000` |
| `npm run build` | Build production site to `./dist/` |
| `npm run preview` | Preview build locally before deploying |
| `npm run astro ...` | Run CLI commands like `astro add`, `astro preview` |

## Architecture

### Content Management
- **Blog Posts**: Located in `src/posts/*.md` - Markdown files with frontmatter containing title, slug, excerpt, date, and author
- **Static Pages**: Located in `src/pages/*.astro` and `src/pages/*.md` for custom pages
- **Dynamic Routing**: `src/pages/[slug].astro` generates individual post pages from markdown files

### Key Components
- **Layout.astro**: Main layout template with Header/Footer and global CSS imports
- **Components**: Reusable Astro components in `src/components/` (Card, Header, Footer, Showcase, Features, Tabs)
- **Styling**: Global CSS in `src/styles/global.css`, component-specific styles are scoped within components

### Content Structure
- **Frontmatter Format**: Each post requires `title`, `slug`, `excerpt`, `date`, and `author` fields
- **Post Listing**: Homepage (`index.astro`) automatically lists all posts sorted by date (newest first)
- **Static Assets**: Images and files stored in `public/` directory, accessible via root URL paths

### Build Configuration
- **Site URL**: Configured for GitHub Pages deployment at `https://hemepositive.github.io`
- **TypeScript**: Basic TypeScript support with ESNext target, no transpilation (Astro handles directly)
- **No Linting/Testing**: This project doesn't include lint or test scripts

## Working with Posts

To add a new post:
1. Create a new `.md` file in `src/posts/`
2. Include required frontmatter fields
3. Write content in Markdown below the frontmatter
4. The post will automatically appear on the homepage and be accessible via its slug

Example frontmatter:
```yaml
---
title: Post Title
slug: post-slug
excerpt: Brief description for the homepage listing
date: YYYY-MM-DD
author: Author Name
---
```

## File Structure Notes

- `public/files/` contains PDF documents referenced in posts
- `src/images/` contains site-specific images used in components
- `astro-website-theme/` appears to be reference materials from the original theme

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hemepositive) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
