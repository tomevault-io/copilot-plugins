## openclaude

> A Discord bot powered by Claude that acts as your personal JARVIS — AI chat + full life management system.

# OpenClaude - JARVIS Life Optimizer Discord Bot

A Discord bot powered by Claude that acts as your personal JARVIS — AI chat + full life management system.

## Setup

1. Install dependencies: `pip install -r requirements.txt`
2. Copy `.env.example` to `.env` and fill in your tokens
3. Run: `python bot.py`

## Required Environment Variables

- `DISCORD_BOT_TOKEN` - Get from https://discord.com/developers/applications
- `ANTHROPIC_API_KEY` - Get from https://console.anthropic.com

## Commands

Type `!commands` in Discord to see all commands. Summary:

**Chat:** Mention the bot or DM it to chat with Claude
- `!new` — Clear conversation | `!model` — Show model | `!soul` — Show personality | `!stats` — Stats

**Life Optimization:**
- `!briefing` — Daily life dashboard
- `!task <title> | <priority> | <due>` — Add task | `!tasks` — List | `!done <id>` — Complete
- `!daily <title> | <time>` — Add recurring daily task | `!dailies` — Today's status | `!checkdaily <id>` — Done
- `!habit <name>` — Track habit | `!habits` — Status & streaks | `!check <id>` — Log today
- `!remind <min> <msg>` — Set reminder | `!reminders` — List pending
- `!journal <entry> | <mood>` — Write entry | `!entries` — Read back
- `!goal <title> | <target> | <unit>` — Set goal | `!goals` — Progress | `!progress <id> <val>` — Update
- `!focus <min> <task>` — Pomodoro timer | `!stopfocus` — Cancel
- `!emails` — Recent inbox | `!email <id>` — Read | `!send to|subj|body` — Send | `!reply <id> <msg>` — Reply | `!searchmail <query>` — Search

## Features

- **JARVIS personality** via `soul.md` — edit to customize
- **Image/file/PDF support** — attach to messages for Claude to analyze
- **Persistent memory** — SQLite-backed, survives restarts
- **Task management** — priorities, due dates
- **Habit tracking** — daily check-ins, streak counting
- **Timed reminders** — automatic ping when time's up
- **Journal** — entries with mood tracking
- **Goal tracking** — progress bars
- **Pomodoro timer** — focus sessions with notifications
- **Gmail integration** — read, send, reply, search emails from Discord
- **Daily recurring tasks** — routines with streak tracking
- **Rate limiting** — configurable per-user limits

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yolajop) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
