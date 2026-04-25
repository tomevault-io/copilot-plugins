## pablosproject-com

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal portfolio/resume website for Paolo Tagliani at https://pablosproject.com/. Built with Hugo static site generator using a custom fork of the hugo-researcher theme.

## Development Commands

```bash
# Serve locally with draft content
hugo server -D

# Build for production
hugo -D

# Create a new post
hugo new posts/POST.md
```

**Requirement**: Hugo Extended version (for SCSS/Sass support)

## Architecture

- **Static Site Generator**: Hugo
- **Theme**: Custom fork of hugo-researcher (git submodule at `/themes/researcher/`)
- **Content**: Markdown files in `/content/`
- **Configuration**: TOML (`config.toml`)
- **Styling**: SCSS in `/themes/researcher/assets/sass/`

### Content Structure

| File | Purpose |
|------|---------|
| `content/_index.md` | Homepage ("About Me") |
| `content/works.md` | Portfolio of projects |
| `content/about.md` | Older About Me page / legacy content |
| `content/blog/_index.md` | Blog landing page |
| `content/blog/*.en.md` | Canonical English blog posts |
| `content/blog/*.it.md` | Optional Italian translations |

### Key Configuration (config.toml)

- Base URL: `https://pablosproject.com`
- Font: Inconsolata
- Page width: 750px
- Raw HTML enabled in Markdown (`unsafe: true`)
- Disabled: Taxonomies, RSS, Sitemap, RobotsTXT

## Theme Customization

The theme is a private fork managed as a git submodule. To modify styling:

1. Edit SCSS in `/themes/researcher/assets/sass/researcher.scss`
2. Theme layout templates in `/themes/researcher/layouts/`
3. Style variables configured in `config.toml` under `[params.style]`

## Static Assets

Place images, favicons, and other static files in `/static/`. They will be served from the site root.

## Multilingual Blog Conventions

The blog supports **English** and **Italian**.

### Language rules

- **English is canonical and required** for every blog post.
- Italian is optional and treated as a translation.
- Do not create an Italian blog post without its English counterpart.

### File naming

Use Hugo language suffixes:

```text
content/blog/post-slug.en.md
content/blog/post-slug.it.md
```

Example:

```text
content/blog/2026-03-31-phishing-recruiter.en.md
content/blog/2026-03-31-phishing-recruiter.it.md
```

### URL behavior

- English posts live at `/blog/...`
- Italian translations live at `/it/blog/...`

### Translation linking

Translation links on blog posts are handled automatically in:
- `layouts/blog/single.html`

Do **not** hardcode “also available in Italian/English” links inside Markdown unless there is a specific reason.

### Publishing workflow

#### English only
Create:

```text
content/blog/my-post.en.md
```

#### English + Italian
Create:

```text
content/blog/my-post.en.md
content/blog/my-post.it.md
```

#### Not allowed
Do not create:

```text
content/blog/my-post.it.md
```

without:

```text
content/blog/my-post.en.md
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pablosproject) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
