## paper-trading-bot-2026

> You are building a fully functional Polymarket paper trading bot with a web dashboard.

# Polymarket Paper Trading Bot — Claude Code Instructions

You are building a fully functional Polymarket paper trading bot with a web dashboard.
This is a paper trading system — NO real money, NO crypto wallet integration.
Build this incrementally in phases. Complete each phase fully before moving to the next.

---

## Project Structure to Build

```
polymarket-paper/
├── CLAUDE.md
├── config.yaml
├── requirements.txt
├── main.py
├── src/
│   ├── __init__.py
│   ├── market/
│   │   ├── __init__.py
│   │   └── fetcher.py
│   ├── strategy/
│   │   ├── __init__.py
│   │   ├── analyzer.py
│   │   └── scorer.py
│   ├── news/
│   │   ├── __init__.py
│   │   └── feed.py
│   ├── risk/
│   │   ├── __init__.py
│   │   └── limits.py
│   └── logger.py
├── dashboard/
│   ├── app.py
│   ├── static/
│   │   ├── style.css
│   │   └── script.js
│   └── templates/
│       └── index.html
├── data/
│   └── trades.csv        # auto-created on first run
└── tests/
    ├── test_fetcher.py
    └── test_analyzer.py
```

---

## Tech Stack

- **Python 3.11+**
- **anthropic** — Claude API for market analysis
- **requests** — Polymarket API calls + news feeds
- **pandas** — trade logging and P&L tracking
- **flask** — web dashboard server
- **apscheduler** — scheduled runs every hour
- **pyyaml** — config management
- **pytest** — tests

---

## Phase 1 — Project Skeleton & Config

Create all folders and empty `__init__.py` files.

Create `requirements.txt`:
```
anthropic>=0.25.0
requests>=2.31.0
pandas>=2.0.0
flask>=3.0.0
apscheduler>=3.10.0
pyyaml>=6.0
pytest>=8.0.0
```

Create `config.yaml`:
```yaml
trading:
  starting_balance: 1000.0         # fake USD to start with
  max_bet_size_usd: 50.0           # max per trade
  min_confidence_gap: 0.08         # only bet if Claude disagrees by 8%+ with market
  min_market_volume: 10000         # skip illiquid markets
  run_interval_minutes: 60         # how often to run
  mode: paper                      # always paper for now

risk:
  max_open_positions: 5            # max simultaneous bets
  max_daily_loss_usd: 100.0        # stop trading if we lose this much in a day
  min_days_to_resolution: 1        # skip markets resolving too soon
  max_days_to_resolution: 90       # skip markets too far out

claude:
  model: claude-opus-4-5-20251001
  max_tokens: 1000
  analysis_temperature: 0.2        # low temp for consistent analysis

news:
  use_rss: true
  feeds:
    - https://feeds.reuters.com/reuters/topNews
    - https://rss.nytimes.com/services/xml/rss/nyt/HomePage.xml
    - https://feeds.bbci.co.uk/news/rss.xml

dashboard:
  host: 127.0.0.1
  port: 5050
  auto_refresh_seconds: 30
```

---

## Phase 2 — Market Fetcher

**File: `src/market/fetcher.py`**

Use the Polymarket CLOB API (no auth required for reads).

Implement these functions:

```python
def get_active_markets(min_volume: float) -> list[dict]:
    """
    Fetch active markets from Polymarket.
    Endpoint: GET https://clob.polymarket.com/markets
    Filter by:
    - active == True
    - volume >= min_volume
    - not yet resolved
    - has YES/NO structure (binary markets only)
    Return list of dicts with keys:
      id, question, yes_price, no_price, volume, end_date, description
    """

def get_market_by_id(market_id: str) -> dict:
    """Fetch a single market's current state."""

def format_market_for_prompt(market: dict) -> str:
    """
    Format market data into a clean string for Claude.
    Include: question, current YES price (= implied probability),
    volume, days until resolution.
    """
```

Handle API errors gracefully — if Polymarket is down, log the error and skip this run. Never crash.

---

## Phase 3 — News Feed

**File: `src/news/feed.py`**

```python
def get_recent_headlines(max_items: int = 20) -> list[str]:
    """
    Fetch recent headlines from RSS feeds in config.
    Parse with xml.etree.ElementTree (stdlib only, no extra deps).
    Return list of headline strings from last 24 hours.
    Deduplicate similar headlines.
    If a feed fails, skip it silently and try the next one.
    """

def filter_headlines_for_market(headlines: list[str], market_question: str) -> list[str]:
    """
    Simple keyword matching — return headlines that seem relevant
    to the market question. Max 5 headlines.
    """
```

