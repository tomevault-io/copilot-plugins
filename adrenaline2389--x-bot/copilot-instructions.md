## x-bot

> This repository is a GitHub Actions driven Twitter/X bot. It is organized as one bot project with isolated feature modules, not as multiple independent bot projects.

# AGENTS.md

## Project Overview

This repository is a GitHub Actions driven Twitter/X bot. It is organized as one bot project with isolated feature modules, not as multiple independent bot projects.

Current features:

- `price-report`: posts crypto market prices. Default symbol is `bitcoin`, configurable through `PRICE_SYMBOLS`.
- `whale-alert`: scans recent Bitcoin blocks and posts transactions whose total output is at or above `WHALE_BTC_THRESHOLD`. Default threshold is `1000` BTC.

## Structure

- `src/x_bot/main.py`: command dispatcher for runnable features.
- `src/x_bot/config.py`: reads and validates environment variables.
- `src/x_bot/twitter_client.py`: shared Twitter/X posting logic.
- `src/x_bot/features/`: one file per bot feature.
- `src/x_bot/services/`: HTTP clients for external APIs.
- `.github/workflows/`: scheduled GitHub Actions jobs.

Use these commands:

```powershell
python -m x_bot price-report
python -m x_bot whale-alert
```

## Configuration

Required Twitter/X secrets:

- `API_KEY`
- `API_SECRET`
- `ACCESS_TOKEN`
- `ACCESS_SECRET`

Optional settings:

- `PRICE_SYMBOLS`: comma-separated CoinGecko ids, default `bitcoin`.
- `PRICE_VS_CURRENCY`: default `usd`.
- `WHALE_BTC_THRESHOLD`: default `1000`.
- `WHALE_SCAN_BLOCKS`: default `6`.
- `WHALE_CACHE_PATH`: default `.cache/posted_whale_txs.json`.
- `X_BOT_DRY_RUN`: set to `true` for local testing without posting.

## Development Notes

- Do not add hardcoded API keys, tokens, or secrets.
- Keep features isolated under `src/x_bot/features/`.
- Put reusable external API calls under `src/x_bot/services/`.
- The bot is designed for GitHub Actions runs, not a local long-running scheduler.
- Prefer small, focused changes and keep README/workflow docs aligned with behavior changes.

## Deletion Rules

Do not batch-delete files or directories, including recursive deletion.

Do not use:

- `del /s`
- `rd /s`
- `rmdir /s`
- `Remove-Item -Recurse`
- `rm -rf`

Do not implement batch deletion through loops, wildcards, or scripts.

When a file must be deleted, delete exactly one explicit file path at a time:

```powershell
Remove-Item "C:\path\to\file.txt"
```

If bulk deletion is needed, stop and ask the user to delete those files manually.

---
> Source: [adrenaline2389/X_bot](https://github.com/adrenaline2389/X_bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
