## node-trans

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Real-time audio translation app using Soniox API. Captures audio from microphone, system audio, or both, translates via Soniox speech-to-text, and stores session history. Runs as a web app (browser) or desktop app (Electron). Supports macOS and Windows.

## Commands

```bash
# Development (runs Express server + Vite dev server concurrently)
npm run dev

# Run only the backend server (with file watching)
npm run dev:server

# Run only the Vite frontend dev server
npm run dev:client

# Production server (serves built frontend from dist/)
npm start

# Build frontend
npm run build

# Electron development
npm run electron:dev

# Electron packaging
npm run electron:build          # Current platform
npm run electron:build:mac
npm run electron:build:win
```

There are no test or lint commands configured.

## Architecture

**ES Modules throughout** — the project uses `"type": "module"` in package.json. All imports use ESM syntax.

### Backend (`src/`)
- **server.js** — Express 5 + Socket.IO server. Entry point for web mode. Uses lazy imports for heavy modules (history, soniox, audio capture, devices) to speed up startup. Manages active translation sessions per socket connection in an in-memory `activeSessions` Map.
- **routes/api.js** — REST API endpoints under `/api`. Also uses lazy imports. API key is masked in GET/PUT settings responses.
- **soniox/session.js** — Wraps Soniox real-time translation API. Creates streaming sessions with callbacks for partials, utterances, and errors.
- **audio/capture.js** — Spawns ffmpeg to capture audio. Uses `avfoundation` on macOS, `dshow` on Windows. Returns a stream with pause/resume/stop controls.
- **audio/devices.js** — Lists audio input/output devices by parsing ffmpeg output. Platform-specific parsing.
- **storage/history.js** — SQLite database (better-sqlite3) for sessions and utterances. DB stored at `~/.node-trans/history.db`.
- **storage/settings.js** — JSON file settings at `~/.node-trans/settings.json`. Loaded synchronously.
- **storage/export.js** — Exports a session to Markdown format.

### Frontend (`client/src/`)
- **React 19 + Vite + Tailwind CSS v4** — SPA with no router.
- **context/SocketContext.jsx** — Central state management using `useReducer`. Manages Socket.IO connection, listening state, utterances, speaker colors, and session selection. This is the main state hub for the app.
- **i18n/** — Bilingual UI (English/Vietnamese). `I18nContext` provides `t()` translation function, `locales.js` has all strings.
- **components/live/** — Real-time transcription UI (controls, transcript display, utterance rendering).
- **components/history/** — Session history browsing, detail view, speaker renaming, export.
- **components/settings/** — Settings form (API key, audio source, devices, languages).
- **utils/api.js** — Fetch wrapper for REST API calls.

### Electron (`electron/`)
- **main.js** — Electron entry point. Starts the Express server internally, then loads the app URL.
- **preload.js** — Preload script for IPC bridge.

### Key Data Flow
1. Client sends `start-listening` via Socket.IO with optional `sessionId` (for resume)
2. Server resolves audio devices, creates/resumes a DB session, starts ffmpeg capture + Soniox streaming session
3. Audio chunks flow: ffmpeg → capture stream → `soniox.sendAudio()` → Soniox API
4. Results flow back: Soniox callbacks → Socket.IO events (`utterance`, `partial-result`) → client reducer → React UI
5. Utterances are persisted to SQLite as they arrive

### Development URLs
- **Vite dev server**: `http://localhost:5173` (proxies `/api` and `/socket.io` to backend)
- **Production/backend**: `http://localhost:3000`

## Configuration

Soniox API key can be set via `.env` file (`SONIOX_API_KEY=...`) or through the in-app Settings UI (takes precedence). User data is stored in `~/.node-trans/`.

---
> Source: [thainph/node-trans](https://github.com/thainph/node-trans) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
