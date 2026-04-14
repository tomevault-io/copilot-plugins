## woodberry-website

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a community website for Woodberry Down Community Building England, featuring information about community programmes for children, youth, and adults. The site is built with Eleventy static site generator and uses Tailwind CSS for styling.

## Technology Stack

- **Static Site Generator**: Eleventy (11ty) v3.1.2
- **Templating**: Nunjucks (.njk files)
- **CSS Framework**: Tailwind CSS (loaded via CDN)
- **Forms**: Netlify Forms integration
- **Deployment**: Netlify
- **Python Tools**: uv package manager, Playwright for screenshots

## Architecture

### File Structure
- `src/` - Source files for Eleventy
  - `_includes/` - Nunjucks templates and partials
    - `base.njk` - Main layout template with Tailwind config and JavaScript
    - `header.njk` - Site navigation header
    - `footer.njk` - Site footer
  - `*.njk` - Page templates with frontmatter
- `_site/` - Generated static site output
- `archive/` - Legacy HTML files and conversion tools
- `docs/` - Project documentation and planning

### Template System
- All pages extend `base.njk` layout
- Frontmatter defines `title` and `description` for each page
- Base template includes:
  - Tailwind CSS configuration with custom colors (primary: #2563eb, secondary: #1e40af, accent: #f59e0b)
  - JavaScript for mobile menu, active page highlighting, and smooth scrolling
  - Responsive navigation with mobile hamburger menu

### Key Components
- Contact form with Netlify Forms integration (`data-netlify="true"`)
- Programme cards with consistent styling and hover effects
- Responsive grid layouts using Tailwind classes
- SVG icons throughout the interface

## Common Development Commands

```bash
# Development server with live reload
npm run dev

# Build static site
npm run build

# Python development (use uv)
uv sync
uv run python script.py

# Take website screenshots
npx playwright codegen
```

## Python Development

- Use uv as a package manager for any Python development
- Dependencies include BeautifulSoup4 and Playwright

## Web Development

- Use npx playwright for taking screenshots of websites
- Tailwind CSS classes should follow the existing color scheme and patterns
- All forms should include `data-netlify="true"` for Netlify Forms
- Maintain responsive design patterns established in existing templates

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ZGurney)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ZGurney)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
