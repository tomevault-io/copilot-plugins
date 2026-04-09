## apex-s44-monitor

> Credit Catalyst Trading System — Master Project Document v8

Credit Catalyst Trading System — Master Project Document v8

**Owner:** Matt Bristow
**Last Updated:** 5 February 2026 (morning)
**Purpose:** This document is the single source of truth for the project. Paste it into any new Claude session to get full context.

---

## 1. Vision: Two Products, One Foundation

### Product 1: Credit Catalyst Monitoring System (Institutional Grade)
**Purpose:** Pitch to hedge funds as a monitoring/alpha generation tool
**Execution:** Manual — system generates ideas, humans trade
**Timeframe:** Medium-term (weeks to months)
**Target:** External clients (hedge funds, asset managers)

### Product 2: Autonomous Trading Agent (Personal)
**Purpose:** Trade your own capital, fully automated
**Execution:** Autonomous — agent trades on your behalf
**Timeframe:** Shorter-term (days to weeks)
**Target:** Your personal account
**Key Feature:** Becomes aggressive when it detects edge

### Shared Foundation
Both products share:
- The Watcher (data collection)
- The Analyst (credit scoring)
- The Scanner (options analysis)
- Knowledge base (19 books + credit docs)
- Company database (202 names)

### Development Path
1. ~~Build shared foundation~~ ✅ COMPLETE
2. ~~Build Product 2 (autonomous agent) for personal trading~~ ✅ COMPLETE
3. **Trade your own capital for 12-24 months — NOW LIVE**
4. Document track record
5. Use results to pitch Product 1 to institutions

### Current Status (as of 4 Feb 2026 evening)

**Phase: 🟢 LIVE TRADING + DASHBOARD LIVE**

**System is deployed and running on VPS (157.245.36.127)**

| Component | Status | Details |
|-----------|--------|---------|
| IB Gateway | ✅ Connected | Account U21710729, Port 4001 (live) |
| Trading System | ✅ Running | Scanning, analyzing, ranking signals |
| API Trading | ✅ Enabled | ReadOnlyApi=no in IBC config |
| Telegram Alerts | ✅ Active | Notifications configured |
| Web Dashboard | ✅ Live | https://dashboard.mb-trading.co.uk |
| Dashboard API | ✅ Live | https://app.mb-trading.co.uk (HTTPS/Let's Encrypt) |
| Excel Dashboard | ✅ Built | `APEX_Trading_Dashboard.xlsx` with live API refresh |

**Dashboard URLs:**
- **Web (Mac + iPhone):** https://dashboard.mb-trading.co.uk (hosted on Vercel)
- **API Backend:** https://app.mb-trading.co.uk (VPS, nginx + Let's Encrypt)
- **Excel:** `python refresh_dashboard.py` or `python live_dashboard.py` for auto-refresh

**To enable trade execution:**
1. Log into https://www.interactivebrokers.com/portal
2. Settings → Account Settings → API
3. Enable "Allow API trading"
4. Confirm via security device

**Live Activity (as of deployment):**
- Signals: 100 raw → 50 analyzed → ranked by edge score
- Top opportunity detected: XLE (equity) Score 0.72, R:R 2.0

**IBKR Credentials (VPS config: ~/ibc/config.ini):**
- Username: mbristow180
- Account: U21710729 (ISA)
- Mode: Live (port 4001)

---

## 2. Core Philosophy

> "Human psychology leaves alpha. My agent will understand psychology and have the autonomy to put trades on under certain guidelines. I want it to become aggressive when it feels it has an edge — THAT IS THE KEY."

### What This Means
- Markets misprice because humans are emotional, slow, and biased
- Credit signals emerge before equity reprices — the lag is the opportunity
- The agent must understand WHY humans misprice, not just THAT they do
- When multiple signals align (credit + psychology + options), the agent sizes up aggressively
- The agent earns the right to be aggressive through conviction, not randomness

---

## 3. The Investment Thesis

### The Macro Setup (2021-2028 Credit Cycle)
- 2021 "free money" era created a wave of covenant-lite HY issuance
- Loan docs were written to favour sponsors/equity, not bondholders
- Many companies (especially European HY) issued when money was cheap with loose documentation
- 2027/2028 maturity wall is coming — these companies need to refinance in a very different rate environment
- Limited European credit defaults historically, so each situation requires rigorous analysis

### Two Distinct Playbooks

#### Playbook A: Aggressive Sponsor Situations

**Characteristics:**
- PE sponsor or controlling shareholder with history of aggressive tactics
- Loose loan docs with weak restricted payments covenants
- Unrestricted subsidiary structures that allow asset movement
- Sponsor incentivised to extract value for equity before bondholders react

**Examples:**
- Altice France / Drahi — moved assets to unrestricted subsidiaries
- Ardagh — similar tactics to benefit equity/sponsors at bondholder expense
- J.Crew, Serta — "creditor-on-creditor violence" playbook

