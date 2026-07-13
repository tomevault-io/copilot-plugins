## mlb-predictor

> This is a personal-use MLB prediction system that pulls live data from free

# MLB Predictor — Project Brief

This is a personal-use MLB prediction system that pulls live data from free
APIs, projects team scores and per-player stats for each game, and ranks
sportsbook bets by edge. It was built across several Claude Code sessions.

**You are reading this because the user asked for a fresh-eyes review.** Don't
rubber-stamp the design. Push back where it's weak. The "Soft spots" section
below points to specific things to scrutinize.

---

## Goal

Given a date, output:
1. Per-game team-score predictions (home & away runs) using park + weather + matchup features.
2. Per-player projections for batters (H, HR, TB, RBI, R, K, BB) and starters (IP, K, BB, H, ER, HR).
3. A ranked leaderboard of +EV bets vs Bovada's lines (or The Odds API consensus if a key is set), including game lines (ML / total / RL) and player props.

Outputs are a CLI (`scripts/predict.py`) and a Streamlit web app (`app.py`).

---

## Data sources (all free, no auth)

- **MLB Stats API** (`statsapi.mlb.com`) — schedule, boxscores, season + by-date-range stats, rosters, handedness. Disk-cached at `data/cache/`.
- **Open-Meteo** — historical (archive) and forecast weather; auto-routed by date. Hourly resolution; we pull the hour containing first pitch. Cached per-park-per-hour.
- **Bovada** (`bovada.lv` JSON coupon endpoint) — game lines + pitcher props (with line) + batter props (yes/no thresholds). No auth, but TOS technically prohibits scraping; treat as personal-use only.
- **The Odds API** (optional) — set `ODDS_API_KEY` env var; gives consensus US-book lines. Falls back to Bovada gracefully.

---

## Pipeline

```
scripts/build_dataset.py       Pull 2026 schedule + stats. Builds weekly Monday snapshots under
                                data/games/snapshots_2026/ for leak-free historical features.
                                Current-day stats also saved to snapshot_2026.json for live use.
scripts/build_dataset_2025.py  One-time: pull 2025 with weekly stat snapshots (~10 min).
scripts/train.py               Fit team-runs model (Poisson GLM + GBT ensemble); 2026-only holdout.
scripts/train_combined.py      Fit team-runs model on 2025+2026 combined (current production model).
scripts/train_props.py         Fit per-stat HistGBT models. Uses per-game weekly snapshots from
                                data/games/snapshots_2026/ — requires build_dataset to have run first.
scripts/fit_dispersion.py      Empirical NegBin dispersion per stat from boxscores.
scripts/backtest.py            Player projection backtest on last-7-day holdout.
scripts/backtest_2025.py       Walk-forward backtest on 2025 full season (5 monthly folds).
scripts/build_props_2025.py    Build props_bat_2025.csv / props_pit_2025.csv — analytical projections
                                vs actuals for 2025. Feeds calibrate_reliability.py for stable r estimates.
                                All API calls are cached so runs ~30 s after build_dataset_2025.py.
scripts/predict.py             CLI: today's slate + value bets.
app.py                          Streamlit: same data + per-game cards + Backtest page.
```

Run order from a clean checkout:
```
python -m scripts.build_dataset         # also generates snapshots_2026/
python -m scripts.build_dataset_2025    # one-time
python -m scripts.train_combined        # produces team_runs.joblib
python -m scripts.train_props           # produces prop_*.joblib (needs snapshots_2026/)
python -m scripts.fit_dispersion        # produces dispersion.json
python -m scripts.build_props_2025      # one-time; generates props_*_2025.csv (~30 s, all cached)
python -m scripts.calibrate_reliability # updates stat_reliability.json from 2025+2026 holdout correlations
python -m streamlit run app.py
```

> **Note:** use `python -m streamlit run app.py` — bare `streamlit` is not on PATH on this machine.
> All models in `data/models/` were last regenerated Apr 29 2026 and are current.

