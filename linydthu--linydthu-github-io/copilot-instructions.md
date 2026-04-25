## linydthu-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an **al-folio** based academic portfolio website - a Jekyll theme for academics and researchers. It provides a professional portfolio with CV, publications, blog posts, projects, and news sections.

- **Site URL:** https://linydthu.github.io
- **Theme:** al-folio (Jekyll-based)
- **Deployment:** GitHub Pages via GitHub Actions

## Common Development Commands

### Local Development (Docker - Recommended)

```bash
# Start development server (runs at http://localhost:8080)
docker compose pull
docker compose up

# Rebuild with updated dependencies
docker compose up --build
docker compose up --force-recreate

# Use slim image (smaller, <100MB)
docker compose -f docker-compose-slim.yml up
```

### Local Development (Native - Not Recommended)

```bash
# Install dependencies
bundle install

# Run development server
bundle exec jekyll serve --port 4000
```

### Code Formatting (Mandatory for PRs)

```bash
# Install prettier
npm install --save-dev prettier @shopify/prettier-plugin-liquid

# Format all files (must run before committing)
npx prettier . --write
```

### Production Build

```bash
# Set environment and build
export JEKYLL_ENV=production
bundle exec jekyll build

# Purge unused CSS (for production)
npm install -g purgecss
purgecss -c purgecss.config.js
```

## Project Architecture

### Key Directories

| Directory | Purpose |
|-----------|---------|
| `_config.yml` | Primary site configuration (title, author, URLs, feature flags) |
| `_bibliography/papers.bib` | BibTeX bibliography for publications |
| `_data/` | YAML data files (socials, coauthors, cv, venues, repositories) |
| `_includes/` | Reusable Liquid template components |
| `_layouts/` | Page layout templates (about, post, bib, distill, cv) |
| `_news/` | News/announcement entries |
| `_pages/` | Static pages (about, cv, publications, projects, teaching) |
| `_posts/` | Blog posts (format: YYYY-MM-DD-title.md) |
| `_projects/` | Project showcase entries |
| `_sass/` | SCSS stylesheets |
| `_scripts/` | JavaScript files |
| `_teachings/` | Course and teaching entries |
| `_plugins/` | Custom Ruby plugins for Jekyll |
| `assets/img/` | Images and profile pictures |

### Content Collections

Collections are defined in `_config.yml` under the `collections` key. Each collection has its own directory prefixed with underscore (e.g., `_news`, `_projects`).

### Key Plugins (from Gemfile)

- `jekyll-scholar` - Bibliography management from BibTeX
- `jekyll-archives-v2` - Archive page generation
- `jekyll-paginate-v2` - Pagination support
- `jekyll-jupyter-notebook` - Jupyter notebook embedding
- `jekyll-tabs` - Tab UI components
- `jekyll-toc` - Table of contents generation
- `classifier-reborn` - Related posts calculation

## CI/CD Pipeline

### GitHub Actions Workflows (`.github/workflows/`)

- **deploy.yml** - Main deployment workflow
  - Triggers on push/PR to main/master (excluding documentation-only changes)
  - Sets up Ruby 3.3.5, Python 3.13, ImageMagick
  - Builds with JEKYLL_ENV=production
  - Runs purgecss for CSS optimization
  - Deploys to gh-pages branch

- **prettier.yml** - Code formatting validation (mandatory for PRs)
- **broken-links.yml** - Link validation
- **axe.yml** - Accessibility testing
- **update-citations.yml** - Automatic Google Scholar citation updates

### Build Requirements

- **Ruby:** 3.3.5 (primary), 3.2.2 (some workflows)
- **Python:** 3.13 (for nbconvert)
- **ImageMagick:** Required for image processing
- **Node.js:** For purgecss and prettier

## Common Configuration Changes

### _config.yml Key Settings

```yaml
title: Donglin Yang
first_name: Donglin
last_name: Yang
url: https://linydthu.github.io
baseurl:  # Leave empty for user site
navbar_fixed: true
footer_fixed: true
search_enabled: true
```

### Content File Formats

**Blog Post Frontmatter (`_posts/`):**
```yaml
---
layout: post
title: Post Title
date: YYYY-MM-DD HH:MM:SS
categories: category-name
---
```

**Project Frontmatter (`_projects/`):**
```yaml
---
layout: page
title: Project Name
description: Short description
img: /assets/img/project-image.jpg
importance: 1  # Display order
---
```

**News Entry (`_news/`):**
```yaml
---
layout: post
date: YYYY-MM-DD HH:MM:SS
inline: true  # If true, shown inline on about page
related_posts: false
---
News content here
```

## Common Pitfalls

### YAML Syntax in _config.yml
- Quote strings containing special characters (`:`, `&`, `#`): `title: "My: Title"`

### URL/baseurl Configuration
- Personal site: `url: https://username.github.io`, `baseurl:` (empty)
- Project site: `url: https://username.github.io`, `baseurl: /repo-name/`

### CSS/JS Not Loading
- Verify `url` and `baseurl` are correct in `_config.yml`
- Clear browser cache (Ctrl+Shift+Del)

### Related Posts Error
- "Zero vectors cannot be normalized": Add meaningful content to posts or set `related_posts: false`

### Port Already in Use
- Docker: `docker compose down && docker compose up`
- Native: `lsof -i :4000 | grep LISTEN | awk '{print $2}' | xargs kill`

## Pre-commit Checklist

Before pushing changes:

1. Run prettier: `npx prettier . --write`
2. Test locally: `docker compose up` and verify at http://localhost:8080
3. Check for Jekyll build errors in the output

## External Services Integration

- **Giscus:** Comments system (configured via `_config.yml` giscus section)
- **Google Analytics:** Add measurement ID (format: G-XXXXXXXXXX)
- **GitHub Stats:** Uses `github-readme-stats.vercel.app` for repository stats
- **Google Scholar:** Automatic citation updates via workflow

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/linYDTHU) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
