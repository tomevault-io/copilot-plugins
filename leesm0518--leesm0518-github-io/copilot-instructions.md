## leesm0518-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Jekyll-based blog using the Chirpy theme for technical writing and daily logs. The site is hosted on GitHub Pages and focuses on documenting experiences and learnings in Korean.

- **Site**: https://leesm0518.github.io
- **Theme**: jekyll-theme-chirpy v7.1.1
- **Language**: Korean (ko)
- **Timezone**: Asia/Seoul

## Build and Development Commands

```bash
# Build the entire project (CSS + JS)
npm run build

# Build CSS only (with PurgeCSS)
npm run build:css

# Build JavaScript for production
npm run build:js

# Watch JavaScript files during development
npm run watch:js

# Lint SCSS files
npm run lint:scss

# Fix SCSS linting issues
npm run lint:fix:scss

# Run tests (currently just SCSS linting)
npm run test
```

## Jekyll Commands

```bash
# Install Ruby dependencies
bundle install

# Serve the site locally for development
bundle exec jekyll serve

# Build the site
bundle exec jekyll build

# Test the site with HTML Proofer
bundle exec htmlproofer _site
```

## Architecture and Structure

### Core Directories
- `_posts/`: Blog posts in Markdown format (YYYY-MM-DD-title.md)
- `_layouts/`: HTML templates for different page types
- `_includes/`: Reusable HTML components
- `_sass/`: SCSS stylesheets
- `_javascript/`: JavaScript modules and components
- `_data/`: YAML configuration files (authors, locales, etc.)
- `_site/`: Generated static site (Git ignored)
- `assets/`: Static assets (images, CSS, JS)
- `templates/`: Post and content templates

### JavaScript Architecture
- Modular JavaScript using Rollup for bundling
- Component-based structure in `_javascript/modules/components/`
- Layout-specific modules in `_javascript/modules/layouts/`
- Page-specific entry points (home.js, post.js, page.js, etc.)

### SCSS Architecture
- Main styles in `_sass/main.scss`
- Component styles in `_sass/addon/`
- Layout-specific styles in `_sass/layout/`
- Color schemes in `_sass/colors/`

### Content Structure
- Posts use Jekyll front matter with categories and tags
- Multi-language support with locale files in `_data/locales/`
- Custom post templates available in `templates/`

## Development Notes

- The site uses PurgeCSS to optimize CSS for production
- JavaScript is bundled with Rollup and Babel for ES6+ support
- SCSS follows the stylelint-config-standard-scss configuration
- Commit messages follow conventional commits format
- The site includes PWA capabilities with service worker
- SEO optimized with jekyll-seo-tag plugin
- Uses Husky for Git hooks and commitlint for conventional commits
- Bootstrap CSS is optimized via PurgeCSS configuration

## Local Development Workflow

```bash
# Full development setup
bundle install && npm install

# Start local development server (Jekyll)
bundle exec jekyll serve

# Watch and build JavaScript during development
npm run watch:js

# Run all linting
npm run test
```

## Build Configuration

- **Rollup**: Configured for modular JavaScript bundling with Babel transpilation
- **PurgeCSS**: Optimizes Bootstrap CSS by removing unused classes
- **Jekyll**: Static site generation with Chirpy theme integration
- **Production builds**: Include minification and banner injection for licensing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LeeSM0518) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
