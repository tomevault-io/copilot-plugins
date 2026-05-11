## pocket-agent

> A persistent desktop AI assistant powered by Claude Agent SDK that runs 24/7 as a system tray application with continuous memory, Telegram integration, browser automation, and scheduled task management.

# Pocket Agent

A persistent desktop AI assistant powered by Claude Agent SDK that runs 24/7 as a system tray application with continuous memory, Telegram integration, browser automation, and scheduled task management.

## Project Structure

```
src/
├── main/           # Electron main process (app lifecycle, tray, windows)
├── agent/          # Claude Agent SDK wrapper and orchestration
├── memory/         # SQLite persistence (messages, facts, embeddings)
├── channels/       # Communication channels (Telegram, desktop)
├── scheduler/      # Cron job management
├── browser/        # 2-tier browser automation (Electron + CDP)
├── tools/          # Agent tool implementations
├── config/         # Configuration and identity loading
├── settings/       # User preferences management
├── auth/           # OAuth flows for integrations
├── permissions/    # System permissions handling (macOS)
├── cli/            # Command-line interfaces
├── mcp/            # Model Context Protocol servers
└── skills/         # Skill system for extensibility

ui/                 # HTML interfaces (chat, settings, cron, facts)
tests/unit/         # Vitest unit tests
assets/             # Tray icons and static assets
.claude/            # Claude Code commands and skills
```

## Organization Rules

**Keep code organized by responsibility:**
- Electron main process → `src/main/`
- Agent logic → `src/agent/`
- Persistence → `src/memory/`
- External channels → `src/channels/`
- Tool implementations → `src/tools/`
- Configuration → `src/config/` and `src/settings/`
- Browser automation → `src/browser/`

**Modularity principles:**
- Single responsibility per file
- Clear, descriptive file names
- Group related functionality together
- Avoid monolithic files

## Code Quality - Zero Tolerance

After editing ANY file, run:

```bash
npm run typecheck && npm run lint
```

Fix ALL errors/warnings before continuing.

**Available scripts:**
- `npm run lint` - ESLint check
- `npm run lint:fix` - Auto-fix lint issues
- `npm run typecheck` - TypeScript type checking
- `npm run format` - Prettier auto-format
- `npm run test` - Run all tests

## Key Architecture

**Memory Layer:** SQLite with messages, facts, and embeddings for persistent conversation

**Browser Automation:** Dual-tier system
- Electron tier: Hidden window for JS rendering
- CDP tier: Chrome DevTools Protocol for authenticated sessions

**Channel System:** Abstracts Telegram and desktop UI communication

**Scheduler:** Cron-based task automation with SQLite persistence

---
> Source: [KenKaiii/pocket-agent](https://github.com/KenKaiii/pocket-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
