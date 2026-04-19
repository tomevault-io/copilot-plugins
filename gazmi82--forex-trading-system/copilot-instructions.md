## forex-trading-system

> A multi-agent AI forex trading system using Claude API as the intelligence layer

# CLAUDE.md — Forex Trading System Project Context
# This file is for Claude Code (VS Code) to stay aligned with all decisions,
# architecture, and progress made in the main Claude.ai chat session.
# Last updated: March 28, 2026

---

## PROJECT OVERVIEW

A multi-agent AI forex trading system using Claude API as the intelligence layer
with rule-based Python execution. Currently in 12-month demo phase.

**Owner:** Gazmir Sulcaj (gsulcaj22@gmail.com)
**Start Date:** March 10, 2026
**Demo Period:** 12 months (ends March 10, 2027)
**Live Trading:** Not before March 2027

---

## LOCKED DECISIONS — DO NOT CHANGE

These decisions were made in the main session and are final:

| Decision | Value |
|----------|-------|
| Focus pair | EUR/USD ONLY |
| Broker | OANDA (demo) → IBKR for futures later |
| Starting capital | $5,000 (when live) |
| Demo capital | $100,000 OANDA V20 demo account ✅ ACTIVE |
| OANDA Account ID | 101-001-38764497-001 |
| Agent approach | Option 1 (System Prompt) + Option 2 (RAG) combined |
| Embedding model | all-MiniLM-L6-v2 (local, FREE) |
| Vector DB | ChromaDB (local, FREE) |
| Demo mode flag | Always TRUE until March 2027 |

---

## HARD TRADING RULES — NEVER MODIFY

These rules are embedded in the system prompt and must NEVER be changed:

```
1. Max risk per trade:     1% of account equity
2. Min Risk:Reward ratio:  1:2 (2.0)
3. Daily loss limit:       2% — triggers auto-stop of all trading
4. Confidence threshold:   <65% = NEUTRAL signal always (no trade)
5. News blackout:          30 min before NFP, CPI, FOMC, ECB decisions
6. Kill Zones only:        London (3-4 AM EST) + NY (8-10 AM EST)
7. Demo mode:              TRUE (hardcoded) until month 12
```

---

## SYSTEM ARCHITECTURE

```
main.py
├── RAGPipeline (app/rag/pipeline.py)
│   ├── ChromaDB vector store (chroma_db/)
│   ├── all-MiniLM-L6-v2 embeddings (local)
│   └── 6 collections: books, research, ict, cot, journal, feedback
│
├── ForexAnalystAgent (app/analysis/agent.py)
│   ├── Option 1: System prompt (embedded trading knowledge)
│   ├── Option 2: RAG retrieval (contextual chunks per analysis)
│   └── Claude API → claude-sonnet-4-20250514
│
├── OANDAClient (app/brokers/oanda.py)
│   ├── Live EUR/USD price feed
│   ├── OHLCV candles (4H, 1H, Daily, Weekly)
│   └── Account summary + open positions (+ stop-loss extraction for stop-based open risk)
│
├── MarketDataBuilder (app/brokers/oanda.py)
│   ├── IndicatorCalculator (EMA, RSI, ADX, ATR)
│   ├── MarketStructureAnalyzer (confirmed swing pivots → HH/HL/LH/LL)
│   └── ICT concepts (Order Blocks, FVGs, P/D zones, Liquidity sweeps)
│
├── TradeExecutor (app/execution/trade_executor.py)
│   ├── Order validation + sizing
│   ├── OANDA order placement / monitoring
│   ├── TP1 at configurable fraction of entry→TP2 (currently 60%)
│   ├── TP1 partial close (tp1_close_percent from config)
│   ├── First-hour early momentum exit for stalled trades
│   ├── ATR-based trailing stop after TP1 (tp2_trail from config)
│   └── Trade outcome feedback loop
│
├── TradeJournal (app/execution/trade_journal.py)
│   ├── Open/closed trade persistence
│   ├── Structured feedback fields per closed trade:
│   │   setup_grade (A/B/C/F), entry_timing, ict_post_hoc,
│   │   root_cause, pattern_tags
│   └── Trade signal timeline per trade (one JSON file from entry to close)
│
├── app/backtesting/
│   ├── HistoricalDataLoader (data_loader.py)
│   ├── SignalReplayEngine (signal_replayer.py)
│   ├── replay_confluence.py
│   ├── OutcomeSimulator (outcome_simulator.py)
│   ├── report.py
│   └── HistoricalFundamentalsProvider (historical_fundamentals_provider.py)
│
└── app/fundamentals/fetcher.py
    ├── DXY intraday signal via Yahoo Finance
    ├── COT positioning via CFTC.gov (weekly, delayed)
    ├── Next high-impact event via Forex Factory
    │   refreshed in fixed 6-hour UTC slots
    └── Latest FX headline via Finnhub or NewsAPI when configured
```

