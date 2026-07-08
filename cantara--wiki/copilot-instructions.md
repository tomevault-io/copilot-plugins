## wiki

> Archive of the Cantara open source community wiki, originally hosted at wiki.cantara.no (Confluence). Recovered from the Wayback Machine and converted to a static MkDocs site hosted on GitHub Pages.

# wiki

## Purpose
Archive of the Cantara open source community wiki, originally hosted at wiki.cantara.no (Confluence). Recovered from the Wayback Machine and converted to a static MkDocs site hosted on GitHub Pages.

## Tech Stack
- Language: Markdown
- Framework: MkDocs with Material theme
- Build: MkDocs (`mkdocs build`)
- Hosting: GitHub Pages at https://cantara.github.io/wiki/

## Architecture
Static documentation site built with MkDocs. Contains archived wiki pages from the original Cantara Confluence instance, organized into a navigable documentation structure. Includes custom overrides and build scripts.

## Key Entry Points
- `docs/` - Markdown documentation source
- `mkdocs.yml` - MkDocs configuration
- `overrides/` - Theme customizations
- `scripts/` - Build and maintenance scripts
- `site/` - Generated static site

## Development
```bash
# Install dependencies
pip install mkdocs mkdocs-material

# Local development
mkdocs serve

# Build
mkdocs build
```

## Domain Context
Community documentation. Preserves the historical Cantara community wiki content covering architecture decisions, installation guides, and development documentation for all Cantara projects.

---
> Source: [Cantara/wiki](https://github.com/Cantara/wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
