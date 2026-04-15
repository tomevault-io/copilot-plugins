## jellydownloader

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Prehrajto Downloader is a web application for searching and downloading movies/series from prehrajto.cz with automatic Jellyfin-compatible naming. It scrapes video files, fetches IMDB/TMDB metadata, and organizes downloads according to Jellyfin naming conventions.

## Commands

```bash
# Install all dependencies (root + frontend)
npm run install:all

# Start production server (port 6565)
npm start

# Development with auto-reload
npm run dev

# Frontend development (Vite dev server)
npm run dev:frontend

# Run both backend and frontend in development
npm run dev:all

# Build frontend for production
npm run build:frontend
```

## Architecture

### Backend (server/)
- **index.js** - Express server entry point, mounts routes and serves React SPA from `public-react/`
- **config.js** - Centralized configuration, reads `.env` for API keys and paths
- **routes/** - Express route handlers:
  - `search.js` - Prehrajto search endpoint
  - `video.js` - Video URL extraction
  - `download.js` - Download orchestration with SSE progress
  - `imdb.js`, `tmdb.js` - Metadata API proxies
  - `settings.js` - User settings and directory browsing
  - `anime.js` - Anime-specific routes (HLS streams)
- **services/** - Business logic:
  - `prehrajto.js` - Scrapes prehrajto.cz, extracts video sources from `var sources` array or Playerjs config
  - `downloader.js` - Core download engine: handles movies, series batches, and HLS via ffmpeg
  - `omdb.js`, `tmdb.js` - External API wrappers
  - `animeApi.js` - Anime site scraping
- **utils/** - Shared utilities:
  - `jobs.js` - Download job management with EventEmitter for SSE progress events
  - `helpers.js` - Filename sanitization, duration/filesize parsing
  - `logger.js` - Structured logging

### Frontend (frontend/)
- React 18 with Vite, no state management library (uses Context)
- **App.jsx** - Main component with mode selection (movie/series/anime)
- **components/** - UI components:
  - `MovieMode.jsx`, `SeriesMode.jsx`, `AnimeMode.jsx` - Mode-specific search and download UIs
  - `DownloadHUD.jsx` - Floating progress indicator
  - `SettingsModal.jsx` - Configuration UI
- **context/** - React Context providers:
  - `DownloadContext.jsx` - Global download state and SSE connection management
  - `ToastContext.jsx` - Notification system
- **services/api.js** - API client functions

### Download Flow
1. User searches → `prehrajto.search()` scrapes results
2. User selects video → `prehrajto.getVideoDetails()` extracts video URLs
3. User initiates download → `downloader.downloadFile()` creates job
4. Backend streams file and emits progress via SSE
5. Frontend connects to `/api/download/progress/:jobId` for real-time updates
6. Files are named using Jellyfin convention: `Title (Year) [imdbid-ttXXXX].ext`

### HLS Downloads (Anime)
- Uses `ffmpeg-static` package
- `downloader.downloadHLS()` spawns ffmpeg to convert HLS → MP4
- Progress parsed from ffmpeg stderr output

## Configuration

Environment variables in `.env`:
- `PORT` - Server port (default: 6565)
- `OMDB_API_KEY` - OMDB API key for IMDB metadata
- `TMDB_API_KEY` - TMDB API key (optional)
- `MOVIES_DIR` - Custom movies download path
- `SERIES_DIR` - Custom series download path
- `ANIME_DIR` - Custom anime download path
- `JELLYFIN_DIR` - Fallback path for all media types

## File Naming Convention

Movies: `Title (Year) [imdbid-ttXXXXXXX].ext`
Series: `Title [imdbid-ttXXXXXXX]/Season XX/Title sXXeYY [imdbid-ttYYYYYYY].ext`

## Key Patterns

- All API routes return `{ success: boolean, data?: any, error?: string }`
- Download progress uses Server-Sent Events (not WebSocket)
- Jobs are stored in memory Map with automatic cleanup after 1-2 hours
- Scraping uses User-Agent spoofing to avoid blocks

Dont start server on your own, type command log "Should i rusn"

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PatrikMana) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
