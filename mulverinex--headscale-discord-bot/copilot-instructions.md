## headscale-discord-bot

> Headscale Discord Bot is a Go application that integrates Discord with Headscale (an open-source VPN server). It enables Discord users to manage a Headscale network directly from Discord with real-time notifications and IP geolocation.

# CLAUDE.md - Development Guide

## Project Overview

Headscale Discord Bot is a Go application that integrates Discord with Headscale (an open-source VPN server). It enables Discord users to manage a Headscale network directly from Discord with real-time notifications and IP geolocation.

## Build & Test Requirements

**IMPORTANT**: Always run a test build after modifying Go code or dependencies:

```bash
CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o headscale-discord-bot .
```

This matches the Docker build command and catches syntax errors before CI fails. Clean up afterward:
```bash
rm -f headscale-discord-bot
```

Run `go fmt ./...` after making changes to ensure consistent formatting.

## Directory Structure

```
headscale-discord-bot/
├── bot/                      # Discord bot core logic
│   ├── bot.go               # Bot initialization, monitoring, notifications
│   ├── commands.go          # Slash command definitions
│   └── handlers.go          # Command execution handlers
├── config/                   # Configuration management
│   └── config.go            # Environment variable loading
├── geolocation/             # IP geolocation module
│   └── geolocation.go       # IP lookup with caching
├── monitor/                 # Log monitoring module
│   └── log_monitor.go       # Journalctl/file tailing
├── main.go                  # Application entry point
├── Dockerfile               # Multi-stage Docker build
├── docker-compose.yml       # Docker Compose configuration
├── .env.example             # Environment variable template
└── VERSION                  # Current version for releases
```

## Key Components

### bot/bot.go
- `New(cfg)` - Initialize bot with Discord session and Headscale client
- `Start()` - Register commands, initialize known nodes, start monitoring
- `Stop()` - Graceful shutdown
- `monitorNodeRegistrations()` - API polling goroutine
- `sendNewNodeNotification()` - Send rich embeds to Discord

### bot/commands.go
Defines 8 slash commands (all disabled by default, admins must grant access):
- `/nodes`, `/users` - List resources
- `/approve-node`, `/rename-node`, `/delete-node` - Node management
- `/create-user`, `/delete-user` - User management
- `/set-notify-channel` - Configure notifications

### bot/handlers.go
Implements command handlers with:
- Autocomplete support for user selection
- Deferred responses for long-running operations
- Reply-to-message support for `/approve-node`

### monitor/log_monitor.go
Two monitoring modes:
1. **Journalctl** (recommended): `journalctl -u <service> -f`
2. **File tailing**: Uses hpcloud/tail library

### geolocation/geolocation.go
- Queries ip-api.com for public IP geolocation
- In-memory caching with thread-safe access
- Detects and skips private IPs

## Configuration

Required environment variables:
- `DISCORD_TOKEN` - Bot token
- `HEADSCALE_API_URL` - Headscale API endpoint (HTTPS)
- `HEADSCALE_API_KEY` - API key
- `GUILD_ID` - Discord server ID

Optional:
- `POLL_INTERVAL_SECONDS` - API polling frequency (default: 5)
- `HEADSCALE_SYSTEMD_SERVICE` - Service name for journalctl
- `HEADSCALE_LOG_FILE` - Log file path

## Development Workflow

### Adding New Commands
1. Define in `bot/commands.go` (add to `commands` slice)
2. Add case in `bot/handlers.go` `handleCommand()` switch
3. Implement handler function
4. **Test build before committing**

### Testing Locally
```bash
export DISCORD_TOKEN=your_token
export HEADSCALE_API_URL=https://your-headscale.com
export HEADSCALE_API_KEY=your_key
export GUILD_ID=your_guild_id
go run .
```

### Releasing
1. Update VERSION file with new version number
2. Push to main - GitHub Actions auto-creates tag
3. Tag triggers Docker build and publish to ghcr.io

## Dependencies

| Package | Purpose |
|---------|---------|
| `discordgo` | Discord API wrapper |
| `headscale-client-go` | Headscale API client |
| `hpcloud/tail` | Log file tailing |

## Docker Build

The Dockerfile uses a multi-stage build:
- **Build stage**: golang:1.24-trixie with static compilation
- **Runtime stage**: debian:trixie-slim with ca-certificates and systemd

## Common Issues

### Syntax errors in CI
Always test build locally before pushing. The most common issues:
- Missing commas in struct literals
- Incorrect indentation (run `go fmt`)
- Mismatched brackets

### Journal access in Docker
Run `./setup-journal-access.sh` or manually configure group_add in docker-compose.yml

### Commands not appearing in Discord
- Verify GUILD_ID is correct
- Commands are disabled by default - grant access in Server Settings > Integrations

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MulverineX) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
