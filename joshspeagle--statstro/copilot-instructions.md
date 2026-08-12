## statstro

> Conference website for **STATSTRO 2026: Sampling, Simulation, and Scientific Discovery**, a 2-day interdisciplinary astrostatistics workshop at the University of Toronto, July 16-17, 2026.

# STATSTRO 2026 Website

## Overview
Conference website for **STATSTRO 2026: Sampling, Simulation, and Scientific Discovery**, a 2-day interdisciplinary astrostatistics workshop at the University of Toronto, July 16-17, 2026.

**Status: the workshop has concluded — the site is now an archive.** Copy is written in the past tense; the top of the page thanks participants and shows the group photo, and tutorial materials are linked. `_config.yml` sets `conference.concluded: true` and all `registration_open`/`in_person_open`/`online_open` flags to `false`, which drive the "concluded" states in `_includes/hero.html` and `_includes/registration.html`. When cloning for a future edition, flip these flags and revert copy to the present/future tense.

## Tech Stack
- **Jekyll 4.x** (plain `jekyll` gem — *not* the legacy `github-pages` meta-gem), Sass for styles, vanilla JS
- **Ruby via [mise](https://mise.jdx.dev/)** — version pinned in `.ruby-version` (3.4.1). `Gemfile.lock` is committed for reproducible builds.
- **Built & deployed by GitHub Actions** (`.github/workflows/jekyll.yml`) on every push to `main` — the repo's Pages source is set to "GitHub Actions", not the legacy branch builder. This keeps local and production on the same current Jekyll/Ruby versions.

## Design System: "Cosmic Cartographer"
Sampling and simulation as exploring/mapping unknown territory. Antique star charts meet modern computational aesthetics.

### Colors
| Token | Hex | Usage |
|-------|-----|-------|
| Navy | `#0B1D3A` | Primary background, dark sections |
| Gold | `#D4A843` | Accents, headings, highlights, links |
| Cream | `#F5F0E8` | Light backgrounds, body text on dark |
| Teal | `#2A9D8F` | Secondary accent, data viz, buttons |
| Coral | `#C45B3E` | Tertiary accent, alerts, highlights |

### Typography
- **Headings:** Playfair Display (Google Fonts)
- **Body:** Inter (Google Fonts)

## Content Editing
All conference content lives in `_data/` YAML files. To update:
- **Speakers:** Edit `_data/speakers.yml` (grouped by `session` id + `role`: keynote/tutorial/contributed/lightning)
- **Schedule:** Edit `_data/schedule_day1.yml` or `_data/schedule_day2.yml`
- **Posters & lightning talks:** Edit `_data/posters.yml` (each has a `session` id)
- **Sessions:** Edit `_data/sessions.yml` — the four thematic tracks (`ml`, `uq`, `sampling`, `sbi`) with their display name and accent `color`. Speakers, posters, schedule items, and the tutorials section all key off these session `id`s and inherit the session color.
- **Organizers:** Edit `_data/organizers.yml`
- **Sponsors:** Edit `_data/sponsors.yml`
- **Tutorial materials:** Add a `tutorial_url` to a speaker (role `tutorial`) in `speakers.yml` — this drives the "Slides & Tutorial Materials" section (`_includes/tutorials.html`) and the link on the speaker card. The same URL is also duplicated on the matching `tutorial` item in the schedule YAML so the schedule stays self-contained.
- **Speaker slides:** `conference.slides_url` in `_config.yml` points at the shared Google Drive folder of talk slides. It renders as the "Speaker Slides" callout at the top of the same section; blank it out to hide the callout entirely.

Organizers can edit these directly in the GitHub web UI — no local setup needed.

**Sections / includes:** `_layouts/default.html` composes the single page from `_includes/*.html`. Archival additions: `thanks.html` (recap + group photo, first in `<main>`) and `tutorials.html` (speaker-slides callout + tutorial materials grid, after the schedule; anchor id stays `#tutorials`, nav label is "Materials"). Section background alternates automatically via `.section:nth-child(even)` in `_sass/_base.scss` — inserting or reordering sections re-flows the light/dark rhythm, which is expected.

## File Organization
```
_config.yml          → Site metadata and Jekyll settings
_data/               → All content (speakers, schedule, sponsors, etc.)
_includes/           → HTML partials for each section
_layouts/            → Page layout templates
_sass/               → Sass partials (design tokens, components)
assets/css/          → Sass entry point
assets/js/           → Hero animation + main scripts
assets/images/       → Speaker photos, sponsor logos
index.html           → Main single-page site
code-of-conduct.md   → Standalone Code of Conduct page
.github/workflows/   → GitHub Actions: build with Jekyll + deploy to Pages
Gemfile(.lock)       → Plain Jekyll 4 toolchain (committed lock)
.ruby-version        → Ruby version for mise / CI (3.4.1)
```

## Local Development
Requires Ruby (managed by `mise`, which reads `.ruby-version`):
```bash
# one-time: install the pinned Ruby, then the gems
mise install            # installs Ruby 3.4.1 per .ruby-version
gem install bundler -v '~> 2.7'   # match Gemfile.lock's BUNDLED WITH (2.7.x)
bundle install

# run the dev server
bundle exec jekyll serve   # → http://localhost:4000
```
If `mise` isn't set up yet, see its docs — or use any Ruby ≥ 3.1; only the `github-pages` gem is gone, plain Jekyll 4 is undemanding.

## Deployment
Push to `main` → the **GitHub Actions** workflow (`.github/workflows/jekyll.yml`) builds with Jekyll and deploys to Pages. No manual steps. The repo's **Settings → Pages → Source must be "GitHub Actions"** (not "Deploy from a branch"). Custom domain statstro.com is preserved via the root `CNAME` file (DNS configured via Squarespace).

## Image Guidelines
- **Speaker photos:** Square crop, ~300x300px, JPEG, placed in `assets/images/speakers/`
- **Sponsor logos:** PNG with transparent background, ~200px wide, in `assets/images/logos/`
- **Group photo:** `assets/images/statstro2026-group.jpg` (display, 2000px wide) with a larger `-large.jpg` (3600px) linked as "view full size". Source lives in Dropbox at `dunlap/Statstro2026/`. Web-optimize the full-res original before committing, e.g.:
  ```bash
  convert SOURCE.jpg -resize 2000x -strip -interlace Plane -sampling-factor 4:2:0 -quality 82 assets/images/statstro2026-group.jpg
  convert SOURCE.jpg -resize 3600x -strip -interlace Plane -sampling-factor 4:2:0 -quality 80 assets/images/statstro2026-group-large.jpg
  ```

---
> Source: [joshspeagle/STATSTRO](https://github.com/joshspeagle/STATSTRO) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
