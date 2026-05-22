## worfin-strategies

> **The project now operates on TWO parallel tracks: Metals (primary, paper) and FX (secondary, paper → live).**

# CLAUDE.md — WorFIn Systematic Trading System
# Master Project Brief | Updated: April 2026
# Tim — CIO/CTO | Systematic Commodity + FX Trading

---

## 🔴 READ THIS FIRST — APRIL 2026 PIVOT

**The project now operates on TWO parallel tracks: Metals (primary, paper) and FX (secondary, paper → live).**

The metals thesis is unchanged. £5k live capital is below the metals futures threshold, so metals continues paper-trading per roadmap while FX is added as the capital-efficient live track on IBKR IdealPro. Read `WorFIn_Pivot_Memo.md` in project knowledge before starting any substantive work.

Do NOT relitigate this decision. Do NOT treat FX as a replacement for metals. If a user request is ambiguous about which book they mean, ASK.

---

## 🎯 PROJECT IDENTITY

We are building a **production-grade systematic trading system** across two asset classes:

- **Metals** (primary): Six statistically-validated strategies on LME base metals and COMEX precious metals. Daily holding periods. Fully automated signal generation. Interactive Brokers execution. Target live capital ≥ £50k.
- **FX** (secondary): Three strategies on G10 FX pairs via IBKR IdealPro. Capital-efficient at £5–50k. Permanent low-correlation diversifier thereafter.

This is a real trading business — every line of code has financial consequences.

**5-Year Roadmap:**
- **Now (Tier 0 → 1):** Backtesting infrastructure + paper trading (both books)
- **Near-term:** Live FX at £5k when paper cleared. Metals stays paper until capital ≥ £50k.
- **Year 2:** Live metals capital £50–100k personal (target H2 2027)
- **Year 4:** Fund launch £2–5m external AUM
- **Year 5:** Cape Town, £10–20m AUM, 3–4 person firm

---

## 🧠 CORE PHILOSOPHY — READ THIS FIRST

1. **Risk management IS the strategy.** Never treat risk as a constraint bolted onto alpha — it IS the central organising principle.
2. **A backtest is a hypothesis, not a result.** Only out-of-sample and live paper trading confirm edge.
3. **Overfitting is the #1 threat.** More parameters = more danger. Simpler is better.
4. **Infrastructure before capital.** Nothing goes live until paper-tested for 60+ trading days (G3 gate).
5. **One strategy done properly beats five done poorly.** S4 (Basis-Momentum) is the metals core. Build it first.
6. **Never size by conviction. Size by volatility.** Always inverse-vol targeting — applies identically to metals and FX.
7. **The kill switch must always work.** Every component must be stoppable in <60 seconds.
8. **FX is a parallel track, not a replacement.** Do not let FX work delay metals validation.

---

## 📐 CURRENT PHASE & ACTIVE WORK

**Phase:** Tier 0 → Tier 1 transition
**Active tracks:**
- Metals: complete S4 Basis-Momentum IS backtest (G0 gate)
- FX: DESIGN ONLY until metals S4 clears G0; do not start FX build yet

**Sequencing (strict order):**

1. Step 0 verification pass on files Tim added (`continuous.py`, `002_pnl_accounting.py`, `003_fx_rates.py`, `pretrade_intergation.py`)
2. S4 Basis-Momentum IS backtest (2005–2017) — must clear G0 (IS Sharpe ≥ 0.50, t-stat ≥ 3.0, max DD ≤ 20%)
3. **Asset-class-agnostic refactor** (`InstrumentSpec` hierarchy, `CostModel` polymorphism, strategy parameterisation)
4. FX data layer (FRED short rates, FX historicals, `FXSpec` instances)
5. FX backtests (FX1, FX2, FX3) — apply G0, G1 gates
6. FX paper trading (alongside metals paper, 60+ trading days)
7. Live FX at £5k when G3 cleared
8. Metals paper continues until capital ≥ £50k and metals G3 cleared, then flip to live

