## flowformlab-com-site

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Source for **flowformlab.com** — a Jekyll site built on the `mmistakes/minimal-mistakes` remote theme (dark skin), hosted on GitHub Pages with a custom domain (`CNAME`). Content is authored either by hand as Markdown files or through the **Decap CMS** admin UI at `/admin` (custom OAuth backend at `ffl-oauth.vercel.app`, not Netlify Identity or the default GitHub App backend).

There is no `Gemfile` / local Jekyll toolchain committed — GitHub Pages builds the site directly (via `jekyll-remote-theme`) on push to `main`. There is no local preview workflow set up; the only local tooling is the front-matter validator below.

## Commands

- Validate post/draft front matter (requires `pip install pyyaml`):
  ```
  python tools/validate_frontmatter.py
  ```
  or on Windows: `.\validate.ps1`. Scans `_posts/` and `_drafts/`, checks required fields (`title`, `date`, `layout`, `categories`, `tags`, `published`), and exits non-zero on any error — writes a machine-readable report to `tools/validate_frontmatter.last.json`.
- `tools/fix_frontmatter.py` — conservative auto-fixer for `_posts/` only: forces `layout: single`, `published: true`, and coerces `date` to a string. Run manually and review the diff; it's not part of any CI gate.

## Content model

- New posts go in `_posts/YYYY-MM-DD-title.md`. Copy the front-matter shape from `_posts/_template.md` (there's a matching `_drafts/_template.md` for drafts).
- **Visibility is controlled by the `published: true/false` front-matter flag**, not by which folder the file lives in — this is a site-specific convention layered on top of Jekyll, enforced only by the validator/CMS UI, not by Jekyll itself.
- **`categories` determines which section a post appears in**, and that's a hard routing dependency, not just metadata:
  - `categories: ["notes"]` (the default) → shows up on the homepage (`_layouts/home.html` filters `site.categories.notes`).
  - `categories: ["fiction"]` → shows up on `/fiction/` (`_layouts/fiction-index.html`, itself set via `taxonomy: fiction` in `fiction.md`, filters `site.categories[page.taxonomy]`).
  - Picking the wrong category silently drops a post from both listings.
- `admin/config.yml` (Decap CMS schema) and `_posts/_template.md` must be kept in sync manually — CMS field changes don't propagate to the template or vice versa.

## Site-wide config

- `_config.yml` carries: site metadata, the `dark` Minimal Mistakes skin, GA4 analytics (`analytics.provider: google-gtag`, tracking ID inline — this is the theme's built-in analytics hook, not a hand-rolled script tag), and `paginate: 10` for the homepage.
- `_includes/head/custom.html` overrides the theme's default `<head>` include — favicons, Open Graph/Twitter meta tags, and a link to `/assets/css/extra.css`. Site-wide `<head>` changes belong here, not in a new file (the remote theme has no local files to edit directly — all customization happens through `_config.yml`, `_data/`, `_includes/` overrides, and `_layouts/` overrides).
- `_data/navigation.yml` drives the top nav (`main`); `_data/authors.yml` defines CMS/AI author profiles referenced from post front matter (e.g. the `llama32_1b` Ghostwriter author).

## Unrelated file

`oci-vpn-hub-public.html` at the repo root is a large standalone static page, unconnected to the Jekyll content pipeline — don't assume it follows any of the conventions above.

---
> Source: [dermawas/flowformlab-com-site](https://github.com/dermawas/flowformlab-com-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
