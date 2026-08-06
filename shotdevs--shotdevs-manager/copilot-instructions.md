## shotdevs-manager

> This file provides guidance to Qoder (qoder.com) when working with code in this repository.

# AGENTS.md

This file provides guidance to Qoder (qoder.com) when working with code in this repository.

## Commands

### Starting the bot
```bash
node index.js
```

### Deploy slash commands
```bash
node deploy.js
```

### Testing (no test framework configured)
No automated tests are currently configured. Manual testing is required by running the bot in a Discord server.

## Architecture

### Entry Point (index.js)
The bot uses discord.js v14 with a license verification system. Startup sequence:
1. License verification via `LicenseManager` (connects to external API)
2. MongoDB connection
3. Command deployment (registers slash commands with Discord)
4. Discord bot login
5. Express web server on port 3000 (serves landing page)

License checks run every 24 hours; bot exits if verification fails.

### Command System
Commands are organized in `/commands` by category:
- `general/`: Bot info, help, ping, server/user info, announcements
- `levels/`: XP/leveling system (rank, leaderboard)
- `moderation/`: Kick, ban, warn, timeout, clear, lock/unlock channels, moderation logging
- `utility/`: Tickets, reaction roles, autoroles, welcome system

Commands must export `data` (SlashCommandBuilder) and `execute` function. The `category` field is auto-assigned from folder name.

### Event System
All events in `/events` follow discord.js event handler pattern. Key events:
- `interactionCreate.js`: Handles slash commands, ticket buttons, reaction role buttons
- `ready.js`: Bot startup
- Guild/member/message events for logging and automation

### Database Models (MongoDB/Mongoose)
- `GuildConfig`: Per-server configuration (roles, channels, ticket settings, welcome system, reaction roles stored as Map)
- `Level`: XP/leveling data (text XP + voice XP, custom backgrounds)
- `MemberProfile`: User warnings

### Ticket System
Buttons with customIds: `create_order_ticket`, `create_enquiry_ticket`, `create_support_ticket`
Creates private channels with permissions for user + staff role
Buttons in ticket: close (saves transcript), claim, transcript (DMs user)
Transcripts sent to `ticketTranscriptChannelId` or fallback to `ticketLogChannelId`
Close delay configurable via `ticketCloseDelaySeconds` in GuildConfig
**All ticket messages use Components V2** with transparent containers, sections, and buttons inside containers

### Reaction Role System
Two implementations exist:
- `configManager.js`: File-based JSON storage in `guild_configs.json`
- `GuildConfig` model: MongoDB-based with `reactionRoles` Map

### Utilities
- `utils/logger.js`: Custom logger with emoji categories (info, success, error, license, command, database, web, bot)
- `utils/LicenseManager.js`: Handles license verification and periodic checks
- `utils/componentsV2Builder.js`: Components V2 builder utilities for creating modern Discord UI (containers, sections, separators, buttons)

### Configuration Management
Dual system (migration may be incomplete):
- File-based: `configManager.js` reads/writes `guild_configs.json`
- Database: `GuildConfig` model in MongoDB

### Web Interface
Express server serves EJS template at `/` with bot branding. No dashboard functionality.

## Environment Variables Required
- `DISCORD_TOKEN`: Bot token
- `CLIENT_ID`: Discord application ID
- `MONGO_URI`: MongoDB connection string
- `LICENSE_KEY`: Product license key
- `LICENSE_API_BASE_URL`: License verification API endpoint
- `PORT`: Web server port (default: 3000)

## Discord Components V2

**Status: FULLY MIGRATED** (as of November 2025)

This bot now uses Discord's Components V2 system for ALL messages. Components V2 provides a modern, container-based layout system with transparent backgrounds and enhanced visual design.

### Components V2 Architecture
- **Builder Utility**: `utils/componentsV2Builder.js` provides reusable functions for creating Components V2 messages
- **IS_COMPONENTS_V2 Flag**: All messages sent by the bot include the flag `1 << 15` (32768) to enable Components V2 rendering
- **REST API**: Messages are sent using Discord's REST API via `sendComponentsV2Message()` and `replyComponentsV2()` helpers

### Component Types Used
- **Container (type 17)**: Top-level component with `accent_color: null` (transparent) for clean appearance
- **Section (type 9)**: Content blocks within containers, can include text displays and accessories
- **Text Display (type 10)**: Formatted text content with full markdown support (bold, italic, headers, code, etc.)
- **Separator (type 14)**: Visual dividers between sections (`divider: true, spacing: 1`)
- **Action Row (type 1)**: Button containers placed inside containers for modern layout
- **Button (type 2)**: Interactive buttons and link buttons
- **Thumbnail (type 11)**: Image accessories for sections (avatars, icons, etc.)

### Design Principles
- **Transparent Containers**: All containers use `accent_color: null` for a modern, clean look
- **Liberal Separators**: Dividers between all sections for clear visual hierarchy
- **Buttons Inside Containers**: Action rows are placed inside containers (not as separate components)
- **Markdown Formatting**: Rich text formatting in all text displays (headers, bold, code blocks, etc.)
- **Accessories**: Bot and user avatars displayed as section accessories

### Builder Functions (utils/componentsV2Builder.js)
- `container({ components })` - Creates container component
- `section({ content, accessory })` - Creates section with text/accessory
- `textDisplay(content)` - Creates formatted text display
- `separator(spacing)` - Creates visual divider
- `button({ custom_id, label, style, emoji })` - Creates button
- `actionRow(components)` - Creates button row
- `thumbnail(url)` - Creates image accessory
- `sendComponentsV2Message(client, channelId, payload)` - Sends message via REST
- `replyComponentsV2(interaction, payload)` - Replies to interaction
- `editComponentsV2Message(message, payload)` - Edits existing message

### Commands Converted
All commands now use Components V2:
- **General**: help, botinfo, server-info, user-info, announce, ping, uptime, avatar
- **Moderation**: ban, warn, kick, timeout, unban, lock, unlock, moderation-log, clear
- **Levels**: leaderboard
- **Utility**: setup-tickets, view-ticket-config
- **Events**: Ticket system (creation, close logs, claim updates), welcome messages (fallback)

### Testing Command
- `/test-components-v2` - Preview Components V2 layouts for various features (admin only)

### Migration Complete
The bot no longer uses traditional `EmbedBuilder`. All embeds have been converted to Components V2 containers with modern, transparent layouts.

---
> Source: [shotdevs/Shotdevs-Manager](https://github.com/shotdevs/Shotdevs-Manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
