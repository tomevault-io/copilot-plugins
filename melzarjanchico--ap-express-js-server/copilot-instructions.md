## ap-express-js-server

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Development Commands
- **dev**: `npm run dev` – runs the server in watch mode using `ts-node-dev`.
- **build**: `npm run build` – compiles TypeScript to JavaScript in the `dist/` folder.
- **start**: `npm start` – runs the compiled server from `dist/index.js`.
- **test**: *(none defined)* – add a test script when tests are introduced.
- **run single test**: once a test script exists, use `npm test -- <test-file>`.

## High‑Level Architecture
- **Entry point (`src/index.ts`)**: sets up Express, CORS, JSON parsing, creates service instances, and mounts controllers.
- **Controllers (`src/components/controllers/*`)**: thin layers exposing HTTP routes; each controller receives a service instance via dependency injection and delegates to it.
- **Services (`src/components/services/*`)**: contain the business logic for external APIs (Spotify, Steam, Letterboxd). Each service may have sub‑modules (e.g., `spotify/auth` for token handling).
- **Models (`src/models` & service‑specific model files)**: TypeScript types and helper utilities used by services.
- **Environment**: configuration via `.env` (loaded in non‑production), including API keys and PORT.
- **Build output (`dist/`)**: compiled JavaScript matching the source layout.

## Development Tips
- Run the server locally with `npm run dev` and access endpoints under the mounted paths (`/spotify`, `/steam`, `/letterboxd`).
- After changing TypeScript files, the dev server restarts automatically.
- Ensure required env vars are present before starting the server.

---
> Source: [melzarjanchico/ap-express-js-server](https://github.com/melzarjanchico/ap-express-js-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