**Options Implications:**
- Equity might SPIKE before eventual collapse (sponsor extracts value)
- Puts alone may not work — timing is treacherous
- Consider: calls (on asset stripping news), straddles (binary outcomes), or avoid entirely
- These situations are harder to trade systematically

#### Playbook B: Maturity Wall / No Aggressive Sponsor

**Characteristics:**
- Company isn't being actively looted
- Simply overleveraged from 2021 issuance
- Faces 2027/2028 maturity wall
- Can't refinance at current spreads without distressed exchange or restructuring
- More predictable deterioration path

**Options Implications:**
- Puts are likely the right strategy
- Timing tied to: maturity schedule, rating actions, covenant breaches
- More predictable — system can identify entry points
- Theta cost acceptable if catalyst timeline is clear

### Critical Research Question

> "Where was the options market pricing when credit analysts first flagged issues?"

This is the edge we're hunting — the lag between:
1. Credit spread widening / analyst concern
2. Options market repricing (IV increase, put skew)

---

## 4. Product 2: Autonomous Trading Agent Specification

### Core Capabilities

| Capability | Description |
|------------|-------------|
| **Autonomous execution** | Opens and closes trades without human approval |
| **Edge detection** | Quantifies conviction level across multiple signals |
| **Aggressive sizing** | Scales position size based on edge score |
| **Psychology understanding** | Predicts human behaviour and market reactions |
| **Risk management** | Hard guardrails that cannot be overridden |
| **Learning loop** | Improves over time based on outcomes |
| **Notifications** | Telegram alerts for all actions with reasoning |

### The Edge Scoring System

The agent quantifies "I have an edge" using a weighted score:
EDGE_SCORE = weighted average of:
- Credit signal strength (0-10)     weight: 25%
- Psychology alignment (0-10)       weight: 25%
- Options mispricing (0-10)         weight: 20%
- Catalyst clarity (0-10)           weight: 15%
- Historical pattern match (0-10)   weight: 15%

#### Credit Signal Strength (0-10)
| Score | Meaning |
|-------|---------|
| 1-3 | Minor concern, no action needed |
| 4-5 | Watching — early warning signs |
| 6-7 | Material deterioration — active monitoring |
| 8-9 | Serious distress — high conviction signal |
| 10 | Imminent default/restructuring |

#### Psychology Alignment (0-10)
| Score | Meaning |
|-------|---------|
| 1-3 | Market already pricing risk — no edge |
| 4-5 | Some awareness, but not fully priced |
| 6-7 | Market complacent — clear mispricing |
| 8-9 | Extreme complacency or denial |
| 10 | Total disconnect between credit reality and equity pricing |

#### Options Mispricing (0-10)
| Score | Meaning |
|-------|---------|
| 1-3 | Options fairly priced — no edge |
| 4-5 | Slight mispricing |
| 6-7 | Clear mispricing — IV too low given credit risk |
| 8-9 | Significant mispricing — cheap protection |
| 10 | Extreme mispricing — gift |

#### Catalyst Clarity (0-10)
| Score | Meaning |
|-------|---------|
| 1-3 | No clear catalyst — vague timing |
| 4-5 | Potential catalyst within 6 months |
| 6-7 | Known catalyst within 3 months |
| 8-9 | Imminent catalyst within 1 month |
| 10 | Catalyst this week |

#### Historical Pattern Match (0-10)
| Score | Meaning |
|-------|---------|
| 1-3 | No similar historical precedent |
| 4-5 | Some similarities to past situations |
| 6-7 | Strong pattern match — similar setup worked before |
| 8-9 | Very close match to successful past trades |
| 10 | Almost identical to high-conviction winner |

### Aggression Logic
POSITION_SIZE = BASE_SIZE x AGGRESSION_MULTIPLIER
AGGRESSION_MULTIPLIER based on EDGE_SCORE:
- Edge score < 5.0: DO NOT TRADE
- Edge score 5.0-5.9: 0.5x (half size, cautious)
- Edge score 6.0-6.9: 1.0x (standard size)
- Edge score 7.0-7.9: 1.5x (increased conviction)
- Edge score 8.0-8.9: 2.0x (high conviction)
- Edge score 9.0+: 2.5x (maximum aggression)

**The agent earns the right to be aggressive by having multiple signals align.**

### Risk Guardrails (Hard Limits — Cannot Be Overridden)

| Guardrail | Limit | Rationale |
|-----------|-------|-----------|
| Max single position | 5% of account | No single bet can blow up account |
| Max portfolio exposure | 30% of account | Always have dry powder |
| Max loss per trade | 50% of position value | Stop-loss required on all trades |
| Max daily loss | 5% of account | Circuit breaker — stop trading |
| Max weekly loss | 10% of account | Circuit breaker — stop trading |
| Max correlation | 3 positions in same sector | Diversification enforced |
| Min edge score to trade | 5.0 | No low-conviction trades |

