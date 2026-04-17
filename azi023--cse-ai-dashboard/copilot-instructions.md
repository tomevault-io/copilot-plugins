## cse-ai-dashboard

> > This file is the single source of truth for Claude Code working on this project.

# CSE AI Investment Intelligence Dashboard — CLAUDE.md

> This file is the single source of truth for Claude Code working on this project.
> Read it fully before writing any code. Re-read it at the start of every session.

---

## 🔴 ABSOLUTE RULES (Non-Negotiable)

1. **The `.env` file must NEVER be touched, modified, read, cat'd, echo'd, or deleted under ANY circumstances.** Use `process.env.*` to access values the app already loads. This rule overrides ALL other instructions.
2. **Never place orders, click buy/sell buttons, or modify account data on ATrad.** All ATrad automation is READ-ONLY (scrape holdings, balances, account data).
3. **Never run destructive database operations** (DROP, TRUNCATE, DELETE without WHERE) without explicit user approval.
4. **Never install npm packages without stating which ones and why first.**
5. **Never commit `.env`, `node_modules`, or personal financial data to git.**

---

## Project Overview

**What:** A personal AI-powered Shariah-compliant investment intelligence platform for the Colombo Stock Exchange (CSE).

**Why:** Increase Sri Lankan household stock market participation through accessible, AI-driven, Shariah-filtered analysis. Currently a personal tool — future public platform.

**Who:** Single user (Atheeque), conservative risk profile, LKR 10,000/month Rupee Cost Averaging strategy, Shariah-compliant only.

**Where:** Runs locally on WSL2 Ubuntu. Managed via PM2 for persistence. No cloud deployment yet.

---

## Tech Stack

| Layer | Technology | Port/Path |
|-------|-----------|-----------|
| Frontend | Next.js 14 + TypeScript + Tailwind + shadcn/ui | localhost:3000 |
| Backend | NestJS + TypeScript | localhost:3001 |
| Database | PostgreSQL 16 | localhost:5433 |
| Cache | Redis 7 | localhost:6379 |
| Browser Automation | Playwright (Chromium) | Headless |
| AI | Claude API (Haiku for digests, Sonnet for analysis) | api.anthropic.com |
| Charts | TradingView Lightweight Charts + Recharts | — |
| Process Manager | PM2 | — |

**Repo:** `~/workspace/cse-ai-dashboard/`
**GitHub:** `https://github.com/Azi023/cse-ai-dashboard.git`

---

## Architecture

```
src/
├── backend/          # NestJS API server
│   └── src/
│       └── modules/
│           ├── cse-data/        # CSE API polling, stock sync, market data
│           ├── ai-engine/       # Claude API integration, briefs, signals, chat
│           ├── portfolio/       # Holdings, P&L, manual add
│           ├── shariah/         # Shariah screening (Almas whitelist)
│           ├── notifications/   # Daily digest, weekly brief, alerts
│           ├── analysis/        # [TO BUILD] Scoring engine, recommendations
│           ├── atrad-sync/      # ATrad Playwright browser automation
│           ├── news/            # RSS feed aggregation
│           ├── journey/         # Investment journey tracking, KPIs
│           ├── dividends/       # Dividend tracking, purification
│           ├── alerts/          # Price alerts, notifications bell
│           ├── signals/         # AI-generated trading signals
│           └── macro/           # CBSL macro data
├── frontend/         # Next.js dashboard UI
│   └── src/app/
│       ├── dashboard/    # Market overview, AI brief
│       ├── stocks/       # Stock browser, detail pages
│       ├── portfolio/    # Holdings, P&L, ATrad sync
│       ├── journey/      # Investment journey, KPIs, goals
│       ├── signals/      # AI signals display
│       ├── alerts/       # Notifications
│       ├── chat/         # AI Strategy Chat
│       └── news/         # News intelligence
├── scripts/          # Standalone scripts (ATrad recon, data tools)
└── data/             # Generated data, AI outputs, ATrad sync dumps
    ├── ai-generated/
    ├── atrad-sync/
    └── tracking/     # KPI tracker (gitignored)
```

