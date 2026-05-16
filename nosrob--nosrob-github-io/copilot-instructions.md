## nosrob-github-io

> Context and guidelines for Claude when working in this repo.

# CLAUDE.md

Context and guidelines for Claude when working in this repo.

## Project

- **What**: Personal music discovery blog at https://nosrob.github.io/
- **Stack**: Hugo static site (theme `lug` in `themes/lug/`), French content (`languageCode = "fr-fr"`)
- **Hosting**: GitHub Pages, auto-deployed from `main` via `.github/workflows/hugo.yml`
- **Content**: Each post embeds a track (YouTube / Bandcamp / Spotify / SoundCloud) with a thumbnail

## Adding a new post

Use Hugo's archetype — it fills frontmatter correctly (date, title, thumb path):

```bash
hugo new content/posts/<slug>.md
```

The slug becomes the URL and drives the auto-filled `title`, `alt`, and `thumb` fields. Use kebab-case, lowercase only.

The `posts` archetype (`archetypes/posts.md`) generates:

```yaml
---
title: "Slug Words Title-Cased"
date: <auto>
draft: false
description: ""
thumb: "images/thumbs/<slug>.jpg"
alt: "Slug Words Title-Cased"
tags: []
categories: []
---

Paste your embed code here (YouTube, Spotify, SoundCloud, etc.) and your description...
```

Then:
1. Drop the matching thumbnail into `static/images/thumbs/<slug>.jpg` (or `.png`). Filename **must** match `thumb:` exactly.
2. Replace the placeholder body with the embed iframe (see Embeds below).
3. Fix the title if special chars or punctuation matter (e.g. `"Stavroz - Merci Éclair"`, `"Nudity - Make-Up"`). Hugo's `title` filter is naive.
4. Add tags (quoted strings) — see Conventions.

## Frontmatter conventions

- `tags`: always quoted strings, lowercase. `tags: ["rock", "punk", "garage"]`. Never `tags: [rock]` (the repo standardized away from that — see commit history).
- `categories`: usually `[]`. Some older posts use `["musique"]`; not required.
- `description`: usually `""`. Optional.
- `draft: false` to publish.
- `date`: leave whatever `hugo new` generates. Do **not** hand-edit the timezone.

### Timezone gotcha

`hugo new` writes the local time with the current offset. If the system clock is in CEST (`+02:00`, summer) but you copy-paste a date with `+01:00` (CET, winter), the post lands in the future at build time → Hugo silently excludes it (no `--buildFuture` in the workflow).

Symptom: post file exists, builds locally, never appears on the deployed site.

Fix: match the actual current offset. In summer (late March → late October) Paris = `+02:00`. In winter = `+01:00`.

## Embeds

Posts are essentially `<iframe>` containers. Examples in the repo:

- **YouTube**: `<iframe width="560" height="315" src="https://www.youtube.com/embed/<id>" ...>` — copy from YouTube's "Share → Embed".
- **Bandcamp**: `<iframe style="border: 0; width: 100%; height: 120px;" src="https://bandcamp.com/EmbeddedPlayer/album=<id>/...">` — copy from Bandcamp's embed widget.
- **SoundCloud / Spotify**: same pattern, use the platform's official embed snippet.

Hugo allows raw HTML because `markup.goldmark.renderer.unsafe = true` is set in `hugo.toml`. No shortcode needed.

## Local dev

```bash
hugo server -D    # serves drafts too
hugo server       # production-like preview
```

Build matches CI when run with `hugo --gc --minify`.

## Deploy workflow

`.github/workflows/hugo.yml` builds on push to `main`. Pinned versions live in the `env:` block — bump them together when refreshing deps. Current pins (as of 2026-05-05): Hugo 0.161.1, Go 1.26.2, Node 24.15.0, Dart Sass 1.99.0.

## Repo customizations beyond the theme

The site has root-level overrides in `layouts/`, `static/`, and CSS — these take precedence over `themes/lug/`. When something looks off, check root first, theme second.

## Conventions for Claude

- **French**: site content is French. Match tone of existing posts when writing copy. Tech commentary in commits/PRs can stay English.
- **Commits**: short conventional style (`fix:`, `chore:`, `feat:`). No co-author trailers, no "Generated with Claude Code" footers — keep messages clean.
- **No new top-level files** unless asked. Don't add READMEs, docs folders, etc. — this is a content repo, not a software project.

---
> Source: [Nosrob/nosrob.github.io](https://github.com/Nosrob/nosrob.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
