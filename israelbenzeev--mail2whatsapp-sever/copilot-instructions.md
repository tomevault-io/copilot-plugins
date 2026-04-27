## mail2whatsapp-sever

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands
```bash
uv pip install -e .
uv run uvicorn app:app --reload
uv run python test.py
```

## Env vars (.env)
`ENVIRONMENT`, `HOST`, `PORT`, `CLIENT_URL`, `SUPABASE_URL`, `SUPABASE_ROLE_KEY`, `OPENAI_API_KEY`, `GOOGLE_CLIENT_SECRET_JSON` (full JSON as string), `TELEGRAM_BOT_TOKEN`

## Architecture
FastAPI backend on Vercel. Bridges Gmail ↔ Telegram via an OpenAI agent (`openai-agents` SDK).

**Flows:**
- `POST /llm/ask-llm/{user_id}` → agent runs Gmail tools, responses rewritten in Hebrew by `style_agent_tool` sub-agent, history stored in Supabase `messages`
- `GET /OAuth/authorize/{user_id}` + `/OAuth/oauth2callback` → Google OAuth, tokens saved to Supabase `user_tokens`
- `POST /telegram/webhook` → links Telegram `chat_id` to `user_id` in `user_tokens`

**Key files:**
- `app.py` — routers registration
- `agent.py` / `styleAgent.py` — agent + Hebrew style sub-agent
- `controllers/agent_controller.py` — runs agent, persists chat
- `controllers/OAuth_Callback_Controller.py` — Google OAuth logic
- `tools_agent_email/gmail_tools.py` — Gmail API as `@function_tool`
- `tools_agent_email/google_apis.py` — builds Gmail service from Supabase tokens (class-level cache `_tokens_cache`)
- `supabase_client.py` — Supabase singleton

**Supabase tables:** `user_tokens`, `messages`, `user_chat_ids`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IsraelBenZeev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
