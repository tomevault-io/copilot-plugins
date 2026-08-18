## devilconnection-web

> > This document is written for AI coding agents who have no prior knowledge of the project. It describes the actual structure, technology stack, runtime behavior, and development conventions observed in the source tree.

# AGENTS.md — DevilConnection Browser Port

> This document is written for AI coding agents who have no prior knowledge of the project. It describes the actual structure, technology stack, runtime behavior, and development conventions observed in the source tree.

## Project overview

This repository is a **browser port of the visual novel *DevilConnection* (恶魔连结 / デビルコネクション)**. It is built on top of the **TyranoScript** visual novel engine (KAG format) and runs as a static HTML5 application in a web browser.

The repository name in `package.json` is `devil-connection-browser`. The original project used Electron + Steamworks; the current browser version replaces the native preload APIs with a browser shim so the same TyranoScript assets can run without Electron or Node.js.

Key facts:

- **No build step is required** to run the game in a browser.
- **No server-side component** exists.
- All game logic is written in TyranoScript scenario files (`.ks`) and vanilla JavaScript.
- The game is intended to be served as static files (`index.html`, `data/`, `tyrano/`).

## Technology stack

- **Frontend runtime**: HTML5, CSS, vanilla JavaScript (ES5-style IIFE modules), jQuery 3.6.0, jQuery UI, jQuery Migrate.
- **Visual novel engine**: TyranoScript (KAG script format, `*.ks`).
- **Audio**: Howler.js + HTML5 `<audio>` for autoplay policy handling.
- **Dialogs / overlays**: SweetAlert2, Remodal, Alertify.js (legacy).
- **Utilities**: JSZip, html2canvas, jsrender, LZString, jsQR, APNG support (`apng.js`, `blob.js`).
- **3D support (mostly unused)**: Three.js and its loaders (the config has `use3D=false`).
- **Text effects**: textillate, animate.css, lettering.js, touchSwipe.
- **Node usage**: only for a small `serve` script in `package.json`; no bundler, transpiler, or test framework is present.
- **Legacy native shell**: `_electron_legacy/` contains the original Electron main/preload/Steam integration, but the current `index.html` does not load it.

## Key configuration files

| File | Purpose |
|------|---------|
| `package.json` | Project metadata. Only defines `scripts.serve` which runs `npx -y serve .`. |
| `index.html` | Entry point. Loads all libraries, the browser API shim, and the Tyrano engine, then waits for the user click on `#tyrano_click_to_start` before initializing the game. |
| `data/system/Config.tjs` | TyranoScript game configuration (screen size, text speed, default volumes, save settings, etc.). Generated/managed by TyranoBuilder. |
| `data/system/KeyConfig.js` | Keyboard/mouse/gesture bindings for the game. |
| `tyrano/lang.js` | In-game text strings. The current file contains Simplified Chinese UI text (with some Japanese names) for this port. |
| `data/scenario/first.ks` | First scenario loaded by TyranoScript after initialization. It loads system macros and jumps to `title_screen.ks`. |
| `browser_api.js` | Browser-only `window.api` shim (file system, storage, dialogs, fullscreen, etc.). |
| `electron_latest.js` | Browser adaptation overrides for TyranoScript core behavior (save integration, patch disabling, `web`/`close` tags). |

## Directory layout

