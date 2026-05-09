## screan

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Screan is a web-based screenshot studio for creating App Store and Google Play listing screenshots. It's a pure frontend application (no backend, no build step) that runs directly in the browser. Users can design screenshots with customizable text, backgrounds, and layouts, then export them as ZIP files in all required store formats.

## Running Locally

No build step required. Open `index.html` in a browser, or serve it locally:

```bash
# Any simple HTTP server works
python3 -m http.server 8000
npx serve .
```

The app is also deployed at https://screan.app

## Architecture

**Module-based vanilla JavaScript** attached to a global `App` namespace. No frameworks, no bundler.

### File Structure

- `index.html` — Complete DOM structure (sidebar, main area, settings panels)
- `app.js` — Entry point, initializes all modules
- `js/config.js` — Constants: device formats, fonts, presets, languages, spacing interpolation
- `js/state.js` — Global state (`App.state`), platform/screenshot selection logic
- `js/storage.js` — IndexedDB persistence layer with debounced saves (500ms)
- `js/render.js` — Canvas 2D rendering engine (preview + export quality)
- `js/export.js` — ZIP export via JSZip (multi-format, multi-language folders)
- `js/screenshots.js` — Screenshot upload, management, thumbnail creation
- `js/events.js` — All UI event handlers and DOM interactions
- `js/localize.js` — Multi-language content management per screenshot
- `js/ai-translate.js` — Claude API (Haiku 4.5) integration for batch translation
- `js/reorder.js` — Drag-and-drop reordering with FLIP animation
- `js/dragdrop.js` — File upload drag-and-drop
- `css/` — Modular CSS with custom properties for dark/light theme

### Key Patterns

- **State**: Centralized in `App.state`, organized by platform (iphone, ipad, mac, android-phone, android-tablet). Each platform has its own screenshots array and export format selection.
- **Rendering**: Canvas-based with dual mode (live preview at reduced scale, export at full resolution). Layout presets (Top/Center/Bottom) control text and screenshot positioning.
- **Persistence**: IndexedDB stores full app state including base64 image data. LocalStorage for API key and theme preference.
- **Localization**: Per-screenshot content object keyed by language code. Global language list shared across platforms.
- **AI Translation**: Direct browser-to-Anthropic API calls (no proxy). API key stored in LocalStorage.

### External Dependencies (CDN)

- **JSZip 3.10.1** — ZIP file generation
- **Lucide Icons** — UI iconography
- **Google Fonts (Inter)** — Typography
- **Vercel Analytics** — Usage analytics

## Conventions

- Vanilla JS with `var` declarations and `App.` namespace pattern
- CSS custom properties for theming (`variables.css`)
- Comments and some UI text in French
- Communicate with the developer in French

---
> Source: [cassardp/Screan](https://github.com/cassardp/Screan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
