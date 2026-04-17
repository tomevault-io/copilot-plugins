## hikehog

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**PackHog** is a zero-build-step static web app for visual packing lists. There is no framework, no package manager, no compilation — just open `index.html` in a browser to run it.

Two pages:
- `index.html` — the main visual packing list app
- `calendar.html` — a trip calendar with color-coded day planning

## Development

Open `index.html` directly in a browser. The app fetches `dist/packhog.csv` via PapaParse (loaded from CDN), so it must be served over HTTP for the CSV fetch to work:

```sh
python -m http.server 8080
# then open http://localhost:8080
```

There is no lint, build, or test command.

## Architecture

### `index.html`
All logic lives inline in a single `<script>` tag. Key concepts:

- **Data source**: `dist/packhog.csv` (columns: `Category`, `Item`, `Image`). Loaded at startup via PapaParse. Image paths in the CSV are relative to `dist/` (e.g., `png/002-sunglasses.png` or `mypng/first aid.png`).
- **State**: Packed item names stored in `localStorage` under key `packhog_packed_list` as a JSON array of strings.
- **Category grid**: Hardcoded HTML table in `.config-panel`. Each cell pairs a checkbox (controls inclusion in summary count) with a tab button (switches the visible item page). The checkbox `value` must match the category name in the CSV (case-insensitive `includes` match).
- **Overnight toggle**: The `#overnight-check` master checkbox dims/enables all cells in column 3 of the category table (`.tab-btn.overnight-btn`). Those categories are: Evening, Bed, Bath, Electronics, Food.
- **Special case**: The `always` category checkbox has `value="always"` and is checked by default. Its items are always included in the summary.
- **Item cards**: Generated dynamically in `initApp()` from filtered CSV rows. Checking an item hides the card and adds the name to localStorage.
- **Summary**: `updateSummary()` counts items whose category matches any checked checkbox value, excluding already-packed items.

### `dist/packhog.csv`
The single source of truth for items. Format:
```
Category,Item,Image
always,sunglasses,png/002-sunglasses.png
```
Blank lines between category groups are ignored (PapaParse `skipEmptyLines: true`).

### Images
- `dist/png/` — numbered stock icons (001–100)
- `dist/mypng/` — custom photos (filenames can contain spaces)

### Adding a new category
1. Add a `<td>` cell to the category table in `index.html` with a checkbox and tab button
2. Add items to `dist/packhog.csv` using the same category name (case-insensitive match)
3. Add any new images to `dist/png/` or `dist/mypng/`

If the category should be part of the Overnight group, add the `overnight-btn` class to its `.tab-btn`.

### `calendar.html`
Standalone. Generates a Mon–Sun grid for a date range. Click-drag selects days; a color picker colors them. Colors saved to `localStorage` under `trip_calendar_colors` keyed by display date string (e.g., `"3/15"`). Note: `makeCalendar()` is defined twice — the second definition (which supports color restore) overrides the first.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/siliconscience) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
