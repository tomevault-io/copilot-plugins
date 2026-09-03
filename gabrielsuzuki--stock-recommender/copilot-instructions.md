## stock-recommender

> Read this before changing anything. It exists so a session does not start cold

# CLAUDE.md — persistent context for this repo

Read this before changing anything. It exists so a session does not start cold
and re-litigate decisions that were already made and written down.

## What this is

A swing-trade screener that pushes a brief to Telegram at 05:00 Pacific on
weekdays. Personal tool, one user, not investment advice.

**Two stages, split on purpose:**

- **Stage A, 01:00 PT** (`pipeline/stage_a_nightly.py`) — deterministic. Refresh
  bars, compute indicators, screen, rank, size, write `candidates.json`.
- **Stage B, 05:00 PT** (`pipeline/stage_b_brief.py`) — regime check, news,
  disqualify, catalysts, theses, render, deliver, journal.

Stage A runs four hours early so a failure can be noticed and fixed before the
brief is due. That margin is the entire reason for the split — do not merge them.

## The rule that governs every change

> **The LLM never computes a number. It reads a table it did not produce, and
> writes prose.**

Indicators, ranking, sizing, regime, backtest stats — all deterministic Python,
unit-tested, reproducible, free. The model's job starts after the numbers exist.
Every number in the brief traces to a field in `candidates.json`.

Three corollaries that have already been enforced against the original design:

1. **The regime verdict is computed, not generated.** A model asked "is the tape
   healthy?" will eventually say yes on a day the rules say no, persuasively.
   The off-switch is not negotiable. `core/regime.py` owns it.
2. **The brief is rendered, not composed.** Letting a model emit MarkdownV2
   means silent formatting failures at 05:00 and numbers that drift in transit.
   `pipeline/agents/editor.py` is a renderer; the model writes one sentence.
3. **Disqualify rules are regexes and form-type checks**, not prompt
   instructions. `mcp/news/disqualify.py`.

## Locked decisions

| | |
|---|---|
| Universe | S&P 500 (+ SPY, fetched but never screened) |
| Screen | Minervini Trend Template, **strict 8-of-8**. An empty list is a valid answer. |
| Horizon | Swing, 1–4 weeks |
| Fundamentals | Deferred to v2 |
| Position log | Telegram reply-to-log (`TOOK AAPL 100 @ 182.50`) |
| Host | Hetzner CX22, timezone `America/Los_Angeles`, systemd timers |
| Data | Alpaca (bars) + Finnhub (news/earnings) + EDGAR (filings) + Marketaux |
| Cost layer | `vendor/tokenwise/`, wired in `pipeline/llm.py` |

**Do not add a "6 of 8" scored variant of the screen.** Conditions 2 and 3 are
logically implied by the others (c1∧c4⟹c2, c2∧c5⟹c3), so a scored version would
triple-count the moving-average stack. This is proven in `core/test_screen.py`.

## Layout

```
core/          indicators, screen, ranking, sizing, regime, synthetic
mcp/market_data/  SQLite cache, Alpaca + offline providers, universe
mcp/news/         Finnhub, EDGAR, offline; mechanical disqualify rules
mcp/journal/      append-only briefs + fills, strict reply parser
pipeline/         stage_a, stage_b, llm, watchdog, fill_listener, weekly_review
pipeline/agents/  base (JSON harness), catalyst, thesis, editor
backtest/         panels, engine, walkforward, metrics, run
vendor/tokenwise/ the cost layer (reconstructed from a shuffled upload)
config/screen.yaml   EVERY threshold. Never hard-code one.
deploy/           bootstrap, systemd units, deploy script
```

## Conventions

- **Thresholds live in `config/screen.yaml`.** If you find yourself typing a
  number into a `.py` file, it belongs in the YAML with a comment.
- **Bars** are a DataFrame indexed by a sorted `DatetimeIndex` with columns
  `open, high, low, close, volume`, split- and dividend-adjusted.
- **`as_of` truncates before anything is computed.** That one line is what makes
  historical replay honest. `indicators.assert_no_lookahead()` proves it.
- **NaN never passes a screen condition.** Unknown and false are distinct
  upstream (insufficient history is *skipped and counted*, not rejected) and
  identical at the condition boundary.
- **Nothing raises into the morning.** Every component has a documented degraded
  mode; `DEGRADED RUN` appears in the brief footer when one fires. The two
  exceptions that *do* abort: stale bars and stale `candidates.json`.
- **Everything offline-runnable.** `--offline` on every entry point. CI has no
  API keys and neither does a new contributor.

## Testing

```bash
./run_tests.sh          # everything
python3 core/demo.py    # screen on synthetic data
```

Write tests that check values computed **by hand in the docstring**, not values
the code currently produces. A test that records current behaviour catches
nothing. See `core/test_screen.py::TestATR` for the shape.

`backtest/test_backtest.py::test_panels_match_production_metrics` is the most
important test in the repo: it asserts the backtest's vectorized panels equal
`compute_symbol_metrics` exactly. If it fails, the backtest is measuring a
different strategy than the one that runs.

## Known unresolved issues

1. **IEX volume — MEASURED AND RESOLVED 2026-08-21.** Alpaca's free feed
   reports **3.5% of consolidated tape** (median over AAPL/MSFT/NVDA/AMZN/SPY),
   with a **3.2x spread across symbols**. Resolved by setting
   `min_avg_volume: 14000` — the floor is now expressed in FEED units rather
   than consolidated units. Deliberately not solved by scaling volume up: a
   constant multiplier is only accurate to a factor of two (NVDA 0.5x, MSFT
   1.6x) and would put an invented number into the brief. For an S&P 500
   universe the gate is close to redundant anyway — index membership is already
   a liquidity filter. Re-measure with `python3 -m pipeline.preflight` if the
   feed or universe changes.
2. **No VIX on the free stack.** The regime uses SPY 20-day realized vol
   instead. Backward-looking, no volatility risk premium — its levels are *not*
   comparable to VIX levels. Do not port VIX rules of thumb into the thresholds.
3. **Survivorship.** `load_universe()` returns *today's* membership. Correct for
   live screening, wrong for backtests. See `mcp/news/universe.py` docstring.
4. **Bars are real; the pipeline has not run on them end to end.** Alpaca
   connectivity, bar freshness, EDGAR and Telegram are all verified live. The
   screen, backtest and paper results in this repo are still from synthetic
   fixtures.

## Getting to a live run

`python3 -m pipeline.preflight` checks every credential, measures the IEX volume
scale, and dry-runs both stages. `docs/launch-checklist.md` is the ordered path.

## Before you act on a backtest number

Read `docs/backtest.md`. Single-split runs are smoke tests. Only walk-forward
output is evidence, only out-of-sample segments are reported, and the SPY null
is printed every time on purpose.

---
> Source: [GabrielSuzuki/Stock-Recommender](https://github.com/GabrielSuzuki/Stock-Recommender) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
