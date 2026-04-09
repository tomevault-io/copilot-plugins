## escritorio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Escritório Virtual Multiplayer** — a real-time 2D multiplayer virtual office built with Next.js, Phaser 3 (game engine), and Socket.io. Users authenticate via GitHub OAuth and interact in a top-down office environment with chat and GitHub Projects integration.

## Development Setup

Two servers must run simultaneously:

```bash
# Terminal 1 — WebSocket server (port 3001)
cd server && npm run dev

# Terminal 2 — Next.js frontend (port 3000)
npm run dev
```

Access at `http://localhost:3000`. Or use `./start.sh` (Linux/Mac) / `start.bat` (Windows).

## Commands

```bash
npm run dev        # Frontend dev server
npm run build      # Production build
npm start          # Production server
npm run lint       # ESLint

cd server && npm run dev    # Backend dev server (nodemon)
cd server && npm start      # Backend production
```

## Environment Variables

Copy `.env.local.example` to `.env.local`:

```
NEXT_PUBLIC_SOCKET_URL=http://localhost:3001
GITHUB_CLIENT_ID=...
GITHUB_CLIENT_SECRET=...
GITHUB_REDIRECT_URI=...
GITHUB_TOKEN=...        # Personal access token (read:project scope for GitHub Projects)
GITHUB_OWNER=...
GITHUB_REPO=...
GITHUB_ORG=...
GITHUB_PROJECT_NUMBER=...
```

## Architecture

Three distinct layers:

**Frontend (Next.js, port 3000)**
- `app/page.tsx` — Login page with GitHub OAuth
- `app/layout.tsx` — Root layout; **Phaser is loaded from CDN here**, not bundled (saves ~400MB)
- `components/Game.tsx` — Main orchestrator: initializes Phaser, manages panel visibility, connects Socket.io
- `game/scenes/MainScene.ts` — Core Phaser scene: player sprites, movement, input, interpolation for remote players
- `game/scenes/EditorScene.ts` — Map editor scene (toggle with `E` key)
- `game/MapLoader.ts` — Loads map JSON from `/api/map`

**WebSocket Server (Express + Socket.io, port 3001)**
- `server/server.js` — Single file: handles player join/movement/disconnect sync and chat relay

**API Routes (Next.js handlers)**
- `app/api/auth/github/` — Initiates GitHub OAuth redirect
- `app/api/auth/callback/` — Handles OAuth callback, exchanges code for token
- `app/api/github/` — Proxies GitHub API calls (issues, PRs, projects)
- `app/api/map/` — Serves map JSON files from `public/maps/`

## Socket.io Event Contract

Client → Server: `player-joined`, `player-movement`, `chat-message`, `player-status-changed`

Server → Client: `current-players`, `new-player`, `player-moved`, `player-disconnected`, `chat-message`

## Key Design Decisions

- **Phaser loaded from CDN** (`app/layout.tsx` script tag) — webpack config in `next.config.js` marks `phaser` as external to avoid bundling it
- **Client-side prediction** in `MainScene.ts` for local player movement; remote players use interpolation
- **Map format** is JSON in `public/maps/` — see `game/types/MapDefinition.ts` for the schema
- **shadcn/ui** components live in `components/ui/` — add new ones with `npx shadcn-ui@latest add <component>`
- **HeadlessMode** (`components/HeadlessMode.tsx`) is a text-based dev mode for testing multiplayer without Phaser, activated from the login page

## Admin Rules

**Admin username: `phrigapov`** (GitHub handle, never the display name).

- The admin is **never blocked**, kicked, or locked out under any circumstance
- `officeLocked` state never applies to the admin — neither on the server nor on the client
- Admin status is always checked against `user.username` (GitHub handle) on the client and against `data.githubUsername` on the server — never against the display name (`user.name`), which can differ
- The `githubUsername` field is sent in every `player-joined` event alongside `username` (display name) specifically for this check
- These rules must be preserved in any refactor

## Architecture Rule — Socket Connection

**The Socket.io connection is created ONCE in `app/page.tsx` when the user logs in, and is passed down as a prop to all modes (`Game`, `HeadlessMode`).**

- Switching between Game mode and Headless mode is **purely a UI change** — it must never create, destroy or reconnect the socket
- `MainScene.ts` reads the socket from Phaser's registry (`this.registry.get('socket')`); it never calls `io()` directly
- `HeadlessMode.tsx` receives the socket as a prop; it never calls `io()` directly
- Only `page.tsx` calls `io()` and owns the socket lifecycle (created on login, destroyed on logout)
- This ensures the server never sees duplicate usernames from the same user across mode switches

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/phrigapov)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/phrigapov)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
