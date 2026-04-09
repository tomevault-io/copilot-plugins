## rafael-sgoncalves-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Academic personal website for Rafael S. Gonçalves (Economics PhD, Princeton). Built with **Jekyll** using the **al-folio** theme. Deployed to GitHub Pages at `rafael-sgoncalves.github.io`.

## Build & Serve

```bash
# Local development
bundle install
bundle exec jekyll serve --lsi

# Docker alternative (serves on port 8080)
docker-compose up

# Production build (used in CI)
JEKYLL_ENV=production bundle exec jekyll build
```

The `--lsi` flag enables latent semantic indexing for related posts (requires `classifier-reborn` gem). The CI build in `.github/workflows/deploy.yml` does not use `--lsi`.

## Deployment

Automatic via GitHub Actions on push to `master` or `source` branches. The workflow builds with Ruby 3.2.2 and deploys the `_site/` output to the `gh-pages` branch.

## Architecture

### Content System

| Content type | Location | Format |
|---|---|---|
| Publications | `_bibliography/papers.bib` | BibTeX |
| Pages | `_pages/*.md` | Markdown + YAML front matter |
| News/announcements | `_news/*.md` | Markdown (output: false) |
| Projects | `_projects/*.md` | Markdown collection |
| Blog posts | `_posts/YYYY-MM-DD-slug.md` | Markdown |
| CV | `assets/pdf/` | PDF file |
| Data files | `_data/` | YAML (coauthors, CV, venues) |

### Publications (jekyll-scholar)

The publication system is the most complex part of the site. It is powered by `jekyll-scholar` and configured in `_config.yml` under `scholar:`.

- BibTeX entries live in `_bibliography/papers.bib`
- The author's name is bolded via `scholar.last_name` / `scholar.first_name` in config
- Coauthor links are configured in `_data/coauthors.yml`
- Custom BibTeX fields: `abstract`, `arxiv`, `bibtex_show`, `html`, `pdf`, `supp`, `blog`, `code`, `poster`, `slides`, `website`, `preview`, `selected`, `altmetric`
- `selected: true` in a BibTeX entry makes it appear on the homepage
- Rendering template: `_layouts/bib.html`
- Filtered keywords (hidden from rendered BibTeX): listed in `_config.yml` under `filtered_bibtex_keywords`

### Layouts

- `_layouts/about.html` — Homepage (profile, news, selected papers)
- `_layouts/bib.html` — Individual publication rendering
- `_layouts/post.html` — Blog posts
- `_layouts/distill.html` — Distill.pub-style scholarly articles

### Custom Plugins (`_plugins/`)

- `details.rb` — Enhanced `<details>` element
- `external-posts.rb` — External blog post sources (e.g., Medium RSS)
- `hideCustomBibtex.rb` — Filters custom BibTeX keywords from output

### Styling

- SCSS files in `_sass/` (variables, themes, custom styles)
- Bootstrap 4.6.1 grid system
- Light/dark mode toggle enabled (`enable_darkmode: true`)
- Theme color customizable via `--global-theme-color` CSS variable

## Key Configuration (`_config.yml`)

- Blog navigation is **disabled** (`blog_nav: false`)
- Announcements are **disabled** (`announcements.enabled: false`)
- MathJax is **enabled** (`enable_math: true`)
- Masonry layout for projects is **enabled** (`enable_masonry: true`)
- Google Analytics: enabled with ID `G--27B5YS472L`

## Common Tasks

- **Add a publication**: Add BibTeX entry to `_bibliography/papers.bib`. Use `selected: true` for homepage visibility.
- **Edit homepage**: Modify `_pages/about.md` (content) or `_layouts/about.html` (structure).
- **Add coauthor link**: Add entry to `_data/coauthors.yml` with name and URL.
- **Update CV PDF**: Replace file in `assets/pdf/`.
- **Modify site metadata**: Edit top of `_config.yml` (requires server restart).

## Gotchas

- Changes to `_config.yml` require restarting the Jekyll server.
- BibTeX syntax errors in `papers.bib` will break the entire build silently or with cryptic errors.
- The abstract display box CSS has been fragile historically (see git history for recent fixes).
- Pre-commit hooks are configured (`.pre-commit-config.yaml`): trailing whitespace, end-of-file fixer, YAML validation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rafael-sgoncalves)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rafael-sgoncalves)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
