## jaredjeya-github-io

> This is an academic personal website for Dr Jared Jeyaretnam, built on the [Academic Pages](https://github.com/academicpages/academicpages.github.io) template (a fork of [Minimal Mistakes](https://mmistakes.github.io/minimal-mistakes/)).

# Agent Instructions

This is an academic personal website for Dr Jared Jeyaretnam, built on the [Academic Pages](https://github.com/academicpages/academicpages.github.io) template (a fork of [Minimal Mistakes](https://mmistakes.github.io/minimal-mistakes/)).

## Development Environment

The site runs in a **dev container** (Docker). Jekyll serves at `http://localhost:4000` with live-reload enabled.

- **Content changes** (posts, publications, talks, pages): live-reload handles rebuilds automatically.
- **Template/layout/include changes**: require restarting the Jekyll server: `jekyll serve -l -H localhost`
- **`_config.yml` changes**: always require a full server restart — Jekyll does not reload this file automatically.

## Content Collections

| Directory | Collection | Notes |
|---|---|---|
| `_publications/` | publications | Academic papers; use `category` frontmatter |
| `_talks/` | talks | Conference/seminar talks |
| `_teaching/` | teaching | Course listings |
| `_pages/` | standalone pages | about, CV, publications listing, etc. |
| `_posts/` | blog posts | Currently unused (`.bak` files only) |

### Markdown syntax

Most pages are written using markdown (`.md` files).

A full guide to Markdown syntax used on this site can be found at https://github.com/academicpages/academicpages.github.io/blob/master/_pages/markdown.md.

In particular, this site uses **MathJax 3** for LaTeX rendering in publications/pages. Use `\\(...\\)` for inline maths, and `$$...$$` for display.

### Publication Frontmatter

Publications must include a `category` field matching one of the keys defined in `_config.yml` under `publication_category`. Current categories:

| Key | Display Title |
|---|---|
| `preprint` | Preprints |
| `manuscript` | Journal Articles |
| `conference` | Conference Papers |
| `thesis` | Phd Thesis |
| `book` | Books |

Example frontmatter:
```yaml
---
title: "Paper title"
collection: publications
category: "preprint"
permalink: /publication/YYYY-MM-DD-short-slug
date: YYYY-MM-DD
venue: 'arXiv'
paperurl: 'https://arxiv.org/abs/...'
citation: 'Author One, Author Two, "Title." Venue (Year)'
---
```

To add a new category, add an entry to `publication_category` in `_config.yml` and restart the server.

For adding inline maths in the frontmatter (e.g. in the `citation` or `title` fields), it may be necessary to use unescaped `\(...\)`.

### Writing Publication Abstracts / Body Content

Publication body text (e.g. abstracts copied from arXiv) is rendered as **Markdown**, not LaTeX. Source abstracts often contain raw LaTeX commands — convert these before saving:

- `\texttt{...}` → Markdown backticks: `` `...` ``
- `\href{URL}{text}` → Markdown link: `[text](URL)`
- `\textit{...}` / `\textbf{...}` → Markdown `*...*` / `**...**`
- Math (`\(...\)`, `$...$`, symbols, subscripts, etc.) → MathJax delimiters: `\\(...\\)` for inline, `$$...$$` for display. Do not leave bare `$...$` or unescaped `\(...\)`.

## Key Customisations vs. Upstream Template

### Google Analytics + Consent Banner

The site uses **Google Tag Manager** (`GTM-W66BSBHD`) with a **Silktide consent manager** for GDPR-compliant cookie consent. The standard Academic Pages analytics provider mechanism is **disabled**.

Relevant files:
- [`_includes/analytics.html`](_includes/analytics.html) — loads GTM script and Silktide CSS (included in `_layouts/default.html`)
- [`_includes/gtm.html`](_includes/gtm.html) — sets `gtag` consent defaults from `localStorage` before GTM fires; includes GTM noscript fallback
- [`_includes/head/cookie-banner.html`](_includes/head/cookie-banner.html) — Silktide banner config (consent update callbacks for `gtag`)
- [`assets/js/silktide-consent-manager.js`](assets/js/silktide-consent-manager.js) — vendored Silktide library
- [`assets/css/silktide-consent-manager.css`](assets/css/silktide-consent-manager.css) — Silktide styles

> **Note**: `_includes/head/cookie-banner.html` is currently commented out in `_layouts/default.html`. The Silktide script is loaded via `_includes/analytics.html` instead.

### Head Extras

[`_includes/head/custom.html`](_includes/head/custom.html) adds:
- Custom favicon set (SVG + PNG variants)

## CV Data

The CV page is driven by [`_data/cv.json`](_data/cv.json), a JSON Resume-compatible file containing work history, education, publications, presentations, and teaching. Edits to this file are reflected on the website CV at `/cv/`. The PDF version of the CV is supplied separately and is not generated from this file.

## Site Configuration

All site-wide settings (author info, social links, analytics, publication categories) are in [`_config.yml`](_config.yml). The site owner is `jaredjeya` — do not change the `url`, `repository`, or GTM ID without explicit instruction.

## Files to Avoid Modifying

- `assets/js/silktide-consent-manager.js` — vendored third-party library; update by replacing from source, not by editing
- `_site/` — generated output, never edit directly
- `*.bak` files — disabled content kept for reference

---
> Source: [jaredjeya/jaredjeya.github.io](https://github.com/jaredjeya/jaredjeya.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