**When I ask you to build something, always check:**
- Which book is it for — metals, FX, or asset-agnostic?
- Does this serve the current phase?
- Is it the simplest implementation that works?
- Does it respect all risk limits defined below?
- Is it testable and auditable?
- Am I about to work on FX before metals S4 has cleared G0? (If yes: STOP.)

---

## 🏗️ TECH STACK

| Component | Choice | Notes |
|-----------|--------|-------|
| Language | Python 3.11+ | Primary for all components |
| Database | PostgreSQL 18 | Run locally for dev; migrate to VPS for live |
| ORM | SQLAlchemy 2.0 | Async-capable; use for all DB interaction |
| Migrations | Alembic | All schema changes versioned — never raw ALTER TABLE |
| Broker API | ib_insync | IBKR connectivity; IB Gateway (not TWS) in production |
| Broker venues | ICEEU (LME), NYMEX/COMEX (metals), IdealPro (FX) | All via single IBKR account |
| Backtesting | vectorbt | Fast vectorised; custom event-driven layer on top |
| Data | pandas + numpy | Standard; use polars for large datasets if needed |
| Stats | scipy + statsmodels | ADF tests, cointegration, regression, GARCH |
| Vol modelling | arch | GARCH volatility estimation |
| Metrics | Custom | empyrical | Sharpe, Sortino, Calmar, drawdown |
| Scheduling | schedule (dev) → APScheduler (prod) → Airflow (Tier 2+) |
| Monitoring | Telegram bot + structured logging | JSON logs with correlation_id; alerts to Telegram |
| Testing | pytest + pytest-asyncio | All strategy logic must have unit tests |
| Environment | pyenv + venv | Never use conda |
| Secrets | python-dotenv (.env file) | Never commit .env |
| Version control | Git + GitHub (private) | Feature branches; no direct commits to main |

---

## 🗄️ DATABASE SCHEMA — CANONICAL STRUCTURE

The schema is asset-class-agnostic. Same tables serve metals and FX.

```sql
-- LAYER 1: Raw ingest (immutable — never modify after insert)
schema: raw_data
  tables: lme_prices, comex_prices, lme_inventory, cftc_cot, macro_indicators,
          fred_rates (NEW: short rates for FX carry),
          fx_prices (NEW: spot + forward points for G10)

-- LAYER 2: Clean, normalised, roll-adjusted
schema: clean_data
  tables: futures_prices, continuous_series, realised_vol, term_structure,
          fx_spot, fx_forwards (NEW)

-- LAYER 3: Computed signals
schema: signals
  tables: carry_signals, momentum_signals, basis_signals, inventory_signals, pairs_signals
  -- Same tables serve metals and FX; disambiguated via instrument_id foreign key

-- LAYER 4: Portfolio & execution
schema: positions
  tables: target_positions, current_positions, position_history

schema: orders
  tables: order_log, fill_log, execution_quality

-- LAYER 5: Audit & monitoring
schema: audit
  tables: system_events, data_quality_flags, reconciliation_log, risk_breaches, roll_log
```

**Rules:**
- Never query across schemas in strategy code — use views
- raw_data is append-only, never update or delete
- Every table has created_at and updated_at timestamps (TIMESTAMPTZ)
- All tables carry `bar_size` and `valid_from`/`valid_until` for intraday-readiness
- P&L tables carry `environment` and `backtest_run_id` columns for run comparison
- FX P&L settles to GBP via daily close rate; audit flag for stale FX rates (>7 days)

---

## 📊 STRATEGY UNIVERSE — QUICK REFERENCE

### Metals Book (primary, paper-only until £50k)

| ID | Name | Type | Sharpe | Hold | Allocation | Status |
|----|------|------|--------|------|------------|--------|
| S1 | Term Structure Carry | Carry | 0.4–0.8 | 5–20d | 20% | Built |
| S2 | Time-Series Momentum | Trend | 0.5–1.0 | 5–15d | 20% | Pending |
| S3 | Cross-Sectional Momentum | RelStr | 0.3–0.6 | 10–30d | 15% | Pending |
| S4 | Basis-Momentum | Composite | 0.6–1.0 | 10–20d | 25% | Built — IS backtest next |
| S5 | Inventory Surprise | Event | 0.3–0.5 | 3–10d | 10% | Pending |
| S6 | Inter-Metal Spreads | StatArb | 0.4–0.7 | 5–20d | 10% | Pending |

