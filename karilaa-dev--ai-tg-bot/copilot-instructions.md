## ai-tg-bot

> An AI-powered Telegram bot with streaming responses, tool support (web search and webpage extraction), conversation persistence, invite-based access control, multi-language support (EN/RU/UK), and optional thinking traces.

# AI Telegram Bot

An AI-powered Telegram bot with streaming responses, tool support (web search and webpage extraction), conversation persistence, invite-based access control, multi-language support (EN/RU/UK), and optional thinking traces.

## Codebase Overview

**Stack**: Python 3.12+, aiogram 3.24+, OpenAI SDK (via OpenRouter), SQLAlchemy 2.0, Tavily, Telegram Bot API 9.3

**Structure**:
- `main.py` - Entry point with multi-language command registration
- `bot/ai/` - OpenRouter client with streaming, tool calling, and Tavily integration
- `bot/database/` - SQLAlchemy models (User, Conversation, Message, InviteCode) and repository
- `bot/i18n/` - Multi-language translations (EN/RU/UK) with auto-detection
- `bot/telegram/` - aiogram Bot, Dispatcher, filters, and handlers (admin, commands, callbacks, inline, messages)
- `bot/utils/` - Formatting utilities

For detailed architecture, see [docs/CODEBASE_MAP.md](docs/CODEBASE_MAP.md).

## Key Commands

```bash
# Run the bot
python main.py

# Install dependencies
uv sync
```

## Environment Variables

Required in `.env`:
- `TELEGRAM_BOT_TOKEN` - Bot token from @BotFather
- `OPENROUTER_API_KEY` - API key from OpenRouter
- `TAVILY_API_KEY` - API key from Tavily
- `ADMIN_IDS` - Comma-separated admin Telegram IDs (e.g., `123456789,987654321`)

Optional:
- `OPENROUTER_MODEL` - Model ID (default: moonshotai/kimi-k2.5)
- `DATABASE_URL` - SQLAlchemy URL (default: sqlite+aiosqlite:///./bot.db)

## Architecture Notes

- All I/O is async (aiogram, aiosqlite, async OpenAI client)
- Singleton pattern for all clients (`bot`, `dp`, `openrouter_client`, `repository`)
- Streaming responses with Telegram draft messages (Bot API 9.3)
- Multimodal support (images, PDFs converted to base64)
- aiogram routers for handler organization with custom filters
- Invite-based access control with admin commands
- Multi-language UI with auto-detection and user preference

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/karilaa-dev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
