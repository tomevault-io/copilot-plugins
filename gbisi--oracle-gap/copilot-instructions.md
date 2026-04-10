## oracle-gap

> This file is the primary instruction set for any AI coding agent working on this repository. Read it fully before writing any code. It defines what to build, how to build it, what not to do, and how to verify your work.

# AGENTS.md — Instructions for the Coding Agent

This file is the primary instruction set for any AI coding agent working on this repository. Read it fully before writing any code. It defines what to build, how to build it, what not to do, and how to verify your work.

---

## 1. Project Mission

You are implementing the **Oracle Gap** research software system. The goal is to compute the theoretical upper bound of achievable returns (the "oracle ceiling") in binary prediction markets, and measure how close real participants come to it, across a parametric investor spectrum.

**Phase 0 scope (your current task):** Build the complete end-to-end pipeline for **Bitcoin 5-minute direction markets on Polymarket** (event class C1). Nothing else. The architecture must be general (class-agnostic), but only the BTC C1 path needs to be wired up and tested.

---

## 2. The Single Most Important Rule

> **The oracle formula is the ground truth. Every line of code that touches it must have a passing unit test before it is considered done.**

The formula is in `docs/MATH_MODEL.md`. If you implement it incorrectly, all downstream results are wrong and the paper cannot be published. Test it obsessively. See `tests/test_oracle.py` for required test cases.

---

## 3. What To Build (Phase 0 Checklist)

Work through these in order. Do not skip ahead. Each item has an exit criterion.

### 3.1 Infrastructure
- [ ] `docker-compose.yml` — services: timescaledb, redis, airflow, dashboard
- [ ] `db/schema.sql` — TimescaleDB hypertables + PostgreSQL tables (see `SCHEMA.md`)
- [ ] `db/migrations/` — Alembic setup with initial migration applying `schema.sql`
- [ ] `.env.example` — all required env vars documented
- [ ] `Makefile` — all targets listed in `README.md`
- [ ] `requirements.txt` — pinned versions matching the stack table in `README.md`

**Exit criterion:** `make dev` starts all services healthy. `make schema` applies migration successfully.

### 3.2 Ingestion Layer
- [ ] `ingestion/base.py` — `BaseIngestor` abstract class with the three required methods
- [ ] `ingestion/polymarket.py` — implements `BaseIngestor` for Polymarket CLOB
- [ ] `ingestion/binance.py` — implements `BaseIngestor` interface pattern for BTC/USDT 1s OHLCV

**Exit criterion:** `make ingest DATE=<any recent date>` populates `trades` and `reference_prices` tables with no errors. At least 200 slot rows result from a single day.

### 3.3 Slot Constructor
- [ ] `pipeline/stages/p3_slots.py` — constructs `slot_master` from raw trades

**Exit criterion:** All 26 columns in `slot_master` are populated. `data_quality_flag=0` rows have no nulls in core fields. BTC return sign matches `outcome_yes` for all flag=0 rows.

### 3.4 Oracle Engine
- [ ] `oracle/bounds.py` — vectorised Polars oracle computation
- [ ] `oracle/breakeven.py` — SciPy break-even solver
- [ ] `tests/test_oracle.py` — all formula unit tests (see Section 6 of this file)

**Exit criterion:** All oracle unit tests pass (100%). `oracle_bounds` table populated for all slots. Runtime < 5s for 10,000 slots.

### 3.5 Monte Carlo
- [ ] `montecarlo/spectrum.py` — Numba JIT investor spectrum

**Exit criterion:** For p=1.0 (oracle): E[R_net] > 0. For p=0.5 (random): E[R_net] < 0. Ruin probability monotonically decreasing in p. Runtime < 60 min for 10k runs on available hardware.

### 3.6 Statistical Analysis
- [ ] `stats/regression.py` — IER OLS regression

**Exit criterion:** Regression runs without error. Outputs LaTeX table stub to `paper/tables/`.

### 3.7 Dashboard
- [ ] `viz/dashboard/app.py` — Plotly Dash live dashboard

**Exit criterion:** Dashboard loads at `:8050`. IER time-series renders. Break-even p* displays in plain language.

### 3.8 Airflow DAG
- [ ] `pipeline/dags/dag_btc_pilot.py` — full P0–P9 DAG

**Exit criterion:** Airflow UI shows green DAG for a 7-day backfill. All stages complete without error.

### 3.9 Reproducibility
- [ ] `make reproduce` target working end-to-end

**Exit criterion:** `make reproduce` succeeds in a clean container from only raw API data.

---

## 4. Architecture Rules (Non-Negotiable)

### 4.1 Class-Agnostic Code

Every module in `oracle/`, `montecarlo/`, `stats/`, and `viz/` must work on the `slot_master` schema regardless of which class or platform produced the data. **No class-specific logic outside `ingestion/` and `config/`.**

