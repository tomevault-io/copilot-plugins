## hibiki

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Hibiki (響)** is a desktop audio companion for Discord, built as an Electron app. It plays music, ambience, and sound effects in Discord voice channels from a desktop UI. Originally designed for Dungeons & Dragons (background music, ambient soundscapes, one-shot effects) but works for any server.

**Current state:** The project is mid-migration from a monorepo (with separate `apps/bot` and `apps/web`) to a unified Electron app. The old structure has been deleted (see git status). The new `app/` package is the single application.

## Common Commands

**IMPORTANT:** Always run `nvm use` before executing any pnpm commands to activate the correct Node.js version (24.14.0).

All commands run from the **repo root**:

```bash
# First, activate the correct Node version
nvm use

# Development
pnpm dev                # Build and launch Electron app
pnpm build              # Build backend (tsc) + frontend (Vite) without launching

# Quality checks (ALWAYS run after code changes)
pnpm lint               # ESLint on backend and frontend
pnpm lint:fix           # Auto-fix linting issues
pnpm test               # Run Jest (backend) + Vitest (frontend) tests

# E2E tests
pnpm run build          # Build first
pnpm test:e2e           # Playwright tests against Electron app

# Distribution (Electron Forge handles everything automatically)
pnpm dist               # Package for current platform (alias for pnpm make)
pnpm dist:mac           # macOS (ZIP for x64 and arm64)
pnpm dist:win           # Windows (Squirrel installer + ZIP)
pnpm dist:linux         # Linux (DEB + ZIP)
pnpm make               # Primary build command (same as pnpm dist)
pnpm package            # Create unpacked app only (for testing)
```

### Testing Individual Components

```bash
# Backend tests (Jest)
cd app && pnpm run test:backend           # All backend tests
cd app && pnpm run test:watch             # Watch mode

# Frontend tests (Vitest)
cd app && pnpm run test:frontend          # All frontend tests
cd app/frontend && vitest run <file>      # Single test file

# Coverage
pnpm run test:coverage                    # Both backend + frontend
cd app && pnpm run test:coverage:backend  # Backend only
cd app && pnpm run test:coverage:frontend # Frontend only
```

## Architecture

### Electron Structure

```text
app/
├── electron/          # Electron main process
│   ├── main.js        # Entry point: boots backend, creates window, IPC handlers
│   ├── preload.js     # Bridge: exposes safe IPC to renderer
│   ├── splash.html    # Splash screen shown during app initialization
│   └── splash-logo.png # Logo for splash screen
├── src/               # Backend (runs in Electron main process)
│   ├── bootstrap-embedded.ts    # Wires services, exposes IPC API
│   ├── discord/       # Discord.js client (login, voice, guild directory)
│   ├── player/        # Playback engine (voice connections, audio streams, volume)
│   ├── sound/         # Sound library (list, upload, delete files)
│   ├── scenes/        # Scene store + import/export (soundboards)
│   ├── audio/         # Audio mixing (guild-specific managers)
│   ├── config/        # Configuration (env + validation)
│   └── persistence.ts # JSON key-value store (app-config.json)
├── frontend/          # Vue 3 renderer (Electron renderer process)
│   └── src/
│       ├── api/       # IPC wrappers (typed frontend API calling backend)
│       ├── audio/     # Web Audio capture (AudioWorklet for browser streaming)
│       ├── stores/    # Pinia stores (player state, guild directory)
│       └── views/     # Pages (Welcome, Scenes, Browser, Media, Settings)
├── web-dist/          # Built frontend (Vite output)
└── dist/              # Compiled backend (tsc output)
```

### Communication: IPC Only (No HTTP)

- **Frontend → Backend:** All communication uses Electron IPC via the `preload.js` bridge.
- **IPC API:** Defined in `bootstrap-embedded.ts` as the `EmbeddedApi` interface.
- **Domains:** `player`, `config`, `sounds`, `scenes` (matching the service structure).
- Frontend wrappers in `frontend/src/api/` provide typed functions (e.g., `joinChannel()`, `listScenes()`).
- Audio streaming (Browser feature) uses chunked IPC: `audio:startStream`, `audio:chunk`, `audio:stopStream`.

### Multi-Guild Architecture

- Bot can be in multiple Discord servers simultaneously.
- **One voice channel per guild** — each guild has independent:
  - Voice connection
  - Playback state (playing/stopped, current sound)
  - Volume levels (music, effects, ambience)
