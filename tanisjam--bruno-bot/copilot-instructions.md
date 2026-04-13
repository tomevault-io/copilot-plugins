## bruno-bot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Bruno Bot is a Discord bot for D&D 5e campaign management that integrates with Nivel20.com. It provides character lookup functionality, automated weekly reminders for game sessions, with plans for weather systems and server-specific shops.

## Essential Commands

### Development
```bash
# Build TypeScript to JavaScript
pnpm run build

# Development mode with auto-reload
pnpm run dev

# Production start
pnpm start

# Deploy Discord slash commands (required after adding/modifying commands)
pnpm run deploy:commands
```

### Docker Deployment
```bash
# Build and run with Docker Compose
docker-compose up -d

# View logs
docker-compose logs -f bruno-bot

# Stop container
docker-compose down
```

## Architecture

### Core Bot Flow
1. **Entry point**: `src/index.ts` → imports `src/bot.ts`
2. **Bot initialization**: `src/bot.ts` creates Discord client, registers commands, sets up event handlers
3. **Command registration**: Commands from `src/commands/` are loaded into `client.commands` Collection
4. **Interaction handling**: `Events.InteractionCreate` handler routes commands to their `execute()` functions

### Command System
- All commands must implement the `Command` interface (`src/types/Command.ts`)
- Each command exports `data` (SlashCommandBuilder) and `execute()` function
- Commands are auto-registered by exporting from `src/commands/index.ts`
- Deploy commands with `pnpm run deploy:commands` after changes

### Key Architectural Patterns

**Service Layer**:
- `src/services/nivel20.service.ts` - Web scraping service for Nivel20.com character data
  - Uses Axios for HTTP requests and Cheerio for HTML parsing
  - CharacterSheet model transforms raw JSON to Discord embeds
- `src/services/database.service.ts` - SQLite database service using better-sqlite3
  - Singleton pattern for single instance across the bot
  - Provides CRUD operations for guilds, reminders, weather, shop inventory
  - See `DATABASE.md` for complete documentation
- `src/services/scheduler.service.ts` - Automated task scheduler using node-cron
  - Runs every minute checking for reminders to send
  - Validates permissions before sending messages
  - Manages start/stop lifecycle

**Interactive Components**:
- Character search uses button-based interaction (60-second timeout)
- Button collectors verify user ownership before responding
- Custom IDs format: `character_{index}` for tracking selections

**Error Handling**:
- Global error handlers in `src/bot.ts` for unhandled rejections and exceptions
- Command errors respond with user-friendly Spanish error messages
- Proper handling of deferred/replied interaction states

**Configuration**:
- Environment variables validated in `src/config.ts` at startup
- Required: `CLIENT_ID`, `TOKEN`, `GUILD_ID`
- Optional: `DB_PATH` (defaults to `./data/bruno-bot.db`)

### Important Implementation Details

**Nivel20 Integration**:
- Campaign URL is hardcoded in `src/services/nivel20.service.ts` (line 16)
- Search endpoint: `{CAMPAIGN_URL}?utf8=%E2%9C%93&q={query}`
- Character details: `{BASE_URL}{character_path}.json`
- Must update `CAMPAIGN_URL` constant when changing campaigns

**TypeScript Configuration**:
- Strict mode enabled with comprehensive null checks
- `noUncheckedIndexedAccess: true` - requires explicit null checking for array/object access
- Module system: CommonJS for Node.js compatibility
- Target: ES2022

**Discord.js Patterns**:
- Client interface extended to include `commands: Collection<string, Command>`
- Intents: Guilds, GuildMessages, MessageContent
- All user-facing text is in Spanish
- DM-only commands use `.setDefaultMemberPermissions(PermissionFlagsBits.ManageGuild)`

**Database & Persistence**:
- SQLite database stored in `./data/bruno-bot.db` (configurable via `DB_PATH`)
- Schema automatically initialized on first run from `src/database/schema.sql`
- Database persists guilds, reminders, weather conditions, and shop inventory
- All tables use foreign keys for referential integrity
- See `DATABASE.md` for full schema documentation

**Scheduler System**:
- `SchedulerService` starts when bot is ready (`Events.ClientReady`)
- Cron job runs every minute: `* * * * *`
- Compares current day/time with enabled reminders
- Sends messages only if bot has `SendMessages` permission in target channel
- Graceful shutdown handlers (SIGINT/SIGTERM) stop scheduler and close DB

## Testing Workflow

1. Make code changes
2. Run `pnpm run build` to compile TypeScript
3. If command definitions changed, run `pnpm run deploy:commands`
4. Run `pnpm start` or `pnpm run dev` to test
5. Monitor logs via `src/utils/logger.ts`

## Implemented Features

### Weekly Reminders ✅
- **Command**: `/recordatorio` with subcommands (crear, listar, activar, desactivar, eliminar)
- **Storage**: SQLite database (`reminders` table)
- **Scheduling**: node-cron checks every minute for matching day/time
- **Permissions**: Requires ManageGuild permission (DM-only)

## Future Features (Planned)

The README outlines planned features: weather system and server shop. These will require:
- Additional command files in `src/commands/` (weather.ts, shop.ts)
- New service files in `src/services/` (weather.service.ts, shop.service.ts)
- Database schema already supports these features (see `DATABASE.md`)

When implementing these, follow the existing command pattern and service architecture.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TanisJam)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/TanisJam)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