```python
# WRONG — class-specific logic in oracle
if class_code == "C1":
    fee = 0.02

# CORRECT — read fee from the slot record
fee = slot["fee_rate"]
```

### 4.2 Ingestor Interface

Every ingestor MUST implement `BaseIngestor`. Do not write a standalone script that directly writes to the DB. The interface is:

```python
# ingestion/base.py
class BaseIngestor(ABC):
    @abstractmethod
    def fetch_trades(self, market_id: str, date: date) -> pl.DataFrame:
        """Returns a DataFrame matching the raw_trades schema."""

    @abstractmethod
    def fetch_resolution(self, market_id: str) -> Resolution:
        """Returns the resolution record for a market."""

    @abstractmethod
    def get_market_ids(self, class_code: str, date: date) -> list[str]:
        """Returns all active market IDs for a class on a given date."""
```

### 4.3 No Raw SQL in Business Logic

All database access goes through `db/queries.py`. No `psycopg2.execute()` calls in pipeline stage files. No inline SQL strings in `oracle/`, `stats/`, or `viz/`.

### 4.4 Polars, Not Pandas

Use Polars everywhere for DataFrame operations. Pandas is not in `requirements.txt`. If you find yourself writing `pd.DataFrame`, stop and rewrite in Polars.

### 4.5 Idempotency

Every pipeline stage must be safe to re-run. All DB writes must use `INSERT ... ON CONFLICT DO UPDATE` (upsert). Never `DELETE` + `INSERT`. The primary key for `slot_master` is `slot_id` (SHA-256 hash — stable across re-ingests).

### 4.6 Seeded Randomness

Every random operation must accept and use an explicit seed. No `np.random.seed()` global calls. Use `np.random.default_rng(seed)` for NumPy and pass `seed` explicitly to Numba.

### 4.7 Type Annotations

All functions must have complete type annotations. Return types are mandatory. Use `from __future__ import annotations` at the top of each file.

---

## 5. Data Rules

### 5.1 The slot_master Schema Is Authoritative

The canonical schema is in `SCHEMA.md` and `db/schema.sql`. If there is any conflict between these files, `db/schema.sql` wins. Do not add columns to `slot_master` without updating both files and creating an Alembic migration.

### 5.2 q* Assignment Rule

`q_star_winner` is the **minimum trade price of the ex-post winning token** during the slot window `[t0_utc, t_close_utc)`. It is computed AFTER resolution is known. It is NEVER the minimum price of both tokens.

```python
# CORRECT
winning_token = "yes" if outcome_yes else "no"
q_star = trades.filter(pl.col("token") == winning_token)["price"].min()

# WRONG — minimum of all trades
q_star = trades["price"].min()
```

### 5.3 Quality Flags

| Flag | Meaning | Include in main analysis? |
|------|---------|--------------------------|
| 0 | Clean | Yes |
| 1 | Missing ticks (>20% gaps or n_trades < 5) | Sensitivity check only |
| 2 | Resolution disputed / BTC sign mismatch | Exclude, report in appendix |
| 3 | Manual exclusion | Exclude always |

### 5.4 Division by Zero Guard

`q_star_winner` must never be zero. Apply this guard in the slot constructor:

```python
q_star = max(q_star_raw, 0.001)
if q_star_raw < 0.01:
    quality_flag = max(quality_flag, 1)
```

---

## 6. Oracle Formula Tests (Required)

These specific tests MUST exist in `tests/test_oracle.py` and MUST pass before any oracle code is considered done.