- `GuildAudioManager` (`src/audio/guild-audio.manager.ts`) manages per-guild state.
- Player (`src/player/player.ts`) coordinates all guild managers.

### Scenes System

**Scene** = soundboard with three categories:
- **Music:** Tracks with volume, loop option
- **Ambience:** Loops with random interval repeats, volume, enabled/disabled toggle
- **Effects:** One-shot sounds

Stored in `scenes.json`. Import/export bundles scenes with their sound files as `.hibiki.zip` archives.

### Storage

Data lives in platform user data directory (e.g., `~/Library/Application Support/hibiki` on macOS):
- `app-config.json` — Discord token (when set in UI), bookmarks, storage path override
- `scenes.json` — Scene definitions
- `music/`, `effects/`, `ambience/` — Sound files (copied on upload, keyed by UUID)

Override paths with env vars: `HIBIKI_STORAGE_PATH`, `HIBIKI_MUSIC_DIR`, `HIBIKI_EFFECTS_DIR`, `HIBIKI_DATA_PATH` (see `.env.example`).

## Key Development Patterns

### Splash Screen

The app shows a splash screen during initialization for better UX:
- **Loads immediately** when app starts (before backend initialization)
- **Frameless transparent window** with app logo and loading spinner
- **Closes automatically** when main window is ready to show
- Files: `electron/splash.html` and `electron/splash-logo.png`
- Handles errors: closes splash if initialization fails

### IPC API Addition

When adding a new backend feature:
1. Add the service method in the appropriate backend file (e.g., `src/player/player.ts`).
2. Expose it in `src/bootstrap-embedded.ts` via the `EmbeddedApi` interface.
3. Register the IPC handler in `electron/main.js` (or use the generic `api` handler).
4. Add a typed wrapper in the frontend (e.g., `frontend/src/api/player.ts`).

### Audio Streaming (Browser Feature)

The Browser tab captures audio from a `WebContentsView` (Electron-managed browser) using Web Audio API + AudioWorklet. Audio chunks are sent via IPC to the backend, which streams them to Discord voice.

- **Backend:** `player.startStream(guildId, stream, metadata)` accepts a Node.js `ReadableStream`.
- **Frontend → Main:** `audio:startStream`, chunked `audio:chunk`, `audio:stopStream`.
- **Main process:** Creates `PassThrough` streams, writes chunks, pipes to backend.

### Scene Playback

Scenes are **not** played directly in a single action. The frontend (SceneView) iterates over scene items and calls the player API individually:
- Music: `playMusic(guildId, soundId, options)`
- Ambience: `playAmbience(guildId, soundId, options)`
- Effects: `playEffect(guildId, soundId)`

The scene is a **template**, not a runtime object. Playback state lives in `GuildAudioManager`.

## Important Rules

### After Every Code Change

**ALWAYS** run `nvm use` first, then `pnpm lint` and `pnpm test` after editing backend or frontend code. Fix any issues before considering the change complete. This is a **blocking requirement** (from `.cursor/rules/verify-after-changes.mdc`).

### Discord Bot Token

- Required for the bot to connect.
- Set via `DISCORD_TOKEN` env var OR in the app's **Settings** page (stored in `app-config.json`).
- Backend reads from config first, falls back to env.

### Node Version

**Node.js 24 required** (see `.nvmrc`). **MUST run `nvm use` before any pnpm commands** to activate the correct version.
- `@discordjs/opus` uses N-API prebuilds (ABI-stable across Node versions).
- Electron 41+ requires Node 24+.
- Always use `source ~/.nvm/nvm.sh && nvm use && <command>` when running commands in new shells.

## Testing

### Backend (Jest)

- Config: `app/jest.config.js`
- Test files: `app/src/**/*.spec.ts`
- Run: `cd app && pnpm run test:backend`
- Watch: `cd app && pnpm run test:watch`

### Frontend (Vitest)

- Config: `app/frontend/vitest.config.ts`
- Test files: `app/frontend/src/**/*.spec.ts`
- Run: `cd app && pnpm run test:frontend`

### E2E (Playwright)

- Located in `e2e/` workspace package.
- Launches the Electron app and drives it via Playwright.
- Requires: `.env` with `DISCORD_TOKEN` and `e2e/.env.e2e` with `E2E_GUILD_ID`, `E2E_VOICE_CHANNEL_ID` (optional).
- **Important:** Build the app first (`pnpm build`) before running E2E.
- See `e2e/README.md` for Electron 33 + Playwright compatibility notes.