---

## FILE STRUCTURE

```
~/forex_trading_system/
├── main.py                    ← Thin CLI entrypoint
├── api_server.py              ← Thin FastAPI entrypoint
├── requirements.txt           ← Dependencies
├── README.md                  ← Setup guide
├── app/
│   ├── analysis/              ← Agent, scheduler, market analysis
│   ├── api/                   ← FastAPI implementation
│   ├── brokers/               ← OANDA market/account data
│   ├── cli/                   ← Runtime CLI implementation
│   ├── core/                  ← Shared config/bootstrap helpers
│   ├── execution/             ← Order execution + monitoring
│   ├── fundamentals/          ← Live fundamentals layer
│   ├── logs/                  ← Signal log helpers
│   └── rag/                   ← RAG pipeline implementation
│
├── documents/
│   ├── books/                 ← Trading books (loaded into RAG)
│   │   ├── DAY TRADING AND SWING TRADING.txt        (160 chunks) ← Kathy Lien
│   │   ├── day-trading-kathy-lien.pdf               (162 chunks) ← duplicate OK
│   │   ├── john_murphy_ocr.txt                      (262 chunks) ← John Murphy (OCR)
│   │   ├── the_forex_trading_course.pdf             (123 chunks) ← Abe Cofnas
│   │   └── trading_in_the_zone_ocr.txt              (157 chunks) ← Mark Douglas (OCR)
│   ├── research/              ← Free BIS/Fed/SSRN papers
│   ├── ict/                   ← ICT transcripts / EUR/USD notes
│   ├── cot/                   ← Manual COT report notes
│   └── journal/               ← Manual trade journal entries
│
├── chroma_db/                 ← ChromaDB vector store (864 chunks total)
├── logs/                      ← Signals, per-trade timelines, closed trades, decision logs
├── feedback/                  ← Auto-generated markdown trade review notes
└── venv/                      ← Python virtual environment
```

---

## KNOWLEDGE BASE STATUS (as of March 10, 2026)

```
✅ books          864 chunks total
   ├── Kathy Lien (TXT)           160 chunks — Forex/EUR/USD sessions
   ├── Kathy Lien (PDF)           162 chunks — duplicate, OK
   ├── John Murphy (OCR)          262 chunks — Classical TA
   ├── The Forex Trading Course   123 chunks — Strategy/COT
   └── Trading in the Zone (OCR)  157 chunks — Psychology/discipline

⚠️  research        0 chunks  ← ADD: bis.org/research PDFs
⚠️  ict             0 chunks  ← ADD: ICT YouTube transcripts
⚠️  cot             0 chunks  ← ADD: CFTC COT reports
⚠️  journal         0 chunks  ← Manual operator notes
⚠️  feedback        0 chunks  ← Populated automatically by closed trades
```

**Target by Month 3:** 2,500+ chunks

---

## ENVIRONMENT SETUP

```bash
# Navigate to project
cd ~/forex_trading_system

# Activate virtual environment (ALWAYS do this first)
source venv/bin/activate

# Required
# ANTHROPIC_API_KEY
# OANDA_API_KEY
# OANDA_ACCOUNT_ID=101-001-38764497-001

# Optional live fundamentals
# FINNHUB_API_KEY
# NEWS_API_KEY
```