### Notification System (Telegram)

#### Trade Opened
TRADE OPENED
Ticker: [ATUS]
Direction: LONG PUT
Strike: $5.00
Expiry: March 2026
Size: 10 contracts (1.5x standard — HIGH CONVICTION)
Cost: $2,340
EDGE SCORE: 7.8/10

Credit signal: 8/10 (covenant breach imminent)
Psychology: 7/10 (market still complacent)
Options: 8/10 (IV 30th percentile — cheap)
Catalyst: 8/10 (earnings in 2 weeks)
Pattern: 7/10 (similar to Ardagh Q2 2024)

THESIS: Credit deterioration not reflected in equity.
Expecting 20-30% downside on earnings miss or guidance cut.

#### Trade Adjusted
TRADE ADJUSTED
Ticker: [ATUS]
Action: ADDED TO POSITION
New size: 15 contracts (was 10)
Reason: Edge score increased to 8.3 after rating downgrade
Updated cost basis: $3,100

#### Trade Closed
TRADE CLOSED
Ticker: [ATUS]
Direction: LONG PUT
Result: +$4,200 (+180%)
Hold time: 18 days
EXIT REASON: Target hit — equity down 35% post-earnings
LEARNINGS:

Credit signal preceded equity move by 6 weeks
Psychology score was accurate — market was in denial
Pattern match to Ardagh was correct
Adding to position on downgrade was right call

This pattern logged for future reference.

#### Daily Summary
DAILY SUMMARY — Feb 3, 2026
Open positions: 3
Total exposure: 18% of account
Daily P&L: +$1,240 (+2.1%)
Weekly P&L: +$3,890 (+6.4%)
POSITIONS:

ATUS Mar $5 puts — +45% — Edge: 7.8
TIIMN Apr 0.30 puts — -12% — Edge: 6.2
VIRFIN Jun 2 puts — +8% — Edge: 7.1

WATCHLIST ALERTS:

INEOS: Credit signal strengthening (now 7/10)
GRIFSA: Rating watch negative — monitoring

No trades today. Edge scores below threshold.

#### High Conviction Alert
HIGH CONVICTION OPPORTUNITY
Ticker: [INEOS]
Edge Score: 8.5/10
This exceeds our 8.0 threshold for aggressive sizing.
Preparing 2.0x position if options liquid enough.
Will notify when trade executed.

### Learning Loop

The agent tracks and learns:

| Metric | What It Tracks | How It Learns |
|--------|----------------|---------------|
| Edge score accuracy | Did high edge scores lead to wins? | Adjusts component weights |
| Signal timing | How long from signal to move? | Refines catalyst clarity scoring |
| Psychology accuracy | Did market react as predicted? | Improves psychology model |
| Pattern matching | Which historical patterns were predictive? | Builds pattern library |
| Risk/reward | Actual vs expected outcomes | Refines position sizing |

After each closed trade:
1. Log all inputs and outputs
2. Compare predicted vs actual outcome
3. Identify what worked and what didn't
4. Adjust scoring weights if needed
5. Store pattern for future matching

---

## 5. Product 1: Institutional Grade Requirements

### What "Institutional Grade" Means

| Requirement | Description |
|-------------|-------------|
| **Track record** | 12-24 months of documented results |
| **Auditable process** | Clear, repeatable methodology |
| **Risk framework** | How drawdowns are managed |
| **Backtested** | Historical analysis showing edge exists |
| **Differentiation** | Why this is better than existing tools |
| **Scalability** | Can handle 100M+ AUM |
| **Compliance** | Can fit into institutional risk limits |

### What Institutions Would Pay For

| Feature | Value Proposition |
|---------|-------------------|
| Credit signal alerts | "Know about distress before the market" |
| Cross-asset linkage | "Credit-to-equity signal generation" |
| Psychology overlay | "When will the market reprice?" |
| European HY coverage | "Underserved market, less competition" |
| Situation classification | "Aggressive sponsor vs maturity wall" |

### Path to Institutional

1. **Year 1:** Build and trade personally (Product 2)
2. **Year 1-2:** Document everything, build track record
3. **Year 2:** Create institutional-facing dashboard (Product 1)
4. **Year 2+:** Pitch with real results

---

## 6. The Six-Agent Architecture

| Agent | Purpose | Product 1 | Product 2 | Status |
|-------|---------|-----------|-----------|--------|
| **The Watcher** | Monitors SEC filings, rating actions, bond prices, news | Yes | Yes | ✅ Running on VPS |
| **The Analyst** | Reads filings, extracts credit signals, classifies situation | Yes | Yes | ✅ Working (situation classifier built) |
| **The Psychologist** | Predicts how/when humans will react | Yes | Yes | ✅ Built (`agents/psychologist.py` - 575 lines) |
| **The Scanner** | Finds mispriced options | Yes | Yes | ✅ Built (`agents/scanners/credit_options_scanner.py`) |
| **The Risk Manager** | Position sizing, exposure limits, guardrails | Optional | Yes | ✅ Built (`agents/risk_manager.py` - 557 lines) |
| **The Runner** | Executes trades via IBKR API | No | Yes | ✅ Built (`agents/execution/trade_executor.py` - 483 lines) |

