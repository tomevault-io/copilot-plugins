## lotto-machine

> AI Hedge Fund - objectives, approach, constraints, verification plan, and open questions for live, DP-aware, breakout/reversal detection with robust rate-limit handling


# AI Hedge Fund – Objectives, Approach, Constraints, Verification Plan

## Objectives
- Deliver a one-button, real-time market intelligence system that:
  - Aggregates multi-source data (quotes, options, DP/block, news) with robust fallbacks.
  - Detects anomalies and clusters (block/DP/option + price/volume + news) and produces actionable alerts.
  - Executes DP-aware breakout/reversal logic and avoids traps at institutional magnets.
  - Generates timestamped, real signal logs and overlaid charts (no mock data).
  - Survives rate limits via caching, backoff, failover, and Yahoo Direct scraping.

## Current Understanding
- Yahoo Direct is our primary public source; RapidAPI caps at ~500; yfinance often rate-limits.
- DP-aware filtering and “magnet” logic are required to confirm signals and avoid traps.
- No mock data in production; slower but authentic polling is preferred.
- Need minute-level (1–5m) replays with rolling features and visual overlays for proof.

## Approach (High-level)
- Data layer: prioritize Yahoo Direct → yfinance fallback → RapidAPI (last resort). Backoff + cache.
- Institutional flow: DP/dark-pool + block trades + options sweeps where feasible; Playwright/Selenium for JS-heavy.
- Detection: rolling maxima/minima breakout windows with adaptive thresholds by regime (UP, DOWN, CHOP).
- DP-aware filter: only act when price action + composite flows agree and not at magnets.
- Verification: 1–5m replays, signal logs, charts; stress-test thresholds and windows; track hit/miss and drawdown.

## Constraints and Practices
- No mock production data; allow slower cadence if necessary.
- Use randomized jitter and user-agent rotation; support proxy rotation when needed.
- Human-in-the-loop for CAPTCHA/session cookies where required.
- Persist results (signal logs, summary JSON, charts) for audit and tuning.

## Deliverables (near-term)
- Minute-level replay for SPY with rolling features, regime, and overlays.
- Timestamped signal log showing exact breakouts/reversals and confidence.
- Regime determination printout (trend, volatility, momentum, SR ratio) per window.
- Robust rate-limit solver integrated into live and replay flows.

## Files (anchors)
- Replay + detection:
  - [replay_intraday_spy.py](mdc:replay_intraday_spy.py)
  - [live_high_resolution_detector.py](mdc:live_high_resolution_detector.py)
  - [real_breakout_reversal_detector_yahoo_direct.py](mdc:real_breakout_reversal_detector_yahoo_direct.py)
  - [real_breakout_reversal_detector_with_solver.py](mdc:real_breakout_reversal_detector_with_solver.py)
- Rate-limit + data sources:
  - [production_rate_limit_solver.py](mdc:production_rate_limit_solver.py)
  - [real_yahoo_finance_api.py](mdc:real_yahoo_finance_api.py)
- DP-aware filter:
  - [dp_aware_signal_filter.py](mdc:dp_aware_signal_filter.py)
- Streamlit/UI components:
  - [src/streamlit_app/ui_components.py](mdc:src/streamlit_app/ui_components.py)

## Verification Plan
- Rerun with live data when rate limits clear: replay today at 1–5m resolution for SPY and peers.
- Produce real, timestamped signal logs and an overlay chart for each run.
- Print regime inputs: trend strength (slope-normalized), volatility (returns std), momentum, SR ratio, volume regime.
- Stress-test windows (5/10/20/30) and thresholds (0.2%–1.5%) and report sensitivity.
- Confirm failover behavior and jitter/backoff timings in logs when sources degrade.

## How to Use This Rule
- Treat this as the operational doctrine for analysis tasks in this repo.
- Before diving into coding, review “Objectives/Approach/Constraints/Verification Plan” and answer “Open Questions”.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fjkiani)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/fjkiani)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
