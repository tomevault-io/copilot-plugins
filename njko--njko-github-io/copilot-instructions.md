## njko-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

"Mon Niko Niko" — a personal mood tracking dashboard hosted on GitHub Pages. It's a zero-dependency, single-file static app (HTML + CSS + vanilla JS) inspired by the Niko Niko calendar from agile practices.

## Development

No build step, no package manager, no dependencies. The entire app lives in `index.html`.

**Local development:** Open `index.html` in a browser, or serve via any HTTP server:
```bash
python -m http.server 8000
```

**Deployment:** Automatic via GitHub Pages on push to main. No CI/CD pipeline.

**No tests or linting configured.**

## Architecture

- **`index.html`** — Single-file app (~27KB) containing all HTML, CSS, and JavaScript. The JS defines a `NikoNiko` class that loads mood data, computes stats, and renders a 3-month calendar grid.
- **`moods.json`** — Data store for mood entries. Keyed by `YYYY-MM-DD` with `value` (1-5), optional `reason` (emoji), and optional `vacation` (boolean).

**Data flow:** Page load → fetch `moods.json` → display today's mood + stats → render 3-month calendar with color-coded days.

**Mood scale:** 1 (😢 Difficile) → 5 (😄 Excellent).

## Content Language

The app UI and README are written in **French**.

## Git Commit Convention

Mood entry commits follow the pattern: `🎭 Humeur du JJ/MM : [emoji]`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Njko)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Njko)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