### Additional Components for Product 2

| Component | Purpose | Status |
|-----------|---------|--------|
| **Edge Scorer** | Combines signals into edge score | ✅ Built (`agents/edge_scorer.py` - 738 lines) |
| **Execution Engine** | Translates decisions to IBKR orders | ✅ Built (`agents/execution/trade_executor.py`) |
| **Learning Loop** | Tracks outcomes, adjusts model | 🔶 Partial (trade history tracking in place, refinement pending) |
| **Notification System** | Telegram alerts with reasoning | ✅ Built (credit-specific Telegram notifications) |

### Supporting Components

| Component | Purpose | Status |
|-----------|---------|--------|
| **Coordinator** | Orchestrates agent interactions | ✅ Built (`agents/coordinator.py`) |
| **ML Predictor** | Machine learning signal generation | ✅ Built (`agents/signals/ml_predictor.py`) |
| **Opportunity Ranker** | Ranks and prioritizes opportunities | ✅ Built (`agents/signals/opportunity_ranker.py`) |
| **Technical Analyzer** | Technical analysis signals | ✅ Built (`agents/signals/technical_analyzer.py`) |
| **European Monitor** | UK/EU regulatory filings (RNS, etc.) | ✅ Built (`monitors/european_monitor.py`) |

---

## 7. Current Infrastructure

### What's Running Now

| Component | Location | Status | Details |
|-----------|----------|--------|---------|
| SEC Filing Monitor | DigitalOcean VPS | Running 24/7 | IP: 157.245.36.127 |
| Company Watchlist | VPS Database | 202 companies | iTraxx Main + Crossover S44 |
| Telegram Alerts | Connected to phone | Working | Bot token and chat ID configured |
| Claude API Analysis | VPS | Basic | Summarises filings |
| Auto-restart | VPS crontab | Configured | Survives reboots |
| Dashboard API | VPS (systemd) | Running 24/7 | FastAPI on port 8000, nginx reverse proxy |
| Web Dashboard | Vercel | Live | Next.js, auto-deploys |
| nginx + HTTPS | VPS | Running | Let's Encrypt cert, auto-renews |

### Database Schema (PostgreSQL/Supabase)

Full schema implemented in `database/schema.sql`:

**Core Tables:**
- `companies` — XO constituents with sector, ownership, sponsor, lifecycle status, distress probability
- `debt_instruments` — Individual bonds/loans with seniority, terms, ratings, ISINs
- `debt_pricing` — Daily pricing with yields, spreads (YTW, Z-spread, OAS)
- `cds_pricing` — CDS spread data by tenor with change tracking
- `financials` — Quarterly/annual metrics (EBITDA, leverage, coverage ratios)
- `maturity_schedule` — Debt maturity wall by year
- `news_alerts` — Credit-relevant news with sentiment and priority
- `credit_opinions` — Analyst opinions with recommendations, risks, catalysts

**Views:**
- `v_latest_pricing` — Most recent pricing per instrument
- `v_company_summary` — Company snapshot with latest financials and CDS

**Data Files (JSON):**
- `data/sponsors.json` — Sponsor database with aggression scores ✅
- `data/maturity_wall.json` — 2027/2028 maturity wall data ✅
- `data/case_studies/` — Altice France, Ardagh analysis ✅

### Hardware

| Device | Purpose | Status |
|--------|---------|--------|
| Mac Mini M4 Pro | Development + running agents | Set up and working |
| MacBook Pro (Intel i7) | Backup development | Working |
| Windows PC | Has audiobook files | Available |
| DigitalOcean VPS | 24/7 monitoring | Running |

### Accounts & Access

| Service | Status | Details |
|---------|--------|---------|
| DigitalOcean | ✅ Active | SSH: `ssh root@157.245.36.127` |
| GitHub | ✅ Active | Repo: bristmatt96-hub/credit-catalyst |
| IBKR | ✅ Connected | Account U21710729, Gateway on port 4001 |
| Telegram Bot | ✅ Working | Token configured, credit notifications active |
| Anthropic API | ✅ Working | Used for filing analysis |
| Vercel | ✅ Active | Project: web, deploys dashboard frontend |
| GoDaddy | ✅ Active | Domain: mb-trading.co.uk |

### Domain & DNS (mb-trading.co.uk via GoDaddy)

