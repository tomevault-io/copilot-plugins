## changelog-watch-telegram-bot

> - Всегда писать в чате на русском.

# AGENTS

- Всегда писать в чате на русском.
- This repo is a single-file Python bot (`bot.py`) with `products.yaml`-driven config, no monorepo boundaries, no additional packages/modules to discover.
- Run commands are the only reliable entrypoints: `python3`/`python` with flags (`--once`, `--dry-run`, `--config`, `--db`) defined in `bot.py`.
- Setup is straightforward but strict: create/activate `.venv`, `pip install -r requirements.txt`, then `cp .env.example .env`, `cp admin-routing.example.yaml admin-routing.yaml`, fill `TELEGRAM_BOT_TOKEN`, and put target chats into `admin-routing.yaml`.
- Dry-run is safe for verification: `python bot.py --once --dry-run` parses and logs candidate posts without Telegram API calls.
- Regular one-shot run is `python bot.py --once`; continuous mode `python bot.py` starts scheduler, runs one immediate check, then sleeps on 30-minute default interval.
- `TELEGRAM_BOT_TOKEN` is only required when `--dry-run` is not set; with dry-run it can be absent. `TELEGRAM_CHAT_ID` is legacy and ignored by routing mode.
- `CONFIG_PATH` and `DB_PATH` env vars override the CLI defaults (and can still be overridden by `--config`/`--db`).
- Config format is strict: top-level `poll_minutes` and a `sources` list in YAML; `load_config` throws if `sources` is missing or not a list.
- `source.id` is the dedupe key in SQLite and controls re-post behavior; changing it will look like fresh items.
- `enabled: false` skips a source without editing code; `post_on_first_run` defaults to false (seed existing items silently), otherwise first run posts up to `first_run_limit` (default 1).
- Supported `type` values are `html_changelog`, `markdown_changelog`, `github_releases`; markdown headings expected like `## [1.2.3] - 2026-01-01` and GitHub URL must be parseable to `owner/repo`.
- Database state is in `DB_PATH` (default `data/posted.sqlite3`), created automatically with `posted_items` and `source_state`; deleting the file resets memory and causes immediate re-send behavior on next first-run handling.
- No test/lint/typecheck/build scripts or CI files are present in this repo; avoid guessing default language-tooling commands and stick to README + docs.
- `systemd/changelog-watch-bot.service` is environment-specific (`User`, absolute paths); edit all host-specific values before `systemctl enable/start`.

---
> Source: [romanilyin/changelog-watch-telegram-bot](https://github.com/romanilyin/changelog-watch-telegram-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