Fundamentals status:
- DXY is auto-fetched intraday from Yahoo Finance
- COT is auto-fetched from CFTC.gov, but remains weekly macro data
- Forex Factory provides the next high-impact USD / EUR event
- Calendar refresh is fixed to 6-hour UTC slots; cached upcoming events roll forward locally between slot refreshes
- Finnhub or NewsAPI provide the latest FX headline when configured
- USD target range is fetched from the official Fed open-market page and ECB key rates are fetched from the official ECB key-rates page
- ECB main refi, marginal lending, and deposit rates are exposed; deposit remains the EUR benchmark for the rate differential
- Retail sentiment auto-fetches from the OANDA EUR/USD position book when available
- Historical replay can load local Fed/ECB rates, DXY, EUR COT, and high-impact USD/EUR calendar CSVs from `backtest_data/fundamentals/`
- Closed trades now generate both:
  - structured feedback stored directly into the feedback RAG collection
  - readable markdown review notes in `feedback/`

---

## RUN COMMANDS

```bash
# Test single analysis
python main.py --mode test

# Test single analysis without placing any orders
python main.py --mode test --dry-run

# Ingest new documents into RAG
python main.py --mode ingest

# Show knowledge base statistics
python main.py --mode stats

# Start continuous demo loop
# Entry windows: every 10 min
# Outside entry windows: every 30 min
python main.py --mode demo
```

---

## DEPENDENCIES

```
# Python packages (all installed in venv)
anthropic          # Claude API
chromadb           # Vector database
sentence-transformers  # all-MiniLM-L6-v2 embeddings
pypdf              # PDF text extraction
pytesseract        # OCR for image-based PDFs
pdf2image          # PDF → images for OCR
pandas             # Data manipulation
ta                 # Technical indicators (replaces pandas-ta — Python 3.11 fix)
requests           # OANDA REST API calls
yfinance           # Intraday DXY signal
pikepdf            # PDF decryption (tried, Murphy still encrypted)

# System tools (installed via Homebrew)
poppler            # Required by pdf2image
tesseract          # Required by pytesseract

# Python version
Python 3.11.4
```

---

## SIGNAL OUTPUT FORMAT

Every analysis produces a JSON signal with this structure:

```json
{
  "timestamp": "ISO-8601",
  "pair": "EUR/USD",
  "timeframe": "4H",
  "session": "NY Kill Zone",
  "macro_bias": {
    "weekly": "BULLISH|BEARISH|NEUTRAL",
    "daily": "BULLISH|BEARISH|NEUTRAL",
    "h4": "BULLISH|BEARISH|NEUTRAL",
    "alignment": "ALIGNED|MIXED|OPPOSED"
  },
  "ict_analysis": {
    "order_block": { "present": bool, "type": str, "level": float, "valid": bool },
    "fair_value_gap": { "present": bool, "type": str, "upper": float, "lower": float },
    "liquidity": { "recent_sweep": bool, "swept_level": float, "direction": str },
    "premium_discount": "PREMIUM|DISCOUNT|EQUILIBRIUM",
    "ote_zone": [float, float]
  },
  "technical_analysis": {
    "ema_bias": "BULLISH|BEARISH|NEUTRAL",
    "rsi_14": float,
    "rsi_signal": "OVERSOLD|OVERBOUGHT|NEUTRAL",
    "adx_14": float,
    "market_regime": "TRENDING|RANGING|HIGH_VOLATILITY",
    "key_levels": { "resistance": [float], "support": [float] }
  },
  "fundamental": {
    "rate_differential": str,
    "dxy_direction": "RISING|FALLING|NEUTRAL",
    "cot_bias": "BULLISH|BEARISH|NEUTRAL",
    "next_news_event": str,
    "news_risk": "HIGH|MEDIUM|LOW"
  },
  "confluence_score": 0-100,
  "signal_strength": "STRONG|MODERATE|NEUTRAL",
  "signal": {
    "direction": "BUY|SELL|NEUTRAL",
    "confidence": 0-100,
    "entry_zone": [float, float],
    "stop_loss": float,
    "take_profit_1": float,
    "take_profit_2": float,
    "risk_reward": float,
    "recommended_lot_size": float,
    "order_type": "LIMIT|MARKET|STOP"
  },
  "reasoning": ["string array of reasoning points"],
  "key_risk": "string",
  "knowledge_sources_used": ["Book Title — relevant concept"],
  "trade_management": {
    "tp1_action": "TP1 sits at 60% of the distance to TP2; close 50% there and move SL to entry",
    "tp2_action": "Trail remaining 50%",
    "time_stop": "Close if -0.5R after 8 hours"
  },
  "do_not_trade_reason": null or "string",
  "demo_mode": true
}
```

---

## SIGNAL QUALITY BENCHMARKS

