## velantrim-eiti-6

> VELANTRIM EITI is a zero-dependency, fully client-side PWA (Progressive Web App) built as a single monolithic `index.html` file (~2.5 MB). There is **no build step, no package manager, no backend, and no external database**. The app uses browser-native storage (IndexedDB, localStorage) and an in-browser SQLite WASM engine (`sql-wasm.js` / `sql-wasm.wasm`).

# AGENTS.md

## Cursor Cloud specific instructions

### Overview

VELANTRIM EITI is a zero-dependency, fully client-side PWA (Progressive Web App) built as a single monolithic `index.html` file (~2.5 MB). There is **no build step, no package manager, no backend, and no external database**. The app uses browser-native storage (IndexedDB, localStorage) and an in-browser SQLite WASM engine (`sql-wasm.js` / `sql-wasm.wasm`).

### Running the app

Serve the repository root with any static HTTP server. Service Workers require `http://` or `https://` (not `file://`).

```
python3 -m http.server 8080 --directory /workspace
```

Then open `http://localhost:8080/index.html` in Chrome.

### Key files

| File | Purpose |
|---|---|
| `index.html` | The entire application (HTML + CSS + JS monolith) |
| `sql-wasm.js` / `sql-wasm.wasm` | SQLite WASM engine for FTS5 search |
| `sw.js` | Service Worker for offline/PWA caching |
| `manifest.json` | PWA manifest |

### Gotchas

- There is no linting, no automated tests, and no build system. The codebase has zero `package.json`, `requirements.txt`, or similar dependency files.
- AI chat features (DeepSeek, Gemini, Grok, OpenRouter) require external API keys configured in the app's Settings tab. The DuckDuckGo AI provider works without an API key.
- The app stores all data in the browser (IndexedDB ~500 MB, localStorage ~5 MB, SQLite WASM for FTS5). There are no external databases.
- When testing, the first load shows an onboarding wizard that must be completed before accessing the main interface.

---
> Source: [velantrian/Velantrim-Eiti-6](https://github.com/velantrian/Velantrim-Eiti-6) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
