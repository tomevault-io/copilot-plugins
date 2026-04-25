## www-cookie-tool

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Electron desktop application for generating cookie-setting URLs across multiple brands and TLDs. Built with React, TypeScript, Vite, and TailwindCSS.

**Core functionality**: Generate URLs in the format `https://www.{brand}.{tld}/global/cookie/set/{encoded}` where `{encoded}` contains URL-encoded cookie data (`c=n={name}&v={value},...`) and optionally a base64-encoded destination URL (`|d={base64}`).

## Development Commands

```bash
# Development mode with hot reload and DevTools
npm run dev

# Build for macOS - both Intel and Apple Silicon (creates .dmg and .zip in release/{version}/)
npm run build:mac

# Build for macOS - Intel only (x64)
npm run build:mac:intel

# Build for macOS - Apple Silicon only (arm64)
npm run build:mac:arm

# Build for macOS - Universal binary (larger, includes both architectures)
npm run build:mac:universal

# Build for Windows (creates NSIS installer in release/{version}/)
npm run build:win

# Build for all platforms and architectures
npm run build

# Preview production build
npm run preview
```

## Build Process

The build is a multi-step process that must run in sequence:

1. `tsc` - Compiles TypeScript for Electron main process (outputs to dist-electron/)
2. `tsc -p tsconfig.preload.json` - Compiles preload script
3. `mv dist-electron/preload.js dist-electron/preload.cjs` - Renames preload to CommonJS format (required by Electron's contextBridge)
4. `vite` or `vite build` - Builds React frontend (outputs to dist/)
5. `electron-builder` (build only) - Packages the app for distribution

The preload script **must** be .cjs because Electron's main process loads it with CommonJS, even though the main process itself uses ES modules.

## Architecture

### System Tray Integration

The app includes a system tray icon that provides:
- **Persistent access**: App remains accessible via tray even when window is closed
- **Quick actions**: Right-click tray icon for Show App/Quit menu
- **Smart behavior**: On macOS, closing the window hides to tray instead of quitting
- **Template icon**: Monochrome icon automatically adapts to light/dark mode on macOS

Tray icon resources are located in `resources/trayTemplate.svg`.

### Electron IPC Architecture

The app uses a secure IPC pattern with strict isolation:

- **Main process** (electron/main.ts): Manages app lifecycle, window creation, IPC handlers, and electron-store access
- **Preload script** (electron/preload.ts): Exposes whitelisted APIs via contextBridge as `window.electronAPI`
- **Renderer process** (React app): Communicates only through `window.electronAPI` methods

**IPC Operations**:
- `copy-to-clipboard` / `open-in-browser`: System integrations
- `save-preset` / `load-presets` / `delete-preset`: Preset management
- `save-to-history` / `load-history` / `clear-history`: URL history (limited to 20 entries)

All IPC calls use `ipcRenderer.invoke()` for async request-response pattern.

### Data Flow

1. **User input** → React components update state via `useUrlGenerator` hook
2. **Generate URL** → `generateCookieUrl()` in utils/urlEncoder.ts encodes cookies and destination
3. **Save to history** → IPC call to main process → electron-store persists to disk
4. **Copy/Open URL** → IPC calls to main process for clipboard and browser operations

### State Management

Primary state is managed in `src/hooks/useUrlGenerator.ts`:
- Form state (brand, TLD, cookies, destination)
- Generated URL and error state
- Operations: addCookie, removeCookie, updateCookie, generateUrl, loadPreset, reset

Persistent state (presets and history) is managed by `src/hooks/useStorage.ts` which wraps IPC calls.

### URL Encoding Logic

Located in `src/utils/urlEncoder.ts`. Critical format:
- Cookies: `c=n={name}&v={value},n={name2}&v={value2}`
- With destination: `c={cookies}|d={base64Destination}`
- Final URL: Base path + URL-encoded payload

### Component Structure

Component hierarchy in src/components/:
- **BrandSelector** / **TldSelector**: Dropdowns for brand and TLD selection
- **CookieList** → **CookieInput**: Dynamic list of cookie name/value pairs
- **DestinationInput**: Optional base64-encoded destination URL
- **GeneratedUrl**: Display with copy/open actions
- **PresetManager**: Save/load/delete configuration presets
- **UrlHistory**: Display last 20 generated URLs with copy functionality

All components are functional with TypeScript props interfaces.

## Configuration Files

- **vite.config.ts**: Configures vite-plugin-electron for main and preload builds, React plugin for renderer
- **electron-builder.json**: Packaging config (appId, output directory, macOS/Windows targets)
- **tsconfig.json**: Main TS config for Electron main process (ES modules, node18)
- **tsconfig.preload.json**: Preload-specific config (outputs .js that gets renamed to .cjs)
- **tsconfig.node.json**: Node tooling config
- **tailwind.config.js** / **postcss.config.js**: TailwindCSS setup

## Data Storage

Uses electron-store for persistent data in JSON format:
- **macOS**: `~/Library/Application Support/cookie-tool/config.json`
- **Windows**: `%APPDATA%\cookie-tool\config.json`

Storage schema:
- `presets`: Object mapping preset names to Preset objects
- `history`: Array of HistoryEntry objects (limited to 20, newest first)

## Constants

Defined in src/utils/constants.ts:
- **BRANDS**: 13 brand domains (alphabetically sorted)
- **TLDS**: ['qa.protected.net', 'xyz', 'com']

## Type Definitions

All types in src/utils/types.ts:
- **Cookie**: { name: string, value: string }
- **Preset**: { brand, tld, cookies[], destination? }
- **HistoryEntry**: { url, timestamp, brand, tld }
- **Window.electronAPI**: Global type augmentation for IPC methods

## Security Notes

- Context isolation enabled (contextIsolation: true)
- Node integration disabled in renderer (nodeIntegration: false)
- All main process operations exposed through whitelisted preload APIs only
- External URLs open in system browser, never in-app webviews

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/msievenpiper) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
