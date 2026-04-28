## json-tools

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pnpm dev        # Watch mode build (rebuilds on file changes)
pnpm build      # Production build → dist/
pnpm check      # TypeScript + Svelte type checking
```

To test changes, load `dist/` as an unpacked extension in Chrome (`chrome://extensions` → Load unpacked). There is no test framework.

## Architecture

This is a Chrome Extension (Manifest v3) that detects JSON pages and replaces them with a formatted UI. It uses three isolated execution contexts required by Chrome's security model:

```
Page Context (page.js)
  └─ Runs in page origin, exposes parsed JSON to window.data

Content Script (content.js)
  └─ Detects JSON, clears page body, mounts Svelte UI
  └─ Bridges IPC between page and background

Background Service Worker (background.js)
  └─ Runs CPU-heavy Prettier formatting off main thread
  └─ Responds to chrome.runtime.sendMessage() calls
```

**Data flow:** Content script detects JSON → injects page.js to expose `window.data` → calls background worker via `callBackgroundWorker.ts` for formatting → renders result in Svelte components.

**IPC pattern:** All background calls go through `src/content/lib/callBackgroundWorker.ts`, which wraps `chrome.runtime.sendMessage()` with type-safe generics. Operations are registered in `src/background/index.ts`.

## Key Files

| File | Purpose |
|------|---------|
| `src/content/index.ts` | Entry point: detects JSON, mounts Svelte app |
| `src/content/lib/detectJSON.ts` | Detects JSON via `<pre>` tag or Content-Type header |
| `src/content/lib/components/App.svelte` | Main UI: mode toggle (raw/prettier), tab width selector |
| `src/content/lib/components/PrettierJSON.svelte` | Responsive Prettier output, recalculates `printWidth` on resize |
| `src/content/lib/charWidth.svelte.ts` | Svelte 5 reactive utility — measures monospace char width via Canvas |
| `src/background/index.ts` | Operation registry + message listener |
| `src/background/formatByPrettier.ts` | Prettier standalone formatter |
| `src/page/index.ts` | Exposes JSON to `window.data` in page context |
| `src/types.ts` | Shared types (`JSONValue`, message types) |
| `public/manifest.json` | Chrome extension manifest v3 |

## Stack

- **Svelte 5** (runes API: `$state`, `$derived`, `$props`)
- **Vite 8** with three entry points → `dist/{content,background,page}.js`
- **TypeScript 6** — strict, `checkJs: true`
- **Prettier 3** bundled into the background worker (~600 KB output)

## Notes

- `analyzeJson.ts` is a stub — the analyze operation is not yet implemented
- The extension sets `css: { injected: true }` in svelte config so styles are inlined into `content.js`
- `chunkSizeWarningLimit: 1000` is intentionally high because Prettier is large
- Minimum Chrome version: 126 (per manifest)
- See `docs/gotchas.md` for non-obvious behaviors (e.g. why body styles are set via JS)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/langpavel) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-17 -->