**Metals universe:**
- LME Base: Copper (CA), Aluminium (AH), Zinc (ZS), Nickel (NI), Lead (PB), Tin (SN)
- COMEX Precious: Gold (GC), Silver (SI), Platinum (PL), Palladium (PA)

### FX Book (secondary, paper → live at £5k)

| ID | Name | Type | Sharpe | Hold | Rebalance | Allocation | Status |
|----|------|------|--------|------|-----------|------------|--------|
| FX1 | FX Carry | Carry | 0.4–0.6 | 5–20d | Weekly | 40% | Design |
| FX2 | FX TSMOM | Trend | 0.6–0.9 | 5–15d | Daily | 40% | Design |
| FX3 | FX XS Momentum | RelStr | 0.3–0.5 | 10–30d | Bi-weekly | 20% | Design |

**FX universe (9 G10 pairs via IBKR IdealPro):**
- EUR/USD, GBP/USD, USD/JPY, USD/CHF, AUD/USD, USD/CAD, NZD/USD, USD/SEK, USD/NOK
- Normalised to "foreign vs USD" internally for ranking
- Target portfolio vol: 6–7% annualised (conservative; step to 10% after live validation)
- Benchmark: equal-weighted long basket of 9 normalised pairs

Full specs: `Metals_Systematic_Trading_Strategies.pdf`, `FX_Strategy_Universe.md`.

---

## 🚀 EXECUTION RULES

### Metals (LME + COMEX)

- LME Base Metals: Execute 14:00–16:00 London (deepest liquidity)
- COMEX Precious: Execute 14:00–16:00 London (London/NY overlap)
- S1/S4: Execute AFTER LME official prices published (~13:30 London)
- S5: Execute 30min AFTER LME inventory report (~09:30 London)
- LME rolls: ALWAYS as single spread order (never leg separately)

### FX (IdealPro)

- FX1 Carry: Execute Monday open (~08:00 London) after Friday-close rebalance
- FX2 TSMOM: Execute 17:00 UTC daily (end of NY session, before Asia open)
- FX3 XS Mom: Execute 17:00 UTC on bi-weekly schedule
- No roll management (spot FX has no expiry)
- Financing via IBKR daily rollover (swap points embedded in P&L)

### Order management (both books, always follow this sequence)

1. Passive limit at mid-price (60 seconds)
2. Aggressive limit at best bid/offer (60 seconds)
3. Market order fallback (should be <10% of executions)

### Pre-trade checks (MANDATORY before every order, both books)

- Order within position limits
- Won't breach gross/net exposure limits
- Within liquidity tier maximum (rolling 20-day ADV for futures; spread tier for FX)
- Price within 2% of current mid (fat-finger protection)
- Daily order count within maximum
- Direction matches signal (prevents sign errors)
- FX-specific: FX rate staleness <7 days (audit flag, never silent fallback)

---

## 📁 PROJECT STRUCTURE (Post-Refactor Target)