```
Confluence Score:
  85-100 = STRONG signal  → consider trading
  65-84  = MODERATE       → trade with caution
  <65    = NEUTRAL        → DO NOT TRADE

Confidence:
  >75%   = High quality
  65-75% = Acceptable
  <65%   = Skip

Risk:Reward:
  >3.0   = Excellent
  2.0-3.0 = Good (minimum accepted)
  <2.0   = Reject trade
```

---

## EUR/USD SPECIFIC KNOWLEDGE EMBEDDED IN AGENT

The following concepts are in the system prompt (Option 1):

```
Sessions & Kill Zones:
  - Asian: 8PM-4AM EST — builds range, observe only
  - London Kill Zone: 3AM-4AM EST — sweeps Asian high/low, prime entry
  - NY Kill Zone: 8AM-10AM EST — continuation or reversal

Key Drivers of EUR/USD:
  1. Fed vs ECB interest rate differential
  2. DXY direction (inverse correlation — EUR is 57.6% of DXY)
  3. Economic data surprises (NFP, CPI, PMI, IFO)
  4. Risk sentiment (S&P500 as proxy)
  5. COT positioning (CFTC Commitments of Traders)

ICT Concepts Applied:
  - Order Blocks (OB) — institutional entry zones
  - Fair Value Gaps (FVG) — price imbalances to fill
  - Liquidity sweeps — stop hunts before real moves
  - Premium/Discount — only buy in discount, sell in premium
  - OTE (Optimal Trade Entry) — 62-79% Fibonacci retracement

Key Weekly Levels to Always Track:
  - Previous Week High (PWH) + Previous Week Low (PWL)
  - Previous Day High (PDH) + Previous Day Low (PDL)
  - Round numbers: 1.0500, 1.0600, 1.0700, 1.0800, 1.0900, 1.1000
  - Current week/day open price
```

---

## OANDA CONNECTOR (March 11, 2026) ✅ ACTIVE

OANDA demo account successfully created and set as the primary broker.