| Subdomain | Type | Points To | Purpose |
|-----------|------|-----------|---------|
| app.mb-trading.co.uk | A | 157.245.36.127 | Dashboard API (VPS) |
| dashboard.mb-trading.co.uk | CNAME | cname.vercel-dns.com | Web dashboard (Vercel) |

### VPS Services (systemd)

| Service | Purpose | Commands |
|---------|---------|----------|
| `xvfb` | Virtual display for IB Gateway | `systemctl status xvfb` |
| `ibgateway` | IB Gateway (via IBC) | `systemctl status ibgateway` |
| `trading-system` | Main trading system | `systemctl status trading-system` |
| `dashboard` | Dashboard API (FastAPI/uvicorn) | `systemctl status dashboard` |
| `nginx` | HTTPS reverse proxy for API | `systemctl status nginx` |

**Quick Commands:**
```bash
# SSH into VPS
ssh root@157.245.36.127

# View live trading logs
journalctl -u trading-system -f

# View IB Gateway logs
journalctl -u ibgateway -f

# Restart everything
systemctl restart xvfb && sleep 2 && systemctl restart ibgateway && sleep 30 && systemctl restart trading-system

# Check all services
systemctl status xvfb ibgateway trading-system

# Take screenshot of IB Gateway
DISPLAY=:1 scrot /tmp/screen.png
```

---

## 8. Code Locations

| Location | Path | What's There |
|----------|------|--------------|
| GitHub | bristmatt96-hub/credit-catalyst | ✅ Source of truth |
| Mac Mini | /Users/matt/projects/credit-catalyst | Main development |
| VPS | /root/apex-s44-monitor | Production monitoring |

### Current State ✅
GitHub (source of truth)
|
v
Mac Mini (development)
|
v
VPS (production, 24/7 monitoring)

---

## 9. Watchlist Details

### Coverage
- **Total companies:** 202
- **iTraxx Crossover (HY):** 75 names — Priority 1
- **iTraxx Main (IG):** 125 names — Priority 2
- **Extras:** ATUS (Altice USA), MANU (Manchester United)

### SEC Filing Coverage (US Filers with CIKs)
Only 14 of 202 companies file with SEC:
- Altice USA, Nokia, Vodafone, BP, Shell, TotalEnergies, HSBC, AstraZeneca, Santander, Deutsche Bank, Barclays, BNP Paribas, Ericsson, Manchester United

### Gap: European Coverage
188 companies are European and don't file with SEC. Need to add:
- UK: RNS feed, Companies House
- EU: National regulatory filings

### Key Names to Watch (Aggressive Sponsor Risk)

| Company | Sponsor | Risk Level | Notes |
|---------|---------|------------|-------|
| Altice France | Drahi | High | History of asset moves |
| Altice USA | Drahi | High | Related party |
| Ardagh | PE-backed | High | Used unrestricted sub tactics |

### Key Names to Watch (Maturity Wall)

| Company | Key Maturities | Risk Level | Notes |
|---------|----------------|------------|-------|
| To be populated | 2027/2028 | | |

---

## 10. Knowledge Library

### Ingested Knowledge by Agent

**Total: 40 documents, ~660+ chunks across all agents**

#### The Analyst — `knowledge/credit/` (19 docs, ~170 chunks)
| Document | Source | Status |
|----------|--------|--------|
| Valuation Process | Moyer | ✅ Ingested |
| Covenants | Moyer | ✅ Ingested |
| Distressed Exchanges | Moyer | ✅ Ingested |
| Financial Issues | Moyer | ✅ Ingested |
| Ranking of Debt | Moyer | ✅ Ingested |
| Maturities and Calls | Moyer | ✅ Ingested |
| Amendments and Consents | Moyer | ✅ Ingested |
| CLOs | Moyer | ✅ Ingested |
| Coupons | Moyer | ✅ Ingested |
| Business Trend Analysis | - | ✅ Ingested |
| Credit Snapshot | - | ✅ Ingested |
| Data Science and Credit Analysis | - | ✅ Ingested |
| Decision Process | - | ✅ Ingested |
| Equity Info | - | ✅ Ingested |
| News Events | - | ✅ Ingested |
| New Issues | - | ✅ Ingested |
| Ownership and Management | - | ✅ Ingested |
| Portfolio Management | - | ✅ Ingested |
| Relative Value Analysis | - | ✅ Ingested |

#### The Psychologist — `knowledge/psychology/` (5 docs, 88 chunks)
| Document | Author | Chunks | Status |
|----------|--------|--------|--------|
| Trade Mindfully | - | 49 | ✅ Ingested |
| Updated Text for Market Wizards | Schwager | 33 | ✅ Ingested |
| Thinking in Bets | Annie Duke | 3 | ✅ Ingested |
| The Ideal Mindset | - | 3 | ✅ Ingested |
| LLM Sentiment Analysis (GenAI) | - | - | ✅ Ingested |

