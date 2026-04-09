## claude-remote-control

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claude Remote Control is a Telegram bot that allows users to interact with Claude Code (via the Agent Client Protocol) through Telegram messages. It spawns a Node.js subprocess running the `claude-agent-acp` adapter, which wraps the Claude Agent SDK.

## Commands

### Python/Telegram Bot

```bash
# Activate virtual environment
source ./venv/bin/activate

# Run the bot
python bot.py

# Lint
ruff check .

# Format
ruff format .
```

### Claude Agent ACP (submodule)

```bash
# Build the submodule
cd claude-agent-acp
npm install
npm run build
cd ..

# Run tests
npm test              # watch mode
npm run test:run      # single run
npm run test:integration  # integration tests
```

## Architecture

```
bot.py              # Telegram bot (entry point)
    ├── Handles Telegram updates, commands, inline menus
    ├── Manages sessions per chat_id
    └── Routes messages to ACP client

acp_client.py       # ACP client wrapper
    ├── Spawns Node.js subprocess running claude-agent-acp
    ├── JSON-RPC communication over stdin/stdout
    ├── Handles permission requests via callbacks
    └── Notifies bot of agent events (tool calls, messages, costs)

claude-agent-acp/   # Git submodule (TypeScript)
    └── ACP adapter wrapping Claude Agent SDK
    └── Communicates with Anthropic API
```

### Communication Flow

1. User sends message to Telegram bot
2. `bot.py` forwards message to `AcpClient.prompt()`
3. `AcpClient` sends JSON-RPC request to Node.js subprocess
4. Claude Agent processes the prompt, may request permissions or call tools
5. Notifications flow back through `notification_callback` and `permission_callback`
6. Bot displays tool calls and responses to user in Telegram

### Key Data Structures

- `sessions`: dict[chat_id, dict] - stores client, session_id, buffer, cwd, tool_messages
- `pending_permissions`: dict[chat_id, asyncio.Future] - tracks pending permission requests

## Environment Variables

- `TELEGRAM_BOT_TOKEN`: Bot API token from @BotFather
- `ACP_PATH`: Path to compiled ACP adapter (default: `claude-agent-acp/dist/index.js`)
- `ALLOWED_USER_IDS`: Comma-separated Telegram user IDs (optional, empty = allow all)
- `ANTHROPIC_API_KEY`: Required by claude-agent-acp (set in submodule or environment)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/manhpham90vn)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/manhpham90vn)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
