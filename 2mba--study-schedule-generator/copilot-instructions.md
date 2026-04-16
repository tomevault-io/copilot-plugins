## study-schedule-generator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A static utility site hosted on GitHub Pages. No build step, no framework, no backend. Open `index.html` directly in a browser to develop.

## Tech Stack

- **HTML** — single `index.html` per tool
- **Tailwind CSS** — loaded via CDN only (no config file, no JIT, no purge)
- **Vanilla JavaScript** — all logic in `js/app.js`

## Development

No install or build required. Open `index.html` in a browser. For live-reload, use any static file server:

```
npx serve .
# or
python -m http.server 8080
```

## Architecture

### Theme System

Themes are CSS custom properties scoped to class variants on `<html>`:

```
html.theme-default { --bg: ...; --accent: ...; ... }
html.theme-pastel  { ... }
html.theme-dark    { ... }
```

`applyTheme(name)` in `app.js` sets `document.documentElement.className = 'theme-' + name` and persists to `localStorage`. Tailwind handles layout/spacing; CSS vars handle all color identity. Do **not** use Tailwind color classes for themed colors — use CSS variables via inline styles or the `.card`, `.field`, `.btn-*` classes defined in the `<style>` block.

### JavaScript Structure (`js/app.js`)

Two sections, clearly delimited by comments:

1. **Shared utilities** — `applyTheme()`, `copyToClipboard()`, `showCopyFeedback()`: reusable across future tools.
2. **Tool logic** — `getInputs()`, `generateSchedule()`, `renderOutput()`, `formatForNotion()`, `formatAsPlainText()`, `showError()`, `clearError()`: specific to the Study Schedule tool.
3. **Event wiring** — single `DOMContentLoaded` listener at the bottom that binds all buttons.

### Adding a New Tool

- Create a new `tool-name.html` using the same header/theme-switcher pattern from `index.html`.
- Add a link in the `<nav>` in both `index.html` and the new tool's header.
- The new tool can have its own `<script>` section or a separate JS file; shared utilities live in `js/app.js` or can be split into `js/utils.js`.

### Schedule Algorithm

Located in `generateSchedule()`. Core logic:
1. Enumerates all days from tomorrow to exam date.
2. Applies rest days using a modulo pattern: within each 7-day cycle, the first `(7 - restDaysPerWeek)` days are work days.
3. Distributes hours evenly up to `maxHoursPerDay`; shows a warning if workload exceeds capacity.

### Print / Export

- `.no-print` — hidden during `@media print`.
- `window._lastSchedule` — holds the current schedule array for export functions.
- `formatForNotion()` — outputs Markdown with `## Day N` headings and `- [ ]` checkboxes.
- `formatAsPlainText()` — plain text for general copy.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/2mba) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
