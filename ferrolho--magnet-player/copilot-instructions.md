## magnet-player

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Magnet Player is a browser-based torrent streaming site built with **WebTorrent v2** and **WebRTC**. Users paste a magnet URI, torrent file URL, or info hash, and the largest file in the torrent is streamed directly in the browser. Only WebRTC-seeded torrents are supported.

Hosted on GitHub Pages at `https://ferrolho.github.io/magnet-player/`.

## Build & Development

This is a **Jekyll** site with **esbuild** for JS bundling.

```bash
# Install Ruby dependencies
bundle install

# Install JS dependencies
npm install

# Bundle JS (main.js -> bundle.js)
npm run build

# Serve locally (note: service worker requires HTTPS, so streaming won't work on localhost)
bundle exec jekyll serve
```

The site is served from the `gh-pages` branch (which is also the main/only branch).

## Architecture

- **Jekyll** handles site generation (`_config.yml`, `_layouts/`, `_includes/`, `_sass/`)
- **`js/webtorrent.js`** — Core torrent logic: registers a service worker, creates a WebTorrent client with `createServer()`, handles form/URL-hash input, downloads torrents, streams the largest file via `file.streamTo()` into a `<video>` element, and updates download statistics
- **`js/main.js`** — UI glue: clipboard (native Clipboard API). Requires `webtorrent.js` and is the esbuild entry point
- **`js/bundle.js`** — esbuild output (committed to repo, must be regenerated after JS changes)
- **`js/webtorrent.min.js`** — Vendored WebTorrent v2.8.5 pre-built browser bundle (ESM converted to IIFE via esbuild, exposes `WebTorrent` global)
- **`sw.min.js`** — WebTorrent service worker for browser-based media streaming (required by `file.streamTo()`)
- No jQuery or Bootstrap — all JS is vanilla, tooltip is CSS-only
- Styling uses the Jekyll **minima** theme with custom SCSS in `_sass/player.scss`
- SEO handled by `jekyll-seo-tag` and `jekyll-sitemap` plugins

## Key Details

- After editing `js/main.js` or `js/webtorrent.js`, run `npm run build` to regenerate the bundle
- Torrent sharing works via URL hash fragment (e.g., `#<infoHash>`), parsed in `onHashChange()` in `webtorrent.js`
- The announce tracker list in `webtorrent.js` is filtered to WebSocket-only (`wss://`) trackers for browser compatibility
- The service worker must be fully activated before `createServer()` is called — the code waits for the `activated` state explicitly
- All DOM text insertion uses `.textContent` (not `.innerHTML`) to prevent XSS via torrent file names

---
> Source: [ferrolho/magnet-player](https://github.com/ferrolho/magnet-player) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
