## m4sport

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

M4 Sport Stream is a containerized Node.js service that scrapes the M4 Sport Hungarian live stream HLS manifest URL using Playwright and serves it via HTTP. Clients (VLC, IPTV apps, browsers) connect to get a redirect to the live stream.

## Commands

```bash
# Install dependencies (requires Node 24+)
npm install

# Install Playwright Chromium browser
npm run install-playwright

# Run locally
npm start

# Run with Docker
docker compose up -d
```

There are no tests, linter, or build step configured.

## Architecture

The app has three phases that run sequentially on startup (`src/main.js`):

1. **Initial scrape** (`src/init.js`) — checks if `config/manifest.json` exists and is fresh (< 24h). If stale/missing, runs the scraper immediately.
2. **HTTP server** (`src/server.js`) — plain `node:http` server serving three endpoints: `/` (hls.js web player), `/stream` (302 redirect to manifest URL), `/playlist.m3u` (M3U playlist).
3. **Cron scheduler** (`src/cron.js`) — uses `node-cron` with `noOverlap: true` to run `src/task.js` as a child process on a configurable schedule.

### Scraper (`src/scraper.js`)

Uses Playwright (headless Chromium) to:
1. Navigate to `https://m4sport.hu/elo`
2. Find the player iframe (`player.php`)
3. Extract the HLS manifest URL from the frame content via regex
4. Write result to `config/manifest.json` as `{ manifestUrl, updatedAt }`

### Configuration (`src/config.js`)

All config is via environment variables with defaults:
- `PORT` (3000), `CRON_SCHEDULE` ("0 0 * * *"), `TZ` ("UTC"), `MANIFEST_FILE_PATH` ("./config/manifest.json")
- Scraping constants (PAGE_URL, PLAYER_FRAME_URL, MANIFEST_URL_PATTERN, MANIFEST_URL_MAX_AGE_MS) are hardcoded.

### Key Constraints

- Requires a Hungarian IP address to access the M4 Sport stream.
- `PLAYWRIGHT_BROWSERS_PATH=0` must be set when running Playwright commands locally (installs browser alongside the package rather than in a shared location).
- The cron task (`src/task.js`) runs as a child process and calls `process.exit()` — it's not imported into the main process at runtime.
- Docker image is based on `node:24.15.0-slim` and publishes to `ghcr.io/balintsoos/m4sport`.

---
> Source: [balintsoos/m4sport](https://github.com/balintsoos/m4sport) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
