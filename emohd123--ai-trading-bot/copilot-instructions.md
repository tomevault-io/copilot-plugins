## ai-trading-bot

> Restart the trading bot after any code or config change so changes take effect


# Restart Bot After Changes

**After making any change** to bot code or config that affects running behavior, **restart the bot** so the changes take effect.

## When to restart

Restart after editing any of:
- `config.py` (thresholds, weights, NO_BUY_IN_DOWNTREND, etc.)
- `dashboard.py`, `main.py`
- `core/*.py` (trader, binance_client, self_healing)
- `ai/*.py` (ai_engine, ml_*, analyzer, etc.)
- `market/*.py`
- `notifications/*.py` (if it affects the running bot)

## How to restart (Windows)

1. **Stop** any running bot:
   ```powershell
   taskkill /F /IM pythonw.exe 2>$null; taskkill /F /IM python.exe 2>$null
   ```
2. **Start** the bot (dashboard) again:
   ```powershell
   Start-Process python -ArgumentList "dashboard.py" -WorkingDirectory "<project_root>" -WindowStyle Minimized
   ```
   Or from project root: `.\scripts\start_bot.bat`

Tell the user: "Restarted the bot. Dashboard: http://localhost:5000"

## Optional

If the user says "don't restart" or "I'll restart myself", skip the restart step.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/emohd123) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
