## apex-predator-neo

> Autonomous triangular arbitrage scanner and AI trading assistant for Binance. Combines all 7 official Binance AI Agent Skills into a unified HFT-grade workflow with real-time triangle detection, spoof hunting, risk management, and natural language control. Built for the Binance OpenClaw Challenge.


# APEX PREDATOR NEO v666 🦈

## Autonomous Triangular Arbitrage Scanner + AI Trading Assistant for Binance

**An OpenClaw skill that orchestrates ALL 7 official Binance AI Agent Skills into one unified, autonomous HFT-grade trading workflow.**

> "7 AI modules. Zero human intervention. Sub-40ms scan cycles."

---

## What It Does

APEX PREDATOR NEO transforms your OpenClaw agent into a professional-grade crypto trading assistant that:

1. **Scans triangular arbitrage opportunities** across 800+ Binance Spot pairs in real-time
2. **Filters signals** through a 6-layer ConfluenceEngine (fake momentum rejection, spoof detection, OI consistency)
3. **Manages risk autonomously** with hard drawdown limits and automatic position freezing
4. **Sweeps profits** into Binance Simple Earn automatically
5. **Reports everything** via your preferred messaging channel (WhatsApp, Telegram, Slack, Discord)

---

## Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `BINANCE_API_KEY` | API Key from Binance (Spot trading enabled) | Yes |
| `BINANCE_SECRET` | API Secret | Yes |
| `APEX_MODE` | `testnet` (default) or `live` | No |
| `APEX_CAPITAL` | Total capital in USDT (default: 22) | No |
| `APEX_MAX_TRADE` | Max per trade in USDT (default: 8) | No |
| `APEX_MAX_DRAWDOWN` | Max drawdown % before freeze (default: 4.0) | No |
| `APEX_SCAN_INTERVAL_MS` | Scan cycle interval (default: 40) | No |
| `APEX_AUTO_EARN` | Enable auto-earn sweep (default: true) | No |
| `APEX_EARN_THRESHOLD` | Min profit to sweep to Earn (default: 0.10) | No |

---

## Binance Skills Integration

This skill orchestrates all 7 official Binance AI Agent Skills:

### 1. CEX Spot Trading Skill
**Used for:** Real-time market data (tickers, depth, candlesticks) and trade execution (market/limit/OCO orders).

```bash
# Fetch current BTC price
curl -s "https://api.binance.com/api/v3/ticker/price?symbol=BTCUSDT" | jq '.price'

# Fetch orderbook depth (15 levels)
curl -s "https://api.binance.com/api/v3/depth?symbol=BTCUSDT&limit=15" | jq '.'

# Fetch all tickers for triangle scanning
curl -s "https://api.binance.com/api/v3/ticker/bookTicker" | jq '.'

# Place testnet order (HMAC-SHA256 signed)
TIMESTAMP=$(date +%s%3N)
QUERY="symbol=BTCUSDT&side=BUY&type=MARKET&quoteOrderQty=8&timestamp=${TIMESTAMP}"
SIGNATURE=$(echo -n "${QUERY}" | openssl dgst -sha256 -hmac "${BINANCE_SECRET}" | cut -d' ' -f2)
curl -s -X POST "https://testnet.binance.vision/api/v3/order?${QUERY}&signature=${SIGNATURE}" \
  -H "X-MBX-APIKEY: ${BINANCE_API_KEY}" | jq '.'
```

### 2. Address Insight Skill
**Used for:** Whale tracking — monitor large wallet movements that could affect triangle spreads.

```bash
# Analyze a whale wallet for holdings and concentration
# Used by EconoPredator module to detect institutional flow
```

### 3. Token Details Skill
**Used for:** Validate token liquidity and trading pairs before including in triangle paths.

### 4. Market Rankings Skill
**Used for:** Dynamic pair selection — auto-select top pairs by volume/volatility for optimal triangle discovery.

### 5. Meme Rush Skill
**Used for:** Detect sudden meme coin surges that create temporary arbitrage dislocations across pairs.

### 6. Trading Signals Skill
**Used for:** Confluence layer — cross-reference APEX signals with Binance's own signal engine for higher confidence.

### 7. Token Contract Audit Skill
**Used for:** Anti-rug module — audit token contracts before including new pairs in triangle scan paths.

---

## 7 AI Modules

### 🧠 ConfluenceEngine — 6-Layer Signal Filter
Before any trade executes, the signal must pass ALL 6 filters:

1. **Tire Pressure** — Measures bid-ask spread compression as a proxy for execution certainty
2. **Lead-Lag Detection** — Identifies which pair in the triangle moves first (information advantage)
3. **Fake Momentum Filter** — Rejects signals driven by wash trading or artificial volume
4. **OI Consistency** — Cross-references Open Interest data to confirm real market conviction
5. **OI Delta/Volume Ratio** — Ensures volume is organic and not inflated by derivatives hedging
6. **Post-Spike Reversal** — Avoids entering after sharp moves that are likely to mean-revert

### 🛡️ Robin Hood Risk Engine
Zero-tolerance capital protection:
- **Hard drawdown limit:** 4% max → triggers 30-minute total freeze
- **Per-trade limit:** $8 max (configurable via `APEX_MAX_TRADE`)
- **No override possible** — survival logic is non-negotiable
- **Auto-recovery:** Resumes scanning after cooldown period

