## architecture-portfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A static IT architecture portfolio site built with [MkDocs](https://www.mkdocs.org/) + the Material theme. There is no application code — content is Markdown under `docs/`, navigation and theme are configured in `mkdocs.yml`, and the published site lives at https://robert-n-myhre.github.io/architecture-portfolio/.

## How content is structured

- `mkdocs.yml` is the source of truth for site navigation, theme, and extra CSS. Adding a new page means creating the `.md` file under `docs/` **and** wiring it into the `nav:` tree in `mkdocs.yml` — pages not listed in `nav` won't appear in the sidebar.
- `docs/projects/` holds the individual project spotlight pages.
- `docs/assets/styles/print.css` and `extra.css` are pulled in via the `extra_css:` block in `mkdocs.yml`. `print.css` exists specifically because the portfolio is also intended to be printed/exported — keep print styles in mind when changing layout.
- `docs/assets/images/` holds the images referenced by the Markdown pages.
- `site/` is the build output. It is gitignored (some legacy files remain tracked) — do not hand-edit it; it is regenerated on every deploy.

## Common commands

Local preview with live reload (default http://127.0.0.1:8000):

```bash
mkdocs serve
```

Build the static site into `site/` (rarely needed locally — CI does this):

```bash
mkdocs build
```

Manual deploy to GitHub Pages (deprecated path — CI normally handles this; see below):

```bash
mkdocs gh-deploy
```

The `origin` remote is configured for SSH, so `gh-deploy` uses the local SSH key rather than prompting for credentials.

## Deployment

`.github/workflows/gh-pages.yml` deploys on every push to `main`:

1. Checks out the repo
2. Installs `mkdocs` and `mkdocs-material` on Python 3.11
3. Runs `mkdocs gh-deploy --force`, which builds `site/` and pushes it to the `gh-pages` branch

There is no test suite, lint step, or PR check — pushing to `main` is the deploy trigger. After a push, watch the Actions tab for status; the live site usually updates within ~1 minute of a green run.

---
> Source: [Robert-N-Myhre/architecture-portfolio](https://github.com/Robert-N-Myhre/architecture-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
