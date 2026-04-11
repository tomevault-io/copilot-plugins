## web-stockbreak

> - Backend API: Node.js + Express

# Stockbreak Agent Guide

## Stack (VPS-friendly)
- Backend API: Node.js + Express
- Database: Postgres
- Frontend UI: Vite + React
- Charting: lightweight-charts
- Background jobs: internal cron (node-cron) with optional n8n later

## Running on a VPS
### Backend
- Dev: `cd backend && npm install && npm run dev`
- Prod: `cd backend && npm install --omit=dev && npm run start`
- Env: copy `backend/.env.example` to `backend/.env`

### Frontend
- Dev: `cd frontend && npm install && npm run dev -- --host`
- Prod build: `cd frontend && npm install && npm run build`
- Preview: `cd frontend && npm run preview -- --host`

## Quickstart (Local)
- Backend dev: `cd backend && npm install && npm run dev` (API on `127.0.0.1:3000`)
- Frontend dev: `cd frontend && npm install && npm run dev -- --host`
- API base: set `VITE_API_BASE_URL` for custom host (defaults to same origin)
- Tests: `cd frontend && npm run test`

## Folder Conventions
- `backend/` API server, ingestion jobs, DB access.
- `frontend/` React UI, charts, and future labeling tools.
- `docs/` architecture, state, and milestone documentation.
- `n8n/` reserved for optional workflow automation.

## Rules
- Every milestone must update `docs/STATE.md` with current state changes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RasyidPuteraA)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/RasyidPuteraA)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
