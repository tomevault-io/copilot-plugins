## maestro-slack

> This repo is a Slack bot that bridges messages to Maestro agents via `maestro-cli`.

# Agent Guide

This repo is a Slack bot that bridges messages to Maestro agents via `maestro-cli`.
CLAUDE.md is a symlink to this file.

## Development workflow

- Install deps: `npm install`
- Run in dev: `npm run dev`
- Deploy slash commands: `npm run deploy-commands`
- Build: `npm run build`
- Production: `npm run build` then `npm start`
- Run tests: `npm test`

## Project layout

- `src/config.ts` — env var loading
- `src/db/index.ts` — SQLite channel registry (agent_channels table)
- `src/services/maestro.ts` — maestro-cli wrapper (listAgents, listSessions, send)
- `src/services/queue.ts` — per-channel FIFO message queue
- `src/services/logger.ts` — logging service
- `src/server.ts` — internal HTTP API server (POST /api/send, GET /api/health)
- `src/commands/` — slash command handlers (health, agents)
- `src/handlers/messageCreate.ts` — Slack message listener → queue
- `src/utils/splitMessage.ts` — splits long messages for Slack's message limits
- `src/deploy-commands.ts` — registers slash commands with Slack API
- `bin/maestro-slack.ts` — CLI tool for agent-to-Slack messaging

## HTTP API

Local API on `127.0.0.1:API_PORT` (default 3457). See [docs/api.md](docs/api.md) for endpoints, request format, and error codes.

## Project notes

- Source lives in `src/` and is TypeScript.
- Env vars are defined in `.env.example`. Keep it in sync with `.env` usage.
- Avoid adding new runtime dependencies unless necessary.
- If you add new slash commands, update the deploy script and README.
- Tests use Node.js built-in test runner (`node --test`), not Jest/Vitest.
- The server uses proper type guards for channel safety (not unsafe casts).

## Expectations for changes

- Follow existing patterns in `src/` before introducing new abstractions.
- Keep changes minimal and focused.
- Update docs when behavior or setup changes.

---
> Source: [scriptease/Maestro-Slack](https://github.com/scriptease/Maestro-Slack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-05 -->
