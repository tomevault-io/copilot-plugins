## mapdownload

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MapDownload is an Electron application for downloading map tiles from various providers (Amap, Baidu, Tencent, OSM, CartoDb, ArcGIS, Tianditu, MapBox). It supports satellite/label layer merging, administrative boundary clipping, and multi-threaded concurrent downloads.

## Build & Development Commands

```bash
# Install dependencies (use Chinese mirror due to large dependencies)
npm install

# Development with hot reload
npm run dev
npm run watch

# Build web assets
npm run build

# Build Electron application (production)
npm run compile

# Run linting
npm run lint

# Run tests
npm run test
```

## Architecture

### Multi-Package Structure

The project follows a standard Electron multi-package architecture:

- `packages/main/` - Electron main process (Node.js environment)
  - Entry: `src/index.js` - Creates BrowserWindow, handles app lifecycle
  - IPC: `src/ipcMain.js` - Handles IPC communication for downloads
  - Worker: `src/downloadWorker.js` - Worker threads for concurrent downloads

- `packages/preload/` - Preload script (bridge between main and renderer)
  - Exposes `window.electron` API via contextBridge

- `packages/renderer/` - Renderer process (Vue 3 application)
  - Entry: `src/index.js` - Vue app initialization
  - Components: `src/components/*.vue` - UI components
  - Utils: `src/utils/` - Core download logic and utilities

### Tile Download Flow

1. **User interaction** (Home.vue): User draws rectangle or selects administrative area
2. **Tile calculation** (fileSave.js → tileTms.js/tileBaidu.js): Calculates tile coordinates based on extent and zoom levels
3. **Download queue** (download.js → DownloadQueue): Manages concurrent downloads with progress tracking
4. **IPC communication**: Renderer sends download requests to main process via `window.electron.ipcRenderer.invoke('save-image', ...)`
5. **Main process** (ipcMain.js): Uses superagent to fetch images and sharp to process/save them

### Key Utility Files

| File | Purpose |
|------|---------|
| `TileLayerCollection.js` | Factory for creating tile layers for different map providers |
| `tilelayers/*.js` | Provider-specific tile URL generators (Amap, Baidu, Tencent, etc.) |
| `download.js` | Core download orchestration: `downloadLoop()`, `downloadClipLoop()`, `retryFailedTask()` |
| `downloadQueue.js` | Concurrent download queue with pause/resume/cancel support |
| `taskManager.js` | Task persistence and state management (localStorage) |
| `failedTilesManager.js` | Tracks failed tiles for retry functionality |
| `config.js` | Download concurrency configuration (1-16 threads) |
| `memoryMonitor.js` | Memory usage monitoring during downloads |

### Map Providers

Different coordinate systems require different handling:
- **TMS/XYZ tiles**: Standard Google/Web Mercator (EPSG:3857) - Most providers
- **Baidu tiles**: Custom BD09 projection - Requires `tileBaidu.js` and `BaiduTileLayer.js`
- **Tianditu**: Requires API key configuration

### Layer Merging

For satellite + label combinations, the app downloads both layers and merges them using sharp's `composite()` operation in the main process.

### Administrative Boundary Clipping

Uses Turf.js (`@turf/boolean-contains`, `@turf/intersect`) to determine tile-polygon relationships:
- `relation === 1`: Fully inside - download normally
- `relation === 2`: Fully outside - skip
- `relation === 3`: Partially inside - clip and download masked image

## Key Patterns

### IPC Communication Pattern

Renderer → Main:
```javascript
// Send (one-way)
window.electron.ipcRenderer.send('ensure-dir', path);

// Invoke (async, returns result)
const result = await window.electron.ipcRenderer.invoke('save-image', params);
```

### Download Configuration

Concurrency is configurable (1-16 threads) via `getDownloadConcurrency()` in `config.js`. Default is CPU cores (clamped to 2-8).

### Memory Management

The `DownloadQueue` explicitly cleans up task references after completion to prevent memory leaks. The `ClipImage` helper is recreated periodically during long downloads.

## Node/Electron Version Requirements

- Node.js >= v16.13
- npm >= v8.1
- Electron 16.0.1

## Important Notes

- Dependencies are large; user may need to use Chinese npm mirrors
- Baidu custom map styles (午夜蓝, 清新蓝, etc.) links are no longer valid
- Downloaded tiles are for personal learning/research only; data belongs to map providers

---
> Source: [Hxy1992/MapDownload](https://github.com/Hxy1992/MapDownload) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
