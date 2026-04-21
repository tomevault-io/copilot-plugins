## translator-bot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Telegram translation bot built with Bun, Telegraf, and Telegram MTProto. The bot automatically detects message language and translates between Indonesian/Malay and English as the primary translation pair.

## Development Commands

```bash
# Development with hot reload
bun --watch .

# Build for production
bun build --target bun src/index.ts --outdir dist

# Run production build
bun run ./dist/index.js

# Docker build and run
docker build -t translator-bot .
docker run --env-file .env translator-bot
```

## Architecture

### Translation Flow
The bot uses a dual-translation approach with fallback:

1. **Primary**: MTProto client (`src/mtproto.ts`) via Telegram's official API
   - Uses `messages.TranslateText` RPC method
   - Requires active Telegram session (SESSION env var)
   - Falls back to google-translate-api-x if MTProto fails

2. **Language Detection**: Uses google-translate-api-x with auto-detection
   - Indonesian/Malay (id, ms) → English
   - All other languages → Indonesian

### Key Components

**src/index.ts** - Main bot logic
- Telegraf bot instance with text message handlers
- Language detection via `detect()` function
- Response formatting with country flags and HTML escaping
- Long message handling (splits at 2048 chars)
- Silent notifications and disabled link previews

**src/mtproto.ts** - MTProto client wrapper
- tgsnake Snake client for raw Telegram API access
- Exports session on startup for debugging
- Single exported `translate()` function

### Environment Variables
Required in `.env` (see `.env.example`):
- `BOT_TOKEN`: Telegram bot token from @BotFather
- `API_HASH`: Telegram API hash from my.telegram.org
- `API_ID`: Telegram API ID from my.telegram.org
- `SESSION`: Telegram user session string for MTProto

## Code Conventions

- TypeScript with strict mode enabled
- Bun runtime and bundler
- ESM modules only (`"type": "module"`)
- HTML escaping for all user content in responses
- Error handling: catch blocks log to console, don't crash bot

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sachnun) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