---

## Phase 4 — Claude Analyzer

**File: `src/strategy/analyzer.py`**

This is the brain of the bot. Use the Anthropic SDK.

```python
def analyze_market(market: dict, relevant_news: list[str], api_key: str) -> dict:
    """
    Send market data + news to Claude.
    Ask Claude to estimate the TRUE probability of YES resolution.
    
    Return dict with keys:
      claude_yes_probability: float (0.0 to 1.0)
      market_yes_price: float (current market implied probability)
      confidence: float (0.0 to 1.0, how confident Claude is)
      reasoning: str (brief explanation)
      recommended_side: str ("YES", "NO", or "SKIP")
      edge: float (abs difference between Claude's prob and market price)
    """
```

Use this system prompt for Claude:
```
You are a prediction market analyst. Your job is to estimate the true probability 
of an event occurring, then compare it to the current market price.

You are NOT allowed to say "I don't know" or refuse to give a number.
You must always output a probability estimate even under uncertainty.
Be calibrated — if you're genuinely uncertain, reflect that with a probability 
near the market price and low confidence score.

Always respond in valid JSON only. No markdown, no explanation outside JSON.
```

Use this user prompt template:
```
Market Question: {question}
Current Market YES Price: {yes_price} (this means the market thinks there's a {yes_price*100:.1f}% chance of YES)
Market Volume: ${volume:,.0f}
Days Until Resolution: {days_remaining}

Recent Relevant News:
{news_items}

Analyze this market and respond with JSON:
{{
  "claude_yes_probability": <your estimated probability of YES, 0.0-1.0>,
  "confidence": <how confident you are, 0.0-1.0>,
  "reasoning": "<2-3 sentence explanation>",
  "recommended_side": "<YES or NO or SKIP>",
  "key_factors": ["<factor 1>", "<factor 2>", "<factor 3>"]
}}
```

Parse the JSON response. If parsing fails, return a SKIP result with the error logged.

---

## Phase 5 — Scorer & Risk Limits

**File: `src/strategy/scorer.py`**

```python
def should_place_bet(analysis: dict, config: dict, current_positions: int, daily_loss: float) -> tuple[bool, str]:
    """
    Final gate before logging a paper trade.
    Returns (True, reason) or (False, reason).
    
    Rules:
    - Skip if recommended_side == "SKIP"
    - Skip if edge < min_confidence_gap
    - Skip if confidence < 0.6
    - Skip if current_positions >= max_open_positions
    - Skip if daily_loss >= max_daily_loss_usd
    """

def calculate_bet_size(analysis: dict, config: dict, current_balance: float) -> float:
    """
    Kelly-inspired sizing (simplified):
    - Base bet = max_bet_size_usd
    - Scale by confidence: bet_size = base * confidence
    - Never exceed 5% of current_balance
    - Round to 2 decimal places
    """
```

**File: `src/risk/limits.py`**

```python
def get_daily_loss(trades_df: pd.DataFrame) -> float:
    """Calculate today's realized + unrealized loss from trades CSV."""

def get_open_positions(trades_df: pd.DataFrame) -> list[dict]:
    """Return list of trades that haven't resolved yet."""

def check_kill_switch(daily_loss: float, max_daily_loss: float) -> bool:
    """Return True if we should stop trading today."""
```

---

## Phase 6 — Trade Logger

**File: `src/logger.py`**

```python
def log_trade(trade: dict) -> None:
    """
    Append a paper trade to data/trades.csv.
    Create the file with headers if it doesn't exist.
    
    CSV columns:
    timestamp, market_id, question, side, bet_size_usd,
    entry_price, claude_probability, market_probability,
    edge, confidence, reasoning, status, exit_price,
    pnl_usd, resolved_at
    
    New trades start with status="OPEN", exit_price=None, pnl_usd=None
    """

def update_resolved_trades(trades_df: pd.DataFrame) -> pd.DataFrame:
    """
    For each OPEN trade, check if the market has resolved.
    If resolved: update status, exit_price, pnl_usd, resolved_at.
    Return updated dataframe and save to CSV.
    """

def get_portfolio_summary(trades_df: pd.DataFrame, starting_balance: float) -> dict:
    """
    Return dict with:
      current_balance, total_pnl, win_rate, total_trades,
      open_positions, best_trade, worst_trade, avg_edge,
      daily_pnl (last 7 days as list for chart)
    """
```

