## electron-react-app

> Electron + React starter kit built on electron-conveyor (typed IPC + cross-window state). This

# electron-react-app

Electron + React starter kit built on electron-conveyor (typed IPC + cross-window state). This
branch (`main`) is the minimal shell; the interactive playground lives on the `demo` branch,
which merges `main` forward — structural changes land here first.

- When writing or editing any source, follow the house style in the `code-style` skill —
  section banner format, comment tone and density.
- Checks: `npm run typecheck`, `npm run lint`, `npm run format`.
- `conveyor/modules/*` run in main only; the renderer imports only `type AppRouter`.

---
> Source: [guasam/electron-react-app](https://github.com/guasam/electron-react-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