---

## Data Sources

| Source | Method | Frequency | Notes |
|--------|--------|-----------|-------|
| CSE Market Data | POST to cse.lk/api/* (22 endpoints) | 5 min during market hours | Reverse-engineered, no auth needed |
| ATrad (HNB Stockbrokers) | Playwright browser automation | On-demand via Sync button | READ-ONLY. Login → scrape holdings + balance |
| Almas Equities Whitelist | Manual / twice-weekly screening | Mon & Thu 9:00 AM | Shariah compliance source of truth |
| RSS News | Economy Next, Daily FT, Google News CSE | Every 30 min (8AM-8PM weekdays) | daily_ft feed frequently fails to parse |
| CBSL Macro | Manual Excel import | As released | OPR, inflation, FX reserves |
| Claude AI | API calls for briefs, signals, digests | Cached with TTLs | Haiku for digests, Sonnet for analysis |

---

## Cron Schedule (All times Sri Lanka Time, UTC+5:30)

| Time | Job | Days | What It Does |
|------|-----|------|-------------|
| 9:25 AM | preMarketWarmup | Mon-Fri | Fetch initial market data before open |
| 9:30-14:30 | pollMarketData | Mon-Fri | Every 5 min: 7 CSE endpoints + trade summary |
| 9:30-14:30 | pollAnnouncements | Mon-Fri | Every 15 min: financial + approved announcements |
| 8:00-20:00 | fetchNews | Mon-Fri | Every 30 min: RSS feeds |
| 9:00 AM | shariahScreening | Mon, Thu | Twice-weekly Shariah compliance check |
| 2:35 PM | postCloseSnapshot | Mon-Fri | Final market data + trade summary after close |
| 2:40 PM | saveMarketSnapshot | Mon-Fri | **[TO BUILD]** Save daily snapshot for analysis |
| 2:42 PM | runStockScoring | Mon-Fri | **[TO BUILD]** Calculate composite stock scores |
| 2:45 PM | generateDailyDigest | Mon-Fri | Haiku: market summary + portfolio P&L |
| 2:55 PM | generateAIRecommendation | Fri | **[TO BUILD]** Sonnet: weekly stock pick |
| 3:00 PM | generateWeeklyBrief | Fri | Sonnet: week review + forward outlook |
| 6:00 PM | afterHoursAnnouncements | Mon-Fri | Single check for late filings |

**Off-hours (nights, weekends): ZERO external API calls.**

---

## Current State (as of March 19, 2026)

### What's Working ✅
- Market data polling with optimized cron (5-min market hours, zero off-hours)
- AI Market Brief (Sonnet, cached 4h)
- AI Signals (5 signals from cache, JSON parsing fixed)
- AI Strategy Chat (live Claude API with market context)
- News Intelligence (RSS from EconomyNext, Google News CSE)
- Announcements (financial + approved, from CSE API)
- ATrad Playwright login + Account Summary scraping (buyingPower, cashBalance confirmed)
- Daily Digest notifications (Haiku, 2:45 PM) — includes crash alerts, portfolio P&L
- Weekly Brief (Sonnet, Friday 3:00 PM) — includes AI recommendation + top 5 scores
- Token budget guard (500K/month, auto-downgrades Sonnet → Haiku)
- Shariah screening (twice-weekly, skips if recent)
- Journey page with deposit tracking, T+2 pending state display
- Portfolio page with manual Add Holding (with broker fees field) + ATrad sync banner
- Premium UI redesign (Bloomberg Terminal dark theme, Inter + JetBrains Mono fonts)
- AEL.N0000 holding added (200 shares @ 69.50 + LKR 155.69 fees = LKR 70.28/share effective)
- **Shariah data seeded**: 11 compliant, 34 non-compliant, remainder pending review
- **Phase 2 AI Analysis Pipeline**: snapshot accumulation started (day 1), scoring ready after 20 market days
- **All 55 backend endpoints tested**: all HTTP 200 (QA pass March 19, 2026)
- **PM2 ecosystem.config.js**: created for persistent process management
- ATrad cron timezone bug fixed (market-hours sync now fires correctly 9:30-2:30 PM SLT)
- Journey thisMonthReturn calculation bug fixed (was double-counting deposits)
- ATrad post-close sync at 2:38 PM SLT (before portfolio snapshot at 2:40 PM)

### What's Pending / Broken ⚠️
- **ATrad Stock Holding scraping returns 0 holdings** — T+2 settlement cleared March 18, need to trigger a manual sync and verify post-settlement `portfolios` array shape. Account Value selector still reads implausible numbers (account number in adjacent field).
- **Stock scoring engine needs 20 market days** — accumulating since March 19. Scoring will be meaningful around April 16, 2026.
- **Shariah Tier 2 screening** — requires quarterly financial ratio data import. Most stocks remain PENDING_REVIEW until that data is available.
- **Backtester `/api/backtester/symbols`** — returns 404. Controller registered but handler route mismatch. Low priority.

### What Needs Building 🔨
- **ATrad holdings selector fix** — post-settlement verification + Account Value field selector repair
- **Shariah Tier 2 financial data** — import quarterly reports to enable ratio screening
- **Historical accuracy tracking** — did past AI recommendations perform? Track in `ai_recommendations` table

---

## User's Investment Profile

- **Capital:** LKR 20,000 initial deposit + LKR 10,000/month RCA
- **Current Holdings:** 200 shares AEL.N0000 @ LKR 69.50 (bought March 16, 2026)
- **Total Cost:** LKR 14,055.69 (incl. fees). Cost basis: LKR 70.28/share
- **Remaining Cash:** LKR 5,944.32
- **Broker:** HNB Stockbrokers, ATrad platform, CDS account active
- **Strategy:** Shariah-compliant Rupee Cost Averaging, conservative risk
- **Screening:** AAOIFI / Meezan / Dow Jones via Almas Equities Whitelist
- **Next Trade:** TJL.N0000 after CBSL rate decision March 25

---

## Workflow Orchestration

### 1. Plan Mode Default
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- Write plan to `tasks/todo.md` with checkable items before coding
- If something goes sideways, STOP and re-plan immediately
- Write detailed specs upfront to reduce ambiguity

### 2. Verification Before Done
- Never mark a task complete without proving it works
- Run `npx tsc --noEmit` after every TypeScript change
- Test endpoints with `curl` after API changes
- Check browser UI after frontend changes
- Ask: "Would a staff engineer approve this?"

### 3. Self-Improvement Loop
- After ANY correction from the user, update `tasks/lessons.md` with the pattern
- Write rules that prevent the same mistake
- Review lessons at session start

### 4. Demand Elegance (Balanced)
- For non-trivial changes: pause and ask "is there a more elegant way?"
- Skip this for simple, obvious fixes — don't over-engineer
- Challenge your own work before presenting it

### 5. Autonomous Bug Fixing
- When given a bug report: just fix it. Don't ask for hand-holding
- Point at logs, errors, failing tests — then resolve them
- Zero context switching required from the user

---

## Task Management

1. **Plan First:** Write plan to `tasks/todo.md` with checkable items
2. **Verify Plan:** Check in before starting implementation
3. **Track Progress:** Mark items complete as you go
4. **Explain Changes:** High-level summary at each step
5. **Document Results:** Add review section to `tasks/todo.md`
6. **Capture Lessons:** Update `tasks/lessons.md` after corrections

---

## Core Principles

- **Simplicity First:** Make every change as simple as possible. Minimal code impact.
- **No Laziness:** Find root causes. No temporary fixes. Senior developer standards.
- **Minimal Impact:** Only touch what's necessary. No side effects with new bugs.
- **Cost Awareness:** Every Claude API call costs money. Cache aggressively. Use Haiku for summaries, Sonnet only for analysis that needs reasoning.
- **Data Integrity:** Financial data must be accurate. Never show mock data as real. Never display misleading P&L figures.

---

## Tasks: Phase 1 — PM2 Setup + Stability

### Task 1.1: PM2 Process Management
```bash
npm install -g pm2
cd ~/workspace/cse-ai-dashboard/src/backend
pm2 start "npm run start:dev" --name cse-backend --watch false
cd ~/workspace/cse-ai-dashboard/src/frontend  
pm2 start "npm run dev" --name cse-frontend --watch false
pm2 save
pm2 startup
```
Create `ecosystem.config.js` in project root for PM2 configuration.

### Task 1.2: ATrad Holdings Verification (Post-Settlement)
- Run the recon script: `cd src/backend && npx tsx ../../scripts/atrad-recon.ts`
- Confirm 200 AEL.N0000 shares appear in `portfolios` array
- If they do, wire the recon logic into `atrad-browser.ts` production sync
- Fix the Account Value selector to avoid reading account number as value

### Task 1.3: Verify All Cron Jobs Fire Correctly
- Start backend, observe logs for one full market day
- Confirm: preMarketWarmup (9:25), market polling (every 5 min 9:30-14:30), postCloseSnapshot (14:35), dailyDigest (14:45)
- Confirm: ZERO polling after 14:35 except news (until 20:00) and announcements (18:00)

---

## Tasks: Phase 2 — AI Analysis Pipeline

### Task 2.1: Data Accumulation Service
Create `src/backend/src/modules/analysis/analysis.service.ts`

**Daily at 2:40 PM SLT:**
- Save market snapshot to `market_snapshots` table:
  - date (unique), aspi_close, aspi_change_pct, sp20_close, volume, turnover, trades
  - top_gainers (jsonb), top_losers (jsonb), sector_performance (jsonb)
- Save portfolio snapshot to `portfolio_snapshots` table:
  - date (unique), total_value, total_invested, unrealized_pl, holdings (jsonb)

**Weekly on Friday at 2:50 PM:**
- Calculate weekly metrics → `weekly_metrics` table:
  - week_start (unique), week_end, aspi_return, portfolio_return, best_holding, worst_holding

### Task 2.2: Stock Scoring Engine
Create deterministic scoring (NO AI needed):

For each Shariah-compliant stock, calculate composite score:
- Dividend yield: 30% weight
- Price momentum (current vs 20-day avg): 20% weight
- Volume trend (today vs 20-day avg): 10% weight
- Volatility (std dev of daily returns): 15% weight
- Sector strength (weekly performance): 15% weight
- Liquidity (avg daily turnover LKR): 10% weight

Store in `stock_scores` table (date, symbol, composite_score, components jsonb).
Run daily at 2:42 PM after market snapshot is saved.
Needs 20+ days of accumulated data to be meaningful — output placeholder scores until then.

### Task 2.3: AI Investment Recommendation (Weekly)
**Friday at 2:55 PM** (after scoring runs):
- Call Claude Sonnet with structured data:
  - This week's market snapshots
  - Current portfolio + cost basis
  - Top 10 compliant stocks by composite score
  - Known upcoming events (CBSL meetings, earnings — from config JSON)
- Request JSON output with: recommended_stock, confidence, reasoning, 3m_outlook, risk_flags
- Save to `ai_recommendations` table
- Create alert notification for the bell

### Task 2.4: Dashboard Integration
- New "AI Advisor" section on Journey page OR dedicated page
- Show: latest recommendation + confidence badge
- Show: stock scoring leaderboard (top 10 compliant)
- Show: data accumulation status ("X days of data collected, need 20 for scoring")
- Show: historical accuracy (did past recommendations perform?)

### Task 2.5: Enhanced Notifications
**Daily digest improvements:**
- Include portfolio P&L when holdings exist
- Include each holding's daily price change
- Flag any stock dropping > 5%
- Flag ASPI dropping > 3% (Crash Protocol reminder)

**Weekly brief improvements:**
- Include AI recommendation from Task 2.3
- Include top 5 stock scores
- Include week-over-week portfolio comparison
- Include next month's suggested action

---

## Known Patterns & Lessons

### ATrad Dojo UI
- Login: `#txtUserName`, `#txtPassword`, `#btnSubmit`
- Client menu: `#dijit_PopupMenuBarItem_4` → dropdown → `#dijit_MenuItem_40` (Stock Holding), `#dijit_MenuItem_41` (Account Summary)
- Account balance: `txtAccSumaryCashBalance`, `txtAccSumaryBuyingPowr`
- Holdings API: POST `/atsweb/client` with `action=getStockHolding&exchange=CSE&broker=FWS&stockHoldingClientAccount=128229LI0&format=json`
- **Single-quote JSON normalization required** before `JSON.parse` on ATrad API responses
- `#gridContainer4 table` is the market watch table, NOT the holdings table — don't confuse them
- Leave `stockHoldingSecurity` empty for all-holdings query
- Logout: `#butUserLogOut`

### CSE API
- 22 endpoints at `https://www.cse.lk/api/*` — all POST, no auth
- `marketSummery` (sic — CSE's typo, not ours)
- `tradeSummary` returns 296 stocks with price, change, volume
- Rate limit unknown — keep polling to 5-min intervals minimum

### AI/Claude API
- Haiku (`claude-haiku-4-5-20251001`): Use for daily digests, simple summaries. ~$0.01/call
- Sonnet (`claude-sonnet-4-6`): Use for weekly briefs, recommendations, deep analysis. ~$0.05/call
- Always request raw JSON output (no markdown fences) for structured data
- Cache aggressively: daily brief 4h, signals 20h, digests 24h
- Monthly budget guard at 500K tokens — auto-downgrade Sonnet → Haiku above threshold

### Common Mistakes to Avoid
- TypeORM 0.3+: `findOne()` REQUIRES a `where` clause — `findOne({ order: ... })` throws
- ATrad returns account number in adjacent fields — filter values > 50M as implausible
- CSE price data in AI briefs can be stale — always validate against live data
- Don't show -100% P&L when portfolio has no holdings (T+2 settlement lag)
- Don't run Strategy Test on mock data — confirm real data pipeline first
- `lastTradedPrice` and `priceChange` are wrong field names in Redis cache — correct: `price` and `change`

---

## Git Workflow

```bash
# After every meaningful change:
npx tsc --noEmit                    # Verify no TS errors
git add -A
git commit -m "feat|fix|refactor(module): description"
# Push only when explicitly asked
```

**Branch strategy:** Work on `main` for now (single developer). Create feature branches for large changes.

**Never commit:** `.env`, `node_modules/`, `data/tracking/`, `data/atrad-sync/*.html`, `*.png` screenshots

---

## File Locations Quick Reference

| What | Where |
|------|-------|
| Backend entry | `src/backend/src/main.ts` |
| Cron schedules | `src/backend/src/modules/cse-data/cse-data.service.ts` |
| AI prompts | `src/backend/src/modules/ai-engine/prompts.ts` |
| ATrad browser | `src/backend/src/modules/atrad-sync/atrad-browser.ts` |
| Notifications | `src/backend/src/modules/notifications/notifications.service.ts` |
| Portfolio | `src/backend/src/modules/portfolio/portfolio.service.ts` |
| Journey | `src/backend/src/modules/journey/journey.service.ts` |
| Shariah | `src/backend/src/modules/shariah/shariah-screening.service.ts` |
| Frontend pages | `src/frontend/src/app/*/page.tsx` |
| Recon script | `scripts/atrad-recon.ts` |
| PM2 config | `ecosystem.config.js` (to create) |
| Task tracking | `tasks/todo.md` (to create) |
| Lessons learned | `tasks/lessons.md` (to create) |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Azi023) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
