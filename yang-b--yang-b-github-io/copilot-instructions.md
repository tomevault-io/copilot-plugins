## yang-b-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is Bo Yang's personal website - a static portfolio website showcasing professional experience, publications, and travel photography.

## Deployment

The site is deployed to GitHub Pages automatically on push to `main`. All static files live at the repository root and are served directly.

- **URL**: https://yang-b.github.io/
- **Source**: GitHub Actions workflow (`.github/workflows/main_boyang.yml`)

To enable GitHub Pages:
1. Go to repository Settings > Pages
2. Under "Build and deployment", select "GitHub Actions" as the source

## Architecture

### Main Components

**Static Content** (repository root):
- HTML pages: `index.html`, `experience.html`, `publications.html`, `interest.html`
- CSS files in `/CSS/` directory
- Static assets in `/Figures/` (travel photos), `/Documents/` (CV, thesis, papers), `/Conference/` (presentations)

**Pages**:
- `index.html` - Home page with professional introduction and external links
- `experience.html` - Education history and work experience
- `publications.html` - Journal articles, patents, and conference papers
- `interest.html` - Travel photography gallery organized by year

**Key Frontend Features**:
- Google Analytics integration (UA-135500287-1)
- Responsive design with navigation between portfolio sections
- Clean, minimal HTML/CSS architecture

## Development Notes

- Pure static website with no backend dependencies
- No build process required
- To preview locally, use any static file server (e.g., `python -m http.server` from the repo root)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Yang-B) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
