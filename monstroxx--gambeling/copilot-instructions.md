## gambeling

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Discord bot named "gameling" (originally "risk-discord-bot") that implements a `/risk` slash command game. Users can execute the command to receive random outcomes ranging from timeouts to a rare jackpot role reward. The bot is designed for deployment on Fly.io.

## Architecture

**Single-file Node.js Application**: The entire bot logic is contained in `index.js` - a straightforward Discord.js v14 implementation with no complex module structure.

**Core Components**:
- Discord.js client with appropriate gateway intents for guild operations
- Slash command registration system using Discord REST API
- Probability-based outcome system with weighted random selection
- Dual command interface supporting both slash commands (`/risk`) and legacy text commands (`?risk`)

**Outcome System**: The `getRandomOutcome()` function implements a weighted probability system:
- 25% safe (no consequence)
- 30% short timeouts (1-5 minutes)
- Escalating timeout probabilities up to 24 hours
- 0.1% jackpot role reward

## Development Commands

**Install dependencies:**
```bash
npm install
```

**Start the bot:**
```bash
npm start
# or
npm run dev
```

**Build and run with Docker:**
```bash
docker-compose up -d --build
```

**View logs:**
```bash
docker-compose logs -f discord-bot
```

**Stop the bot:**
```bash
docker-compose down
```

## Environment Configuration

The bot requires these environment variables in `.env`:
- `DISCORD_TOKEN`: Bot authentication token
- `CLIENT_ID`: Discord application client ID  
- `GUILD_ID`: Target Discord server ID
- `JACKPOT_ROLE_ID`: ID of the role awarded for jackpot outcomes

## Bot Permissions Required

- `Moderate Members` (for timeout functionality)
- `Manage Roles` (for jackpot role assignment)
- `Send Messages`
- `Use Slash Commands`

## Deployment

**Docker Configuration**: The bot is containerized for easy deployment on any host:
- Multi-stage Docker build for optimized image size
- Node.js 20.18.0 slim base image
- Production environment settings
- Docker Compose setup with automatic restart policy

**Hetzner Deployment**: Copy files to server, configure `.env`, then run `docker-compose up -d --build`

---
> Source: [Monstroxx/gambeling](https://github.com/Monstroxx/gambeling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
