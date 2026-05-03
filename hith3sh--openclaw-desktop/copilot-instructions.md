## openclaw-desktop

> **OpenClaw Desktop** is a Windows-first (also supports macOS) Electron desktop application that wraps [OpenClaw](https://github.com/openclaw). It provides a guided onboarding UI, installs platform dependencies (WSL, Node.js, Homebrew on macOS), and manages the OpenClaw gateway process — all without requiring the user to touch a terminal.

# Copilot Instructions for openclaw-desktop

## Project Overview

**OpenClaw Desktop** is a Windows-first (also supports macOS) Electron desktop application that wraps [OpenClaw](https://github.com/openclaw). It provides a guided onboarding UI, installs platform dependencies (WSL, Node.js, Homebrew on macOS), and manages the OpenClaw gateway process — all without requiring the user to touch a terminal.

- **License:** AGPL-3.0-or-later
- **App ID:** `ai.openclaw.desktop`
- **Custom protocol:** `openclawdesktop://`

---

## Architecture

The app follows standard Electron architecture with three distinct layers:

```
src/
├── main/               # Electron main process (Node.js, TypeScript)
│   ├── main.ts         # Entry point: BrowserWindow, Tray, IPC handlers, protocol registration
│   └── services/       # Business logic services (see below)
├── preload/            # Electron preload scripts (context bridge)
├── renderer-react/     # React + Tailwind UI (Vite-built)
│   └── src/
│       ├── App.tsx     # Root component, all major UI views and state
│       └── components/ # Reusable UI primitives (shadcn/ui-style)
├── renderer/           # Legacy plain HTML/CSS renderer (kept for reference)
└── shared/
    └── types.ts        # Shared TypeScript types used by both main and renderer
```

### Main Process Services (`src/main/services/`)

| Service | Responsibility |
|---|---|
| `environment.ts` | Detect/install WSL, Node.js, Homebrew; build WSL commands |
| `setup-orchestrator.ts` | Multi-step setup state machine (WSL → runtime → openclaw → onboarding) |
| `setup-store.ts` | Persist setup state to disk |
| `config-store.ts` | Read/write user app config (auth, provider, model, etc.) |
| `auto-updater.ts` | electron-updater integration for auto-update |
| `workspace-files.ts` | Read/write OpenClaw workspace config files |
| `command-runner.ts` | Spawn child processes and stream output |
| `parsers.ts` | Parse CLI output (gateway status, etc.) |
| `windows-startup.ts` | Windows auto-launch on login |

### IPC Communication

All renderer↔main communication goes through `ipcMain`/`ipcRenderer` with a typed bridge defined in the preload script. IPC channel names are string constants in `main.ts`. Always use the context bridge (`window.electronAPI.*`) in the renderer — never access Node APIs directly from renderer code.

---

## Technology Stack

- **Electron** (latest stable) — desktop shell
- **TypeScript** — all source files (strict mode)
- **React 19** + **Vite** — renderer UI
- **Tailwind CSS v3** + **PostCSS** — styling
- **Radix UI** — accessible UI primitives (`@radix-ui/react-*`)
- **lucide-react** — icons
- **electron-builder** — packaging and distribution
- **electron-updater** — auto-update
- **Node built-in `node:test`** — test runner (no Jest/Mocha)

---

## Build & Development Commands

```bash
# Install dependencies
npm install

# Development (builds then launches Electron)
npm run dev

# Development with experimental React UI
npm run dev:react

# Build only (no Electron launch)
npm run build

# Run tests (builds first, then runs node:test)
npm run test

# Package for distribution
npm run dist:win      # Windows NSIS installer
npm run dist:mac      # macOS dmg + pkg + zip
npm run dist:all      # All targets

# Verify release artifacts
npm run verify:artifacts
```

Tests live in `tests/*.test.js` and are run with Node's built-in `node:test` module after compiling TypeScript (`npm run build:test`). Tests import from `dist/main/services/`.

---

## Code Conventions

### TypeScript
- **Strict mode** is enabled (`tsconfig.json`). All types must be explicit — avoid `any`.
- Use types from `src/shared/types.ts` for anything shared between main and renderer.
- Main process uses CommonJS (`"module": "CommonJS"` in `tsconfig.main.json`).
- Renderer uses ES modules via Vite.

### Services
- Services are classes instantiated once in `main.ts` and passed or called directly.
- Async methods should return typed results, never throw for expected errors — return `{ ok: boolean, ... }` result objects (see `CommandResult` in `shared/types.ts`).

### React Renderer
- All major app state lives in `App.tsx` and is passed down as props.
- Use Tailwind utility classes for styling. Do not use inline styles.
- Component files go in `src/renderer-react/src/components/`.
- Dark/light theme is toggled via a `data-theme` attribute on `<html>` and CSS variables.
- The `OPENCLAW_UI_EXPERIMENT=react` env variable switches to the React renderer in dev mode.

### IPC
- Define all IPC channel names as string constants at the top of `main.ts`.
- All IPC handlers must be registered before `app.whenReady()` resolves.
- Use `ipcMain.handle` for request/response patterns; `webContents.send` for push events.

### File Naming
- TypeScript source files: `kebab-case.ts`
- React components: `PascalCase.tsx`
- Test files: `kebab-case.test.js`

---

## Testing

- Tests are in `tests/` and use Node's built-in `node:test` + `node:assert/strict`.
- Tests operate on compiled JS in `dist/main/` — always run `npm run build:test` before `npm test`, or use `npm test` which does both.
- Tests mock service methods directly on instances (no mocking libraries) — e.g., reassign `service.runWslBash = async () => { ... }`.
- Do not modify or delete existing tests unless the underlying behavior has changed.
- Add a test for every non-trivial service method change.

---

## Release & CI

Tag pushes matching `v*` trigger `.github/workflows/build-release.yml`, which:

1. Builds Windows artifacts on `windows-latest`
2. Builds macOS artifacts on `macos-latest`
3. Verifies expected installer files before publishing
4. Publishes all artifacts to the GitHub Release

Do not manually edit release artifacts. Use `npm run verify:artifacts` locally to check.

---

## Platform Notes

### Windows
- Primary target platform.
- Setup installs WSL 2 (Ubuntu), then Homebrew inside WSL, then Node.js via `nvm`, then `openclaw` via npm.
- The app uses `WindowsStartupService` to manage auto-launch via the Windows registry.
- WSL commands are run via `wsl.exe` with a specific distro and user.

### macOS
- Supported as a secondary target.
- macOS builds produce `.dmg`, `.pkg`, and `.zip` for both `x64` and `arm64`.
- `hardenedRuntime` is disabled; `gatekeeperAssess` is disabled (for dev/CI builds).

---

## Security Considerations

- The app handles OAuth tokens for authentication. Never log or expose tokens.
- The custom protocol handler (`openclawdesktop://`) is used for OAuth callbacks. Validate `state` parameters before processing.
- Config files stored on disk may contain sensitive credentials — do not log config objects.
- IPC handlers should validate all inputs from the renderer before passing to Node.js APIs.

---

## Key Files for Orientation

| File | Purpose |
|---|---|
| `src/main/main.ts` | Electron entry, all IPC handlers, window/tray management |
| `src/shared/types.ts` | All shared TypeScript interfaces and types |
| `src/main/services/environment.ts` | WSL/runtime detection and installation |
| `src/main/services/setup-orchestrator.ts` | Full onboarding state machine |
| `src/renderer-react/src/App.tsx` | Entire React UI |
| `tests/` | All unit tests |
| `.github/workflows/build-release.yml` | CI/CD release pipeline |

---
> Source: [hith3sh/openclaw-desktop](https://github.com/hith3sh/openclaw-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
