## davecurry-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Site Overview

This is a personal Hugo static site for Dave Curry (davecurry.net), built with the PaperMod theme. The site serves as both a professional portfolio and personal blog for a Sr. Design Director at Warner Bros. Discovery.

## Development Commands

### Local Development
- `hugo server` - Start development server (typically on http://localhost:1313)
- `hugo server -D` - Start server including draft posts
- `hugo server --disableFastRender` - Start server without fast render for theme development

### Building
- `hugo` - Build production site to `./public/`
- `hugo --minify` - Build with minification (production build)
- `hugo --baseURL "https://davecurry.net/"` - Build with specific base URL

### Content Management
- `hugo new posts/post-name.md` - Create new blog post
- `hugo new pages/page-name.md` - Create new page

## Architecture

### Site Configuration
- **Main config**: `config.yml` (not config.toml)
- **Theme**: PaperMod (as git submodule in `themes/PaperMod/`)
- **Base URL**: https://davecurry.net (recently migrated from evad.lol)

### Content Structure
- **Posts**: `content/posts/` - Blog posts with frontmatter (title, date, tags, draft)
- **Pages**: `content/pages/` - Static pages like About
- **Static files**: `static/` - Images, favicons, etc.

### Theme Configuration
The site uses PaperMod's "Home-Info Mode" with:
- Custom home page content in `config.yml` under `homeInfoParams`
- Professional title and description
- LinkedIn social icon
- Disabled sharing buttons and post navigation
- Enabled reading time, breadcrumbs, and code copy buttons

### Deployment
- **Platform**: GitHub Pages via GitHub Actions
- **Workflow**: `.github/workflows/hugo.yml`
- **Hugo Version**: 0.148.2 (specified in workflow)
- **Build**: Automatic on pushes to `main` branch

### Content Patterns
- Posts use tags like "working well", "productivity", "writing"
- Dates in ISO format with timezone (e.g., `2024-04-25T11:57:19-07:00`)
- Draft status controlled by `draft: false/true` in frontmatter
- Professional focus on design, technology, and work setup content

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davecurry) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
