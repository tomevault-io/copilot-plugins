## yt2site

> This file tells the agent (Cursor) how to operate this scaffolding. When the user

# AGENTS.md — Playbook for building a site from YouTube videos

This file tells the agent (Cursor) how to operate this scaffolding. When the user
asks you to "build the site," follow these steps exactly.

## Inputs you must read first

1. `inputs/website-spec.md` — the human's brief: what the website is about,
  audience, tone, desired pages/sections, and any do's/don'ts.
2. `inputs/videos.txt` — the YouTube URLs that are the source material.
3. `content/transcripts/*.txt` — the fetched transcripts (one file per video).
  If this folder is empty, tell the user to run
   `py scripts/fetch_transcripts.py` first (or run it yourself if appropriate).

## Your job: turn videos → website material

You are the bridge between raw transcripts and a polished site. The build script
is deterministic and only renders Markdown — **you** write that Markdown.

### Step 1 — Understand the brief

Read `website-spec.md`. Identify: the site's purpose, the audience, the tone, and
the page structure the user wants. If the brief is vague, make reasonable
choices and note them; ask the user only if something is genuinely blocking.

### Step 2 — Mine the transcripts

For each transcript in `content/transcripts/`, extract the substantive ideas,
explanations, steps, examples, and quotes relevant to the brief. Do **not** paste raw transcripts. Synthesize, organize, and rewrite in clean prose and in language the reader can understand. Transcripts are messy (filler words, timestamps removed already) — your output should read like original, well-edited writing.

### Step 3 — Author the pages

Write one Markdown file per page into `content/pages/`. Each page needs YAML
front matter:

```markdown
---
title: How It Works
slug: how-it-works      # used for the filename: output/how-it-works.html
order: 2                # nav order (lower = earlier); home is usually order 0/1
nav: true               # show in the top navigation?
summary: One-line description used on cards/SEO.
---

# How It Works

Your synthesized content here, in Markdown...
```

Guidelines for the content:

- The page with `slug: index` (or the lowest `order`) becomes the homepage.
- Group related ideas across videos rather than making one page per video,
unless the brief asks for per-video pages.
- Attribute material to its source where useful: link back to the video.
Each transcript file starts with a header line containing its source URL.
- Keep headings meaningful; use lists, callouts, and short paragraphs.
- Match the tone requested in the brief.

### Step 4 — Fill site config

Update `config/site.config.yaml` from the brief: `title`, `tagline`,
`description`, `accent_color`, `author`, and `footer`. Keep nav driven by the
pages' `nav`/`order` front matter (the build handles that automatically).

### Step 5 — Build & sanity-check

Run `py scripts/build_site.py`. Confirm `output/index.html` and one or two pages
render, nav links work, and content reflects the videos. Report what you made.

## Rules

- Never invent facts that aren't supported by the transcripts or the brief. If a
transcript lacks needed detail, say so rather than fabricating.
- Don't dump raw transcripts into pages. Always synthesize.
- Keep all generated website material under `content/pages/`; never hand-edit
files in `output/` (they're overwritten on every build).
- If `inputs/videos.txt` is empty or `website-spec.md` is still the template,
prompt the user to fill them in before building.

---
> Source: [TehGhost99/yt2site](https://github.com/TehGhost99/yt2site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
