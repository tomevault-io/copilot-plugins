## hytale-docker

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Hytale Docker is a production-ready Docker container for running Hytale dedicated servers. The runtime is built with Bun + TypeScript, compiled to standalone binaries for the container.

## Commands

### Runtime Development
```bash
bun run src/main.ts          # Run server entrypoint locally
bun run src/hytale.ts        # Run CLI locally
bun run typecheck            # TypeScript type checking
bun run fmt                  # Format code with Biome
bun run lint                 # Lint code with Biome
```

### Docker
```bash
bun run build                # Build Docker image
bun run build:clean          # Build without cache
bun run run:docker           # Run container locally
docker exec -it hytale-server hytale <command>  # CLI inside container
```

### Documentation Site (docs/)
```bash
cd docs && pnpm install      # Install docs dependencies
cd docs && pnpm dev          # Run docs dev server
cd docs && pnpm build        # Build docs
cd docs && pnpm lint         # Lint docs with Biome
```

## Architecture

### Two Entry Points
- `src/main.ts` - Container entrypoint (compiled to `hytale-server` binary). Handles auth, downloads, updates, and spawns the Java server process.
- `src/hytale.ts` - CLI tool (compiled to `hytale` binary). Provides commands for auth, profile management, and server control.

### Module Structure (`src/modules/`)

**Core** (`core/`):
- `Config.ts` - Environment variable parsing and path configuration
- `Logger.ts` - Colored console output

**Auth** (`auth/`):
- `OAuthClient.ts` - RFC 8628 Device Code Flow for Hytale authentication
- `TokenStore.ts` - Persists OAuth/session tokens to filesystem
- `ProfileManager.ts` - Game profile selection
- `SessionManager.ts` - Game session lifecycle (create/terminate)
- `AuthService.ts` - Orchestrates auth flow, background token refresh

**Server** (`server/`):
- `VersionService.ts` - Detects available updates
- `UpdateManager.ts` - Update scheduling logic
- `DownloadManager.ts` - Downloads server via hytale-downloader binary
- `ServerProcess.ts` - Spawns and manages Java server process
- `CurseForgeManager.ts` - Optional mod synchronization from CurseForge

**CLI** (`cli/`):
- `HytaleCli.ts` - Main CLI command router
- `AuthCli.ts` - Auth-related subcommands

### Types (`src/types/`)
Centralized TypeScript interfaces for OAuth tokens, sessions, profiles, config, and server options.

### Docker Build
Multi-stage Dockerfile:
1. `build` stage - Compiles TypeScript to standalone Bun binaries
2. `downloader` stage - Fetches official hytale-downloader
3. `runtime` stage - Eclipse Temurin JRE 25 Alpine with tini init

### Environment Variables
Key configuration (see `Config.ts` for full list):
- `JAVA_OPTS` - JVM memory settings
- `SERVER_PORT` - UDP port (default: 5520)
- `PATCHLINE` - release/beta channel
- `AUTO_UPDATE` - Enable automatic server updates
- `CF_API_KEY`, `CF_MODS` - CurseForge mod support

## Key Technical Details

- Hytale uses QUIC over UDP (port 5520), not TCP
- OAuth tokens stored in `/server/.hytale/tokens/`
- Background daemon refreshes tokens every 30 days
- Server JAR path: `/server/Server/HytaleServer.jar`

---
> Source: [Romariin/hytale-docker](https://github.com/Romariin/hytale-docker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
