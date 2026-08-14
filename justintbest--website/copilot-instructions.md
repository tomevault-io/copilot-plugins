## website

> Portfolio site for Justin Best (Concept Art + Architecture).

# Website — Notes for Claude

Portfolio site for Justin Best (Concept Art + Architecture).

## Structure

- `index.html` — Concept Art page, driven by `gallery-data.js`
- `Architecture.html` — Architecture page, driven by `architecture-data.js`
- `images/project-<slug>/` — one folder per project, holding that project's images
- Both data files share the same shape: `HERO_SLIDES` (array of image paths for the hero slideshow) and `GALLERY` (array of project entries)

## Adding a new project

1. Create `images/project-<slug>/` and move/rename the new images into it with descriptive lowercase-hyphenated filenames (avoid spaces, "Untitled-xxx", etc.)
2. Add an entry to the relevant `GALLERY` array:
   - `title`, `project`, `category` (display label), `cat` (filter key — Architecture page only has `academic` / `speculative`), `year`
   - `thumb`/`full` — cover image (shown on the grid card and as the lightbox hero)
   - `series` — array of `{ thumb, full }` for additional images shown in the lightbox below the cover
   - `desc` (optional, HTML string) — project description shown in the lightbox below the cover image, above the series grid. Use `<p>` for body text and `<p class="lb-desc-credit">` for exhibition credits / press mentions / software lists (renders smaller, monospace, uppercase).
3. Commit and push directly to `main` (per current workflow — no PRs).

## Series image layout

By default, series images stack in a single column at full width.

For a per-project alternate layout (e.g. "Rethinking Monolithic Architecture"), the lightbox JS checks `p.t === '<title>'` and toggles a `.flow` class on `.lb-series`, which switches to a 2-column masonry layout (CSS `column-count: 2`, 1 column on mobile). Use this pattern if a project needs a denser image grid instead of the default stack.

## Git workflow

- Work directly on `main` and push there (per user instruction in this repo).
- If push is rejected due to remote changes, `git fetch origin main && git rebase origin/main` then push again.

---
> Source: [justintbest/Website](https://github.com/justintbest/Website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