```
worfin_strategies/
├── CLAUDE.md                        ← This file (root)
├── WorFIn_Pivot_Memo.md             ← April 2026 pivot context
├── FX_Strategy_Universe.md          ← FX strategy spec
├── .env                             ← Secrets (never commit)
├── .env.example                     ← Template (commit this)
├── pyproject.toml
├── alembic/
│   └── versions/
├── src/worfin/
│   ├── config/
│   │   ├── CLAUDE.md
│   │   ├── settings.py              ← All config via pydantic-settings
│   │   ├── instruments.py           ← InstrumentSpec base class
│   │   ├── metals.py                ← MetalSpec + metal instances
│   │   ├── fx.py                    ← FXSpec + FX pair instances
│   │   └── calendar.py              ← LME + COMEX + FX holiday calendars
│   ├── data/
│   │   ├── CLAUDE.md
│   │   ├── ingestion/
│   │   │   ├── lme.py
│   │   │   ├── comex.py
│   │   │   ├── fred.py              ← FX rates + short rates for FX carry
│   │   │   └── ibkr_hist.py         ← Unified historical via IBKR
│   │   ├── pipeline/
│   │   └── models.py                ← Asset-class-agnostic schema
│   ├── strategies/
│   │   ├── CLAUDE.md
│   │   ├── base.py                  ← Abstract base; universe-parameterised
│   │   ├── carry.py                 ← Used by S1 (metals) + FX1
│   │   ├── tsmom.py                 ← Used by S2 (metals) + FX2
│   │   ├── xs_momentum.py           ← Used by S3 (metals) + FX3
│   │   ├── basis_momentum.py        ← S4 — metals-specific
│   │   ├── inventory.py             ← S5 — metals-specific
│   │   └── pairs.py                 ← S6 — metals-specific
│   ├── costs/
│   │   ├── model.py                 ← CostModel interface
│   │   ├── futures.py               ← FuturesCostModel (tick-based)
│   │   └── fx.py                    ← FXCostModel (bps-based)
│   ├── risk/
│   │   ├── CLAUDE.md
│   │   ├── limits.py                ← All hard limits defined here
│   │   ├── sizing.py                ← Vol targeting (asset-agnostic)
│   │   ├── monitor.py               ← Real-time risk monitoring
│   │   └── circuit_breakers.py     ← Automated drawdown triggers
│   ├── execution/
│   │   ├── CLAUDE.md
│   │   ├── engine.py                ← Core execution loop
│   │   ├── orders.py                ← Order management
│   │   ├── broker/
│   │   │   ├── base.py              ← Broker interface
│   │   │   └── ibkr.py              ← IBKR adapter — futures + FX
│   │   └── pretrade_checks.py       ← All pre-trade validations
│   ├── backtest/
│   │   ├── CLAUDE.md
│   │   ├── engine.py                ← Walk-forward backtester
│   │   ├── metrics.py               ← All performance metrics (no empyrical)
│   │   └── validation.py            ← WFER, PBO, stress tests
│   └── monitoring/
│       ├── alerts.py                ← Telegram + email alerts
│       ├── reconciliation.py        ← Daily position reconciliation
│       ├── data_quality.py          ← Staleness, outlier detection
│       └── reporting.py             ← Daily P&L report generation
└── tests/
    ├── test_strategies/
    ├── test_risk/
    ├── test_execution/
    ├── test_data/
    └── test_fx/                     ← NEW
```

---

## 🔑 CODING STANDARDS

**General:**
- Type hints on ALL functions — no exceptions
- Docstrings on all public functions and classes
- No magic numbers — all constants in `config/`
- All monetary values as `Decimal`, never `float`
- All dates as `datetime` with TIMESTAMPTZ, UTC internally
- Never hardcode FX rates (no `1.27`), never hardcode DTE (no `91`), never hardcode anything universe-specific

**Strategy code specifically:**
- Every strategy inherits from `BaseStrategy` and accepts a `universe` parameter
- Metals-specific strategies (S4, S5, S6) explicitly type-constrain their universe to `Metal`
- Generic strategies (carry, tsmom, xs_momentum) accept any `InstrumentSpec` universe
- Signals return values in range [-1, +1] (normalised z-score)
- Position sizing happens in `risk/sizing.py`, NEVER inside strategy code
- Strategies are stateless — they take data in, return signals out
- No `print()` statements — use `logging` module with `correlation_id` throughout

**Risk code specifically:**
- Hard limits in `risk/limits.py` are constants — never modify at runtime
- Circuit breakers run as a SEPARATE process from the signal engine
- Every risk breach is logged to `audit.risk_breaches` with full context
- Kill switch must work even if the main execution process is hung

