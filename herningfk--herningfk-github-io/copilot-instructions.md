## herningfk-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Hugo static site for **Herning Faldskærmsklub** (Herning Skydiving Club). All content is in Danish. Hosted on GitHub Pages at herningfk.dk.

## Build and Development Commands

```bash
# Local development server (with draft posts)
hugo server --buildDrafts

# Production build (outputs to ./docs/)
HUGO_ENV=production hugo
```

Requires Hugo installed locally (`brew install hugo`).

## Deployment

There is no CI/CD. The site is built locally and the `docs/` directory is committed directly to git. GitHub Pages serves from the `docs/` folder on `main`. After making changes:

1. Run `HUGO_ENV=production hugo`
2. Commit the updated `docs/` directory along with source changes

The custom domain (`herningfk.dk`) is set via `docs/CNAME`.

## Architecture

- **Hugo config**: `hugo.toml` is the primary config. `config.toml` is a legacy leftover and can be ignored.
- **Theme**: Ananke v2.12.1 via Go Modules (`go.mod`). No git submodules despite `.gitmodules` existing (empty file).
- **Content**: All under `content/da/` (single language, Danish). Uses YAML front matter (`---`). Posts are in `content/da/post/`.
- **Custom shortcodes** in `layouts/shortcodes/`:
  - `button.html` — styled button with `text` and `link` params
  - `hfkform.html` — contact form using Formspree
  - `rawhtml.html` — raw HTML passthrough
- **Images**: Hosted externally on Cloudinary (`res.cloudinary.com/dzanurchx/`), not stored in the repo.
- **Static assets**: Favicons and PDFs (meeting minutes, bylaws) in `static/`.

## Content Conventions

- Front matter fields: `date`, `publishDate`, `lastmod`, `featured_image`, `author`, `tags`, `title`, `description`
- Use `<!--more-->` tag in posts for summary truncation
- The archetype (`archetypes/default.md`) uses TOML front matter, but existing content uses YAML — use YAML for consistency with existing posts
- External bookings link to Burble (`bookings.burblesoft.eu`)

## Commit and PR Conventions

- Conventional commits: `feat:`, `fix:`, `chore:`
- PR titles must also follow semantic commit format (e.g. `feat: add new page`)
- NEVER include `Co-Authored-By` or any other mention of Claude/AI in commits

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/herningfk)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/herningfk)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
