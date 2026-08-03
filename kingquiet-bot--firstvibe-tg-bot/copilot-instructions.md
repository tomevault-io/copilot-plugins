## firstvibe-tg-bot

> * `python-telegram-bot` (v20.x) for Telegram integration

# Lead Generation Chatbot

## Stack
* Python 3
* `python-telegram-bot` (v20.x) for Telegram integration
* `sqlite3` (built-in) for local database
* `anthropic` or `openai` for LLM integration
* `python-dotenv` for managing API keys
* `pandas` for exporting SQLite data to CSV/Excel

## Conventions
* Indent: 4 spaces
* Variables/Functions: snake_case (e.g., `save_lead`, `user_state`)
* Keep API keys in a `.env` file (never hardcode them)
* Use clear async/await syntax for Telegram bot handlers

## Don't
* Don't build a Web UI or Admin Dashboard (CLI and Telegram interface only)
* Don't use complex databases like PostgreSQL/MySQL (stick to local SQLite)
* Don't build payment gateways or checkout systems
* Don't guess the company products; wait for `company_knowledge.txt`

## Commands
# Install dependencies
pip install python-telegram-bot anthropic python-dotenv pandas

# Run the bot
python bot.py

# Export data to CSV
python export_data.py

## Files
* `bot.py` (Main Telegram bot logic & AI routing)
* `database.py` (SQLite connection, save chat history & leads)
* `export_data.py` (Script to export SQLite tables to CSV)
* `company_knowledge.txt` (Product info for the bot to read)
* `.env` (Stores TELEGRAM_TOKEN and AI_API_KEY)

## Owner Notes
* Target audience: Potential customers asking about products on Telegram.
* Must save all conversation logs to a `chat_history` table.
* The bot must use "Function Calling" or "State Management" to sequentially collect Name, Phone, and Email when purchase intent is detected.

---
> Source: [kingquiet-bot/FirstVibe-tg-bot](https://github.com/kingquiet-bot/FirstVibe-tg-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
