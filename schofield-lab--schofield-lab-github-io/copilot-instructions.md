## schofield-lab-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

The website for Steven R. Schofield's research group at UCL ([schofield-lab.github.io](https://schofield-lab.github.io)), built with Jekyll 4.3 using the [Lab Website Template](https://greene-lab.gitbook.io/lab-website-template-docs).

## Commands

**Serve locally:**
```sh
bundle exec jekyll serve --livereload
```

**Build site:**
```sh
bundle exec jekyll build
```

**Update citations** (regenerates `_data/citations.yaml` from `_data/sources.yaml`):
```sh
cd _cite
pip install -r requirements.txt
python cite.py
```

**Docker alternative** (serves on port 4000):
```sh
.docker/run.sh
```

## Architecture

### Content Collections

- `_members/` — Lab member profiles. Each file has YAML front matter (`name`, `role`, `image`, `aliases`, `links`) followed by a bio. The `aliases` list must match the author name as it appears in paper metadata for auto-linking on citations.
- `_posts/` — Blog posts. Files are named `YYYY-MM-DD-slug.md`. Posts tagged `news` appear automatically on the homepage (up to 4 most recent).
- `research/` — Research area pages. Linked from `_data/research.yaml`.

### Publication / Citation System

Publications flow: `_data/sources.yaml` → `_cite/cite.py` → `_data/citations.yaml`

- **Do not manually edit** `_data/citations.yaml` — it is auto-generated.
- To add a paper, add an entry to `_data/sources.yaml` with the DOI:
  ```yaml
  - id: doi:10.xxxx/xxxxx
    type: paper
    image: /images/publications/filename.jpg
    description: Optional abstract override.
  ```
- `cite.py` runs plugins (`google-scholar`, `pubmed`, `orcid`, `sources`) and fetches metadata via Manubot/DOI APIs, caching results under `_cite/__pycache__/`.
- On GitHub, citation updates run automatically on push to `main` (requires `GOOGLE_SCHOLAR_API_KEY` secret).

### Data Files (`_data/`)

| File | Purpose |
|------|---------|
| `sources.yaml` | Raw DOI/identifier list to pull into citations |
| `citations.yaml` | Auto-generated; full citation metadata |
| `alumni.yaml` | Former group members |
| `authors.yaml` | Author display name overrides |
| `presentations.yaml` | Talks and poster presentations |
| `research.yaml` | Research area metadata |

### Layouts and Includes

- `_layouts/default.html` — base layout; wraps all pages
- `_layouts/member.html` — member profile pages
- `_layouts/post.html` — blog post pages
- `_includes/citation.html` — renders a single citation (`style="rich"` or plain)
- `_includes/list.html` — renders a filtered list of citations or other data
- `_includes/feature.html`, `feature_text.html` — hero/feature sections used on the homepage
- `_includes/section.html` — horizontal rule / section divider

### Styles and Scripts

- SASS lives in `_styles/` (configured as `sass_dir`)
- JavaScript lives in `_scripts/`

### CI (GitHub Actions)

- **on-push.yaml**: On push to `main`, runs citation update then builds and deploys the site.
- **on-schedule.yaml**: Periodically updates citations.
- **build-preview.yaml** / **on-pull-request.yaml**: Build preview for PRs.

## Typical Content Changes

- **New blog post**: create `_posts/YYYY-MM-DD-slug.md` with front matter `title`, `description`, `tags`, `image`, `author`.
- **New publication**: add entry to `_data/sources.yaml`; citations are rebuilt by CI.
- **New team member**: create `_members/name.md` following the pattern of existing members.
- **News on homepage**: posts tagged `news` appear automatically; the homepage shows the 4 most recent.

---
> Source: [schofield-lab/schofield-lab.github.io](https://github.com/schofield-lab/schofield-lab.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