### 👁️ SpoofHunter — L2 Defense
Real-time Level 2 orderbook analysis:
- Detects ghost orders (large orders placed then cancelled within 200ms)
- Identifies fake walls and layering patterns
- Flags wash trading activity
- **Any signal contaminated by spoof activity is automatically rejected**

### 📊 EconoPredator — Macro Intelligence
Monitors external factors that affect crypto volatility:
- CPI/FOMC/NFP economic calendar events
- Funding rates across major pairs
- Open Interest shifts
- Long/Short ratio imbalances
- On-chain whale flows (via Address Insight Skill)

### 📡 Active Position Manager (APM)
Real-time position monitoring at 100ms intervals:
- **VPIN** (Volume-Synchronized Probability of Informed Trading) tracking
- **Alpha Decay** measurement — auto-exits when edge deteriorates
- Partial fill detection with atomic rollback

### 💰 Auto-Earn Hook
Idle capital optimization:
- Sweeps profits > $0.10 into Binance Simple Earn
- Finds highest APR product automatically
- Capital generates passive yield 24/7 between trades

### 🌐 Distributed Executor Network
Multi-region execution architecture:
- Scanner runs locally (your machine / Curitiba node)
- Redis Pub/Sub broadcasts signals with nanosecond timestamps
- Singapore + Tokyo executors for <10ms latency to Binance matching engine
- Atomic state machine with rollback on partial fills

---

## Architecture

```
Your Machine (OpenClaw)
  └── APEX PREDATOR NEO Skill
       ├── LOB Service ── WebSocket @depth@100ms ──→ Binance API
       │    └── Shared memory (0ms read)
       ├── Scanner ── ConfluenceEngine (6 filters)
       │    ├── Dynamic Fee Manager (real-time + BNB discount)
       │    ├── SpoofHunter (L2 analysis)
       │    └── EconoPredator (macro + on-chain)
       ├── Redis Pub/Sub (orjson + nanosecond timestamps)
       │    ├── strike_zone_{symbol} channels
       │    └── execution confirmations
       ├── Executors (Singapore/Tokyo AWS)
       │    └── Atomic State Machine + Rollback
       └── Post-Trade
            ├── Robin Hood Risk (drawdown check)
            ├── Auto-Earn Hook (sweep to Simple Earn)
            └── Report to OpenClaw chat (WhatsApp/Telegram/Slack)
```

---

## Usage Examples

### Quick Status Check
> "What's the APEX status?"

Returns: Active triangles, current spread, drawdown level, scan cycle time, module health.

### Triangle Scan
> "Show me the best triangle opportunities right now"

Returns: Top 10 triangle paths sorted by spread, with confluence scores and fee analysis.

### Risk Check
> "What's my risk exposure?"

Returns: Current drawdown, capital deployed, active positions, Robin Hood status.

### Market Analysis
> "Give me a market analysis for BTC"

Uses EconoPredator + Binance Trading Signals to provide funding rates, OI, long/short ratios, and whale activity.

### Wallet Analysis
> "Analyze this whale wallet: 0x..."

Uses Address Insight to break down holdings, 24h changes, and concentration metrics.

### Token Safety Check
> "Is this token safe? Contract: 0x..."

Uses Token Contract Audit to check for rug pull indicators, honeypot patterns, and suspicious code.

### Force Exit
> "Emergency exit all positions"

Triggers immediate market-order close on all active positions.

### Enable Auto-Earn
> "Sweep my idle USDT to Simple Earn"

Activates Auto-Earn Hook to find best APR product and subscribe.

---

## Tech Stack

- **Python 3.12** — Core runtime
- **CCXT Pro** — Exchange connectivity
- **Redis 7.2** — Message bus (Pub/Sub)
- **Docker Swarm** — Orchestration
- **uvloop** — High-performance event loop
- **orjson** — Fast JSON serialization
- **WebSocket** — Real-time market data streams
- **Rust FFI** — Optional native LOB processing
- **FastAPI** — Dashboard API
- **Loguru** — Structured logging
- **Pydantic v2** — Configuration validation

---

## Security

- **Testnet by default** — Must explicitly enable live trading
- **API key scoping** — Only Spot trading permissions required, no withdrawal access
- **Rate limiting** — Respects Binance rate limits (2400 req/min) with intelligent batching
- **No hardcoded secrets** — All credentials via environment variables
- **Audit trail** — Every scan, signal, and trade logged with nanosecond precision
- **Human-in-the-loop** — Optional confirmation mode for trades above threshold

---

## Installation

```bash
# Via ClawHub
clawhub install apex-predator-neo

# Or paste this GitHub URL in your OpenClaw chat:
# https://github.com/leoklein/apex-predator-neo

# Configure
export BINANCE_API_KEY="your_key"
export BINANCE_SECRET="your_secret"
export APEX_MODE="testnet"
```

Then tell your OpenClaw agent:
> "Activate APEX PREDATOR NEO and start scanning triangles"

---

## Disclaimer

This skill is for educational and research purposes. Cryptocurrency trading involves substantial risk of loss. Always use testnet mode first. Never trade with money you cannot afford to lose. Past performance does not guarantee future results. The authors are not responsible for any financial losses.

---

## Built For

**Binance OpenClaw Challenge 2026**
#BinanceOpenClaw #CryptoAI

Built by **@leoklein** — Curitiba, BR 🇧🇷

---
> Source: [ofockus/apex-predator-neo](https://github.com/ofockus/apex-predator-neo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
