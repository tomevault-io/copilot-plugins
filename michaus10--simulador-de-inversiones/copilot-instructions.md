## simulador-de-inversiones

> This file provides guidance for AI assistants (Claude Code and similar tools) working on this repository.

# CLAUDE.md — Simulador de Inversiones

This file provides guidance for AI assistants (Claude Code and similar tools) working on this repository.

---

## Project Overview

**simulador-de-inversiones** is a Claude-powered NYSE stock analysis and portfolio simulation agent with a Streamlit web UI. Users can chat with the agent to get real-time stock quotes, technical analysis, backtest portfolios, and compare NYSE stocks.

- **Repository**: MICHAUS10/simulador-de-inversiones
- **Stack**: Python 3.11+, Streamlit, Anthropic SDK (`claude-opus-4-6`), yfinance
- **Data source**: Yahoo Finance via `yfinance` (free, no API key required)
- **AI Model**: `claude-opus-4-6` with adaptive thinking and tool use

---

## Directory Structure

```
simulador-de-inversiones/
├── app.py                   # Streamlit application entry point
├── agent/
│   ├── __init__.py
│   ├── tools.py             # Tool implementations (yfinance data + calculations)
│   ├── schemas.py           # Tool JSON schemas for the Claude API
│   └── prompts.py           # System prompt for the agent
├── requirements.txt         # Python dependencies
├── .env.example             # Environment variable template
└── CLAUDE.md                # This file
```

---

## Setup & Running

### 1. Install dependencies

```bash
pip install -r requirements.txt
```

### 2. Set API key

```bash
cp .env.example .env
# Edit .env and set ANTHROPIC_API_KEY=your_key_here
```

### 3. Run the app

```bash
streamlit run app.py
```

The app opens at `http://localhost:8501`.

---

## Architecture

### Agent Loop (`app.py :: stream_agent`)

The agent follows a standard tool-use loop:

1. User sends a message via Streamlit chat input or sidebar button
2. Claude is called with `claude-opus-4-6`, adaptive thinking, and the tool schemas
3. If Claude requests tool use (`stop_reason == "tool_use"`), tools are executed locally
4. Tool results are fed back and Claude continues — loop repeats until `stop_reason == "end_turn"`
5. Text deltas stream to the UI in real time via the `stream` context manager

### Tools (`agent/tools.py`)

| Tool | Description |
|------|-------------|
| `get_stock_quote` | Current price, P/E, market cap, sector |
| `get_historical_prices` | OHLCV summary and recent closes |
| `calculate_technical_indicators` | RSI, MACD, SMA/EMA, Bollinger Bands + signals |
| `simulate_portfolio` | Backtest allocation vs S&P 500 with full risk metrics |
| `compare_stocks` | Normalized performance, correlation, fundamentals |
| `get_market_overview` | US indices, VIX, sector ETF performance |
| `search_stock` | Find NYSE ticker by company name |

All tools return plain `dict` objects (JSON-serializable). The `chart_data` key in tool results triggers chart rendering in the Streamlit UI.

### Chart Rendering (`app.py :: render_chart`)

Tools that produce visual output include a `chart_data` dict with a `type` field:
- `"portfolio_performance"` → line chart of portfolio vs S&P 500
- `"stock_comparison"` → normalized multi-stock performance chart
- `"technical"` → price chart with SMA and Bollinger Band overlays

Charts are rendered using Plotly via `st.plotly_chart`.

### Session State

| Key | Type | Purpose |
|-----|------|---------|
| `api_messages` | `list[dict]` | Full Claude conversation history (includes tool use/result blocks) |
| `display_messages` | `list[dict]` | Simplified history for UI rendering (text + chart data) |
| `portfolio_df` | `list[dict]` | Current portfolio builder state |
| `pending_message` | `str` | Message queued by sidebar button (processed on next render) |

---

## Development Conventions

### Branch & Commit Style

- Feature branches: `feature/<description>`
- Bug fixes: `fix/<description>`
- AI-assisted: `claude/<task-description>`
- Commit messages: imperative, prefixed (`feat:`, `fix:`, `refactor:`, `docs:`, `test:`)

### Adding a New Tool

1. **Implement** the function in `agent/tools.py` — return a plain `dict`
2. **Register** it in `execute_tool()` dispatch table at the bottom of `tools.py`
3. **Add schema** to `TOOL_SCHEMAS` list in `agent/schemas.py`
4. **Update** system prompt in `agent/prompts.py` if the tool needs behavioral guidance
5. **Optionally** add chart rendering logic to `render_chart()` in `app.py`

### Financial Domain Rules

- All monetary values in USD
- Be explicit about rate periods (annual, monthly, daily) in comments and docstrings
- Annualized volatility = daily_std × √252
- Sharpe ratio uses 5% annualized risk-free rate
- Portfolio weights must sum to ≈ 1.0 (normalized automatically in `simulate_portfolio`)
- Never use floating-point equality checks for weight validation — use `abs(sum - 1.0) > 0.05`
- Always note that analysis is educational, not financial advice

### Error Handling

- All tool functions catch exceptions and return `{"error": "..."}` — never raise
- The agent reads error messages from tool results and responds accordingly
- Streamlit UI catches `anthropic.AuthenticationError`, `RateLimitError`, and `APIStatusError`

---

## Dependencies

| Package | Purpose |
|---------|---------|
| `anthropic` | Claude API client |
| `streamlit` | Web UI |
| `yfinance` | Yahoo Finance market data |
| `pandas` | Data manipulation |
| `numpy` | Numerical calculations |
| `plotly` | Interactive charts |
| `python-dotenv` | `.env` file loading |

---

## Updating This File

Update this file when:
- New tools are added or existing tools change significantly
- The agent model or API parameters change
- New Python dependencies are introduced
- Architecture patterns change

---
> Source: [MICHAUS10/simulador-de-inversiones](https://github.com/MICHAUS10/simulador-de-inversiones) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
