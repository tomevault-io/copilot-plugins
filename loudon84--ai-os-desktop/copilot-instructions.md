## 001-electron-architecture

> Electron three-process architecture and Main/Preload/Renderer boundaries


# Electron Architecture Rules

This project is an Electron desktop shell for Hermes Agent.

## Hard constraints

- Renderer must not access Node.js APIs.
- Renderer must not import `fs`, `path`, `child_process`, `better-sqlite3`, `electron`.
- Renderer must communicate only through `window.hermesAPI` (and `window.profileRuntime`, `window.profileEntry`, `window.aiosBrowser` where applicable).
- Preload is the only bridge between Renderer and Main.
- Main process owns filesystem, process spawning, SQLite, gateway lifecycle.

## New backend capability flow

1. Implement domain module in `src/main/`
2. Register `ipcMain.handle` in `src/main/index.ts` (or dedicated `*-ipc.ts` registered from index)
3. Expose typed wrapper in `src/preload/index.ts` (or dedicated `*-api.ts`)
4. Declare type in `src/preload/index.d.ts`
5. Consume from Renderer

## Do not

- Create alternative IPC channels outside the established preload → main contract.
- Bypass `profileHome()` for profile-scoped data paths.
- Modify Gateway start/stop logic without reading `src/main/hermes.ts` and profile-runtime modules first.

---
> Source: [loudon84/ai-os-desktop](https://github.com/loudon84/ai-os-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
