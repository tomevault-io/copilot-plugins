## gis-whatsup-helper

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

WhatsApp GIS bot that listens to WhatsApp groups, captures location data and messages (with Hebrew report commands), and exports them to GIS formats. Two-service architecture: a Node.js/TypeScript WhatsApp listener forwards messages via HTTP to a Python/Flask GIS processor that stores data in SQLite and serves a Leaflet.js dashboard.

## Commands

### Setup
```bash
cd gis-processor && pip install -r requirements.txt
cd whatsapp-listener && npm install
```

### Running Services
```bash
# Python GIS service (runs on localhost:5000)
cd gis-processor && python app.py

# WhatsApp listener (TypeScript dev mode)
cd whatsapp-listener && npm run dev

# Or both via tmux:
./start.sh
```

### WhatsApp Listener NPM Scripts
```bash
npm run build    # Compile TypeScript
npm run start    # Run compiled JS
npm run dev      # Run with ts-node
npm run clean    # Remove dist/, auth_info/, cache
```

## Architecture

```
WhatsApp Groups → Node.js Listener (whatsapp-web.js) → HTTP POST → Flask API → SQLite → GIS Exports
                  port: N/A (WhatsApp client)                        port: 5000
```

**whatsapp-listener/** (TypeScript, CommonJS, ES2020 target, strict mode):
- `src/index.ts` — WhatsApp client initialization, QR auth, startup watchdog
- `src/messageHandler.ts` — Message parsing, report command detection (תד/סד), media encoding, forwarding to Python service
- `src/reportTracker.ts` — Per-user report state machine with timeout/grace period logic
- `src/config.ts` — YAML config loader (reads `../config.yaml`)
- `src/logger.ts` — Pino structured logging

**gis-processor/** (Python/Flask):
- `app.py` — REST API server, all endpoints, media serving, dashboard
- `models.py` — SQLAlchemy ORM models: Message, Location, Report (SQLite)
- `gis_export.py` — GeoPandas-based export to GeoJSON, Shapefile, KML, GeoPackage
- `static/index.html` — Dashboard UI (Leaflet.js maps, Hebrew RTL, WhatsApp-themed dark UI)

**config.yaml** — Shared config at repo root: target WhatsApp groups, Python service host/port, storage paths, log level.

## Key Domain Concepts

- **Report flow**: Users start a report with `תד` (תחילת דיווח) and end with `סד` (סוף דיווח). Messages between are aggregated. Reports require a location to be marked "complete" vs "invalid". One active report per sender, with 120s inactivity timeout + 60s grace period.
- **Deduplication**: Messages are deduplicated by `message_id` before storage.
- **Visibility**: Locations and reports have `is_visible` flags controlling dashboard display and GIS export inclusion.
- **Tags**: JSON arrays stored on locations/reports, used for filtering.

## REST API

Data receipt: `POST /message`, `POST /report`
CRUD: `GET /messages`, `GET /locations`, `GET /reports`, `PATCH/DELETE /api/reports/<id>`, `PATCH/DELETE /api/locations/<id>`
Exports: `GET /export/{geojson,shapefile,kml,gpkg}`, `GET /export/reports/geojson`
Live data: `GET /api/geojson`, `GET /api/tags`
Dashboard: `GET /dashboard`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davidelka)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/davidelka)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
