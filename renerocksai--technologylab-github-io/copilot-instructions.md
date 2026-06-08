## technologylab-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the website for technologylab.ai, built with [Zine](https://zine-ssg.io) - a Zig-based static site generator.

## Build Commands

```bash
# Local development (serves site with hot reload)
zine  # no further args

# Production build (outputs to public/)
zine release
```

## Architecture

**Zine Configuration**: `zine.ziggy` defines site metadata, directory paths, and static assets.

**Directory Structure**:
- `content/` - Page content in SuperMD format (`.smd` files)
- `layouts/` - Page templates (`.shtml` files)
- `layouts/templates/` - Base templates that layouts extend
- `assets/` - CSS, fonts, images
- `zig-out/` - Development build output (gitignored)
- `public/` - Production build output (gitignored)

**Template Hierarchy**: Layouts in `layouts/` extend `layouts/templates/base.shtml` using `<extend template="base.shtml">` syntax.

**Content Files**: Each `.smd` file has YAML-like frontmatter (using Ziggy syntax) specifying title, date, author, layout, and draft status.

## Deployment

Automatically deployed to GitHub Pages on push to `master` via `.github/workflows/gh-pages.yml`. Uses Zine v0.10.1.

---
> Source: [renerocksai/technologylab.github.io](https://github.com/renerocksai/technologylab.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-08 -->
