## youtube-local

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

YouTube Local is an Electron + React desktop application for managing and playing a personal video library stored on local or external drives. Videos are indexed in-place (never copied) with metadata persisted in SQLite.

## Development Commands

```bash
npm run dev              # Start Vite + Electron concurrently (main workflow)
npm run dev:vite         # Vite dev server only (port 5173)
npm run dev:electron     # Electron only (requires Vite running)
npm run build            # Build React/Vite production bundle
npm run build:electron   # Create distributable packages via electron-builder
npm run rebuild          # Rebuild native modules (better-sqlite3)
```

After `npm install`, the postinstall script automatically runs `electron-rebuild` for better-sqlite3.

## Architecture

### Process Model (Electron)
- **Main process** (`src/main/`): Node.js runtime - handles database, file system, FFmpeg operations
- **Renderer process** (`src/renderer/`): React app - UI and user interactions
- **Preload** (`src/preload/index.js`): Secure IPC bridge exposing `window.electronAPI`

### Key Main Process Modules
| File | Purpose |
|------|---------|
| `index.js` | Window creation, app lifecycle |
| `database.js` | SQLite schema, better-sqlite3 connection, prepared statements |
| `scanner.js` | Recursive video file discovery |
| `fileWatcher.js` | Chokidar-based real-time monitoring |
| `diskDetection.js` | Multi-disk detection and reconnection handling |
| `thumbnailGenerator.js` | FFmpeg thumbnail extraction |
| `ipc/*.js` | IPC handlers grouped by feature |

### IPC Communication Pattern
All renderer-to-main communication uses `ipcRenderer.invoke()` through the preload bridge:
```javascript
// Renderer calls
const videos = await window.electronAPI.getVideos(filters);

// Main process handles via ipcMain.handle()
ipcMain.handle('get-videos', async (event, filters) => { ... });
```

Event listeners return unsubscribe functions:
```javascript
const unsubscribe = window.electronAPI.onSyncProgress(callback);
// Later: unsubscribe();
```

### Database (SQLite via better-sqlite3)
- Location: `database/youtube-local.db`
- Uses WAL mode and prepared statements
- Core tables: `videos`, `watch_folders`, `categories`, `tags`, `playlists`
- Junction tables: `video_categories`, `video_tags`, `playlist_videos`
- Videos use `file_hash` (MD5 of filepath + fileSize) for identity tracking across disk locations

### React Frontend Structure
```
src/renderer/src/
├── App.jsx           # Router and main layout
├── components/       # Reusable UI components
├── pages/            # Route-based views (Home, Video, Settings, etc.)
├── context/          # React Context (SearchContext)
├── hooks/            # Custom hooks (usePagination)
└── utils/            # Helpers (videoSortFilter)
```

## Important Patterns

### File Tracking
- Videos are **never deleted** from DB - marked `is_available = 0` when file missing
- `disk_identifier` + `relative_filepath` enable reconnection detection
- Hash-based tracking allows videos to be found if moved within same disk

### Database Operations
- Always use prepared statements: `db.prepare(sql).run(...args)`
- Wrap multi-step operations in transactions: `db.transaction(() => { ... })()`
- Check existing indices before adding query filters

### IPC Events
When state changes in main process, broadcast to renderer:
```javascript
mainWindow.webContents.send('sync-complete', data);
```

### Security Context
- `contextIsolation: true`, `nodeIntegration: false`
- `webSecurity: false` required for local `file://` video playback
- `sandbox: false` required for better-sqlite3 native module

## Codebase Notes

- Code comments mix Spanish and English
- Pagination uses 24 items per page via `usePagination` hook
- Thumbnails stored in `thumbnails/` directory (gitignored)
- FFmpeg installed via `@ffmpeg-installer/ffmpeg`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/epjuan21) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
