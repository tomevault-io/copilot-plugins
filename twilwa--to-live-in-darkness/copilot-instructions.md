## to-live-in-darkness

> - **pnpm** (primary) for package management and scripts

# Redbot - Discord Voice Bot

## Project Setup

This project uses:

- **pnpm** (primary) for package management and scripts
- **mise** (optional) for environment and task management
- **TypeScript** for development
- **trunk** for code quality checks

## Development Workflow

### Task Management

Preferred (pnpm direct):

- `pnpm install` - Install dependencies
- `pnpm build` - Build the project
- `pnpm dev` - Run in development mode
- `pnpm test` - Run tests

Optional (mise tasks, if desired):

- `mise tasks` - List available tasks
- `mise pnpm-install` - Install dependencies
- `mise pnpm-build` - Build the project
- `mise pnpm-dev` - Run in development mode
- `mise pnpm-test` - Run tests

### Code Quality

- **trunk check** is enabled for pre-commit hooks
- If pre-commit hooks fail, document issues in `@scratchpads-logs/`

## Architecture

### Tech Stack

- **Discord.js** with @discordjs/voice for Discord integration
- **Deepgram SDK** for both speech-to-text and text-to-speech
- **E2B Code Interpreter** for sandboxed code execution
- **Obsidian Local REST API** for vault integration
- **Zellij/tmux** for terminal pane management
- **Carapace** for shell completion specs

### Voice Pipeline

- Continuous listening (no push-to-talk)
- Deepgram Voice Agent integration with Discord.js
- Real-time STT → LLM → TTS pipeline
- Target latency: < 2 seconds end-to-end

### Data Sources

- **Obsidian Clippings**: Weekly clippings from `/Clippings/` subdirectory
- **LLM**: OpenAI or Anthropic for conversation and code generation
- **Sandboxed Execution**: E2B for running user code safely

### Terminal Integration

- **Glow**: Simple CLI execution in Zellij/tmux panes
- **Pane Management**: Automated layout creation and labeling
- **CLI Generation**: Go Cobra tools with Carapace completions

## Environment Variables Required

```bash
# Discord
DISCORD_BOT_TOKEN=
DISCORD_APPLICATION_ID=
DISCORD_PUBLIC_KEY=

# Voice & AI Services
DEEPGRAM_API_KEY=
OPENROUTER_API_KEY= # For LLM (OpenAI-compatible)
OPENROUTER_MODEL= # e.g., claude-3-sonnet, gpt-4, etc.

# Obsidian
OBSIDIAN_API_KEY=

# Code Execution
E2B_API_KEY=
```

## File Structure

```
redbot/
├── src/
│   ├── discord/     # Discord client and voice handling
│   ├── voice/       # Deepgram voice pipeline
│   ├── obsidian/    # Vault integration
│   ├── terminal/    # Zellij/tmux/glow integration
│   ├── code/        # E2B code execution
│   └── cli/         # CLI generation and Carapace specs
├── tests/           # Test suites for all modules
├── spec/            # Detailed specifications
└── scratchpads-logs/ # Development notes and findings
```

## Implementation Priority

1. **MVP**: Discord voice bot with continuous listening
2. **Core**: Obsidian weekly clippings integration
3. **Enhanced**: Terminal glow integration for markdown display
4. **Advanced**: Code execution and CLI generation

## Notes

- Manual muting preferred over push-to-talk implementation
- Clippings are pre-filtered by date in Obsidian vault structure
- Simple glow integration - just execute CLI in terminal panes
- Deepgram Voice Agent provides integrated STT/TTS pipeline

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/twilwa)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/twilwa)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
