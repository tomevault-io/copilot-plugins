## e-learning-front

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # Start dev server (http://localhost:3000) with Turbopack
npm run build    # Production build
npm run start    # Run production build
npm run lint     # ESLint (flat config, eslint.config.mjs)
```

No test framework is configured.

## Environment

Copy `.env.template` to `.env.local` and set:
```
NEXT_PUBLIC_API_URL=http://localhost:8000
```

The API client (`src/services/api.ts`) falls back to `http://localhost:8000` if unset.

## Architecture

**Next.js App Router** with three routes:
- `/` — Course catalog dashboard (`AuthGuard`-wrapped)
- `/auth` — UID entry/generation page
- `/player/[videoId]` — Video player with notes and summary

All pages are Client Components (`'use client'`). There is no server-side rendering in practice.

### State Management (Zustand)

Four stores in `src/stores/`:
- `auth.store.ts` — UID auth state; reads/writes `localStorage` key `user_uid`
- `catalog.store.ts` — Formations list; fetches from API with `.formations`/`.data`/`.items` fallback
- `theme.store.ts` — `'light' | 'dark' | 'system'` preference; persisted to `localStorage`; listens to `window.matchMedia` for system changes
- `player.store.ts` — Playback position; `updateProgress()` is debounced 500 ms before calling API

### API Layer (`src/services/api.ts`)

Axios instance that injects the `X-User-UID` header from `localStorage` on every request. A response interceptor clears the UID on 401/403. All API methods live in the exported `apiService` object.

Key endpoints:
- `POST /auth/generate` — Create UID
- `GET/POST /progress/{videoId}` — Load/save video position
- `GET /notes/{videoId}`, `POST`, `PUT /notes/{noteId}`, `DELETE /notes/{noteId}`
- `GET /videos/{videoId}/summary`, `PUT /videos/{videoId}/summary`
- `GET /videos/{videoId}/stream` — Video stream

### Styling

MUI (`@mui/material` v7) is the only styling system. Tailwind is **not** used. The MUI theme is configured in `src/app/theme-provider.tsx` and toggled via `theme.store.ts`. The markdown editor (`@uiw/react-md-editor`) requires `data-color-mode` attribute and extensive CSS overrides for theme consistency.

### Video Player

`src/components/VideoPlayer.tsx` imperatively initializes a `video.js` instance. The player is hardcoded to `video/mp4`. See `docs/SUPPORT_MP3_MP4.md` for notes on MP3 support.

### Authentication Model

No JWT/sessions. The user is identified by a 64-char lowercase hex UID stored in `localStorage`. `AuthGuard` checks this on mount and redirects to `/auth` if absent.

### Path Alias

`@/*` maps to `./src/*` (configured in `tsconfig.json`).

---
> Source: [EdwinAlkins/e-learning-front](https://github.com/EdwinAlkins/e-learning-front) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