```python
# Test 1: Layer-1 oracle (α=1, perfect outcome)
# Input: q0=0.50, q*=0.35, f=0.02, τ=0.26
# Expected: R_L1 = (1/0.50) * (1-0.02) * (1-0.26) - 1 = 0.4504
assert abs(R_L1(q0=0.50, q_star=0.35, f=0.02, tau=0.26, alpha=1.0) - 0.4504) < 1e-4

# Test 2: Layer-2 oracle (α=0, perfect outcome + timing)
# Input: q0=0.50, q*=0.35, f=0.02, τ=0.26
# Expected: R_L2 = (1/0.35) * (1-0.02) * (1-0.26) - 1 = 1.0721
assert abs(R_L2(q0=0.50, q_star=0.35, f=0.02, tau=0.26) - 1.0721) < 1e-4

# Test 3: Timing premium
# ΔR = R_L2 - R_L1 = (1/0.35 - 1/0.50) * (1-0.02) * (1-0.26)
assert abs(delta_R_timing(q0=0.50, q_star=0.35, f=0.02, tau=0.26) - 0.6217) < 1e-4

# Test 4: Break-even at α=1 (near-50/50 market, Italian tax)
# p* = 0.50 / [(1-0.02)*(1-0.26)] = 0.50 / 0.7252 = 0.6895
assert abs(break_even_p(q0=0.50, q_star=0.35, f=0.02, tau=0.26, alpha=1.0) - 0.6895) < 1e-4

# Test 5: Random investor always loses money (p=0.5, any α, any positive f)
# E[R_net](p=0.5, α=1) with f=0.02, τ=0 should be negative
assert expected_return(p=0.5, alpha=1.0, q0=0.50, q_star=0.35, f=0.02, tau=0.0) < 0

# Test 6: Oracle at α=0 always beats oracle at α=1 (q* < q0)
assert R_L2(q0=0.50, q_star=0.35, f=0.02, tau=0.0) > R_L1(q0=0.50, q_star=0.35, f=0.02, tau=0.0, alpha=1.0)

# Test 7: Fee drag is multiplicative, not additive
# (1-f)(1-τ) < (1-f-τ) for f,τ > 0
f, tau = 0.02, 0.26
assert (1-f)*(1-tau) < (1-f-tau+f*tau)  # always true; confirms compound is worse

# Test 8: Division by zero guard (q*=0 must be caught upstream)
# The oracle function should raise ValueError for q_star <= 0
import pytest
with pytest.raises(ValueError):
    R_L2(q0=0.50, q_star=0.0, f=0.02, tau=0.26)

# Test 9: IER is finite and meaningful when R_L2 > 0
# Note: IER = E[R_crowd]/R_L2. Since E[R_crowd] < 0 (fees drain crowd), IER < 0.
ier = compute_ier(r_crowd=-0.2748, r_l2=1.0721)
assert ier < 0  # Crowd always loses money relative to oracle

# Test 10: Parametric oracle interpolates between L1 and L2
r_l1 = R_L1(q0=0.50, q_star=0.35, f=0.02, tau=0.26, alpha=1.0)
r_l2 = R_L2(q0=0.50, q_star=0.35, f=0.02, tau=0.26)
for alpha in [0.0, 0.25, 0.5, 0.75, 1.0]:
    r = R_parametric(q0=0.50, q_star=0.35, f=0.02, tau=0.26, alpha=alpha)
    assert r_l2 >= r >= r_l1 - 1e-9  # monotone in alpha
```

---

## 7. File-by-File Implementation Notes

### `ingestion/polymarket.py`

- Use `httpx.AsyncClient` for all REST calls. Never use `requests` (blocking).
- The CLOB API endpoint for trade history: `GET https://clob.polymarket.com/trades?market={condition_id}&after={timestamp}`
- For resolution, cross-check with the Polygon UMA oracle contract. The `UmaCtfAdapter` contract address is in `config/markets.yaml`.
- Rate limit: max 10 requests/second. Implement token-bucket limiter.
- All HTTP errors must be caught, logged, and retried (max 3 attempts, exponential backoff with jitter).
- Store raw API responses as JSON to `data/raw/{date}/{market_id}.json` before parsing. This enables re-parsing without re-fetching.

### `ingestion/binance.py`

- Use `ccxt.binance` async client. Symbol: `BTC/USDT`.
- Timeframe: `1s` (1-second candles). Fetch all candles in `[t0_utc - 10s, t_close_utc + 10s]` for each slot.
- The extra ±10s buffer ensures alignment with Polymarket slot boundaries.
- If `1s` candles are unavailable for historical dates, fall back to `1m` candles and interpolate linearly. Flag any interpolated slot with `data_quality_flag = max(flag, 1)`.

### `oracle/bounds.py`

- Must use **Polars LazyFrame** throughout. Never call `.collect()` until the final write step.
- The alpha grid and tax grid must be defined as module-level constants, not hardcoded in functions.
- The IER formula: `IER = R_crowd / R_L2`. `R_crowd` is the expected return of an investor who bets at `q0` proportionally to `q0` itself (treats market as fair). For a correct prediction: `R_crowd = (1/q0)(1-f)(1-τ) - 1`. E[R_crowd] = q0 · R_crowd_correct + (1-q0) · (-1). Simplify: `E[R_crowd] = (1-f)(1-τ) - 1 + f(1-τ)q0 + ...` — see `docs/MATH_MODEL.md` for the full expression.
- Never use Python loops over DataFrame rows. All computation must be Polars expressions.

### `montecarlo/spectrum.py`

- The Numba function signature MUST match exactly what `pipeline/stages/p6_montecarlo.py` passes. See `docs/MATH_MODEL.md` Section 4 for the exact expected inputs.
- Pre-warm the Numba JIT cache at module import time using a tiny 10-slot synthetic dataset. This prevents 5-minute cold-start in the Airflow DAG.
- Output format: HDF5 file at `data/mc_results/{run_id}.h5` with groups `wealth_paths`, `summary_stats`, `parameters`.

### `viz/dashboard/app.py`

