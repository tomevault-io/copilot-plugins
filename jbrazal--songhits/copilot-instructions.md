## songhits

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# userEmail
The user's email address is j.e.brazal@gmail.com.
# currentDate
Today's date is 2026-05-27.

      IMPORTANT: this context may or may not be relevant to your tasks. You should not respond to this context unless it is highly relevant to your task.

## Commands

```bash
npm run site:build   # build static site → _site/
npm run site:serve   # build + watch + local dev server
npm run build        # compile VS Code extension (esbuild)
npm run watch        # compile extension in watch mode
```

## Architecture

This repo is two things sharing the same source files:

**1. VS Code extension** — provides syntax highlighting and a live preview panel for `.chordmark` files. Entry point: `src/extension.ts`, compiled to `out/extension.js` via `esbuild.js`. The preview panel (`src/previewPanel.ts`) renders chord charts using the `chord-mark` npm library.

**2. Eleventy static site** — generates a browsable chord chart website. Config: `eleventy.config.js`. Output: `_site/`.

### Site page types

| URL pattern | Template | Source |
|---|---|---|
| `/` | `index.njk` | lists all songs + setlists |
| `/songs/{slug}/` | `_includes/song.njk` | one page per `.chordmark` file |
| `/setlist/` | `setlist/index.njk` | lists all setlists |
| `/setlist/{name}/` | `_includes/setlist.njk` | inline all songs for a setlist |

### How `.chordmark` files become song pages

`chordmark/chordmark.11tydata.js` is an Eleventy directory data file — it applies to every `.chordmark` file in that directory. It sets `layout: song.njk`, derives `title` and `artist` from the filename slug (format: `song-title--artist-name.chordmark`), and sets `permalink: /songs/{slug}/`. The raw file content is passed as `content` to the template via a custom Eleventy extension registered in `eleventy.config.js`.

**Critical**: in `song.njk`, the SOURCE must be output as `{{ content | dump | safe }}` — `dump` produces a JSON string, `safe` prevents Nunjucks from HTML-escaping it (which would break the JS).

### Setlist data model

`setlist/{name}/index.md` uses YAML frontmatter with a `parts` array. Each part has a `name` and a `songs` array. Songs with a `slug` field get a rendered chart; songs without one show "Chart coming soon" or "TBD".

### Chord-mark rendering

All rendering happens client-side via `/assets/chord-mark.js` (copied from `media/chord-mark.js` at build time). The global is `window['chord-mark']` with `.parseSong(source)` and `.renderSong(parsed, options)`.

### Setlist page: songs embedded at build time

`eleventy.config.js` provides a `setlistSongsJson` shortcode that reads all referenced `.chordmark` files at build time and embeds them as a `{ slug: source }` JSON map. The setlist page then renders each chart client-side on load.

### `.chordmark` file format

Section headers: `#v` verse, `#c` chorus, `#p` pre-chorus, `#b` bridge, `#i` intro, `#u` outro, `#o` other. Custom names also work (e.g. `#Post-Chorus`). Each section has alternating chord lines and lyric lines; `_` in a lyric line marks where the chord above it falls.

### Filename conventions

Slug format for files with a known artist: `song-title--artist-name.chordmark`. Files without `--` get no artist attribution. Some legacy files use spaces or mixed case (e.g. `Love.chordmark`, `Foollish Heart.chordmark`) — these produce URL-encoded paths.

---
> Source: [jbrazal/songhits](https://github.com/jbrazal/songhits) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
