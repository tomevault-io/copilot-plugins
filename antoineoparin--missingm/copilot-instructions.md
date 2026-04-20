## missingm

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MissingM is an AI patient simulator with a React frontend and Express backend. The backend uses the Anthropic SDK to generate realistic patient responses.

## Common Commands

### Backend

```bash
cd backend
npm run dev      # Start dev server with hot reload (tsx watch)
npm run build    # Compile TypeScript to dist/
npm start        # Run compiled server from dist/
```

The backend runs on port 3000 by default (configurable via `PORT` env var).

### Frontend

```bash
cd frontend
npm run dev      # Start Vite dev server
npm run build    # Type check and build for production
npm run preview  # Preview production build
npm run lint     # Run ESLint
```

The Vite dev server proxies `/api` requests to `http://localhost:3000`.

## Architecture

### Backend (TypeScript/Express)

- **Entry point**: `backend/src/server.ts`
- **Controllers**: `backend/src/controllers/` - Handle HTTP requests/responses
- **Routes**: `backend/src/routes/` - Route definitions
- **Services**: `backend/src/services/` - Business logic (Anthropic API integration)
- **Compiled output**: `backend/dist/` (gitignored)

The backend expects `CLUADE_API_KEY` in the environment (from `.env` at project root) to authenticate with Anthropic.

### Frontend (React + Vite + TypeScript)

- **Entry point**: `frontend/src/main.tsx`
- **Router**: `frontend/src/App.tsx` (simple single-page setup)
- **Pages**: `frontend/src/pages/`
- **Components**: `frontend/src/components/`
- **Services**: `frontend/src/services/api.ts` - Axios instance pointing to backend API
- **Styling**: Tailwind CSS v4 (configured via `@import "tailwindcss"` in `index.css`)
- **Assets**: `frontend/src/assets/` and `frontend/assets/emotions/` (patient emotion images)

### API Flow

1. Frontend sends POST to `/api/chat` → proxied to backend
2. Backend receives at `/api/chat`, calls Anthropic API
3. Patient responses generated with `claude-sonnet-4-5` model using roleplay system prompt

## TypeScript Configuration

- **Backend**: Uses `nodenext` module resolution, outputs to `dist/`, generates `.d.ts` and source maps
- **Frontend**: Project references setup with separate `tsconfig.app.json` and `tsconfig.node.json`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AntoineOparin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
