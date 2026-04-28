## ncaa26

> Read this file at the start of every session. Follow these rules exactly.

# CLAUDE.md — NCAA26 Project Rules

Read this file at the start of every session. Follow these rules exactly.

---

## Project Mission

Build a March Madness prediction system that beats KenPom in forecast accuracy:
1. **Kaggle** — March Machine Learning Mania 2026 (Brier score, $50K prize, deadline **March 19 2026**)
2. **ESPN** — 10-person bracket pool (round multiplier: R64=1, R32=2, S16=4, E8=8, F4=16, Champ=32)
3. **DraftKings/FanDuel** — only if evidence of edge after ESPN/Kaggle locked in

Selection Sunday: **~March 15, 2026** | First games: **March 20, 2026**

### Data Freshness Checklist (re-check before final submission)

Kaggle raw data was last refreshed on **2026-03-16**. The following files may update after Selection Sunday:

- [ ] **`MNCAATourneySeeds.csv` / `WNCAATourneySeeds.csv`** — 2026 seeds (required for submission). Not yet available as of 2026-03-16.
- [ ] **`SampleSubmissionStage2.csv`** — Stage 2 matchup IDs for 2026. May update.
- [ ] **Regular season results** — refreshed 2026-03-15; re-pull if conference tournament games are still ongoing.
- [ ] **`MGameCities.csv` / `WGameCities.csv`** — refreshed 2026-03-15; may add conference tourney city data.

**To refresh**: `kaggle competitions download -c march-machine-learning-mania-2026`, then re-run `scripts/02_compute_features.py` and `scripts/03_train_models.py`.

---

## IMPORTANT: No External Ratings as Model Inputs

KenPom and Barttorvik are VALIDATION TOOLS only — never model features.
We build everything from raw box scores. KenPom is used ONLY to validate our efficiency engine (r > 0.95).

**Exception**: Massey Ordinals (`MMasseyOrdinals.csv`) are approved as model features.
These are Kaggle-provided competition data aggregating ~196 public computer ranking systems.
We use the consensus (median rank) rather than individual systems.

---

## Tech Stack (locked)

- **Python**: 3.13, **Package manager**: Poetry
- **Linting**: ruff (PostToolUse hook auto-runs on every file write)
- **Storage**: Parquet via pyarrow (not CSV, except Kaggle submission)
- **Primary model**: XGBoost Cauchy loss on point spread + UnivariateSpline calibration
- **Tuning**: Optuna (100 trials), **CV**: Season-grouped GroupKFold — NEVER shuffle seasons

---

## Git Branch Strategy

Branch naming: `agent/{phase}{letter}-{task}` (e.g., `agent/03a-feature-store-glm`)
Always tell agents: "Work on branch agent/{name}. Do not merge to main."
Merge (linear history): rebase feature branch onto main, then fast-forward:
```bash
git checkout agent/{branch} && git rebase main
git checkout main && git merge --ff-only agent/{branch}
```
**NEVER merge to main without explicit user approval.**

---

## How to Run

```bash
export VENV="VIRTUAL_ENV=/Users/kniu91/Documents/projects/ncaa26/.venv"
$VENV poetry run python scenarios/<test_file> [--fixtures]
$VENV poetry run python scripts/<script_file>
```

### Ad-Hoc Analysis Rule

Any ad-hoc analysis or exploration kicked off directly by the agent in-chat (e.g., inline Python scripts, one-off ablation runs) **must complete within ~5 minutes**. Before running:
- Use early stopping, reduce rounds, subsample, or limit folds to stay within budget.
- If the task cannot reasonably finish in 5 minutes, **tell the user** with an estimated runtime and get approval before executing.
- When replicating pipeline logic (e.g., CV loops), use the existing library functions (`run_season_cv`, etc.) rather than hand-rolling — they already have early stopping and other safeguards.

---

## Data Paths

```
PROJECT_ROOT = /Users/kniu91/Documents/projects/ncaa26
RAW_KAGGLE = data/raw/kaggle/march-machine-learning-mania-2026/
RAW_KENPOM = data/raw/kenpom/
RAW_BARTTORVIK = data/raw/barttorvik/
PROCESSED = data/processed/
SUBMISSIONS = data/submissions/

# Prior year reference
NCAA25_ROOT = /Users/kniu91/Documents/projects/ncaa25
```

All hyperparameters and paths live in `src/ncaa26/utils/constants.py` — do not change without CV evidence.

---

## Source Code Layout

```
src/ncaa26/
├── ingestion/         # kaggle_downloader, kenpom_scraper, espn_scraper
├── features/          # game_preparation, four_factors, momentum, garbage_time,
│                      #   tourney_distance, massey_ordinals, auto_qualifier
├── rating/            # elo_ratings, srs_ratings, rating_pipeline
├── models/            # feature_store, xgb_spread_model, logistic_model,
│                      #   calibration, cross_validator
├── competition/       # kaggle_submission, bracket_simulator, bracket_optimizer
└── utils/             # city_utils, slot_matchups, constants
```

---

## Known Gotchas

