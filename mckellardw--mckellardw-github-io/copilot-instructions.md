## mckellardw-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal academic/professional website for David W. McKellar built with Hugo and deployed to GitHub Pages. Live at https://mckellardw.github.io/

## Development Commands

Local development uses a conda environment named `hugo`:

```bash
# Activate conda environment
conda activate hugo

# Run local development server
bash serve.sh
# Or manually:
hugo server -D

# Production build
hugo --gc --minify
```

## Deployment

Automated via GitHub Actions on push to `main` branch. The workflow (`.github/workflows/hugo.yaml`) builds with Hugo v0.148.1 extended and Dart Sass v1.89.2.

## Architecture

**Hugo Theme**: `hugo-paper` (git submodule in `themes/hugo-paper/`)
- Clone with `--recursive` to fetch theme

**Custom Layouts** (override theme defaults):
- `layouts/index.html` - Custom homepage with profile styling
- `layouts/shortcodes/gcal_button.html` - Google Calendar scheduling button

**Content Structure**:
- `content/_index.md` - Homepage content
- `content/publications/index.md` - Publications page
- `static/images/` - Profile pictures and graphics
- `static/pdfs/` - Research PDFs

## Configuration

Main config in `config.toml`:
- Theme: `hugo-paper` with dark color scheme
- Markdown rendering allows raw HTML (`unsafe = true`)
- Single menu item: Publications

## Custom Shortcode

Google Calendar button usage in markdown:
```
{{< gcal_button url="https://calendar.google.com/..." color="#039BE5" label="Book a call" >}}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mckellardw) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
