## logspect

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

**Development Setup:**

- `pnpm install` - Install dependencies
- `overmind start` - Start both Vite dev server and Electron app (recommended)
- `pnpm dev` - Start Vite development server only
- `pnpm start` - Start Electron in development mode

**Building:**

- `pnpm build` - Build Vue.js frontend for production
- `pnpm dist` - Build and package Electron app
- `pnpm release` - Full release build with publishing

**No test framework is currently configured** - tests would need to be set up if testing is required.

## Architecture Overview

**Logspect** is an Electron-based Rails log viewer with a modular architecture:

### Core Architecture

- **Electron Main Process** (`src/`) - Handles file watching, log parsing, and data storage
- **Vue.js Frontend** (`src-vue/`) - Modern UI built with Vue 3, Vite, and Tailwind CSS
- **IPC Communication** - Secure communication between main and renderer processes via preload script

### Key Modules (src/modules/)

- `app.js` - Main Electron app and window management
- `logWatcher.js` - File watching and incremental log reading (watches Rails log/development.log)
- `logParser.js` - Parses Rails logs, extracts UUIDs, JIDs, HTTP requests, and system logs
- `logStorage.js` - In-memory storage with Map-based grouping by UUID/JID
- `projectManager.js` - Rails project validation and directory selection
- `ipcHandlers.js` - IPC communication handlers
- `devtools.js` - Development environment setup

### Log Processing Pipeline

1. **File Watching** - Real-time monitoring of Rails development.log
2. **Parsing** - Extracts three log types:
   - Web requests (with UUID) - grouped by request ID
   - Background jobs (with JID) - grouped by Sidekiq job ID
   - System logs - grouped by time-based UUID (5-second windows)
3. **Storage** - Groups related log entries by identifier
4. **Frontend** - Vue components display grouped logs with filtering and search

### Frontend Structure (src-vue/)

- `stores/logStore.js` - Pinia store managing project state, log data, and search
- `components/` - Vue components for log viewing, entry details, and toolbar
- `App.vue` - Main application component with welcome screen and log viewer

### Build Configuration

- **Vite** builds Vue frontend to `public/` directory
- **electron-builder** packages the complete app
- Development uses Vite dev server with hot reload
- Production loads built files from `public/`

### Key Data Structures

- Log entries grouped by UUID/JID with metadata (type, success status, timing)
- Real-time streaming from main process to renderer
- Efficient incremental file reading (only reads new content)

---
> Source: [BilalBudhani/Logspect](https://github.com/BilalBudhani/Logspect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
