## finance-dashboard

> This file contains specific rules and conventions for this project that should be followed by AI assistants.

# Project Rules & Customizations

This file contains specific rules and conventions for this project that should be followed by AI assistants.

## File & Directory Conventions

- **Temporary Files & Logs**: All test scripts, logs, and temporary outputs should be stored in the `temp/` directory within the project root. Operations within this directory are considered safe and do not require user approval.

## Automation & AI Rules

- **Safe Operations**: Reading, writing, and executing files within `temp/` and `tools/` is permitted without explicit user confirmation (`SafeToAutoRun: true`).
- **Communication Language**: Always respond in Chinese (繁體中文) to the user's requests unless specified otherwise.
- **Git Commit Messages**: All git commit messages should be written in English.
- **Development Workflow**: Use the `dev` branch for all active development. Sync to `main` only after verification.

## Architecture & Data Rules

- **Market Data Color Convention**: For price changes, use **Red** for increases (上漲) and **Green** for decreases (下跌) to align with Taiwan/Asian market conventions.
- **Market Quotes Stability**: For real-time ticker data, prefer `yfinance`'s **`fast_info`** over full history downloads (`yf.download`) to ensure stable baseline calculations and avoid price volatility between updates.
- **Symbol Formatting**: Numerical symbols for Taiwan trackers (e.g., 0050) should always be suffixed with `.TW` or `.TWO` when used in backend logic.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thomas-chu123)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/thomas-chu123)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