#### The Scanner — `knowledge/options/` (3 docs, 325 chunks)
| Document | Author | Chunks | Status |
|----------|--------|--------|--------|
| Option Volatility and Pricing | Natenberg | 198 | ✅ OCR'd & Ingested |
| Art & Science of Technical Analysis | Grimes | 122 | ✅ Ingested |
| Volume Price Analysis | Coulling | 5 | ✅ Ingested |

#### System Design — `knowledge/system_design/` + `knowledge/genai/` (14 docs, ~70 chunks)
| Document | Author | Chunks | Status |
|----------|--------|--------|--------|
| Quantitative Trading | Ernie Chan | 53 | ✅ Ingested |
| Inside the Black Box | Rishi Narang | 4 | ✅ Ingested |
| GenAI/ML Trading PDFs (12 docs) | Various | ~13 | ✅ Ingested |

#### The Risk Manager — `knowledge/credit/` (1 doc, ~10 chunks)
| Document | Source | Status |
|----------|--------|--------|
| Portfolio Management | - | ✅ Ingested |

### Books — Still to Process

**Audiobooks (on Windows PC — need transfer + Whisper transcription)**
| Book | Author | Agent | Status |
|------|--------|-------|--------|
| Trading in the Zone | Mark Douglas | Psychologist | Audiobook on Windows PC |
| Best Loser Wins | Tom Hougaard | Psychologist | Audiobook on Windows PC |
| One Good Trade | Mike Bellafiore | Psychologist | Audiobook on Windows PC |
| Alpha Trader | Brent Donnelly | Psychologist + Risk | Audiobook on Windows PC |
| The Predictive Edge | Alejandro Lopez-Lira | System Design | Audiobook on Windows PC |

**Kindle (need Calibre conversion)**
| Book | Author | Agent | Status |
|------|--------|-------|--------|
| Trading Options Greeks | Dan Passarelli | Scanner | Purchased — convert via Calibre |

---

## 11. What's Not Built Yet

| Component | Priority | Product | Notes |
|-----------|----------|---------|-------|
| ~~Enable IBKR API trading~~ | ~~Critical~~ | ~~Product 2~~ | ✅ Already enabled (ReadOnlyApi=no) |
| ~~Learning loop refinement~~ | ~~Medium~~ | ~~Product 2~~ | ✅ Implemented — edge component learner, pattern learner, trade_closed emission |
| Historical options research | Medium | Both | Pull past IV data for backtesting |
| Audiobook transcription | Medium | Both | 5 books on Windows PC, need transfer + Whisper |
| Passarelli Kindle conversion | Low | Scanner | Need Calibre to convert Trading Options Greeks |

### Recently Completed (v9 — 5 Feb 2026)
- ✅ Learning loop — EdgeComponentLearner adapts component weights from trade outcomes (24hr cycle, 20-trade min)
- ✅ Pattern learner — Grows pattern database from closed trades with auto-detected characteristics
- ✅ trade_closed emission — TradeExecutor now emits trade_closed with full edge score snapshot
- ✅ Coordinator fan-out — trade_closed flows to 4 learning systems (adaptive weights, edge learner, pattern learner, model manager)
- ✅ EdgeLearningConfig — Configurable guardrails (5-40% weight bounds, 15% max shift/day)
- ✅ Seed patterns — 4 hardcoded patterns moved to `data/patterns.json`

