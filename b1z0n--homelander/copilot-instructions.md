## homelander

> Homelander is an Electron desktop app (v1.1.4) that automates apartment applications on ImmobilienScout24. It uses React 19 + Vite for the UI, Puppeteer 24 for browser automation, and better-sqlite3 for local storage.

# GitHub Copilot Instructions — Homelander

Homelander is an Electron desktop app (v1.1.4) that automates apartment applications on ImmobilienScout24. It uses React 19 + Vite for the UI, Puppeteer 24 for browser automation, and better-sqlite3 for local storage.

## When making changes

- **Electron main process** lives in `electron/`. `main.js` manages lifecycle, daemon, config, and IPC. `chrome.js` wraps Puppeteer CDP. `preload.cjs` is the contextBridge — keep it CommonJS (`.cjs`), do NOT convert to ESM.
- **Daemon** lives in `engine/`. It's forked from main, communicates via stdout JSON lines + IPC. Two loops: `pollLoop` (IS24 mobile API) and `applyLoop` (CDP form filling). Config is hot-reloaded via IPC patches.
- **Renderer** lives in `src/`. React 19, Zustand store (`appStore.js`), Tailwind CSS 4, i18n via `LocaleContext`. IPC goes through `window.homelander.*` (from preload).
- **Tests** in `test/`. Run with `npm test`. Set `HOMELANDER_TEST_FAST=1` to skip jitter. CI uses `PUPPETEER_SKIP_DOWNLOAD=true`.

## Code conventions

- ESM with `"type": "module"` in `package.json` — EXCEPT `preload.cjs` which MUST remain CommonJS
- No TypeScript — plain JavaScript with JSDoc
- Zustand store is the single source of truth in the renderer
- Daemon events are JSON objects with `type` field: `stats`, `listing`, `captcha_wall`, `paused`, `resumed`, `error`, `poll_error`, `session_expired`, `chrome_dead`
- Config lives at `~/.homelander/config.json` — NOT the old `config/autoapply.config.yaml` (Fredy-based, obsolete)
- Data at `~/.homelander/homelander.db` (SQLite WAL mode)

## Brand

- Gold `#D9A441` on white `#FFFFFF`. Never recolored except black/white utility variants.
- macOS `.icns` MUST be opaque white background (not transparent).
- Never use `app.dock.setIcon()` — bypasses squircle mask.

## Critical invariants

- Never touch the system Chrome — always use Puppeteer-bundled Chromium
- Don't poll IS24 login/SSO pages — they're bot-sensitive
- `preload.cjs` stays CommonJS forever
- Release is `workflow_dispatch` only — never auto-triggered
- Use `contextBridge.exposeInMainWorld()` for IPC, never expose `ipcRenderer` directly
- Support bundles: HTML snapshots are raw-copied (no redaction in Electron main)

---
> Source: [B1Z0N/homelander](https://github.com/B1Z0N/homelander) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
