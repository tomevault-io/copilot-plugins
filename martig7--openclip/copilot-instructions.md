## openclip

> OpenClip is a Windows Electron + React desktop app for automatic game recording with OBS Studio. The main code lives in `electron-app/`. The native OBS C plugin in `obs-plugin/` is optional and requires CMake + MSVC to build.

# AGENTS.md

## Cursor Cloud specific instructions

### Project overview

OpenClip is a Windows Electron + React desktop app for automatic game recording with OBS Studio. The main code lives in `electron-app/`. The native OBS C plugin in `obs-plugin/` is optional and requires CMake + MSVC to build.

### Development commands

All commands run from `electron-app/`:

- **Install deps:** `npm install`
- **Dev server (frontend only):** `npx vite` (serves React at http://localhost:5173)
- **Dev server (full Electron):** `npm run dev` (requires a display; launches Vite + Electron)
- **Lint:** `npm run lint`
- **Unit tests:** `npm run test:unit` or `npx vitest run`
- **E2E tests:** `npm run test:e2e` (Playwright; needs `npx playwright install chromium` first)
- **Full test suite:** `npm run test` (includes vitest + Playwright + integration tests)
- **Integration (OBS WS + plugin harness):** `npm run test:integration` — headless OBS WebSocket tests under `tests/integration/obs/`, plus **mock OpenClip plugin HTTP** contract tests under `tests/integration/plugin/` (no OBS/DLL required).
- **Build:** `npm run build` (runs `build:plugin` then Vite production build to `dist/`)
- **OBS plugin only:** `npm run build:plugin` (CMake Release → `resources/obs-plugin/openclip-obs.dll`)
- **Plugin incremental (tests):** `npm run build:plugin:ensure` — rebuilds only when `obs-plugin/` sources are newer than the staged DLL (used by `pretest`, `test:integration`, `test:e2e:integration`). On non-Windows this no-ops. Set **`OPENCLIP_SKIP_PLUGIN_BUILD=1`** to skip the ensure step.
- **Plugin watch (dev):** `npm run build:plugin:watch` — rebuild on every change under `obs-plugin/src` and `obs-plugin/include` (Windows only; run in a second terminal).

### OpenClip OBS plugin test harness

- The native plugin speaks **HTTP** (`POST /api` JSON `{ method, params }`) on a port written to `%APPDATA%\open-clip\runtime\plugin_port` (see `electron/pluginHttpTransport.js`).
- **`tests/integration/plugin/pluginHarness.mjs`** starts an in-process mock that implements the same wire format — use **`OPENCLIP_PLUGIN_HTTP_PORT`** so `resolvePluginPort()` targets the harness without Electron.
- **`electron/pluginHttpTransport.js`** is Electron-free and shared with the Electron `obsPlugin.js` client; integration tests import it directly.

### Linux / Cloud VM caveats

- **koffi FFI failures are expected on Linux.** The `electron/winUtils.js` module loads Windows DLLs via koffi at import time. Tests that transitively import it (all `tests/api/*` suites, plus `fileManager.*` and `moveFileSafe` unit tests) will fail on Linux with "Failed to load shared library". This is not a code bug — those tests only pass on Windows.
- **Electron cannot launch headless on this VM** (no display + Windows-only features). Use `npx vite` alone for frontend dev, or run Playwright E2E tests which start their own Chromium.
- The Playwright E2E tests use mocked data and route interception, so they work fully on Linux without OBS or Electron.
- **Passing test scope on Linux:** 41 vitest suites pass (385 tests), 110 Playwright E2E tests pass. The 12 failing vitest suites (47 tests) all fail due to the koffi/Windows dependency, not code bugs.

### Frontend / viewer testing note

- **`electron-app/src/viewer/components/TestMediaSidebar.jsx`** is a **test-only** sidebar mock (used by `tests/components/TestMediaSidebar.test.jsx`). The real recordings/clips UI uses **`MediaSidebar.jsx`**. The mock exists so Vitest can assert sort/group/search/list behavior in jsdom without importing production sidebar code that is awkward to test when **canvas** (and related APIs) are involved or poorly supported in the test environment.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/martig7)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/martig7)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