- The plain-language break-even sentence must read: *"To break even in this market today [jurisdiction], you need to be right on more than X% of your bets."*
- The (p, α) heatmap must use a diverging colorscale: red (negative E[R_net]) → white (break-even) → green (positive).
- The IER panel must show: raw IER per slot (scatter, light), 7-day rolling mean (line, bold), and ±1σ band.

### `pipeline/stages/p3_slots.py`

- The `slot_id` must be: `hashlib.sha256(f"{platform}:{market_id}:{t0_utc.isoformat()}".encode()).hexdigest()`
- This must be deterministic and stable — the same slot ingested twice must produce the same `slot_id`.
- The BTC return sign check: if `btc_return_pct > 0.1` (>0.1% up, in percent units) and `outcome_yes == False`, set `data_quality_flag = 2`. The 0.1 threshold (in percent) avoids flagging near-zero moves where direction is ambiguous.

---

## 8. What NOT To Do

- **Do not** write a single-file script that does ingestion + computation + output. Follow the module structure.
- **Do not** use pandas. Polars only.
- **Do not** put secrets in code or config files. All secrets go in `.env`.
- **Do not** add class-specific logic to `oracle/`, `montecarlo/`, `stats/`, or `viz/`. Those modules must not import from `ingestion/`.
- **Do not** call `.collect()` on a Polars LazyFrame more than once per pipeline stage.
- **Do not** skip writing the unit test first if you are implementing any formula.
- **Do not** use `global` state or module-level mutable variables.
- **Do not** catch broad `Exception` without re-raising or logging with full traceback.
- **Do not** write to `slot_master` from any module other than `pipeline/stages/p3_slots.py`.
- **Do not** delete and re-insert data. Upsert only.
- **Do not** implement Phase 1 features. Stubs are acceptable (raise `NotImplementedError`), but no working code for classes other than C1 in Phase 0.

---

## 9. Testing Requirements

- All tests live in `tests/`. No test files elsewhere.
- Run with: `make test` (which runs `pytest tests/ --cov=. --cov-report=term-missing`)
- Coverage requirements:
  - `oracle/bounds.py`: **100%** (no exceptions)
  - `oracle/breakeven.py`: **100%**
  - `montecarlo/spectrum.py`: **≥90%**
  - `pipeline/stages/p3_slots.py`: **≥90%**
  - All other modules: **≥80%**
- Every test that touches the oracle formula must use the exact numerical cases from Section 6.
- Integration tests (those requiring a live DB) must be marked `@pytest.mark.integration` and skipped in CI unless `INTEGRATION_TESTS=1` is set.

---

## 10. Logging and Error Handling

Use Python's `logging` module. No `print()` statements in production code (only in `__main__` blocks).

```python
import logging
logger = logging.getLogger(__name__)

# In each pipeline stage:
logger.info("P3 slot constructor: processing %d trades for date %s", n_trades, date)
logger.warning("Slot %s: q_star < 0.01, setting quality_flag=1", slot_id)
logger.error("P1 ingestion failed for market %s: %s", market_id, exc, exc_info=True)
```

Airflow task failure policy: retry 3 times with 5-minute delay. On third failure, send alert (configured via Airflow email connection).

---

## 11. Commit and PR Guidelines

- Commit messages: `type(scope): description` — e.g., `feat(oracle): add vectorised alpha grid expansion`
- Types: `feat`, `fix`, `test`, `refactor`, `docs`, `chore`
- One logical change per commit
- Before any PR: `make test` and `make lint` must both pass
- Every new formula implementation needs a corresponding test in the same commit

---

## 12. How to Verify Your Work End-to-End

After completing Phase 0, run this validation sequence:

```bash
# 1. Clean environment
docker-compose down -v && docker-compose up -d
make schema

# 2. Ingest one week of BTC data
make backfill START=2026-03-01 END=2026-03-08

# 3. Run oracle computation
make oracle

# 4. Spot-check break-even (should be ~0.69 for IT, α=1, BTC near 50/50)
psql $DB_URL -c "SELECT alpha, tau_label, p_star FROM break_even_surface WHERE class_code='C1' ORDER BY alpha, tau_label;"

# 5. Run Monte Carlo
make mc SEED=42

# 6. Check: oracle (p=1) must have E[R_net] > 0; random (p=0.5) must have E[R_net] < 0
# (Check mc_results HDF5 summary_stats group)

# 7. Run statistics
make analyse

# 8. Verify dashboard
make dashboard
# → Open :8050 and confirm: IER time-series visible, p* sentence displays

# 9. Generate paper outputs
make figures
ls paper/tables/ paper/figures/  # must be non-empty

# 10. Full reproducibility test
make reproduce
```

If all 10 steps succeed, Phase 0 is complete.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GBisi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/GBisi)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
