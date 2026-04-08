## claude-discord-bridge

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Essential Commands

### Development
```bash
# Install dependencies
npm install

# Run in development mode with auto-reload
npm run dev

# Build TypeScript to JavaScript
npm run build

# Start production server
npm start
```

### Testing & Quality
```bash
# Run tests
npm test

# Run tests in watch mode
npm test:watch

# Lint TypeScript files
npm run lint

# Format code with Prettier
npm run format
```

### Setup & Maintenance
```bash
# Initialize SQLite database
npm run db:init

# Register Discord slash commands
npm run register

# Run system diagnostic checks
npm run doctor

# Clean and rebuild
npm run rebuild
```

## Architecture Overview

This is a Discord bot that bridges Claude Code CLI with Discord, providing interactive AI assistance through Discord's UI components. The architecture uses a session-based approach with multiple backend options.

### Core Flow
1. **Discord interactions** (slash commands, buttons, modals) are received by the main bot client in `src/index.ts`
2. **SessionManager** (`src/claude/session-manager.ts`) orchestrates multiple concurrent Claude sessions with SQLite persistence
3. **Session backends** execute Claude CLI:
   - **TmuxManager** (primary): Uses tmux for robust session management
   - **PTYManager** (fallback): Direct pseudo-terminal interface
4. **OutputParser** (`src/claude/output-parser.ts`) processes Claude's responses for Discord formatting
5. **ComponentHandler** (`src/interactions/component-handler.ts`) manages Discord's interactive components (buttons, select menus, modals)

### Key Design Patterns
- **Session persistence**: SQLite database stores session state across restarts
- **Streaming architecture**: Real-time output capture from CLI processes
- **Component-based interactions**: Every Claude response includes interactive control panels
- **Thread management**: Auto-creates Discord threads for organized conversations

### Session Management Strategy
- Sessions are identified by unique IDs and mapped to Discord users/channels
- Each session maintains its own Claude CLI process (via tmux or PTY)
- Output is captured in real-time and streamed to Discord with intelligent parsing
- Sessions can be restored after bot restarts if persistence is enabled

### Configuration
The bot is configured via environment variables (see `.env.example`). Key settings:
- `SESSION_TYPE`: Choose between `tmux` (recommended) or `pty`
- `CLAUDE_CLI_PATH`: Path to the Claude Code CLI executable
- Feature flags control streaming, threading, persistence, and other behaviors

### Discord Integration Points
- **Slash commands** defined in `src/discord/commands.ts`
- **Interactive components** (buttons, menus) handled by `src/interactions/component-handler.ts`
- **Real-time streaming** updates Discord messages as Claude generates output
- **File attachments** and **code blocks** are properly formatted for Discord

### Critical Implementation Notes
- Always check if tmux is available before defaulting to that session type
- Session cleanup is crucial to prevent resource leaks
- Discord API rate limits must be respected when streaming updates
- The OutputParser handles various Claude output formats (code blocks, lists, tables)

## Specialized Agents

This project includes specialized Claude Code agents for different development tasks. Each agent has focused expertise in specific areas of the codebase.

### Available Agents
Located in `.claude/agents/`:

1. **Discord Bot Agent** - Discord integration, commands, and UI components
2. **Session Management Agent** - Claude session lifecycle and persistence
3. **Process Management Agent** - Tmux/PTY process handling
4. **Output Processing Agent** - Claude output parsing and formatting
5. **DevOps Agent** - Deployment, monitoring, and system health
6. **Configuration Agent** - Environment variables and feature flags
7. **Testing Agent** - Test creation and maintenance
8. **Documentation Agent** - Project documentation

### Using Agents
When working with Claude Code, you can:
- Request specific agents by mentioning the relevant area (e.g., "Help with Discord commands")
- Explicitly invoke agents (e.g., "Use the Testing Agent to write tests")
- Chain agents for complex tasks

See `.claude/agents/README.md` for detailed agent documentation and usage patterns.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thcapp)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/thcapp)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
