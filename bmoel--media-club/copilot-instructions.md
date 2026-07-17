## media-club

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A monorepo for a website that tracks anime/manga/light novel statistics from AniList for a media club group. Live at https://mediaclub.cool.

```
apps/
  media-club-api/   # Rust backend (AWS Lambda + DynamoDB)
  web-ui/           # React frontend (Vite + TypeScript)
```

## Commands

### Frontend (`apps/web-ui`)

```bash
npm run dev        # Start dev server (http://localhost:5173)
npm run watch      # Alias for dev (vite)
npm run build      # Type-check and build with Vite/Rolldown
npm run lint       # Run ESLint
npm run lint-fix   # Auto-fix lint issues
npm run preview    # Preview production build
npm run test       # Run tests once (vitest run)
npm run test:watch # Run tests in watch mode (vitest)
```

### Backend (`apps/media-club-api`)

```bash
make db                  # Start DynamoDB Local in Docker
make seed TABLE=<name>   # Seed a DynamoDB table from JSON
make dev                 # Start local Lambda server (port 9000)
make query URL=<path>    # cURL the running API

cargo lambda watch       # Hot-reload Lambda server
cargo lambda build       # Build for ARM64
cargo lambda deploy      # Deploy to AWS Lambda
cargo test               # Run unit tests
```

Local API base URL: `http://localhost:9000/lambda-url/media-club-api/`

## Architecture

### Frontend Stack

- **React 19** + **TypeScript** + **Vite/Rolldown**
- **Redux Toolkit + RTK Query** for state and data fetching (two API clients: `anilistApi` for GraphQL, `mediaClubApi` for REST)
- **Material-UI (MUI) v7** + Emotion for UI components and styling
- **React Router v7** for routing
- **i18next + react-i18next** for internationalization

Routes: `/` (gallery), `/media/:id` (detail), `/stats`, `/registration`, `/auth/callback`

Data flow: Page components → custom hooks (`hooks/`) → RTK Query → API. Key hook: `useAnilistHomeMedia` merges data from both APIs.

Frontend env vars (`.env.development`):
```
VITE_MEDIA_CLUB_API_BASE_URL
VITE_ANILIST_APP_CLIENT_ID
VITE_ANILIST_APP_REDIRECT_URI
```

### Backend Stack

- **Rust** (2021 edition) + **Axum 0.8** + **Tokio**
- Deployed as an AWS Lambda "lambdalith" (single function handles all routes)
- **DynamoDB** via `aws-sdk-dynamodb` + `serde_dynamo`

Layered architecture: Handlers → Services → Repositories → DynamoDB/AniList GraphQL API

API routes:
```
GET  /                                    - Welcome/health check
GET  /media                               - Fetch all media items
GET  /users                               - Fetch all registered users
GET  /users/{id}/favorites/{media_id}     - Get a user's favorite for a specific media item
POST /auth/sync                           - Register AniList user (OAuth code exchange)
POST /auth/remove                         - Unregister user
```

All responses use `ApiResponse<T> { success, data, error }`. Custom `MyError` enum maps to HTTP status codes (502 for AniList/network errors, 500 for DB/internal errors).

### Authentication Flow

1. Frontend redirects user to AniList OAuth
2. AniList redirects to `/auth/callback` with `code`
3. Frontend POSTs code to backend `/auth/sync`
4. Backend exchanges code for AniList access token, fetches user ID, stores in DynamoDB

### Infrastructure

- **Frontend**: AWS Amplify (CI/CD automated)
- **Backend**: AWS Lambda (ARM64) + DynamoDB in `us-east-2`
- **Local DB**: DynamoDB Local via Docker (`make db`)

---
> Source: [Bmoel/media-club](https://github.com/Bmoel/media-club) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