1. **Gender encoding**: TeamID < 3000 = Men's (1), >= 3000 = Women's (0).
2. **Submission ID format**: `{Season}_{lower_TeamID}_{higher_TeamID}` — prob from lower TeamID's perspective.
3. **Season labels**: KenPom ending year = Kaggle Season (2024-25 = Season 2025).
4. **`DayNum > 118` cutoff**: Last 14 days of regular season for momentum stats.
5. **Cauchy loss `c=5000`**: Calibrated for ncaa25 scale. May need re-tuning.
6. **`num_parallel_tree=10`**: Each XGBoost "round" = 10 trees. Effective LR much lower than `eta`.
7. **Poetry venv workaround**: Prefix `poetry run` with `VIRTUAL_ENV=.venv`. See gotcha #9 in git history for details.
8. **Stale editable installs after worktree merge**: Re-run `poetry lock && poetry install` from main.
9. **Opponent four factors**: Compute from T2 columns grouped by T1_TeamID (not by swapping symmetrized data).
10. **KenPom scraping**: Requires session cookie in `.env`; expires periodically.
11. **Seed format**: Strings like "W01", "X16a" — extract `int(seed_str[1:3])`, strip play-in suffixes.
12. **Season 2020**: No tournament data. Skip in all tournament-related processing.
13. **Feature leakage from `prepare_data()`**: Tournament detailed results produce per-game box score columns (`T1_FGM`, `T2_DR`, etc.) — these are the actual game's stats and MUST be excluded from features. Use `_GAME_BOX_COLS` in `feature_store.py`.
14. **Game city data starts at 2010**: `MGameCities.csv`/`WGameCities.csv` have no tournament entries before 2010. Distance features are NaN for seasons 2003-2009 (448 games). Training seasons 2010+ have 100% coverage.

---

## Reference Docs (@ imports)

@docs/runbook.md
@docs/validation_benchmarks.md
@specs/feature_backlog.md

---

## Specs Index

| Phase | Spec | Branch | Status |
|-------|------|--------|--------|
| P1 | `specs/01a_kaggle_download.md` | `agent/01a-kaggle-download` | DONE |
| P1 | `specs/01b_utility_ports.md` | `agent/01b-utility-ports` | DONE |
| P1 | `specs/01c_scenario_scaffolding.md` | `agent/01c-scenario-scaffolding` | DONE |
| P2 | `specs/02a_elo_srs_ratings.md` | `agent/02a-elo-srs-ratings` | DONE |
| P2 | `specs/02b_feature_wrappers.md` | `agent/02b-feature-wrappers` | DONE |
| P2 | `specs/02a_h1_srs_stability.md` | `agent/02a-h1-srs-fix` | DONE |
| P3 | `specs/03a_baseline_model_pipeline.md` | `agent/03a-feature-store-glm` | DONE |
| P3 | `specs/03b_model_training_pipeline.md` | `agent/03b-model-training-pipeline` | DONE |
| P3 | `specs/03c_distance_features.md` | `agent/03c-distance-features` | DONE |
| P3 | `specs/03d_diff_features.md` | `agent/03d-diff-features` | DONE |
| P3 | `specs/03e_elo_improvements.md` | `agent/03e-elo-improvements` | DONE |
| P3 | `specs/03f_massey_ordinals.md` | `agent/03f-massey-ordinals` | DONE |
| P3 | `specs/03g_srs_recency.md` | `agent/03g-srs-recency` | DONE (no improvement) |
| P4 | `specs/04a_model_manifest.md` | `agent/04a-model-manifest` | DONE |
| P4 | `specs/04b_tourney_distance_2026.md` | `agent/04b-tourney-distance-2026` | DONE |
| P4 | `specs/04c_kaggle_submission.md` | `agent/04c-kaggle-submission` | DONE |
| P4 | `specs/04e_optuna_tuning.md` | `agent/04e-optuna-tuning` | NOT STARTED |
| P5 | `specs/05a_bracket_simulator.md` | `agent/05a-bracket-simulator` | DONE |
| P5 | `specs/05b_espn_crowd_picks.md` | `agent/05b-espn-crowd-picks` | DONE |
| P5 | `specs/05c_bracket_optimizer.md` | `agent/05c-bracket-optimizer` | DONE |
| P6 | `specs/06_improvement_tinkering.md` | (main) | DONE (06a AQ shipped, 06c no change) |
| — | `specs/fix_bracket_optimizer_inference.md` | (main) | DONE |

See `specs/feature_backlog.md` for full component inventory and feature roadmap.

---

## Agent Communication Protocol

When completing a session:
1. Update `specs/feature_backlog.md` — move items to Completed, check off In Progress
2. Record benchmark results in `docs/validation_benchmarks.md`
3. Add any new gotchas to Known Gotchas above
4. Update `docs/runbook.md` if new scripts or tests were added/verified
5. Update Specs Index above if a spec's status changed
6. Add `.vscode/launch.json` config if new scripts or scenario tests were created

When starting a new session:
1. Read this CLAUDE.md first
2. Read the relevant spec for the component you're implementing
3. Run existing pipeline to verify current state before making changes

---

## Key References

- **ncaa25 port sources**: `kev_25.ipynb` (cells 1-40), `bracketMaker.ipynb`, `explore/scoring/`
- **Orchestrator guide**: `docs/ORCHESTRATOR_GUIDE.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kevniu95) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-15 -->