```
.
├── index.html              # Application entry point
├── package.json            # Minimal Node metadata / serve script
├── browser_api.js          # Browser shim for Electron preload APIs
├── electron_latest.js      # Tyrano runtime browser patches
├── favicon.ico
├── data/                   # Game assets and scripts
│   ├── bgimage/            # Background images
│   ├── fgimage/            # Foreground / character images
│   ├── image/              # UI / misc images
│   ├── sound/              # Sound effects
│   ├── bgm/                # Background music
│   ├── video/              # Movies
│   ├── others/             # Custom JavaScript, plugins, and game data
│   │   ├── plugin/         # TyranoBuilder-style plugins (init.ks + JS)
│   │   └── *.js            # Game-specific helpers (master_data, loading overlay, etc.)
│   ├── scenario/           # Main KAG scenario files
│   │   ├── system/         # System macros and auto-generated backups (`_*.ks`)
│   │   └── *.ks            # Story scripts
│   └── system/             # Tyrano system files (Config.tjs, KeyConfig.js)
├── tyrano/                 # TyranoScript engine
│   ├── tyrano.js           # Core bootstrap
│   ├── tyrano.base.js      # Base layer / screen scaling
│   ├── libs.js             # jQuery / utility extensions
│   ├── lang.js             # In-game strings
│   ├── libs/               # Third-party libraries
│   ├── plugins/kag/        # KAG engine plugin (kag.js, tags, parser, menu, etc.)
│   ├── css/                # Engine styles
│   └── images/             # Engine default images
└── _electron_legacy/       # Original Electron + Steamworks files (not used by browser build)
```

## Runtime architecture

1. The browser loads `index.html`.
2. `browser_api.js` runs first and creates `window.api`, a browser-compatible replacement for the Electron preload API (storage, file dialogs, fullscreen, etc.).
3. Core libraries (jQuery, jQuery UI, Howler, SweetAlert2, etc.) and the Tyrano engine (`tyrano/tyrano.js`, `tyrano/tyrano.base.js`, `tyrano/plugins/kag/*.js`) are loaded.
4. `electron_latest.js` patches Tyrano core methods for the browser:
   - Forces `configSave = 'webstorage'` so saves use the browser shim.
   - Disables patch application and web-patch checks.
   - Overrides the `web`, `close`, and `check_web_patch` KAG tags.
   - Hooks `TYRANO.init` so IndexedDB storage is ready before the game starts.
5. The user must click the `#tyrano_click_to_start` overlay. This satisfies browser autoplay policies, resumes the Howler `AudioContext`, and then calls `TYRANO.init()`.
6. `TYRANO.init()` loads the `kag` plugin, which reads `data/system/Config.tjs` and starts `data/scenario/first.ks`.
7. `first.ks` loads system macros (`system/tyrano.ks`, `system/builder.ks`, `system/chara_define.ks`), sets up the message window, loads plugins, and jumps to `title_screen.ks`.

### Save / storage system

- The browser version stores save data in **IndexedDB** (`tyrano_browser_storage` / `kv`) via `window.api.storage`.
- IndexedDB is loaded into memory at startup; writes are flushed asynchronously on a short timer and on `beforeunload` / `pagehide` / `visibilitychange`.
- If IndexedDB is unavailable, it falls back to `localStorage`.
- On first run, existing `localStorage` save keys are migrated into IndexedDB.
- Save data is JSON-encoded and percent-encoded (matching the Steam/Electron `.sav` format). `validateSaveData` parses the decoded JSON; if parsing fails, **all storage is cleared** (controlled by `TYRANO.clear_on_corrupt_save`).

## Browser port layer

The two files that make the browser port possible are:

- **`browser_api.js`**: Implements the `window.api` contract expected by the Tyrano engine. Includes file-system shims, save-file download (`saveFile`), dialog shims, fullscreen, and the IndexedDB-backed `storage` object.
- **`electron_latest.js`**: Implements Tyrano runtime behavior that differs between Electron and the browser. It overrides `kag.init`, `checkUpdate`, `applyPatch`, and several KAG tags, and extends jQuery with browser-compatible storage helpers.

Both files are written as immediately-invoked function expressions (IIFE) so they can be loaded as plain `<script>` tags. They rely on globals such as `jQuery`, `TYRANO`, `Howler`, `Swal`, and `LZString`.

## Legacy Electron / Steam build

The `_electron_legacy/` directory contains the original native shell:

- `main.js` — Electron main process (obfuscated/minified). Handles window creation, IPC, file I/O, encryption, patch application, and Steamworks integration.
- `preload.js` — Exposes the native `window.api` to the renderer.
- `steam.js` — Initializes the Steamworks client with a hardcoded Steam App ID.

The current `index.html` does **not** reference these files. They are kept for reference or for producing a future Electron build.