**Testing:**
- Unit tests for every signal calculation
- Property-based tests for position sizing (always within limits)
- Integration tests for the full backtest pipeline
- Never mock the risk limits — test against real limits
- FX tests must not assume futures semantics (no expiry, no roll, no lot size)

---

## ⚠️ THINGS CLAUDE SHOULD NEVER DO

1. **Never bypass risk limits** — not even "just for testing"
2. **Never hardcode API keys, passwords, or credentials**
3. **Never hardcode FX rates or DTE** — they must be dynamic
4. **Never modify the OOS/Holdout data splits** after they're set
5. **Never optimise parameters on OOS data** — it becomes contaminated IS data
6. **Never submit live orders without all pre-trade checks passing**
7. **Never assume a signal is correct** — validate data quality first
8. **Never delete from raw_data** — it's immutable
9. **Never add complexity** that the current phase doesn't require
10. **Never skip transaction costs** in backtests — "fiction without them"
11. **Never use `float`** for monetary calculations
12. **Never use `empyrical`** — it's broken on Python 3.12; use custom metrics
13. **Never start FX build before metals S4 has cleared G0** — roadmap discipline
14. **Never relitigate the April 2026 pivot** — the decision is made and documented
15. **Never conflate the metals and FX books** — when ambiguous, ask which one

---

## 📚 KEY REFERENCES (embedded in project docs)

**Metals:**
- Gorton & Rouwenhorst (2006) — carry premium evidence
- Moskowitz, Ooi & Pedersen (2012) — time-series momentum
- Bakshi, Gao & Rossi (2019) — basis-momentum (t-stat 4.14)
- Harvey, Liu & Zhu (2016) — multiple testing, t-stat ≥ 3.0 threshold
- de Prado (2018) — CPCV, PBO methodology
- Figuerola-Ferretti & Gonzalo (2010) — LME cointegration

**FX:**
- Koijen, Moskowitz, Pedersen & Vrugt (2018) — carry across asset classes
- Lustig, Roussanov & Verdelhan (2011) — HML-FX factor
- Menkhoff, Sarno, Schmeling & Schrimpf (2012) — currency momentum
- Asness, Moskowitz & Pedersen (2013) — value/momentum everywhere (includes FX)
- Moskowitz, Ooi & Pedersen (2012) — TSMOM in FX (shared with metals)

---

## 🧑‍💻 ABOUT THE DEVELOPER

- **Tim** — CIO/CTO. Front-office metals support background (Macquarie).
  ARPM-trained. Production engineering experience.
- Building this part-time (10–15 hours/week evenings and weekends)
- Solo until 2029 — every milestone must be achievable by one person
- **James** joins 2029 as COO/Head of Capital — investor relations, fundraising
- **Jared** — legal advisor (corporate structuring, FSCA)
- Target HQ: Cape Town by 2031

**Working style preferences:**
- Favour simple, correct code over clever, complex code
- Always explain the "why" behind implementation choices
- Flag when I'm about to build something the current phase doesn't need
- If something is risky or could lose money, say so explicitly
- When uncertain between two approaches, show both with trade-offs
- Ask clarifying questions before assuming on structural decisions
- Never build ahead of verification (Step 0 discipline)

---

*This file is the single source of truth for project context.
Update it whenever the phase, stack, priorities, or strategic direction changes.
Cross-references: `WorFIn_Pivot_Memo.md` (April 2026 pivot rationale),
`FX_Strategy_Universe.md` (FX strategy spec), `Metals_Systematic_Trading_Strategies.pdf` (metals spec).*

---

## 📝 Revision Log

- **March 2026** — Initial master brief, metals-only.
- **April 2026** — FX book added. Asset-class-agnostic refactor planned. Pivot memo linked at top. Metals S4 IS backtest flagged as gate for FX build start. Tech stack corrected: empyrical banned, PostgreSQL 16 + TimescaleDB confirmed.

---
> Source: [timwfitnum/worfin_strategies](https://github.com/timwfitnum/worfin_strategies) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
