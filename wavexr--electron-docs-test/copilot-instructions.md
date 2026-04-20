## electron-docs-test

> An Electron + React desktop app that reads Google Sheets/Docs data, displays it in a tabbed dashboard, uses Claude Haiku for analysis, and writes back to sheets.

# CLAUDE.md — Instructions for Claude Code

## What This Project Is
An Electron + React desktop app that reads Google Sheets/Docs data, displays it in a tabbed dashboard, uses Claude Haiku for analysis, and writes back to sheets.

## Key Rules
- All Google API and Anthropic API calls happen in the **main process only** (never in renderer)
- Renderer communicates with main process via IPC (contextBridge)
- Credentials are loaded from `.env` using dotenv — never hardcode secrets
- Use electron-forge with the Vite + React template
- Keep the UI clean and minimal — this is a demo/PoC

## Reference
See PROJECT-SPEC.md for full architecture, IPC channels, data shapes, and tech stack details.

## When Building
- Always check that `npm start` works after making changes
- Handle errors gracefully — return error objects through IPC, don't crash the app
- Keep Google Sheets, Google Docs, and LLM logic in separate files under src/main/
- Share the Google auth instance between sheets and docs modules

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wavexr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
