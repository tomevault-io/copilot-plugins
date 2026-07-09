## ariel-villanea

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Villanea** is a professional portfolio and documentation website built with Docusaurus v3.9.2. It serves as a platform to showcase projects, write blog posts, and host comprehensive documentation.

**Deployment:**
- Production URL: https://villanea.com
- GitHub Pages (fallback): https://ariel-villanea.github.io/ariel-villanea/
- Organization: ariel-villanea
- Project: ariel-villanea

## Essential Commands

### Development
```bash
npm start              # Start development server at http://localhost:3000/
npm run build          # Generate static site in build/
npm run serve          # Serve built site locally
npm run typecheck      # Run TypeScript type checking
npm run clear          # Clear Docusaurus cache
```

### Deployment
```bash
npm run deploy         # Deploy to GitHub Pages
```

### Docusaurus Utilities
```bash
npm run swizzle                    # Customize Docusaurus components
npm run write-translations         # Generate translation files
npm run write-heading-ids          # Add IDs to markdown headings
```

## Architecture

### Configuration System
- **docusaurus.config.ts**: Main site configuration (title, URL, theme, navbar, footer, plugins)
- **sidebars.ts**: Documentation sidebar configuration (currently auto-generated from docs/ folder structure)
- **tsconfig.json**: TypeScript configuration (extends @docusaurus/tsconfig)

### Content Organization
- **docs/**: Documentation files (MDX/Markdown) - automatically generates sidebar navigation
- **blog/**: Blog posts with metadata (authors.yml, tags.yml)
- **src/pages/**: Custom React pages (index.tsx is the homepage)
- **src/components/**: Reusable React components (e.g., HomepageFeatures)
- **src/css/**: Custom stylesheets (custom.css)
- **static/**: Static assets served directly (images, favicons, etc.)

### Key Technical Details
- **React 19.0** + **TypeScript 5.6.2**
- **MDX Support**: Write JSX in Markdown for docs and blog
- **Theme**: Docusaurus Classic preset with dark mode support (respects system preference)
- **Syntax Highlighting**: Prism with GitHub light/Dracula dark themes
- **Node.js**: Requires >= 20.0

### Component Structure
- Homepage uses `Layout` from `@theme/Layout` wrapper
- `HomepageFeatures` component displays three-column feature grid
- Type-safe imports using `@site/` alias for project root
- CSS Modules for component-scoped styling
- **src/theme/Root.tsx**: Custom theme wrapper for GTM noscript fallback

### Content Frontmatter
Blog posts and docs support MDX frontmatter for metadata (title, author, tags, date, etc.). Docusaurus enforces best practices with warnings for inline tags/authors and untruncated blog posts.

### SEO & Analytics
- **Sitemap**: Auto-generated at `/sitemap.xml` via Docusaurus sitemap plugin (configured in docusaurus.config.ts)
- **JSON-LD Structured Data**: Implemented for SEO in `src/pages/index.tsx` (Person, WebSite schemas) and `docs/about-me.mdx` (ProfilePage schema)
- **Google Tag Manager**: Container GTM-NGKD6Z9F loaded via `headTags` in docusaurus.config.ts with noscript fallback in Root.tsx

## Development Notes

- Built files go to `build/` and `.docusaurus/` (both gitignored)
- Development server supports hot reloading
- Edit links point to GitHub repository (main branch)
- Blog includes RSS/Atom feeds with XSLT
- Search functionality is built-in via Docusaurus

## Content Writing Guidelines

**Markdown Line Breaks:**
When editing Markdown/MDX files in `docs/` or `blog/`, always add **two spaces** at the end of a line when a single line break is expected in the rendered output. Without the two trailing spaces, Markdown will merge consecutive lines into a single paragraph.

Example:
```markdown
**Duration:** February 2013 – October 2016··
**Employment Type:** Full-time · On-site
```
(where ·· represents two spaces)

---
> Source: [ariel-villanea/ariel-villanea](https://github.com/ariel-villanea/ariel-villanea) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
