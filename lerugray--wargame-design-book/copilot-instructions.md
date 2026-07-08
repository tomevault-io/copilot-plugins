## wargame-design-book

> A free online version of "A Contemporary Guide to Wargame Design" by Ray Weiss, hosted on GitHub Pages using Jekyll with the `just-the-docs` theme. The full book (23 chapters, 6 appendices, acknowledgements) is already converted to Jekyll-compatible markdown in `docs/_chapters/`. Diagrams are in `docs/assets/images/`.

# A Contemporary Guide to Wargame Design — GitHub Pages Site

## What This Is

A free online version of "A Contemporary Guide to Wargame Design" by Ray Weiss, hosted on GitHub Pages using Jekyll with the `just-the-docs` theme. The full book (23 chapters, 6 appendices, acknowledgements) is already converted to Jekyll-compatible markdown in `docs/_chapters/`. Diagrams are in `docs/assets/images/`.

The source manuscript lives in a PRIVATE repo (`passive-income-hub`). This repo is PUBLIC — it's the reader-facing website.

## Who the User Is

The user (Ray Weiss) is not a programmer. He's a wargame designer building passive income streams. Explain any commands or steps clearly. He pays for Claude Max ($100/month) — do not suggest API costs.

## Current State

The Jekyll site is **scaffolded but not yet deployed**. What's done:
- `docs/_config.yml` — Jekyll config with `just-the-docs` remote theme
- `docs/_chapters/` — all 30 content files (23 chapters + 6 appendices + acknowledgements) with front matter
- `docs/index.md` — homepage with table of contents
- `docs/_layouts/chapter.html` — chapter layout with prev/next navigation
- `docs/assets/images/` — 11 diagram PNGs
- `docs/Gemfile` — Ruby dependencies

## What Needs Doing

### Phase 1: Get it Live (do this first)
1. **Push everything to GitHub** (the initial commit with all content)
2. **Enable GitHub Pages** in repo settings → point to `docs/` folder on `main` branch
3. **Verify the site builds** — check at https://lerugray.github.io/wargame-design-book/
4. **Fix any build issues** — Jekyll can be finicky with remote themes and collections
5. **Enable GitHub Discussions** for reader feedback

### Phase 2: Diagram Embedding
The diagrams are copied to `docs/assets/images/` but NOT yet referenced inline in the chapter text. Each diagram needs an image tag inserted at the right place in its chapter:

| Diagram | Chapter File | Where to Insert |
|---------|-------------|-----------------|
| hex-scale-comparison.png | chapter_05.md | Where hex scale is discussed |
| counter-anatomy.png | chapter_06.md | Where counter components are explained |
| hex-zoc-basic.png | chapter_07.md | Where zones of control are introduced |
| hex-zoi-scaling.png | chapter_07.md | Where ZOI scaling is discussed |
| hex-facing.png | chapter_08.md | Where facing mechanics are introduced |
| hex-flanking.png | chapter_08.md | Where flanking is discussed |
| sedan-combat-matrix.png | chapter_09.md | Where the CRT example appears |
| sop-flowchart.png | chapter_10.md | Where sequence of play flow is described |
| scaled-victory.png | chapter_12.md | Where scaled victory is discussed |
| pcs-airstrike-flowchart.png | chapter_13.md | Where the PCS air system is described |

Use this format for images:
```markdown
![Description]({{ site.baseurl }}/assets/images/filename.png)
```

### Phase 3: Visual Polish (later — Claude in Chrome)
This will be done separately using Claude in Chrome with frontend design plugins. Not for this session unless the user asks.

## Important Rules

- **Do NOT modify the manuscript text** — only add front matter, image references, and structural markup. Content edits happen in the source repo.
- **No real name on public-facing content** — wait, actually the book IS under "Ray Weiss" so that's fine for this repo. But do NOT reference "Conflict Simulations LLC" anywhere on the site.
- **Keep it simple** — this is a book website, not a web app. Clean reading experience is the goal.
- **The repo is PUBLIC** — do not put API keys, personal info, or anything sensitive here.

## Maintenance Hazards

### Chapter content lives in TWO places

Every chapter exists at both `docs/chapter_NN.md` AND `docs/_chapters/chapter_NN.md`. They are NOT the same file — they have diverged historically. This is a quiet footgun:

- **Online site (Jekyll)** renders from `docs/chapter_NN.md`. The non-prefixed copies. `_config.yml` line 37 EXCLUDES `_chapters/` from the site build.
- **Print pipeline (Pandoc → LuaLaTeX)** assembles `docs/_chapters/chapter_NN.md`. The prefixed copies. See `print/build.sh` and `print/prepare-chapters.py`.

If you edit ONE copy, the other surface drifts. Concrete example: 2026-04-28 a misplaced Sedan combat-matrix figure in chapter 9 needed two commits to fully remove — `85cb1f2` fixed the print source (`docs/_chapters/`), `e12c33b` fixed the site source (`docs/`). Either commit alone would have left one surface broken.

**Whenever you edit chapter content, always update both files.** Or pick one to delete + adjust `_config.yml`/`build.sh` to read from the other. The current divergence is technical debt; deleting one side cleanly is preferable to maintaining the parallel sync.

### `print/build.sh` has CRLF line endings

`bash print/build.sh` fails with `$'\r': command not found` on Windows because the file has CRLF. Workaround: run the underlying command sequence directly (`python print/prepare-chapters.py` + Pandoc + LuaLaTeX). Permanent fix: normalize the file to LF, or rewrite the build as a `.ps1` / Python script.

## File Structure

```
wargame-design-book/
├── README.md
├── CLAUDE.md              ← you are here
├── docs/
│   ├── _config.yml        ← Jekyll config
│   ├── _chapters/         ← all 30 content files with front matter
│   │   ├── chapter_01.md through chapter_23.md
│   │   ├── appendix_a_recommended_reading.md through appendix_f_glossary.md
│   │   └── acknowledgements.md
│   ├── _layouts/
│   │   └── chapter.html   ← chapter template with prev/next nav
│   ├── assets/
│   │   └── images/        ← 11 diagram PNGs
│   ├── index.md           ← homepage with TOC
│   ├── Gemfile            ← Ruby dependencies
│   └── .gitignore
```

## Chapter Front Matter Format

Each chapter file has:
```yaml
---
title: "Chapter Title"
chapter_number: N        # only on chapters, not appendices
nav_order: N             # 1-23 for chapters, 100-106 for appendices/acknowledgements
layout: chapter
---
```

## Quick Commands

```bash
# Local preview (if Ruby/Jekyll installed)
cd docs && bundle install && bundle exec jekyll serve

# Check build status after push
gh run list --limit 5
```

---
> Source: [lerugray/wargame-design-book](https://github.com/lerugray/wargame-design-book) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
