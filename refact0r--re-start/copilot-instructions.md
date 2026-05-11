## re-start

> TUI-style browser extension startpage built with Svelte 5.

# re-start

TUI-style browser extension startpage built with Svelte 5.

## Build

- `npm run dev` — dev server at localhost:5173
- `npm run build` / `npm run build:chrome` — production builds to `dist/firefox` or `dist/chrome`
- `npm test` — vitest

## Guidelines

- Write clean, minimal code
- Follow Svelte 5 conventions, do not use deprecated Svelte features
- Make sure styling is consistent with existing styles

## Notes

- **Build manifest post-processing**: `scripts/build-manifest.js` runs after each build via a custom Vite plugin. It mutates `public/manifest.json` per browser — removing `oauth2`/`identity` for Firefox, removing `browser_specific_settings` for Chrome. `CLIENT_ENV` env var selects dev/prod OAuth client ID.
- **Theme injection at build time**: The `injectThemeScript()` Vite plugin embeds theme CSS custom properties directly into HTML to prevent FOUC. Theme definitions live in `src/lib/themes.js`.
- **Google Tasks is Chrome-only**: Depends on `chrome.identity.getAuthToken()` which has no Firefox equivalent. Gated by `src/lib/browser-detect.js`.

---
> Source: [refact0r/re-start](https://github.com/refact0r/re-start) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
