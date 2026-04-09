## my-cv

> Personal CV / resume site for Andrey Kotko, hosted on GitHub Pages.

# Project: my-cv

Personal CV / resume site for Andrey Kotko, hosted on GitHub Pages.

## Tech Stack

- **Static site generator:** Jekyll with `jekyll-theme-minimal`
- **Content:** Markdown (`index.md`)
- **Hosting:** GitHub Pages (branch: `gh-pages`)
- **CI/CD:** GitHub Actions — build with Jekyll, deploy to GitHub Pages
- **OG/meta tags:** Custom `_includes/head-custom.html`

## Project Structure

- `index.md` — CV content (Markdown with YAML front matter)
- `_config.yml` — Jekyll configuration
- `_includes/head-custom.html` — Open Graph and Twitter Card meta tags
- `assets/` — images (avatar, og-image) and PDF version of CV
- `.github/workflows/ci.yml` — CI pipeline: build, link check, deploy

## Key URLs

- Live site: https://kotkoa.github.io/my-cv/
- Repository: https://github.com/Kotkoa/my-cv

## Conventions

- Main branch is `gh-pages` (not `main`)
- Commit messages start with `chore:` for non-feature changes
- Keep `index.md` as the single source of truth for CV content
- OG image is at `assets/img/og-image.png` with cache-busting via `?v=` param

## Workflow

- Edit `index.md` to update CV content
- Push to `gh-pages` triggers CI build and deploy
- Jekyll builds the site into `_site/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kotkoa)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Kotkoa)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
