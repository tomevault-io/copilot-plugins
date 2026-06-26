## crypto-ai-research

> Open-source AI research lab for crypto markets. Multi-language, built on Solana.

# Orin.LAB — Project Context

## What is this
Open-source AI research lab for crypto markets. Multi-language, built on Solana.
GitHub: https://github.com/nujar00t/Orin.LAB

## Stack
- **Python** — main agents, Telegram bot, TA engine, utilities
- **TypeScript** — Auto Poster (Twitter/X), Solana SDK
- **Go** — Signal CLI
- **Rust** — Core SDK (high-performance computation)

## Key modules
| Path | Role |
|------|------|
| `bot/` | Telegram bot (python-telegram-bot async) |
| `agents/signal_engine.py` | BUY/SELL/HOLD signal generator |
| `agents/technical_analysis.py` | Pure-Python TA — RSI, MACD, BB, EMA, ATR |
| `agents/whale_tracker.py` | Solana whale wallet monitor |
| `agents/alert_manager.py` | Async threshold alert dispatcher |
| `agents/signal_history.py` | JSON signal log |
| `agents/post_writer.py` | Natural post generator for Twitter/X |
| `agents/dashboard.py` | Rich live terminal dashboard |
| `utils/ai_client.py` | AI provider adapter (Anthropic / DeepInfra / OpenAI-compatible) |
| `utils/rate_limiter.py` | Sliding-window rate limiter |
| `utils/cache.py` | In-memory TTL cache |
| `utils/config.py` | Typed env-var config loader |
| `utils/helpers.py` | Formatting, parsing utilities |
| `orinlab/cli.py` | `orinlab` CLI entry point (pip installable) |
| `cli_setup.py` | Interactive setup wizard |

## AI provider
- Switch via `AI_PROVIDER` env var: `anthropic` | `deepinfra` | `openai`
- All AI calls go through `utils/ai_client.chat()` — never call SDK directly
- Rate limiting via `utils/rate_limiter.anthropic_limiter`

## Price data
- CoinGecko free API (no key needed) — replaces Jupiter which requires auth
- OHLCV via `agents/technical_analysis.fetch_ohlcv()`

## Install
```bash
pip install orinlab
orinlab setup   # wizard
orinlab bot     # start
```

## Config location
- Installed users: `~/.orinlab/.env`
- Dev: `.env` in project root

## Bot commands
`/start`, `/signal`, `/ta`, `/analyze`, `/post`, `/history`, `/help`
+ photo handler for chart analysis

## Tests
```bash
pytest tests/ -v
py -m pytest tests/ -v   # Windows
```
35+ unit tests — all pure math/logic, no API calls in tests

## Commit style
```
feat(agents): add X
fix(bot): fix Y
docs: update README
chore: update deps
```
Always add: `Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>`

## Branch strategy
- `main` — stable
- `develop` — integration
- `feature/*`, `hotfix/*`, `chore/*`

## Things to remember
- Jangan call Anthropic SDK langsung — pakai `utils/ai_client.chat()`
- Jangan pakai Jupiter API — sudah deprecated, pakai CoinGecko
- Windows encoding issue: hindari Unicode box-drawing chars di terminal output
- `py` bukan `python` di Windows ini

---
> Source: [cryptosignalorg/crypto-ai-research](https://github.com/cryptosignalorg/crypto-ai-research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