## Migration Context

The project is transitioning from a Docker-based monorepo to an Electron app. See `docs/electron-migration-plan.md` for the full migration plan (phases 1-5). **Current status:** Phases 1-3 mostly complete (permissions and slash commands removed, repo flattened to `app/` + `e2e/`, Electron shell added).

**What was removed:**
- NestJS backend (replaced with plain TypeScript services in `src/`)
- HTTP API (replaced with IPC)
- Docker (Dockerfile, docker-compose)
- Permission system and slash commands (app-only control now)
- Monorepo `apps/bot` and `apps/web` (merged into `app/`)

**Key differences from old structure:**
- No HTTP server; all communication is IPC.
- Bot runs in Electron main process, not a separate Node service.
- Token configurable in UI, not just env.

## Tech Stack

- **Electron 41** — Main + renderer processes
- **Discord.js 14** — Bot, voice connections
- **Vue 3** — Frontend (Composition API, `<script setup>`)
- **Pinia** — State management
- **Vue Router** — Frontend routing
- **Vite** — Frontend build
- **TypeScript** — Backend and frontend
- **Jest** — Backend testing
- **Vitest** — Frontend testing
- **Playwright** — E2E testing
- **Web Audio API** — Browser audio capture (AudioWorklet)
- **@discordjs/voice** — Discord voice streaming
- **node-audio-mixer** — Audio mixing for multiple streams

## Common Pitfalls

- **Don't add HTTP endpoints** — This is an Electron app; use IPC only.
- **Per-guild state** — Always think guild-first. Most player methods take `guildId`.
- **Scene is not a runtime object** — Scenes are templates; playback state is in `GuildAudioManager`.
- **Build before E2E** — E2E tests launch the built app, not dev mode.
- **Audio streaming is chunked IPC** — High-volume data; use the existing `audio:chunk` pattern, not single IPC calls.

## References

- **README.md** — User-facing setup, download, quick start
- **app/README.md** — Architecture details, local dev instructions
- **CONTRIBUTING.md** — Setup, lint/test requirements, PR workflow
- **docs/electron-migration-plan.md** — Full migration roadmap
- **e2e/README.md** — E2E test setup, Electron + Playwright compatibility

## Design Context

### Users
D&D / TTRPG Game Masters running tabletop sessions. They're switching scenes, triggering sound effects, and managing ambience in real-time while simultaneously running a game. The UI must be quick to operate under pressure — one hand on the mouse, eyes mostly on the table. Clarity and speed matter more than exploration.

### Brand Personality
**Immersive, Crafted, Warm** — Like a well-made instrument or a tavern fireplace. The app should feel purposeful and inviting, not sterile or overly technical. It's a companion to storytelling, not a recording studio.

### Aesthetic Direction
- **Visual tone:** Dark, warm, atmospheric — a game companion aesthetic inspired by D&D Beyond. Rich but not gaudy.
- **Theme:** Dark mode only. Deep backgrounds (`#0c0c0e`), elevated cards (`#161618`/`#1a1a1d`), warm accent opportunities.
- **Accent color:** Cyan (`#06b6d4`) is the primary accent. Consider warm secondary accents (amber/gold) for adventure-themed elements — the welcome screen already uses `#fbbf24` gold.
- **Typography:** Plus Jakarta Sans (400–800). Clean, modern, slightly rounded — fits the "crafted" personality.
- **Anti-references:** Avoid sterile enterprise dashboards, neon-heavy gamer aesthetics, or clinical white-space minimalism. Should not look like a generic admin panel.

### Design Principles
1. **Session-ready** — Every interaction should be fast and confident. GMs use this mid-game; minimize clicks, maximize clarity. No ambiguity in controls.
2. **Atmospheric, not decorative** — Design should evoke the feeling of a well-set game table. Use subtle depth, warm shadows, and muted textures rather than ornamental fantasy elements.
3. **Sound-first hierarchy** — Audio controls (play, stop, volume) are the primary actions. They should be the most visually prominent and easiest to reach.
4. **Quiet confidence** — The UI stays out of the way until needed. Subtle transitions, restrained color use, no unnecessary animation. Let the content (scenes, sounds) be the focus.
5. **Accessible by default** — Good contrast ratios, keyboard navigation, semantic HTML. No specific WCAG target, but standard best practices throughout.

---
> Source: [PhyberApex/hibiki](https://github.com/PhyberApex/hibiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
