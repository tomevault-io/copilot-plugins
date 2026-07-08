## personal-portfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A [Quarto](https://quarto.org) website: Linh Ngo's personal portfolio/blog. Content is authored in `.qmd` files (Markdown + optional embedded R code chunks) and rendered to static HTML.

## Commands

- Render the full site: `quarto render`
- Live preview with auto-reload: `quarto preview`
- Render a single page (faster iteration): `quarto render posts/<slug>/index.qmd`

There is no test suite, linter, or CI configured for this repo — rendering successfully is the correctness check.

## Architecture

- **`_quarto.yml`** is the site config: navbar, theme (`litera`), and `project.output-dir: docs`. Rendered output is written to `docs/`, which is what gets served (e.g. via GitHub Pages). A separate `_site/` directory also exists in the repo from an earlier config and is stale — don't treat it as the render target.
- **`index.qmd`** is the homepage (About-style landing page using Quarto's `trestles` about-template).
- **`blog.qmd`** is the blog listing page. It uses Quarto's `listing` feature to auto-populate from `posts/*/index.qmd`, plus one manually hardcoded listing entry that links out to an external GitHub repo (not a local post) — check this file when adding/removing posts, since external entries need manual upkeep.
- **`posts/<slug>/`** — each post is self-contained in its own directory with `index.qmd` plus any local images, data files, or citation assets (`citations/*.bib`, `*.csl`) it needs. Follow this per-post-directory convention when adding a new post.
- **`posts/_metadata.yml`** applies shared front-matter to every post, notably `freeze: true` (see [Quarto freeze](https://quarto.org/docs/projects/code-execution.html#freeze)) and `title-block-banner: true`. Computational output for posts with R code chunks is cached under `_freeze/` and only re-executed when the source changes — if a post's rendered output looks stale after an edit, check whether `_freeze/` needs to be regenerated via `quarto render`.
- Posts that include R analysis (e.g. `posts/price_to_safety_index_melb/`) use knitr code chunks with the tidyverse/geospatial R stack (`tidyverse`, `sf`, `ggplot2`, `ggiraph`, `DT`, `kableExtra`, `sugarbag`). Other posts (e.g. `posts/prefviz/`, `posts/mixpanel-implementation/`) are narrative-only with no executable code.
- **`styles.css`** holds custom theme overrides on top of the `litera` Quarto theme.
- `docs/`, `_site/`, and `_freeze/` are all committed to git (only `.quarto/` is gitignored) — this is a static-output-in-repo setup, so rendered artifacts are expected to be part of commits touching content.

---
> Source: [linhngo66/personal-portfolio](https://github.com/linhngo66/personal-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
