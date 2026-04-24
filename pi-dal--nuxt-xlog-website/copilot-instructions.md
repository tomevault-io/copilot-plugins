## nuxt-xlog-website

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **static website generator** for xLog blogs built with Vue 3 + Vite, using the vite-ssg plugin for static site generation. The project syncs content from xLog (a decentralized blogging platform) and generates an optimized static website.

**Key Architecture:**

- **Vue 3 + Vite**: Modern frontend framework with fast build tooling
- **vite-ssg**: Static site generation for Vue/Vite applications
- **File-based routing**: Uses `unplugin-vue-router` with pages in the `pages/` directory
- **Markdown support**: Full markdown processing with frontmatter, syntax highlighting, and Vue components
- **xLog integration**: Fetches blog content directly from the Crossbell xLog GraphQL API
- **UnoCSS**: Atomic CSS framework for styling
- **TypeScript**: Full type safety throughout the codebase

## Development Commands

```bash
# Start development server (opens on http://localhost:3333)
pnpm dev

# Build for production (includes OG image generation, font copying, RSS generation, and redirects)
pnpm build

# Preview production build
pnpm preview

# Run linting (uses @antfu/eslint-config with automatic fixes)
pnpm lint

# Image compression (for photos using Sharp)
pnpm compress

# Photo management (rename/organize photos with EXIF data)
pnpm photos

# Prepare git hooks
pnpm prepare
```

## Build Process

The build command runs multiple steps:

1. `tsx ./scripts/generate-xlog-og.ts` - Generate Open Graph images for xLog posts
2. `vite-ssg build` - Static site generation with Vue 3 + Vite
3. `tsx ./scripts/copy-fonts.ts` - Copy font files to public assets
4. `tsx ./scripts/rss.ts` - Generate RSS/Atom feeds (XML, Atom, JSON formats)
5. `cp _redirects dist/_redirects` - Copy redirect rules for deployment

> `pnpm build` exports `NO_WEBFONT_FETCH=1` so UnoCSS skips remote Google Fonts downloads during CI/SSG runs. Unset this variable locally if you prefer UnoCSS to fetch web fonts on demand.

## Configuration

### xLog Configuration

The site can be configured via two methods:

1. **Web interface**: Visit `/config` to set xLog handle through UI
2. **Environment variable**: Set `XLOG_HANDLE` in `.env` file

The xLog handle is stored in localStorage (client-side) and falls back to environment variables (server-side).

### Key Configuration Files

- `vite.config.ts`: Main build configuration with extensive markdown processing, plugins, and SSG setup
- `unocss.config.ts`: CSS framework configuration with custom shortcuts and web fonts
- `tsconfig.json`: TypeScript configuration with strict type checking
- `eslint.config.js`: Uses @antfu/eslint-config with custom rule overrides
- `package.json`: Uses catalog entries for dependency version management with pnpm workspaces

## Architecture Details

### Data Flow

1. **xLog API**: Content is fetched via the Crossbell GraphQL indexer, no SDK required
2. **Local Storage**: xLog handle configuration is stored client-side
3. **Static Generation**: vite-ssg pre-renders pages at build time
4. **Markdown Processing**: Full markdown pipeline with syntax highlighting and Vue components

### Key Directories

- `src/components/`: Vue components including xLog-specific components
- `src/logics/`: Business logic and API clients for xLog integration
- `pages/`: File-based routing with both `.vue` and `.md` files
- `scripts/`: Build scripts for RSS, image processing, and font management
- `public/`: Static assets including generated OG images and fonts

### xLog Integration

- **API Client**: GraphQL helpers in `src/logics/xlog-direct.ts` with caching and resilience
- **Direct API**: `src/logics/xlog-direct.ts` - Direct API calls for operations not covered by SDK
- **Site State**: `src/logics/site.ts` - Global site information management
- **Types**: `src/types.ts` - Comprehensive TypeScript interfaces for xLog data structures, including raw API types and enhanced post types
- **Error Handling**: `src/logics/errors.ts` - Centralized error handling with retry logic and timeouts
- **Caching**: `src/logics/cache.ts` - In-memory caching system with configurable TTL
- **Metadata**: `src/logics/metadata.ts` - Post enhancement system for adding extra metadata to xLog posts

### Markdown Processing

The markdown pipeline includes:

- **Shiki**: Syntax highlighting with dual themes (light/dark)
- **Frontmatter**: YAML metadata processing
- **Anchor links**: Automatic heading anchors
- **Table of contents**: Auto-generated TOC
- **GitHub Alerts**: Support for GitHub-style alerts
- **Magic Links**: Automatic link enhancement for known repositories

### Image Handling

- **Photo Management**: Automated photo processing with EXIF data extraction
- **Compression**: Sharp-based image compression with quality optimization
- **OG Images**: Automatic Open Graph image generation for posts
- **Image Preview**: Click-to-zoom functionality for images in posts

## Development Notes

### Package Management

- Uses **pnpm** as package manager
- Configured with workspace support via `pnpm-workspace.yaml`
- Uses catalog entries for dependency version management

### Git Hooks

- Pre-commit hooks run `lint-staged` which applies ESLint fixes to all files
- Configured via `simple-git-hooks` with automatic setup on `pnpm prepare`
- ESLint configuration removes several default rules for more flexible development

### Error Handling

- xLog API calls are wrapped with comprehensive error handling
- Graceful fallbacks for missing content or API failures
- Client-side configuration validation

### Performance Optimization

- Static site generation for fast loading
- Image compression and optimization
- Font subsetting and local font loading
- CSS atomic classes for minimal bundle size

## Testing & Deployment

### Local Testing

Run `pnpm dev` to start the development server on port 3333 (opens automatically). The `/config` page allows testing xLog connectivity and handle configuration.

### Build Validation

After running `pnpm build`, verify:

- Static files are generated in `dist/`
- RSS feeds are created (`feed.xml`, `feed.atom`, `feed.json`)
- Font files are copied to `public/assets/fonts/`
- Redirects file is in place

### Deployment

The `dist/` directory can be deployed to any static hosting service. The build process is optimized for platforms like Vercel, Netlify, or GitHub Pages.

## Important Development Practices

### Type Safety

- All xLog API operations use strict TypeScript interfaces defined in `src/types.ts`
- Raw API responses are transformed to ensure consistent data structures
- Enhanced post types support metadata supplements for extended functionality

### Error Handling Pattern

- Use `withErrorHandling()` wrapper for all async operations that may fail
- Implement graceful fallbacks for missing data or API failures
- Log errors with context using the centralized logger system

### Caching Strategy

- API responses are cached using `withCache()` with configurable TTL values
- Cache keys include relevant parameters to ensure proper invalidation
- Different cache durations for different data types (site info, posts, etc.)

### Performance Monitoring

- Use `perfTracker.measure()` to track operation performance
- API timeouts and retry logic are configured per operation type
- Build process includes performance optimizations for fonts, images, and assets

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pi-dal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
