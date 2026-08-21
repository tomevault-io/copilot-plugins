## slack-weekly-trivia

> Use caveman skill for all work on this project. Talk like caveman. Be brief.

# slack-daily-trivia

## Comm

Use caveman skill for all work on this project. Talk like caveman. Be brief.

## Rules

- NEVER commit or push code. Present changes, let user review and commit.

# slack-daily-trivia

Slack app that posts a daily trivia question to configured channels. Users
answer privately via ephemeral buttons. Stats and leaderboard track results.

## Architecture

```
src/
├── app.py              # Slack wiring — events, slash commands, actions
├── trivia_api.py       # HTTP adapter for the-trivia-api.com
├── trivia_service.py   # Game logic — questions, answers, stats, blocks
├── stats_store.py      # SQLite persistence — answers, leaderboard, configs
└── scheduler.py        # APScheduler — daily trivia + weekly leaderboard
```

**`app.py`** is the integration layer. It knows Slack (Bolt, Socket Mode) but
delegates all logic to `TriviaService`. Business logic lives in
`trivia_service.py`. Data lives in `stats_store.py`.

## Slash commands

| Command | Scope | Description |
|---------|-------|-------------|
| `/post-trivia` | Where bot is member | Post daily question to this channel (public) |
| `/set-trivia-channel` | Where bot is member | Set this channel for scheduled daily posts |
| `/set-trivia-time HH:MM` | Anywhere | Set post time (ET, e.g. `12:00`). 24-hour. |
| `/stats` | Anywhere | Personal stats — accuracy, by category, by difficulty |
| `/leaderboard` | Anywhere | Top 3 by accuracy. Dropdown filters for category/difficulty. |

## Interactive components

| Action ID | Trigger | Result |
|-----------|---------|--------|
| `start_answer` | "Answer" button in public post | Opens ephemeral A/B/C/D choices |
| `posted_trivia_answer_[a-e]` | Answer button in posted flow | Replaces ephemeral with correct/wrong |
| `leaderboard_category` | Dropdown change | Re-filters leaderboard |
| `leaderboard_difficulty` | Dropdown change | Re-filters leaderboard |

## Style

- **Type hints** — use Python type hints on all function signatures (params + return).

## Key design decisions

- **Socket Mode** — no public URL needed. App connects outbound to Slack.
- **Ephemeral answers** — `chat_postEphemeral` ensures answers are private per user.
- **SQLite with no WAL** — compatible with DB Browser for SQLite for direct editing.
- **`check_same_thread=False`** — SQLite connection shared across Bolt event threads.
- **APScheduler** — handles daily trivia (user-configured time, Mon-Fri ET) and
  weekly leaderboard (Friday noon Eastern). Re-syncs config from DB every 5 minutes.
- **`StatsStore` is the single writer to SQLite** — no concurrent write issues.

## Database schema (SQLite)

### `answers` — every answer ever submitted
```
user_id | question_id | category | difficulty | correct | selected | timestamp
PRIMARY KEY: (user_id, question_id)
```

### `asked_questions` — dedup per channel
```
channel_id | question_id
PRIMARY KEY: (channel_id, question_id)
```

### `workspace_configs` — per-workspace settings
```
team_id | channel_id | post_time (default '09:00')
PRIMARY KEY: team_id
```

## Secrets (.env)

| Variable | Required | Purpose |
|----------|----------|---------|
| `SLACK_BOT_TOKEN` | Yes | `xoxb-...` Bot OAuth token |
| `SLACK_APP_TOKEN` | Yes | `xapp-...` App-level token (Socket Mode) |
| `SLACK_SIGNING_SECRET` | Yes | Request signing verification |
| `TRIVIA_DB_DIR` | No | Override SQLite path (default: `src/data/`) |

## Deployment

See `DEPLOY.md` for full Fly.io guide. Quick steps:

```bash
fly apps create slack-daily-trivia
fly volumes create trivia_data --region iad --size 1
fly secrets set SLACK_BOT_TOKEN=... SLACK_APP_TOKEN=... SLACK_SIGNING_SECRET=...
fly deploy
```

## Slack app settings checklist

- **Socket Mode**: ON
- **App Token**: `connections:write`
- **Bot Token Scopes**: `app_mentions:read`, `chat:write`, `commands`
- **Event Subscriptions**: `app_mention` (Socket Mode handles delivery)
- **Slash Commands**: `/post-trivia`, `/stats`, `/leaderboard`,
  `/set-trivia-channel`, `/set-trivia-time`
- **Interactivity**: ON (required for buttons)
- **Install to workspace**

## Development

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
# Edit .env with real tokens
python src/app.py
```

SQLite DB created automatically at `src/data/trivia_stats.db` on first answer.
DB file is in `.gitignore`.

---
> Source: [wmaethner/slack-weekly-trivia](https://github.com/wmaethner/slack-weekly-trivia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
