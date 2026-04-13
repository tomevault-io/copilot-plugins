## connection-machine

> This file gives coding agents working in this repository the minimum context they need to make safe changes without reverse-engineering the whole project first.

# AGENTS.md

This file gives coding agents working in this repository the minimum context they need to make safe changes without reverse-engineering the whole project first.

## Project Overview

Connection Machine is a LinkedIn automation worker built around a persistent browser session, a SQLite-backed task queue, and OpenRouter-powered text generation.

Today the codebase supports three practical automation flows:

- Sending LinkedIn connection requests from queued database tasks
- Creating LinkedIn posts from queued database tasks
- Generating feed comments autonomously when the account is idle and an OpenRouter key is configured

The browser layer is built on Patchright plus Playwright, with human-like interaction helpers to reduce obvious automation patterns.

## High-Level Runtime

```text
src/main.py
  -> load .env
  -> launch persistent Chromium context
  -> restore LinkedIn session or log in
  -> initialize database
  -> start TaskDispatcher loop

TaskDispatcher
  -> polls pending DB tasks
  -> enforces spacing and per-day limits
  -> runs matching task handler
  -> records success / failure / cooldown state
  -> opportunistically runs autonomous feed comments when no DB task is runnable
```

## Main Components

- `src/main.py`: entry point, browser startup, LinkedIn auth recovery, debug CLI modes, graceful shutdown
- `src/dispatcher.py`: polling loop, task selection, spacing windows, zombie cleanup, autonomous comment scheduling
- `src/db.py`: SQLAlchemy engine, `linkedin_tasks` table, enums, DB initialization
- `src/tasks/invite.py`: profile visit, connect-button discovery, optional AI-generated note, invitation confirmation
- `src/tasks/post.py`: opens the supplied LinkedIn composer URL and submits a post
- `src/tasks/comment.py`: scans the feed for safe posts, asks the LLM for a comment, optionally publishes it, and stores local comment history
- `src/llm.py`: OpenRouter calls for connection notes, feed comments, and connect-action detection
- `src/human_actions.py`: randomized sleeps, hover behavior, typing cadence, and click simulation
- `src/notifications.py`: optional outbound notifications

## Task Model

Database-backed task types in `src/db.py`:

- `send_invite`
- `create_post`
- `comment_feed_post`

Current dispatcher behavior:

- `send_invite` and `create_post` are handled as normal queued DB tasks
- `comment_feed_post` DB rows are treated as legacy and cleaned up
- feed comments now run autonomously from `TaskDispatcher.maybe_run_autonomous_comment()`

Task statuses:

- `pending`
- `processing`
- `completed`
- `failed`

## Operational Behavior

### Dispatcher cadence

- Poll interval is 10 seconds
- Stuck `processing` tasks are reset to `pending` on startup
- stale pending `create_post` tasks older than one hour are deleted
- legacy DB-backed feed-comment tasks are deleted

### Rate limits and spacing

The dispatcher uses randomized spacing rather than fixed cron-like scheduling.

- invites: 10 per rolling 24 hours
- posts: 50 per rolling 24 hours
- autonomous feed comments: 12 per rolling 24 hours

Spacing between actions is derived from the daily cap, then randomized in the `0.7x` to `1.3x` range.

Additional cooldowns are applied for known LinkedIn skip reasons:

- invite weekly limit: 24 hours
- invite withdrawal cooldown: 6 hours
- no safe commentable feed posts: 30 minutes
- autonomous comment hard failure: 30 minutes

## Persistence and Local State

- browser session data: `data/connection-machine-chrome`
- SQLite database: whatever `DATABASE_URL` points to
- autonomous feed-comment history: `data/feed_comment_history.json`

The browser runs in a persistent context, so successful login state is meant to survive restarts.

## Environment Variables

The application loads `.env` automatically via `python-dotenv`. Copy `.env.example` to `.env` and adjust values.

Required for a normal worker run:

- `LINKEDIN_USERNAME`: LinkedIn login email or username
- `LINKEDIN_PASSWORD`: LinkedIn password
- `DATABASE_URL`: SQLAlchemy database URL. For local development use `sqlite:///data/connection_machine.db`

Optional but important:

- `OPENROUTER_API_KEY`: enables AI-generated invite notes, connect-action detection, and autonomous feed comments
- `HEADLESS`: browser visibility toggle. Defaults to `true`
- `SOCKS_PROXY`: optional proxy passed through Chromium as `--proxy-server=...`
- `TELEGRAM_NOTIFICATIONS_URL`: notification endpoint
- `TELEGRAM_CHAT_ID`: notification target
- `TELEGRAM_API_KEY`: bearer token for the notification endpoint

Behavior notes:

- invite tasks still work without `OPENROUTER_API_KEY`, but personalized notes and some selector-detection behavior degrade
- autonomous feed comments require `OPENROUTER_API_KEY`
- notifications are silently skipped unless all three Telegram-related variables are set

## Local Development

```bash
# install dependencies
uv sync

# run the main worker
uv run python src/main.py

# run one invite directly without using the DB queue
uv run python src/main.py --debug-invite "https://linkedin.com/in/profile-url"

# run one invite directly and skip the AI-generated note
uv run python src/main.py --debug-invite "https://linkedin.com/in/profile-url" --no-message

# generate one feed comment without submitting it
uv run python src/main.py --debug-feed-comment

# generate one feed comment and actually submit it
uv run python src/main.py --debug-feed-comment --submit-comment

# populate DB tasks from CSV
uv run python utils/populate_db.py
```

## Docker

The container expects `.env` through `docker-compose.yaml` and mounts `./data` into `/app/data`.

```bash
docker-compose up -d
docker-compose logs -f connection-machine
```

## Agent Guidance

When changing this codebase:

- preserve persistent-session behavior unless the task explicitly requires auth changes
- keep automation conservative; the dispatcher intentionally spaces work out
- prefer documenting real behavior from code, not planned behavior
- do not reintroduce DB-backed feed comments unless that is the explicit task
- treat `data/` as runtime state, not source-controlled configuration
- if you add a new env var, update both `.env.example` and this file in the same change

## Common Pitfalls

- `DATABASE_URL` must be set or imports from `src/db.py` will fail immediately
- a missing browser binary may cause Patchright to fall back to local Chrome
- LinkedIn auth can land on `/checkpoint`; that is treated as not authenticated
- autonomous comments are intentionally disabled when no OpenRouter key is present

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/artbred) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
