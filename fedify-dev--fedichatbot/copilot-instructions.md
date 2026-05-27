## fedichatbot

> This file provides guidance to LLM-based coding agents when working with code in this repository.

# AGENTS.md

This file provides guidance to LLM-based coding agents when working with code in this repository.

## Project Overview

FediChatBot is an LLM-powered chatbot for the fediverse, built with Deno. It uses BotKit (a framework for building fediverse bots on top of Fedify) and Google's Gemini 3 Flash model via LangChain.

## Development Commands

```bash
# Run development server with hot reload
deno task dev

# Type check, lint, and format check
deno task check

# Individual checks
deno check *.ts      # Type checking
deno lint            # Linting
deno fmt --check     # Format check
deno fmt             # Auto-format
```

## Architecture

The entire bot is contained in a single file `bot.ts` (~350 lines). Key components:

- **Bot Setup**: Uses `@fedify/botkit` to create a fediverse bot with Deno KV for persistence
- **LLM Integration**: Uses LangChain with `ChatGoogleGenerativeAI` (Gemini 3 Flash)
- **Event Handlers**:
  - `bot.onFollow`: Sends greeting when someone follows
  - `bot.onMention`: Responds to mentions (not replies)
  - `bot.onReply`: Handles threaded conversations with context
- **Prompt Templates**: Located in `prompts/` directory (system.txt, follow.txt, mention.txt, reaction.txt)
- **Image Handling**: Caches images in Deno KV with 3-hour expiry, converts to data URLs for LLM

## Key Dependencies

- `@fedify/botkit`: Fediverse bot framework (docs: https://botkit.fedify.dev/llms.txt)
- `@fedify/fedify`: ActivityPub federation library (docs: https://fedify.dev/llms.txt)
- `@langchain/google-genai`: Google Gemini integration
- `@logtape/logtape`: Structured logging
- `tinyld`: Language detection for responses

## Environment Variables

Requires Google AI API key for Gemini access (loaded via `@std/dotenv`).

## License

AGPL-3.0-only

---
> Source: [fedify-dev/fedichatbot](https://github.com/fedify-dev/fedichatbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
