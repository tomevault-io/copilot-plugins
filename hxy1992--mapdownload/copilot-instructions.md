## mapdownload

> Guidance for agent tools working in this repo. See also `CLAUDE.md` (architecture detail) and `QWEN.md`.

# AGENTS.md

Guidance for agent tools working in this repo. See also `CLAUDE.md` (architecture detail) and `QWEN.md`.

## Commands

```bash
npm install          # dependencies are large; use a Chinese npm mirror if slow
npm run watch        # dev with hot reload (alias: npm run dev)
npm run build        # vite-build all 3 packages (main, preload, renderer); uses --max_old_space_size=8192
npm run compile      # electron-builder --dir, asar disabled; runs production build first via precompile
npm run lint         # eslint . --ext js,ts,vue
npm run test         # playwright tests; runs `npm run build` first via pretest (build output is required)
```

## Architecture

Three Vite-built packages, each with `/@/` aliased to its own `src/`:

- `packages/main/` — Electron main process. Entry `src/index.js`; IPC in `src/ipcMain.js` and `src/ipHandle.js`; concurrent download via worker_threads in `src/downloadWorker.js`. Downloads fetched with superagent, processed with sharp. Builds to CJS `.cjs`.
- `packages/preload/` — contextBridge bridge exposing `window.electron`. Builds to CJS `.cjs`.
- `packages/renderer/` — Vue 3 + Naive UI + maptalks UI. Core download logic lives in `src/utils/` (NOT components): tile calc (`fileSave.js` → `tileTms.js`/`tileBaidu.js`), queue (`download.js`, `downloadQueue.js`, `downloadCascadeTiles.js`), state (`taskManager.js`, `failedTilesManager.js`), `config.js`, `memoryMonitor.js`.

`package.json` `main` points at `packages/main/dist/index.cjs`; electron-builder packs `packages/**/dist/**`. `dist/` is gitignored.

## Key flow / patterns

- Renderer computes tile coordinates, then sends each tile to main via `window.electron.ipcRenderer.invoke('save-image', ...)`; main does network + sharp processing + file save.
- Layer merge (satellite+labels) uses sharp `composite()` in main.
- Boundary clipping: Turf.js relation checks — `1` fully inside (download), `2` outside (skip), `3` partial (clip/mask with `clipImage.js`).
- Concurrency 1–16 via `getDownloadConcurrency()` in `config.js` (default: CPU cores clamped 2–8).
- Memory leaks have been a recurring issue (see git history): `DownloadQueue` cleans task refs on completion; avoid accumulating per-tile references.

## Gotchas

- Baidu tiles use the custom BD09 projection (`tileBaidu.js`, `BaiduTileLayer.js`); never treat them as plain XYZ/TMS.
- Tianditu needs an API key.
- Baidu custom map style URLs (午夜蓝, 清新蓝, etc.) are dead — do not "fix" them by guessing.
- Toolchain is old and pinned: Electron 16.0.1, Vite 2.6, Vue 3.2. Node must be `>=16.13.0 <19` — Node 19+ breaks the build (confirmed on Node 24). Enforced via `package.json` engines + `.npmrc` `engine-strict=true`; `.nvmrc`/`.node-version` pin 16.13.0. Do not upgrade tools casually; `.electron-vendors.cache.json` drives build targets.
- Lint style: semicolons required, single quotes, trailing commas in multiline (`eslint:recommended`).
- Recent fixes touch Google tile download uncaught errors and download memory leaks — keep those paths in mind when touching download code.

---
> Source: [Hxy1992/MapDownload](https://github.com/Hxy1992/MapDownload) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