## Build and run commands

```bash
# Serve the project as static files (uses `serve` via npx)
npm run serve
```

Because the project is static, any static file server works. For example:

```bash
npx serve .
# or
python3 -m http.server 3000
```

There is **no production build**, **no bundler**, and **no transpilation step**.

## Development conventions

### Scenario scripts (`.ks`)

- TyranoScript commands are written in KAG format: `[command param=value]` or `@command param=value`.
- Comments start with a semicolon: `; this is a comment`.
- Labels use `*label_name`.
- Macros are defined with `[macro name=...] ... [endmacro]` and are typically placed under `data/scenario/system/`.
- JavaScript embedded in a scenario uses `[iscript] ... [endscript]`.
- `data/scenario/system/_*.ks` files appear to be auto-generated backup / preview copies of the main scenario files.

### Custom plugins (`data/others/plugin/`)

- Most plugins contain an `init.ks` that defines KAG macros or loads a JS file, plus a `main.js` that registers custom tags on `TYRANO.kag.ftag.master_tag`.
- Some plugins also contain a `*.builder.js` file. These are TyranoBuilder component definitions written as CommonJS modules exporting a `plugin_setting` class.

### JavaScript style

- Engine and shim code uses ES5-style function declarations, `var`, and IIFEs.
- `let`/`const` appear only in newer shim code (`browser_api.js`, `electron_latest.js`).
- Global namespaces are heavily used: `TYRANO`, `TYRANO.kag`, `TYRANO.kag.ftag`, `TYRANO.kag.variable.sf/tf`, `jQuery` (`$`).
- File paths in the browser are resolved relative to `location.href` via `getBasePath()` in `browser_api.js`.

### Assets

- Image/audio/video paths referenced from KAG are relative to `data/` (e.g. `storage="image/menu/op.png"` resolves to `data/image/menu/op.png`).
- Default media format is configured as `ogg` in `Config.tjs`.
- Screen design size is `1280x960` with `ScreenRatio=fix` and `ScreenCentering=true`.

## Testing / debugging

- There is **no automated test suite** (no Jest, Mocha, Vitest, Cypress, etc.).
- Manual testing is done by running the game in a browser and exercising the scenario flow.
- `data/system/Config.tjs` has `debugMenu.visible=true`, and `data/scenario/config.ks` loads `debug_menu.js`, so an in-game debug menu is available from the config screen.
- `data/scenario/tester.ks` and files prefixed with `AAAA_` or `_AAAA_` appear to be debug / scratch scenarios.
- To inspect runtime state, use the browser DevTools and look at globals such as `TYRANO.kag.variable.sf`, `TYRANO.kag.stat`, and `TG`.

## Deployment

Deploy the following as static files:

```
index.html
favicon.ico
browser_api.js
electron_latest.js
data/
tyrano/
```

Notes for deployment:

- The server must serve `.ks` files with a text MIME type (or the browser will still load them as text because they are fetched with `fetch` / `$.loadText`).
- Audio files (`.ogg`) must be served with the correct audio MIME type.
- The game does not require any backend API.
- Make sure autoplay policy is satisfied by the click-to-start overlay; do not remove `#tyrano_click_to_start`.

## Security considerations

- The game executes JavaScript from `[iscript]` blocks in scenario files and from arbitrary `.js` files loaded via `[loadjs]`. Treat all `.ks` and `.js` content as trusted.
- There is no Content Security Policy defined in `index.html`.
- `browser_api.js` uses `fetch` to load binary/text resources and `document.createElement('a')` with `.download` for photo export. Paths are resolved relative to the page URL.
- Save data is stored in the client’s IndexedDB / localStorage. The browser shim validates JSON on read and clears all storage if data appears corrupted.
- The original Electron build uses `shell.openExternal`, native dialogs, file I/O, and Steamworks; do not enable those paths in the browser build.
- Patch application (`applyPatch`) is explicitly disabled in the browser build to prevent arbitrary file extraction.

---
> Source: [lllhhh2282/devilconnection-web](https://github.com/lllhhh2282/devilconnection-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
