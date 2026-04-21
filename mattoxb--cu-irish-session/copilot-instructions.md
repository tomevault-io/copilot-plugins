## cu-irish-session

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a static website for the Champaign-Urbana Irish Music Session built using Quartz v4, a TypeScript-based static site generator. The site provides information about the session and hosts a collection of traditional Irish tune sheet music.

## Build Commands

```bash
# Install dependencies
npm ci

# Build the site (outputs to public/)
npx quartz build

# Build and serve locally with live reload
npx quartz build --serve

# Build docs specifically (alternative)
npm run docs

# Type checking
npm run check  # runs tsc --noEmit && prettier --check

# Format code
npm run format

# Run tests
npm test
```

## Development Requirements

- Node.js >= 22
- npm >= 10.9.2

## Architecture

### Core Configuration Files

- **quartz.config.ts**: Main Quartz configuration including plugins, theme, and site metadata
- **quartz.layout.ts**: Page layout definitions (shared, content pages, list pages)
  - Defines component placement (left sidebar, right sidebar, beforeBody, etc.)
  - Uses a component-based architecture where components are arranged by position

### Directory Structure

- **content/**: Markdown source files
  - **content/Music/**: Tune categorization pages (Reels.md, Jigs.md, Hornpipes.md, etc.)
  - **content/assets/music/**: PDF sheet music files
- **quartz/**: Quartz framework code
  - **quartz/components/**: Preact components (ArticleTitle, Explorer, Search, etc.)
  - **quartz/plugins/**: Build pipeline plugins
    - **transformers/**: Process markdown (FrontMatter, GitHubFlavoredMarkdown, Latex, etc.)
    - **filters/**: Filter content (RemoveDrafts)
    - **emitters/**: Generate output (ContentPage, Assets, Static, etc.)
  - **quartz/styles/**: SCSS stylesheets
- **public/**: Build output (generated, not versioned except via GitHub Pages)

### Plugin Architecture

Quartz uses a three-stage plugin system configured in quartz.config.ts:

1. **Transformers**: Process markdown content (run sequentially in order)
   - FrontMatter, CreatedModifiedDate, SyntaxHighlighting, ObsidianFlavoredMarkdown, GitHubFlavoredMarkdown, TableOfContents, CrawlLinks, Description, Latex

2. **Filters**: Exclude content from being published
   - RemoveDrafts

3. **Emitters**: Generate output files
   - ComponentResources, ContentPage, FolderPage, TagPage, ContentIndex, Assets, Static, Favicon, NotFoundPage, CustomOgImages

### Component System

Components are Preact-based (JSX) and placed in layout positions defined in quartz.layout.ts:
- **left**: PageTitle, Search, Darkmode, ReaderMode, Explorer
- **right**: TableOfContents, Backlinks (Graph commented out)
- **beforeBody**: Breadcrumbs, ArticleTitle, ContentMeta, TagList
- **shared**: Head, Footer

Components are imported from `./quartz/components` and configured in the layout file.

### Styling

Custom styles are in quartz/styles/ and use SCSS. The theme colors and typography are configured in quartz.config.ts under the `theme` key.

## Deployment

The site deploys to GitHub Pages via `.github/workflows/pages-deploy.yml`:
1. Runs on push to main
2. Installs dependencies with `npm ci`
3. Builds with `npx quartz build`
4. Deploys the `public/` directory to GitHub Pages

## Content Management

To add new sheet music:
1. Add PDF to content/assets/music/
2. Update the appropriate category page in content/Music/ (e.g., Reels.md)
3. Add alphabetically under the appropriate letter heading

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mattoxb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
