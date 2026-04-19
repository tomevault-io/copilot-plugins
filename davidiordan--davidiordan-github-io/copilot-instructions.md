## davidiordan-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a static personal portfolio website hosted on GitHub Pages. There is no build system, package manager, or server-side code — all files are served directly as-is.

## Development

Open `index.html` in a browser to preview locally. For accurate rendering of any path-relative assets, serve with a local HTTP server:

```bash
python3 -m http.server 8000
```

Deploying is done by pushing to the `master` branch; GitHub Pages serves the site automatically.

## Architecture

The entire site is a **single-page application** contained in `index.html`. The design is a man page / CLI terminal aesthetic with no images, no third-party JS libraries, and minimal CSS.

- **`css/style.css`**: The only stylesheet. Uses CSS custom properties for theming — no frameworks.
- All JS is inlined at the bottom of `index.html` (dark/light toggle + `localStorage` persistence). There are no external JS dependencies.
- The files in `js/` and `css/plugins.css` are legacy and no longer used by the site.

## Theming

Two themes toggled by a fixed `[light]` / `[dark]` button; preference is saved in `localStorage` under the key `theme`. The `<body>` carries either class `dark` (default) or `light`.

CSS custom properties in `css/style.css`:

| Variable    | Purpose                        |
|-------------|--------------------------------|
| `--bg`      | Page background                |
| `--fg`      | Body text                      |
| `--bold`    | Bold text and `<b>` tags       |
| `--heading` | Section headings (`h2`)        |
| `--link`    | Anchor color                   |
| `--muted`   | De-emphasized text (toggle btn)|

Colors are sourced from Xcode's default dark/light editor themes:
- **Dark**: `#1e1e1e` bg, `#ffffff` text, `#67c7a4` headings, `#5dd8ff` links, `#6c7986` muted
- **Light**: `#ffffff` bg, `#000000` text, `#1a7f37` headings, `#0b4f79` links, `#5d6c79` muted

## Content Structure

`index.html` is structured as a Unix man page with these sections (in order):

- Man page header/footer: `DAVID(1)` — `User Commands` — `DAVID(1)`
- `NAME` — one-line description
- `SYNOPSIS` — flag-style summary of links
- `DESCRIPTION` — about / bio
- `EXPERIENCE` — work history
- `PORTFOLIO` — past projects with links
- `OTHER VENTURES` — personal ventures
- `SEE ALSO` — social/profile links
- `AUTHOR`

## Key Conventions

- Indentation follows man page convention: `.i1` = 7ch (section body), `.i2` = 14ch (sub-entries).
- Font is `'Courier New', Courier, monospace` throughout.
- No images are used anywhere in the layout.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davidiordan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