### Recently Completed (v8 — 5 Feb 2026)
- ✅ Psychologist knowledge — Ingested 4 new books: Thinking in Bets, Trade Mindfully, The Ideal Mindset, Market Wizards (88 chunks total)
- ✅ Scanner knowledge — Ingested 3 books: Natenberg (198 chunks, OCR'd from scanned PDF), Art & Science of TA (122 chunks), Volume Price Analysis
- ✅ System Design knowledge — Ingested Inside the Black Box + Quantitative Trading (57 chunks)
- ✅ Natenberg OCR — Decrypted encrypted PDF, ran OCR on 475 pages, extracted 198 chunks
- ✅ Knowledge index rebuilt — Full agent mapping in `knowledge/index.json`

### Completed in v7 (4 Feb 2026)
- ✅ Web dashboard — Next.js frontend deployed to Vercel at https://dashboard.mb-trading.co.uk
- ✅ Dashboard API — FastAPI backend with REST + WebSocket, running on VPS as systemd service
- ✅ HTTPS setup — nginx reverse proxy + Let's Encrypt on VPS (https://app.mb-trading.co.uk)
- ✅ Domain configured — mb-trading.co.uk (GoDaddy), `app` and `dashboard` subdomains
- ✅ Mobile access — Dashboard works on iPhone via https://dashboard.mb-trading.co.uk
- ✅ Excel dashboard — `APEX_Trading_Dashboard.xlsx` with 4 sheets (Dashboard, Opportunities, Trade History, Risk Monitor)
- ✅ Live Excel refresh — `refresh_dashboard.py` (one-shot) and `live_dashboard.py` (auto-refresh in Numbers)

### Previously Completed (v6)
- ✅ IBKR API connection — Gateway connected on VPS, port 4001
- ✅ VPS deployment — 24/7 operation with systemd services
- ✅ IBC automated login — Credentials configured in ~/ibc/config.ini
- ✅ GitHub sync — repo active at `bristmatt96-hub/credit-catalyst`
- ✅ Sponsor database — `data/sponsors.json`
- ✅ Maturity wall tracker — `data/maturity_wall.json`
- ✅ Situation classifier — integrated into analyst
- ✅ European regulatory monitoring — `monitors/european_monitor.py`
- ✅ Options Scanner — `agents/scanners/credit_options_scanner.py`
- ✅ Edge Scorer — `agents/edge_scorer.py`
- ✅ Psychologist agent — `agents/psychologist.py`
- ✅ Execution Engine — `agents/execution/trade_executor.py`
- ✅ Risk Manager — `agents/risk_manager.py`
- ✅ Notification System — Telegram integration complete

---

## 12. Action Plan

### Phase 1: Foundation ✅ COMPLETE
- ✅ VPS running with basic monitoring
- ✅ Mac Mini set up
- ✅ Code pulled from VPS
- ✅ .claude.md file in place
- ✅ GitHub repo synced

### Phase 2: Knowledge Base ✅ COMPLETE
- ✅ Created /knowledge folder organised by agent
- ✅ Ingested 19 credit PDFs (Moyer chapters + analysis docs)
- ✅ Ingested 12 GenAI/ML trading PDFs
- ✅ Knowledge index built (`knowledge/index.json`)
- 🔶 Audiobooks still to transcribe (Tier 1-2 psychology books)
- 🔶 Options books still to digitise (Natenberg, Passarelli)

### Phase 3: Credit Research ✅ COMPLETE
- ✅ Built sponsor database (`data/sponsors.json`)
- ✅ Identified aggressive sponsor risk names
- ✅ Built maturity wall tracker (`data/maturity_wall.json`)
- ✅ Researched Altice France timeline (`data/case_studies/altice_france.json`)
- ✅ Researched Ardagh timeline (`data/case_studies/ardagh.json`)
- ✅ Built situation classifier

### Phase 4: European Coverage ✅ COMPLETE
- ✅ Built European monitor (`monitors/european_monitor.py`)
- ✅ RNS/UK regulatory coverage implemented

### Phase 5: Options Integration ✅ COMPLETE
- ✅ Built Scanner agent (`agents/scanners/credit_options_scanner.py`)
- ✅ Options flow scanner (`agents/scanners/options_flow_scanner.py`)
- ✅ IV percentile calculation implemented
- ✅ Situation-dependent strategy logic in place

### Phase 6: Autonomous Agent ✅ COMPLETE
- ✅ Built Edge Scorer (`agents/edge_scorer.py` - 738 lines)
- ✅ Built Psychologist agent (`agents/psychologist.py` - 575 lines)
- ✅ Built Risk Manager with guardrails (`agents/risk_manager.py` - 557 lines)
- ✅ Built Execution Engine (`agents/execution/trade_executor.py` - 483 lines)
- ✅ Built Notification System (Telegram integration)
- ✅ Built Coordinator (`agents/coordinator.py`)
- ✅ IB Gateway deployed on VPS (IBC automated login)
- ✅ Trading system running 24/7
- 🔶 Learning Loop — basic tracking in place, refinement pending

### Phase 7: Dashboard & Monitoring ✅ COMPLETE
- ✅ Built web dashboard (Next.js + Tailwind, mobile-responsive)
- ✅ Built dashboard API (FastAPI + WebSocket for real-time updates)
- ✅ Deployed frontend to Vercel (https://dashboard.mb-trading.co.uk)
- ✅ Set up HTTPS on VPS (nginx + Let's Encrypt, https://app.mb-trading.co.uk)
- ✅ Configured domain (mb-trading.co.uk via GoDaddy)
- ✅ Built Excel dashboard with live API refresh
- ✅ Mobile access working (iPhone + Mac)

### Phase 8: Scale & Track Record 🟢 IN PROGRESS
- ✅ Configure IBKR API credentials (done: mbristow180 / U21710729)
- ✅ Deploy to VPS with 24/7 operation
- ✅ IB Gateway connected (port 4001, live mode)
- ✅ Dashboard live on web + mobile + Excel
- ⬜ Enable API trading in IBKR Client Portal (pending user action)
- ⬜ Monitor first trades
- ⬜ Document all trades and outcomes
- ⬜ Refine edge scoring based on results
- ⬜ Build 12-24 month track record
- ⬜ Prepare institutional pitch materials (Product 1)

---

## 13. Key Commands

### SSH Access
```bash
ssh root@157.245.36.127
```

### Trading System (on VPS)
```bash
# View live trading logs
journalctl -u trading-system -f

# Check all services
systemctl status xvfb ibgateway trading-system

# Restart trading system
systemctl restart trading-system

# Restart IB Gateway (if disconnected)
systemctl restart ibgateway

# Restart everything
systemctl restart xvfb && sleep 2 && systemctl restart ibgateway && sleep 30 && systemctl restart trading-system

# Stop everything
systemctl stop trading-system ibgateway xvfb
```

### IB Gateway Debugging
```bash
# View Gateway logs
journalctl -u ibgateway -f

# Check IBC logs
tail -50 /root/ibc/logs/ibc*.txt

# Take screenshot of Gateway GUI
DISPLAY=:1 scrot /tmp/screen.png

# Check if port is listening
ss -tlnp | grep 4001
```

### Dashboard
```bash
# View dashboard API logs
journalctl -u dashboard -f

# Check nginx (HTTPS proxy)
systemctl status nginx

# Renew SSL cert (auto, but manual if needed)
certbot renew

# Redeploy web dashboard
cd ~/Projects/credit-catalyst/web && npx vercel --prod --yes
```

### Excel Dashboard (Mac)
```bash
# One-time refresh from API
cd ~/Projects/credit-catalyst && source venv/bin/activate && python refresh_dashboard.py

# Auto-refresh while Numbers is open
python live_dashboard.py         # every 30s
python live_dashboard.py 60      # every 60s
```

### Old Monitor (legacy)
```bash
# View SEC filing monitor logs
cat /root/apex-s44-monitor/monitor.log
```

---

## 14. Key Learnings / Mistakes to Avoid

### From Ardagh / CDS Experience
- Buying CDS protection on LME candidates doesn't work as expected
- Asset Package Delivery mechanism fails for corporate LMEs
- Protection buyers ended up with illiquid equity they couldn't use
- **Lesson:** Focus on equity options, not CDS basis trades

### From General Experience
- Being early feels exactly like being wrong — need conviction to hold
- Credit signals can fire weeks/months before equity reprices
- Aggressive sponsors can make equity GO UP before collapse (asset stripping)
- Not every credit deterioration = buy puts

### Format Lessons
- Technical books (options, quant, credit) — ebook/PDF preferred
- Narrative books (psychology, memoirs) — audiobook fine
- Kindle — Calibre + DeDRM
- Paperback — scan or find PDF

---

## 15. Success Metrics

### Product 2 (Autonomous Agent) — Year 1 Targets

| Metric | Target | Rationale |
|--------|--------|-----------|
| Win rate | >55% | Edge exists if consistently above 50% |
| Average win/loss ratio | >2:1 | Let winners run, cut losers |
| Max drawdown | <20% | Survive to trade another day |
| Sharpe ratio | >1.5 | Risk-adjusted returns matter |
| Edge score correlation | >0.6 | High scores should predict wins |
| Trades per month | 5-15 | Patient, selective trading |

### Product 1 (Institutional) — Year 2 Targets

| Metric | Target | Rationale |
|--------|--------|-----------|
| Track record length | 18+ months | Institutions want history |
| Documented trades | 100+ | Enough data to analyse |
| Signal accuracy | >60% | Credit signals that led to moves |
| Unique value prop | Clear | Why this vs Bloomberg/existing tools |

---

## 16. Web Dashboard Architecture

### Frontend (Vercel)
- **URL:** https://dashboard.mb-trading.co.uk
- **Stack:** Next.js 14, Tailwind CSS, TypeScript
- **Pages:** Dashboard (P&L + positions), Opportunities, Positions detail, Trade History
- **Auto-refresh:** Every 30 seconds via polling
- **Source:** `web/` directory
- **Deploy:** `npx vercel --prod --yes` from `web/`

### Backend API (VPS)
- **URL:** https://app.mb-trading.co.uk
- **Stack:** FastAPI, uvicorn, WebSocket support
- **Endpoints:** `/api/pnl`, `/api/positions`, `/api/opportunities`, `/api/health`
- **Source:** `dashboard/` directory
- **HTTPS:** nginx reverse proxy → localhost:8000, Let's Encrypt cert (expires 5 May 2026, auto-renews)
- **nginx config:** `/etc/nginx/sites-enabled/dashboard`
- **Cert:** `/etc/letsencrypt/live/app.mb-trading.co.uk/`

### Excel Dashboard (Mac)
- **File:** `APEX_Trading_Dashboard.xlsx`
- **Sheets:** Dashboard, Opportunities, Trade History, Risk Monitor
- **Refresh scripts:** `refresh_dashboard.py` (one-shot), `live_dashboard.py` (auto-refresh in Numbers)
- **LaunchAgent:** `~/Library/LaunchAgents/com.apex.live-dashboard.plist` (disabled by default, load to auto-start)

---

End of document. Version 8.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bristmatt96-hub)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bristmatt96-hub)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
