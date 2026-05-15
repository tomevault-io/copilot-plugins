## accessible-web-widget

> This repository ships a browser widget as IIFE bundles from modular source.

# AGENTS Guide

This repository ships a browser widget as IIFE bundles from modular source.

## Quick Flow

1. Install: `npm install`
2. Build: `npm run build`
3. Smoke test: `npm run test:smoke`
4. Full CI locally: `npm run ci`

## Source of Truth

- Entry: `src/index.js`
- Build output: `dist/accessible-web-widget.js`, `dist/accessible-web-widget.min.js`
- Dist files are committed intentionally (CDN/release consumption).

## Editing Rules

- Keep runtime compatibility for script-tag users.
- Preserve auto-init and `window.AccessibleWebWidget` unless explicitly changing API.
- Run `npm run build` after source edits.
- Keep smoke tests green.

## Key Docs

- `src/README.md`
- `src/constants/README.md`
- `src/styles/README.md`
- `tests/README.md`

---
> Source: [ifrederico/accessible-web-widget](https://github.com/ifrederico/accessible-web-widget) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
