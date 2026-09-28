## unnamed-skill

> The app is delivered as a static SPA served from `index.html`. UI layout and styling live in `assets/css/style.css`. Core client logic is split across the `assets/js` folder: `app.js` drives navigation/state, `db.js` wraps IndexedDB for offline storage, `crypto.js` handles PBKDF2/AES-GCM encryption, and `exporter.js` covers import/export workflows. Keep feature-specific helpers co-located in `assets/js` and expose a single entry function that `index.html` can import. The schema prototype in `silent_voice_schema.sql` documents the future server-side tables; update it whenever the client data model evolves.

# Repository Guidelines

## Project Structure & Module Organization
The app is delivered as a static SPA served from `index.html`. UI layout and styling live in `assets/css/style.css`. Core client logic is split across the `assets/js` folder: `app.js` drives navigation/state, `db.js` wraps IndexedDB for offline storage, `crypto.js` handles PBKDF2/AES-GCM encryption, and `exporter.js` covers import/export workflows. Keep feature-specific helpers co-located in `assets/js` and expose a single entry function that `index.html` can import. The schema prototype in `silent_voice_schema.sql` documents the future server-side tables; update it whenever the client data model evolves.

## Build, Test, and Development Commands
No build step is required. For local preview use `python3 -m http.server 4173` (or `npx serve .`) from the repository root and open `http://localhost:4173`. Clear browser storage with `localStorage.clear()` between sessions when testing seeded data. When modifying SQL, validate syntax with `sqlite3 silent_voice_schema.sql`.

## Coding Style & Naming Conventions
JavaScript uses ES modules with 2-space indentation, semicolons, and double quotes. Keep arrow functions for inline callbacks and prefer `const`/`let` over `var`. DOM IDs/classes follow `kebab-case`, while exported functions stay camelCase (e.g. `exportAuditCSV`). CSS tokens sit at the top of `style.css`; extend them instead of hard-coding colors. Maintain compact spacing used in existing files and group related functions with banner comments when needed.

## Testing Guidelines
Automated tests are not yet wired in; prioritize consistent manual smoke tests in Chromium and Safari: verify report creation, dashboard filters, encrypted bundle export/import, and print view in `#page-flow`. When adding regression scripts, place them under a new `tests/` directory and document the command in this guide. Aim to keep localStorage/IndexedDB backward compatible, incrementing object store versions cautiously.

## Commit & Pull Request Guidelines
The repo currently has no public history, so adopt Conventional Commit prefixes (`feat`, `fix`, `docs`, `refactor`) and keep subject lines under 72 characters. Each pull request should link to its issue, summarize UI/UX impacts, attach before/after screenshots for frontend changes, and list manual test steps executed. Flag any schema adjustments from `silent_voice_schema.sql` in both the PR body and changelog entry.

---
> Source: [yukitagushi/-](https://github.com/yukitagushi/-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
