## thisislegit-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Jekyll-based academic website built on GitHub Pages using the academicpages template (forked from Minimal Mistakes Jekyll Theme). It serves as a personal academic portfolio for Xu Chen, a Ph.D. student at University of Electronic Science and Technology of China.

## Development Commands

### Local Development
```bash
# Install Ruby dependencies
bundle install

# Clean up directory (if needed)
bundle clean

# Serve locally with live reload
bundle exec jekyll serve
```

The local server will automatically rebuild and refresh pages on change at `localhost:4000`.

### Build Assets
```bash
# Build minified JavaScript
npm run build:js

# Watch for JS changes and rebuild
npm run watch:js
```

## Site Architecture

### Jekyll Structure
- `_config.yml` - Main site configuration
- `_config.dev.yml` - Development-specific overrides
- `_layouts/` - Page templates (default, single, archive, talk, etc.)
- `_includes/` - Reusable components and partials
- `_sass/` - SCSS stylesheets
- `_data/` - Site data files (navigation, UI text, comments)

### Content Collections
The site uses Jekyll collections for organized content:
- `_pages/` - Static pages (about, CV, publications, etc.)
- `_posts/` - Blog posts
- `_publications/` - Academic publications
- `_talks/` - Conference talks and presentations
- `_teaching/` - Teaching experience
- `_portfolio/` - Portfolio items

### Content Generation
The `markdown_generator/` folder contains Jupyter notebooks and Python scripts to generate markdown files from TSV data for publications and talks. This allows bulk import of structured academic content.

### Key Configuration
- Site owner: Xu Chen (Thisislegit.github.io)
- Theme: Modified Minimal Mistakes
- Analytics: Google Universal Analytics
- Collections have specific permalinks and layouts defined in `_config.yml`
- Author profile data configured in `_config.yml` under `author:`

### Dependencies
- Ruby gems managed via Gemfile (github-pages, jekyll-feed, jekyll-sitemap, hawkins)
- JavaScript dependencies managed via package.json for build tools
- Uses GitHub Pages compatible plugins only

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Thisislegit) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