**Account Status:** LIVE DEMO ✅
- Account ID: `101-001-38764497-001`
- Account Type: V20 (OANDA's modern REST API)
- Balance: $100,000.00 USD
- NAV: $100,000.00
- Unrealized P/L: $0.00
- Activated: March 11, 2026

**Connector file:** `app/brokers/oanda.py`

**What it does:**
- Fetches real-time EUR/USD bid/ask/mid/spread
- Downloads OHLCV candles for 4H, 1H, Daily, Weekly timeframes
- Calculates all indicators (EMA, RSI, ADX, ATR) from real data
- Detects ICT concepts (Order Blocks, FVGs, P/D zones, liquidity sweeps)
- Identifies market structure from confirmed swing pivots across all timeframes
- Uses ATR-based tolerance to avoid treating tiny wick differences as new structure
- Keeps the same public `weekly_structure` / `*_trend` fields used by the API and frontend
- Reads account balance, equity, open positions from OANDA
- Fails closed when live broker data is unavailable

**Live fundamentals layered on top:**
- DXY intraday signal from Yahoo Finance (`DX-Y.NYB`)
- next high-impact USD / EUR event from Forex Factory
- latest FX headline from Finnhub or NewsAPI when configured
- COT positioning from CFTC.gov for delayed weekly macro bias
- retail sentiment is sourced only from the OANDA EUR/USD position book

**Auth:** Bearer token — OANDA API key in Authorization header

**Instrument name:** `EUR_USD` (OANDA format, not `EURUSD`)

**To activate:**
```bash
# 1. OANDA demo account created ✅ (March 11, 2026)
# 2. Generate API key: My Account → Manage API Access → Generate
# 3. Set in terminal (add to ~/.zshrc for persistence):
export OANDA_API_KEY="your-api-key"
export OANDA_ACCOUNT_ID="101-001-38764497-001"

# 4. Test connection:
python main.py --mode test

# 5. Run live demo:
python main.py --mode demo
```

**Failure behavior:** If OANDA credentials are not set or live market data fails,
the system stops and prints actionable warnings. It does not substitute static data.

---

## NEXT STEPS (Priority Order)

See `IMPROVEMENT_ROADMAP.md` for the full implementation history. Current
state summary below.

```
COMPLETED (March 2026):
  ✅ Build OANDA broker layer and demo runtime
  ✅ Mechanical confluence scorer + execution gate
  ✅ Weekly loss limit enforcement
  ✅ Historical data loader, replay engine, outcome simulator, and reports
  ✅ Historical fundamentals provider with local rates / DXY / COT / calendar CSVs
  ✅ Structured trade feedback + per-trade timeline logging
  ✅ TP1 partial close from config
  ✅ ATR-based trailing stop after TP1
  ✅ First-hour early momentum exit for stalled trades

CURRENT PRIORITIES:
  ⬜ Improve replay realism further
     → sequential one-account simulation
     → spread / slippage modeling
  ⬜ Continue hardening live fundamentals infrastructure
  ⬜ Add more ICT transcripts and remaining research PDFs
  ⬜ Build a cleaner performance dashboard on top of existing API/log outputs
```

---

## 5-YEAR PROFIT PROJECTIONS ($5,000 start)

| Year | Start | Net Profit | Return | End Capital |
|------|-------|-----------|--------|-------------|
| 1 | $5,000 | +$3,000 | +60% | $8,000 |
| 2 | $8,000 | +$6,150 | +77% | $14,150 |
| 3 | $14,150 | +$13,500 | +95% | $27,650 |
| 4 | $27,650 | +$28,000 | +101% | $55,650 |
| 5 | $55,650 | +$63,000 | +113% | $118,650 |

Conservative: $67,000 | Moderate (most likely): $118,650 | Optimistic: $310,000
$100k milestone target: ~Month 59 (Year 5 Q3)

---

## KNOWN ISSUES & FIXES APPLIED

```
Issue: pandas-ta not compatible with Python 3.11
Fix:   Replaced with 'ta==0.11.0' library
Status: ✅ Resolved

Issue: 3 of 4 books had encrypted PDFs (Murphy, Kathy Lien PDF, Trading in Zone)
Fix:   OCR with tesseract + pdf2image + poppler
Status: ✅ All 4 books loaded (864 chunks)

Issue: Trading in Zone OCR killed by memory (all 237 pages at once)
Fix:   Process in 20-page batches at dpi=150
Status: ✅ Resolved — 157 chunks loaded

Issue: total_chunks KeyError in the RAG pipeline
Fix:   Changed to result.get("total_chunks", 0)
Status: ✅ Resolved

Issue: API key from wrong workspace (Claude Code workspace, not Individual Org)
Fix:   Created new key in Default workspace under Gazmir's Individual Org
Status: ✅ Resolved

Issue: RSI/ADX crash on flat or zero-range markets
Fix:   Extract scalars before division; guard NaN/zero with explicit checks
Status: ✅ Resolved — March 20, 2026

Issue: FVG false positives — impulse candle retracing back through gap
Fix:   Validate c2 (impulse candle) does not breach back through c1 boundary
Status: ✅ Resolved — March 20, 2026

Issue: Premium/Discount almost always returning EQUILIBRIUM
Fix:   Switched from price-relative 0.2% band to range-relative 60/40 threshold
Status: ✅ Resolved — March 20, 2026

Issue: Liquidity sweep lookback too narrow (10 candles)
Fix:   Extended to 48 candles to catch full session sweeps
Status: ✅ Resolved — March 20, 2026

Issue: Greedy JSON regex failing on nested Claude responses
Fix:   Replaced with brace-depth tracker that handles nested objects + strings
Status: ✅ Resolved — March 20, 2026

Issue: Session label using Claude's inferred value, not clock-derived value
Fix:   Overwrite signal["session"] from fund.get("active_session") in validator
Status: ✅ Resolved — March 20, 2026

Issue: Daily loss check not projecting new trade's risk before blocking
Fix:   Block when daily_pnl - max_trade_risk_pct < -max_daily_loss_pct
Status: ✅ Resolved — March 20, 2026

Issue: Open risk masking offsetting positions (abs of net sum)
Fix:   Sum absolute per-position exposure instead of abs(net)
Status: ✅ Resolved — March 20, 2026

Issue: Daily PnL resetting to 0 after process restart
Fix:   _get_daily_start_balance() reads from persistent daily_state.json
Status: ✅ Resolved — March 20, 2026

Issue: trades_today showing open count only, not daily total
Fix:   _count_closed_today() scans closed_trades.jsonl for today's entries
Status: ✅ Resolved — March 20, 2026

Issue: Session loss streak not surviving process restarts
Fix:   Falls back to closed_trades.jsonl when in-memory feedback is empty
Status: ✅ Resolved — March 20, 2026

Issue: OANDA candle fetch failing silently on network errors
Fix:   3-attempt retry with 2s/4s exponential backoff
Status: ✅ Resolved — March 20, 2026

Issue: Fundamentals MANUAL_CHECK strings passed as-is to Claude
Fix:   Neutral fallback values (NEUTRAL, None available) for all fields
Status: ✅ Resolved — March 20, 2026

Issue: tp1_close_percent and tp2_trail in config never used by executor
Fix:   Wired tp1_close_percent into _apply_tp1_if_needed;
       added _apply_trailing_stop_if_needed with one-directional ATR trail
Status: ✅ Resolved — March 20, 2026

Issue: Dead wrapper methods in TradeExecutor and unused aliases in agent
Fix:   Removed 8 dead wrappers; removed feedback_dir/feedback_memory aliases;
       consolidated ALLOWED_ENTRY_SESSIONS to single definition in scheduler.py
Status: ✅ Resolved — March 20, 2026
```

---

## READING LIST (Books Already Downloaded)

```
✅ Day Trading and Swing Trading the Currency Market — Kathy Lien
   Status: LOADED (160+162 chunks)
   Key value: EUR/USD session behavior, fundamental drivers

✅ Trading in the Zone — Mark Douglas
   Status: LOADED (157 chunks)
   Key value: Psychology, discipline, consistency rules

✅ Technical Analysis of the Financial Markets — John Murphy
   Status: LOADED (262 chunks — OCR)
   Key value: Classical TA, moving averages, chart patterns

✅ The Forex Trading Course — Abe Cofnas
   Status: LOADED (123 chunks)
   Key value: Strategy rules, COT analysis

⬜ Quantitative Trading — Ernest Chan (get month 6)
⬜ Algorithmic Trading — Ernest Chan (get month 7)
⬜ The Zurich Axioms — Max Gunther (get month 3)
⬜ Antifragile — Nassim Taleb (get month 4)
```

---

## CLAUDE CODE INSTRUCTIONS

When working on this project in VS Code, follow these rules:

1. **Never change hard risk rules** — 1% max, 1:2 RR, 2% daily limit are sacred
2. **Never set demo_mode = False** — only change after March 2027
3. **Always test with --mode test before --mode demo**
4. **All new features must fail closed on missing live data** — print actionable warnings, never use example market data
5. **Never delete logs/** — every signal is training data
6. **Never delete chroma_db/** — rebuilding takes 30+ minutes
7. **Always activate venv first** — `source venv/bin/activate`
8. **Config changes go in app/core/config.py only** — not hardcoded in other files
9. **New documents go in documents/[category]/** then run --mode ingest
10. **Keep the OANDA broker layer separate** — never merge `app/brokers/oanda.py` into the CLI entrypoint
11. **New features must follow IMPROVEMENT_ROADMAP.md phase order** — do not skip phases; each builds on the last
12. **Allowed sessions are defined once** — always import `ALLOWED_ENTRY_SESSIONS` from `app/analysis/scheduler.py`, never hardcode the set
13. **Confluence scoring** — when the mechanical scorer (Phase 1) is built, it becomes the execution gate; Claude's self-reported score is logged only for comparison

---

## API COSTS REFERENCE

```
Claude API (claude-sonnet-4-20250514):
  Input:  $3.00 per million tokens
  Output: $15.00 per million tokens
  
Per analysis (approx):
  ~2,000 tokens input + ~500 tokens output
  Cost: ~$0.01-0.02 per analysis
  
Demo loop (every 30 min, 24h):
  48 analyses/day × $0.02 = ~$0.96/day
  Monthly: ~$29/month
  
With $10 credit: ~10 days of continuous demo
Recommended: Add $20-50 to platform.claude.com billing
```

---

## CONTACT & ACCOUNTS

```
Claude API:   platform.claude.com — Gazmir's Individual Org — Default workspace
OANDA:        oanda.com — demo account ACTIVE ✅ (101-001-38764497-001, $100k)
              → API key still needed: My Account → Manage API Access
GitHub:       (not set up yet — recommended for version control)
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gazmi82) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