---

## Architecture

### Team-runs model (`src/model.py`)

- One row per (game, batting team) — long form gives 2 rows per game.
- **PoissonRegressor** GLM (log link) for calibrated mean runs.
- **HistGradientBoostingRegressor** with Poisson loss for non-linearities. Blend weight learned on holdout (currently ends up at 1.0 GLM-only — GBT not adding signal at this sample size).
- Features: offense (RPG, OPS, wOBA, BABIP, BB%), recent form (14d), opposing starter (xFIP, FIP-, BB/9, HR/9), bullpen ERA, park factors, weather (runs/HR multipliers, wind to CF, temp), engineered interactions (off×park, opp_sp×off), is_home.
- Wired for **isotonic recalibration** but turned off (didn't help on small sample).
- Trained on **2025 + 2026 combined (~2,800 games)** — gives stable coefficients.

### Player projections (`src/projections.py`)

Two layers stacked:

1. **Analytical** — empirical-Bayes shrunk rate stats × expected PA × matchup multipliers (opp pitcher quality, park, weather, L/R platoon split).
2. **ML stack** (`src/prop_models.py`) — per-stat HistGBT trained on 2026 boxscore data. Each model has a holdout-tuned blend weight stored on its `StatModel.blend_weight`. Some stats prefer pure ML (HR, runs, RBI), some prefer pure analytical (hits), some blend.

Switch hitters: bat side flips opposite the pitcher's throw. L/R splits applied as shrunk-ratio multipliers on K%, BB%, HR%, AVG.

### Value engine (`src/value.py`)

- American-odds → implied prob → de-vigged via two-way normalization.
- Game lines (ML / total / RL) priced from joint independent-Poisson grid.
- Player props priced from Negative Binomial with **empirically-fit dispersion** per stat (`data/models/dispersion.json`), conditional on projection mean.
- One-sided props (Bovada's "Yes" prices on HR/Hits/etc.) compared to implied prob minus 6% juice estimate.
- Each ValueBet gets `confidence` and `score` populated by `annotate()`:
  - `confidence = stat_reliability × info_factor`
  - `score = edge_pct × confidence`
  - `info_factor = 4·p·(1-p)` peaks at 50% probability.
- Leaderboard can be ranked by **Score** (default) or **EV/$** via sidebar toggle in app.py.

### Bet ranking philosophy

Stat reliability weights are loaded from **`data/models/stat_reliability.json`** (seeded from hardcoded defaults, editable after backtest runs). To update after a backtest:
```python
from src.value import write_stat_reliability
write_stat_reliability({"prop_pitcher_k": 0.62, ...})
```
Pitcher props get higher weights than batter props because (a) we predict them better and (b) Bovada offers two-sided pricing on K's.

---

## Backtest numbers (current best)

### 2025 walk-forward (full season, train on months 1..K, test on K+1)

| Test month | n  | MAE  | Bias  |
|------------|----|------|-------|
| May        | 412| 2.57 | +0.07 |
| Jun        | 398| 2.55 | -0.08 |
| Jul        | 371| 2.47 | -0.05 |
| Aug        | 422| 2.55 | -0.22 |
| Sep        | 374| 2.38 | +0.01 |
| **Mean**   |    |**2.50 (std 0.08)**|**-0.05**|

ML accuracy on the Sep single-split: **52.4%** (n=374). Slight edge over coin flip.

### 2026 last-7-day holdout (combined-trained model)

- Team runs MAE: **2.36**
- Game total MAE: **3.27**, RMSE 4.19
- ML accuracy: **51.6%** (n=91)
- Per-stat batter MAE: H 0.67, HR 0.20, TB 1.30, K 0.63
- Per-stat pitcher MAE: K 1.63, outs 2.96, ER 1.48 (R² 0.31, 0.31, 0.17)

### 4/28/26 spot-check (15 games, NOT a backtest fold)

- ML accuracy 10/15 = 67% — likely lucky variance
- Total runs bias +0.95, away bias +0.97 — within single-day noise (2025 mean bias is -0.05)

---

## Soft spots — please scrutinize

### Fixed (as of Apr 29 2026 session)

- ~~**Feature leakage in 2026 training data**~~ — `build_dataset.py` now pulls weekly Monday snapshots to `data/games/snapshots_2026/`. Historical games use the prior-Monday snapshot; `train_props.py` also uses these per-game snapshots for analytical projections. `snapshot_2026.json` is still current-day for live prediction.
- ~~**Run-line ValueBet stored wrong `line`**~~ — `value.py:evaluate_game_lines` now correctly stores `home_line`/`away_line` (±1.5) and no longer NameErrors when a book has run_line but no total.
- ~~**STAT_RELIABILITY was hardcoded**~~ — now loaded from `data/models/stat_reliability.json`; `write_stat_reliability()` in `value.py` updates it from backtest output.
- ~~**No EV/$ ranking option**~~ — sidebar radio in `app.py` lets user switch leaderboard between Score and EV/$.

### Still open

1. ~~**Score formula penalised locks**~~ — `value.py:annotate` and `score_bet` now use `score = edge_pct × rel / sqrt(p*(1-p))`. High-confidence picks with the same edge score higher, consistent with Kelly. `confidence` still uses `4p(1-p)` (display metric for outcome uncertainty; not used for ranking).

2. ~~**Statcast not used**~~ — Statcast integrated Apr 29 session 4. Game model now uses `off_xwoba_sc` (team xwOBA, EB-shrunk) and `opp_sp_xera_sc` (starter xERA, EB-shrunk); `off_woba` and `off_barrel_rate` removed (they went negative once xwOBA entered — multicollinearity). Prop models now include `sc_barrel_pct` and `sc_hard_hit` (player-level). **52% ML accuracy** still low — remaining gaps:
   - Bullpen by leverage spot — we use staff-wide ERA.
   - Umpire effects (K% varies ~5% by umpire crew).
   - Rest days / pitcher TTO penalties.

3. ~~**`off_rpg_recent` had a NEGATIVE coefficient**~~ — dropped from `model.FEATURES`. 2025 training data fills it with the season value (constant on 70% of rows), so the GLM learned a spurious contrast. Removing it requires re-running `train_combined.py`. `off_rpg` (full-season RPG) is still in the feature set.

4. **Switch hitter platoon splits at April sample sizes** may be net noise (shrinkage cap is PA=200 → 60% weight on split data). Not empirically tested. Does removing splits change holdout MAE?

5. **Pitcher BB/K final blend weights are 0.0** (pure analytical). Either the analytical model has all the signal or the GBT isn't trained well. GBT feature importance would clarify.

6. ~~**Lineups guessed from PA leaderboard**~~ — `mlb_api.extract_lineups()` now parses `game["lineups"]` (already in the `schedule()` hydrate). `predict_core.py` and `scripts/predict.py` pass confirmed `lineup_ids` to every `get_likely_batters` call; falls back to PA leaderboard when lineup not yet posted. `schedule()` TTL lowered to 900s so lineup data stays fresh.

7. **Bovada juice is wider than US books.** A 5% edge vs Bovada may vanish at DK/FD. Edges should be labelled as "vs Bovada" and the user should expect haircut at sharper books.

8. ~~**Prop model blend weights tuned on reported holdout**~~ — `_train_one` in `prop_models.py` now uses a **3-way temporal split**: `train` (70%) fits the GBT, `blend` (15%) tunes the weight, `test` (15%) reports honest MAE. The `blend_weight` is still a holdout quantity but no longer contaminates the reported number. Note: `model.py`'s team-model blend has the same structure but its GBT weight ends up 1.0 (pure GLM) so the leak is dormant there.

9. ~~**`_get_prop_models` singleton not cleared by `st.cache_data.clear()`**~~ — `projections.reload_prop_models()` resets the module-level cache; `app.py` Refresh button now calls it alongside `run_prediction.clear()`.

---

## Files of interest

| File | Purpose |
|------|---------|
| `src/predict_core.py` | The main `predict_slate()` entrypoint used by both CLI and Streamlit. **Single source of truth for the prediction pipeline.** |
| `src/features.py` | Feature engineering for game-level rows (advanced metrics, weather adjustment, interactions). |
| `src/model.py` | Team-runs Poisson GLM + GBT ensemble. |
| `src/projections.py` | Analytical + ML-stacked player projections. |
| `src/prop_models.py` | Per-stat HistGBT models. |
| `src/dispersion.py` | Empirical NegBin dispersion fitting. |
| `src/value.py` | Odds math, prop probabilities, ValueBet construction, scoring. `STAT_RELIABILITY` now loaded from `data/models/stat_reliability.json`. |
| `src/bovada.py` | Free Bovada JSON scraper. |
| `src/parks.py` | Ballpark coordinates / park factors / orientation. |
| `src/weather.py` | Open-Meteo client. |
| `src/name_match.py` | Cross-feed player name matching. |
| `app.py` | Streamlit entrypoint. Sidebar has Score / EV/$ ranking toggle. |
| `pages/2_Backtest.py` | Streamlit Backtest page. |
| `data/games/snapshots_2026/` | Weekly Monday stat snapshots (one JSON per Monday). Built by `build_dataset.py`, consumed by `train_props.py`. Key for leak-free training. |
| `data/models/stat_reliability.json` | Per-market reliability weights for bet scoring. Edit directly or call `value.write_stat_reliability(weights)`. |
| `src/statcast.py` | Baseball Savant CSV fetcher. `get_team_batting(year, player_team_map)` and `get_pitcher_stats(year)`. Disk-cached, 6-hr TTL for current season. |
| `src/bet_tracker.py` | Logs top-10 confidence picks to `data/bets/bet_log.json`. `evaluate_outcomes()` fills W/L from boxscores. `get_track_record(days)` returns summary. |
| `data/bets/bet_log.json` | Running log of top-confidence picks with outcomes. Auto-populated by `predict_core.py` on live-date slate runs. |

---

## Things the user is likely to ask next

- "How did today's top picks do?" — `src/bet_tracker.py` now logs top-10 confidence picks to `data/bets/bet_log.json` automatically. Outcomes are evaluated against `box_2026.csv`/`games_2026.csv` on the next run. Track Record section is at the bottom of the Streamlit app.
- "Why is one game dominating the leaderboard?" — concentration warning fires at 40% threshold; user can dial down via slider. Underlying cause is usually the team-runs model strongly disagreeing with the book on a specific game.
- "Will this work on opening day next season?" — early-season is where shrinkage matters most. Priors at PA=30 / BF=50 are aggressive and may overfit to small-sample All-Stars.

---

## What "review" probably means

If the user is asking for a fresh-eyes review, useful angles:
- Audit the feature set against published MLB modeling literature (Statcast adoption is the obvious gap).
- Check the prop-model evaluation pipeline for leakage. Specifically: `train_props.py` builds analytical projections then trains a residual model, but the analytical projections use `recent_stats` from the SAME snapshot used for the test rows. There may be subtle leakage.
- Sanity-check the `score` formula against actual betting outcomes if any are tracked.
- Stress-test edge cases: doubleheaders (gamePk handling), suspended games, opener-style starters, pitchers being announced late.
- Re-derive whether the variance-weighted leaderboard is a `good idea` or just a heuristic that hides poor model calibration.

Don't be afraid to say "this part is over-engineered" or "this assumption is wrong" — the user explicitly wants pushback.

---
> Source: [kheefry/Mlb-Predictor](https://github.com/kheefry/Mlb-Predictor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-13 -->
