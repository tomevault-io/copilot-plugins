## odug-web-hugo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the Hugo static site for the **Oregon Delphi User Group** (odug.org). It contains monthly meeting announcements dating back to 2000, an About page, and presentation archives.

- **Hugo config:** `hugo.toml` — theme is `pascalinia` (a custom Hugo theme as a git submodule)
- **Remote:** `https://github.com/corneliusdavid/odug-web-hugo.git`
- **Published via:** GitHub Actions

## Build Commands

```bash
hugo server        # Local dev server (from repo root)
hugo               # Build static site to public/
```

## Content Structure

All meeting posts live in `content/events/`. There are two content formats:

- **Simple posts:** `content/events/YYYY-MM.md` — a single Markdown file
- **Page bundles:** `content/events/YYYY-MM/index.md` — used when the post includes images or other resources (e.g. presentation files) in the same directory

The section index is `content/events/_index.md` (just a title, "Meetings").

The About page is a page bundle at `content/about/index.md`.

## Event Post Conventions

Front matter uses TOML (`---` delimiters) with these fields:
- `title`: Meeting topic title (in quotes)
- `date`: Full ISO timestamp, e.g. `2026-01-21T10:54:16-08:00`
- `tags`: YAML list — always includes the year (e.g. `- 2026`), optionally speaker names or topics
- `image`: (optional) Filename of a co-located image in the page bundle

Posts use `<!--more-->` to separate the summary from the full content.

Common content sections within a post:
- Opening description/blurb
- Resource links (presentations, PDFs, GitHub repos)
- `## PRESENTER(S) ##` — speaker name(s) and bio(s)
- `## DATE/TIME ##` — meeting date and time
- `## LOCATION ##` — venue name, address, link

## Layout Customizations

The site overrides the theme via `layouts/partials/inject/`:
- `head.html` — adds a meta refresh tag (hourly auto-reload)
- `header-after.html` — (empty, placeholder for injection)

## Static Assets

- `static/` — site favicon/icons (`delphi-software.*`) and `presentations/` archive
- `assets/events/` — additional event assets
- Presentation files linked from posts may live in `static/presentations/` or co-located in page bundles

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/corneliusdavid) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
