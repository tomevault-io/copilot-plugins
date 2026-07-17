## personal-website

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal portfolio website with an interactive Langton's Ant simulation. React/TypeScript frontend + Python Flask backend.

## Commands

### Frontend (in `client/`)
```bash
npm start         # Dev server on localhost:3000
npm run build     # Production build
npm test          # Run tests
```

### Backend (in `server/`)
```bash
pip install -r requirements.txt
python server.py  # Dev server on localhost:5000
```

### Docker
```bash
docker-compose up  # Flask on :5000, PostgreSQL on :5432
```

## Architecture

**Monorepo** with two independent services:

- `client/` — React 19 + TypeScript SPA (Create React App / webpack). Two routes: `/` (portfolio home) and `/langton` (Langton's Ant simulator).
- `server/` — Flask 3 REST API. Primary endpoint: `GET /api/langton/ant_config/<n_colors>` which calls OpenAI (o4-mini) to generate Langton's Ant configs. Deployed via Gunicorn + Docker Compose alongside PostgreSQL. All py files use tabs not spaces.

**Frontend state**: `AppContext.tsx` provides a thin global context (`options`/`setOption`). Most state is local to components.

**Langton's Ant simulation** (`client/src/Langton/`):
- `Langston.ts` — Core simulation classes: `LangstonAnt` (classic binary ant), `CustomAnt` (multi-color with JSON-configurable rules)
- `LangstonGrid.tsx` — Canvas-based grid + fetches configs from backend via axios
- `Stepper.tsx` — Animation loop / stepping controller
- `Renderer.tsx` — Canvas drawing logic
- `Decoder/` — Parser for ant config language

Ant config schema:
```json
{
  "rules": { "0": { "turn": "right|left|straight|reverse", "flipTo": 1 }, "1": {...} },
  "startDirection": "N|E|S|W",
  "startColor": 0
}
```

**Styling**: SCSS; animated backgrounds (`WaveBackground`, `LeafBackground`) live in `client/src/static/backgrounds/`.

---
> Source: [KeenanSegenchuk/personal-website](https://github.com/KeenanSegenchuk/personal-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
