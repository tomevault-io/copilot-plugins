## photos

> Static photo portfolio built with Eleventy 3, adapted from the Halide theme. Deployed to Cloudflare Pages.

# Portfolio

Static photo portfolio built with Eleventy 3, adapted from the Halide theme. Deployed to Cloudflare Pages.

## Tech stack

- **SSG**: Eleventy (11ty) v3
- **Templates**: Nunjucks
- **CSS**: Vanilla CSS (Halide-based design system)
- **Package manager**: Yarn 4 (node-modules linker)
- **Deployment**: Cloudflare Pages via wrangler (local deploy)

## Commands

```bash
yarn dev        # start dev server at localhost:8080
yarn build      # production build → _site/
yarn clean      # remove _site/
yarn deploy     # deploy _site/ to Cloudflare Pages
```

## First run

```bash
corepack enable
corepack use yarn@4
yarn install
```

## Adding a project

1. Create `projects/<name>/` folder
2. Drop TIFF (or JPEG/PNG) images in the folder — sorted alphabetically, first image = cover
3. Optionally add `projects/<name>/meta.yaml`:

```yaml
title: My Project Title   # defaults to folder name
description: Optional description (markdown supported)
year: 2024
```

The `projects/` folder is gitignored — images live only on the local machine.

## Architecture

- `_data/projects.js` — scans `projects/` at build time, returns array of project objects
- `project-page.njk` — pagination template, generates one page per project at `/project/<slug>/`
- `home.njk` — home page, lists all projects from global `projects` data
- `_includes/components/projectList.njk` — grid of project thumbnails

## Image processing

- Source: TIFF (preferred), JPEG, PNG
- Output: JPEG + WebP + AVIF (configured in `_data/settings.yaml`)
- First build is slow (TIFF → AVIF via Sharp); subsequent builds use `.cache/`
- Run `yarn clean` only when necessary (clears the image cache)

## Deployment

Images stay on the local machine and are never committed to git. Workflow:

```bash
yarn build && yarn deploy
```

First deploy requires:
```bash
wrangler login
wrangler pages project create
```

## Settings

Edit `_data/settings.yaml` to change:
- Site title, description, subtitles
- Image quality settings

---
> Source: [cinan/photos](https://github.com/cinan/photos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
