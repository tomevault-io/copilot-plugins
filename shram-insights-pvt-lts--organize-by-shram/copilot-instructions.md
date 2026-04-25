## organize-by-shram

> Chrome MV3 extension that automatically organizes browser tabs into logical groups using local, privacy-first AI. No data leaves the user's browser.

# Organize by Shram - Chrome Extension

## Overview
Chrome MV3 extension that automatically organizes browser tabs into logical groups using local, privacy-first AI. No data leaves the user's browser.

## Architecture

```
popup.html/js  →  background.js (service worker)  →  offscreen.js (WebLLM)
                        ↓
              lib/dbscan.js (clustering)
              src/utils/smartCategorizer.js (domain lookup)
```

- **popup.js** — UI: "Organize" and "Ungroup" buttons, progress display
- **background.js** — Orchestrator: receives messages from popup, manages offscreen document, runs categorization + clustering, creates Chrome tab groups
- **offscreen.js** — Isolated context for WebLLM WASM model (`snowflake-arctic-embed-m-q0f32-MLC-b4`), generates text embeddings
- **lib/dbscan.js** — DBSCAN clustering algorithm using cosine similarity
- **src/utils/smartCategorizer.js** — 200+ domain-to-category mappings, 13 categories

## Two-Tier Grouping Strategy
1. **Fast path**: Domain lookup against `DOMAIN_CATEGORY_MAP` (O(n))
2. **AI path**: Only uncategorized tabs get embeddings → DBSCAN clustering (O(n²))

## Build

```bash
bun run build      # Build to dist/
bun run dev        # Build + watch
bun run package    # Build + zip for distribution
```

- Build tool: **Bun** (with esbuild)
- `build.ts` bundles `background.js` and `offscreen.js` (ESM, no minification, no splitting)
- `popup.js` is vanilla JS — copied as-is, no bundling
- Output goes to `dist/`

## Key Conventions

- **No external APIs** — single production dependency is `@mlc-ai/web-llm`
- Service worker is declared as `"type": "module"` in manifest (required for ES imports)
- CSP includes `wasm-unsafe-eval` for WebLLM's WASM execution
- Offscreen document required for WebLLM (service workers can't run WASM directly)
- Message passing between popup → background → offscreen via `chrome.runtime.sendMessage`

## Loading for Development
1. `bun run build` (or load source directory directly — both work since manifest declares module type)
2. `chrome://extensions/` → Developer mode → Load unpacked → select `dist/` (or root)

## File Structure
```
manifest.json              # MV3 manifest
background.js              # Service worker entry
offscreen.js               # WebLLM embeddings (module)
offscreen.html             # Offscreen document shell
popup.html                 # Extension popup UI (320px, dark mode support)
popup.js                   # Popup event handlers
lib/dbscan.js              # DBSCAN + suggestEpsilon exports
src/utils/smartCategorizer.js  # Domain maps + category definitions
build.ts                   # Bun build script
src/icons/                 # Extension icons (16-128px)
dist/                      # Build output (gitignored)
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Shram-Insights-Pvt-Lts) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