---

## Phase 7 — Main Loop

**File: `main.py`**

```python
"""
Main entry point. 

Usage:
  python main.py          # start bot + dashboard
  python main.py --once   # run analysis once and exit
  python main.py --dash   # dashboard only, no trading
"""
```

The main loop should:
1. Load config.yaml
2. Load existing trades from CSV (or create empty)
3. Check kill switch — if daily loss exceeded, skip trading
4. Fetch active markets filtered by volume
5. For each market (max 10 per run):
   a. Get relevant news headlines
   b. Run Claude analysis
   c. Score the opportunity
   d. Log paper trade if criteria met
6. Update resolved trades
7. Print summary to console with emoji indicators
8. Schedule next run via APScheduler

Console output format:
```
🔍 Analyzing 8 markets...
  ✅ "Will X happen?" — Claude: 73% | Market: 61% | Edge: 12% → BET YES $42.00
  ⏭️  "Will Y happen?" — Claude: 52% | Market: 55% | Edge: 3% → SKIP (low edge)
  ⏭️  "Will Z happen?" — Claude: 48% | Market: 46% | Edge: 2% → SKIP (low confidence)

📊 Portfolio: $1,087.50 (+8.75%) | Open: 3 | W/L: 7/3
```

---

## Phase 8 — Web Dashboard

**File: `dashboard/app.py`**

Flask app with these routes:
- `GET /` — main dashboard page
- `GET /api/summary` — JSON portfolio summary
- `GET /api/trades` — JSON list of all trades
- `GET /api/chart` — JSON daily P&L data for chart

**File: `dashboard/templates/index.html`**

Build a dark-themed trading dashboard. Design requirements:
- Dark background (#0a0a0f or similar deep dark)
- Monospace font for numbers (Fira Code, JetBrains Mono, or similar)
- Green for profits, red for losses (classic trading terminal aesthetic)
- Auto-refreshes every 30 seconds via JavaScript fetch

Dashboard sections:
1. **Header bar** — "POLYMARKET PAPER TRADER" + current time + bot status (RUNNING/STOPPED)
2. **Stats row** — 4 big cards: Current Balance | Total P&L | Win Rate | Open Positions
3. **P&L Chart** — last 7 days bar chart (use Chart.js from CDN)
4. **Live Trade Feed** — scrollable table of all trades with color-coded P&L
   - Columns: Time | Question (truncated) | Side | Bet | Entry | Claude% | Market% | Edge | Status | P&L
5. **Bot Log** — last 10 console messages (updated via polling)

Make it look like a real trading terminal. Green glow effects on positive numbers. 
The design should feel like Bloomberg Terminal meets crypto dashboard.

---

## Phase 9 — Tests

**File: `tests/test_fetcher.py`**
- Test that `get_active_markets` returns a list
- Test that markets have required keys
- Test graceful handling of API timeout (mock the request)

**File: `tests/test_analyzer.py`**
- Test JSON parsing of Claude response
- Test fallback behavior when JSON is malformed
- Test that edge calculation is correct

---

## Phase 10 — README

Create `README.md` with:
1. What this project does
2. Setup instructions (python -m venv, pip install, set ANTHROPIC_API_KEY)
3. How to run (`python main.py`)
4. How to view dashboard (open http://127.0.0.1:5050)
5. How paper trading works
6. Config options explained
7. How to read the trade log

---

## Important Rules

1. **Never make real trades.** All execution is simulated. No wallet code.
2. **Always read config.yaml** — never hardcode values.
3. **ANTHROPIC_API_KEY comes from environment variable** — never hardcode it.
4. **Graceful degradation** — if any API fails (Polymarket, news, Claude), log the error and continue. Never crash.
5. **Idempotent logging** — never log the same market twice in the same run.
6. **The dashboard must work even when the bot is not running** — it just reads the CSV.
7. **All money values are fake USD** — make this obvious in the UI.

---

## How to Run After Build

```bash
# Setup
python -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate
pip install -r requirements.txt

# Set your API key
export ANTHROPIC_API_KEY=your_key_here

# Run the bot + dashboard
python main.py

# Open dashboard
open http://127.0.0.1:5050
```

---

Start with Phase 1. Confirm completion of each phase before proceeding to the next.
Ask me if anything is ambiguous. Let's build this.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Buddy-Lu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Buddy-Lu)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
