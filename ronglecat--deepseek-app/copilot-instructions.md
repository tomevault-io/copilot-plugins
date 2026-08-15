## deepseek-app

> Sibling of Grok App. Visual tokens come from Grok App. Agent runtime is DeepSeek Harness (`dsh web`).

# DeepSeek App

Sibling of Grok App. Visual tokens come from Grok App. Agent runtime is DeepSeek Harness (`dsh web`).

Handoff for the next developer: [docs/HANDOFF.md](docs/HANDOFF.md).

## Rules

- All user-facing strings go through `createT(locale)` in `src/renderer/i18n`.
- Do not call DSH `/api` from the renderer. Use `window.desktop.dsh`.
- Do not add Grok-only surfaces (official xAI account, Imagine, Remote IM).
- New Harness Web capability = new row in `docs/FEATURES.md` and `src/renderer/features/catalog.ts`.
- No `window.confirm` / `alert` in product paths if a modal already exists; first-run danger confirm is temporary.

## Layout

- `src/main` — window + dsh spawn + HTTP/WS bridge
- `src/preload` — contextBridge
- `src/renderer` — Grok-styled workbench

---
> Source: [RongleCat/deepseek-app](https://github.com/RongleCat/deepseek-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-15 -->
