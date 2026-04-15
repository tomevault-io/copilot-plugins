## musicgy

> Musicgy is a collaborative music listening platform built with Next.js 16. It allows multiple users to join a room, search for songs from Netease Cloud Music, add them to a shared queue, and listen in sync. The application uses Redis for real-time state management (queue, current song, user presence, votes) and Next.js App Router for both frontend and backend logic.

# Project Context: Musicgy

## Overview
Musicgy is a collaborative music listening platform built with Next.js 16. It allows multiple users to join a room, search for songs from Netease Cloud Music, add them to a shared queue, and listen in sync. The application uses Redis for real-time state management (queue, current song, user presence, votes) and Next.js App Router for both frontend and backend logic.

## Tech Stack
- **Framework:** Next.js 16 (App Router)
- **Language:** TypeScript
- **Frontend:** React 19, Tailwind CSS
- **Backend:** Next.js API Routes
- **Database/State:** Redis (via `ioredis`)
- **External API:** `@neteasecloudmusicapienhanced/api` (Netease Cloud Music)
- **Containerization:** Docker (for Redis)

## Architecture

### State Management
The application relies heavily on **Redis** as the single source of truth for the room's state. There is no traditional database (SQL/NoSQL) for persistent data; the room state is ephemeral but persisted in Redis.
- **Room Service** (`services/room.ts`): Manages all Redis interactions.
    - **Queue:** Stored as a Redis List (`room:queue`).
    - **Current Song:** Stored as a JSON string (`room:current`).
    - **Users:** Managed via a Set (`room:users`) and Heartbeats (`user:{id}:heartbeat`) to detect active users.
    - **Votes:** Skip votes are stored in a Set (`room:votes`).
- **Synchronization:** The frontend likely polls the `/api/state` endpoint to sync with the server state (based on the presence of heartbeat logic and lack of explicit WebSocket server code in the initial scan).

### Music Service
- **Netease Service** (`services/netease.ts`): Wraps the Netease Cloud Music API.
    - Handles searching, song details, and fetching audio URLs.
    - Implements an audio proxy mechanism to bypass potential referral/CORS restrictions when playing music in the browser.
    - Supports QR code login to unlocking higher quality audio or user-specific features.

### Project Structure
- `app/`: Next.js App Router structure.
    - `app/api/`: Backend API endpoints (e.g., `queue`, `state`, `stream`, `auth`).
    - `app/page.tsx`: Main application view.
- `components/`: React UI components (`Player`, `Queue`, `SearchModal`, etc.).
- `services/`: Business logic and external service integrations.
    - `room.ts`: Redis logic for room state.
    - `netease.ts`: Netease API wrapper.
- `lib/`: Shared utilities (`api.ts` for frontend API calls).

## Development Workflow

### Prerequisites
1.  **Node.js** (v18+ recommended)
2.  **Redis**: Must be running locally or available via connection string.
3.  **Bun**: Always use bun instead of npm.

### Setup & Run
1.  **Start Redis**:
    ```bash
    docker-compose up -d redis
    ```
2.  **Install Dependencies**:
    ```bash
    npm install
    # or
    bun install
    ```
3.  **Start Development Server**:
    ```bash
    npm run dev
    # or
    bun run dev
    ```
    The app will be available at `http://localhost:3000`.

### Environment Variables
- `REDIS_URL`: Connection string for Redis (default: `redis://localhost:6379`).
- `.env.local`: Local environment variables.

## Key Files
- `services/room.ts`: Core logic for the collaborative room. Crucial for understanding how the state changes.
- `services/netease.ts`: Interface with the music provider.
- `app/api/stream/route.ts`: Proxies audio streams to the client.

## Conventions
- **API Routes**: All backend logic resides in `app/api/*`.
- **Service Layer**: Complex logic is delegated to `services/` rather than being written directly in route handlers or components.
- **Redis Keys**: All Redis keys are defined as constants within `services/room.ts` to prevent collisions and typos.
- **Styling**: Tailwind CSS is used for all styling.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/li-sky)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/li-sky)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
