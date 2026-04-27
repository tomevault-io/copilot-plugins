## just-text

> Minimal web text editor. Open, edit, close — content persists automatically.

# just-text

Minimal web text editor. Open, edit, close — content persists automatically.

**Domain:** justtext.dev (Cloudflare Pages)

## Tech Stack

- Vite + Vue 3 + TypeScript (Composition API, `<script setup>`)
- CodeMirror 6 via vue-codemirror
- No UI framework — hand-crafted toolbar

## Architecture

```
src/
  App.vue                — Main layout: toolbar + editor + footer
  components/
    Editor.vue           — CodeMirror 6 wrapper, custom Tab keymap
    Toolbar.vue          — Theme toggle, language selector, copy-with-metadata
  composables/
    useStorage.ts        — Abstract storage layer, debounced auto-save
    useTheme.ts          — Auto/light/dark theme, system preference tracking
  storage/
    adapter.ts           — StorageAdapter interface
    create-adapter.ts    — Factory: IndexedDB → localStorage fallback
    indexeddb.ts         — IndexedDB implementation
    local.ts             — localStorage implementation
  theme/
    light.ts             — CodeMirror light theme
    dark.ts              — CodeMirror dark theme
public/
  .well-known/llms.txt   — LLM crawler metadata
```

## Key Patterns

- **Storage adapter pattern:** All persistence goes through `StorageAdapter` interface. `createAdapter()` picks IndexedDB (more reliable on mobile/Android) with localStorage fallback.
- **Auto-save:** Debounced 500ms after last keystroke, serialized saves with snapshot deduplication.
- **Theme:** Three-way toggle (Auto/Light/Dark). Auto follows OS `prefers-color-scheme` with live listener. Inline script in index.html pre-pins `data-theme` to avoid FOUC.
- **LLM-friendly:** `.well-known/llms.txt` for crawlers. "Copy with metadata" button exports content with YAML frontmatter (language, chars, words, lines, timestamp).

## Commands

```bash
npm run dev          # Dev server
npm run build        # Production build
npm run preview      # Preview production build
npm test             # Run tests (vitest)
npm run test:watch   # Watch mode
```

## Deployment

- Cloudflare Pages (auto-deploy on push to master)
- Build command: `npm run build`
- Output directory: `dist`
- GitHub: github.com/LZong-tw/just-text

## Conventions

- Vue SFC with `<script setup lang="ts">`
- Composables in `src/composables/`, prefixed with `use`
- Storage adapters implement `StorageAdapter` interface in `src/storage/`
- No CSS framework — scoped styles + CSS variables for theming
- Keep dependencies minimal — this is a lightweight tool

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LZong-tw) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
