## transmission-electron-gui

> This repository is an Electron + Vite + React + TypeScript desktop client for Transmission daemon RPC.

# Copilot Instructions

This repository is an Electron + Vite + React + TypeScript desktop client for Transmission daemon RPC.

- Keep the Electron main/preload split secure; do not enable renderer Node integration.
- Use the typed IPC surface exposed through `window.transmission` for renderer-to-main communication.
- Keep Transmission RPC request/response types in `src/shared/types.ts`.
- Prefer focused, dense desktop UI patterns over marketing-style layouts.
- Run `npm.cmd run build` or `./build.cmd` on Windows before publishing changes.
- Do not commit generated build output, `node_modules`, local secrets, or Electron user data.

---
> Source: [adameat/transmission-electron-gui](https://github.com/adameat/transmission-electron-gui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
