## personal

> A personal assistant that lives in Telegram, powered by Claude and built in Elixir. It manages a personal knowledge base stored as markdown files, encrypted at rest with age.

# KbaseBot — Personal Knowledge Base Assistant

## What this is

A personal assistant that lives in Telegram, powered by Claude and built in Elixir. It manages a personal knowledge base stored as markdown files, encrypted at rest with age.

## Architecture

- **Elixir/OTP** application running on the BEAM
- **Two-layer LLM design**:
  - **Manager**: handles conversation, decides what to do, spawns tasks, manages schedules
  - **Task**: background workers that search/read the knowledge base and report back
- **SQLite** (via Exqlite) for persistence: conversation history, tasks, schedules, journal entries
- **QMD** for semantic search over the knowledge base (can be disabled via `QMD_ENABLED=false`)
- **Telegram** as the sole interface (via ExGram)
- **Cron scheduler** for recurring events (daily briefings, reminders)

## Knowledge base

The knowledge base is a collection of markdown files covering training, nutrition, medical history, and personal notes. Files are encrypted with age before committing (`.vault/*.age`), decrypted on the server at deploy time.

## Key directories

- `kbase_bot/` — the Elixir application
  - `lib/kbase_bot/tools/` — all LLM tools (each implements the `KbaseBot.Tool` behaviour)
  - `lib/kbase_bot/scheduler/` — cron scheduling
  - `lib/kbase_bot/tasks/` — background task execution
  - `priv/prompts/` — system prompts for manager/task/briefing layers
- `knowledge_base/` — the actual content (gitignored, only encrypted vault tracked)
- `infra/` — Terraform + Ansible for VPS provisioning
- `scripts/` — encryption/decryption helpers

## Tools available to the assistant

The assistant has tools split by layer:
- **Manager tools**: respond, journal_entry, spawn_task, list/cancel/message tasks, create/list/cancel/update schedules, get_current_time, refresh_context, send_gif, request_tool
- **Task tools**: search_knowledge, read_file, list_files, notify_user, request_tool

## Conventions

- Responses should be concise — this is Telegram
- The assistant has a personality inspired by Hanekawa from Monogatari
- Health/medical info is sensitive — never expose it outside the encrypted knowledge base
- When the assistant wishes it had a tool it doesn't have, it should use `request_tool` to log the request

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PsicoThePato) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
