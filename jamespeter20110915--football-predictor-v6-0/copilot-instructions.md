## football-predictor-v6-0

> Read `PROJECT.md`, `CURRENT_TASK.md`, `TASKS.md`, `DECISIONS.md`, and `README.md` before changing code. On Windows, explicitly enter this Unicode path before running commands:

# Repository Agent Guide

## Read first

Read `PROJECT.md`, `CURRENT_TASK.md`, `TASKS.md`, `DECISIONS.md`, and `README.md` before changing code. On Windows, explicitly enter this Unicode path before running commands:

```powershell
Set-Location -LiteralPath 'D:\Coding\足球\v6'
```

Inspect `git status --short`, `git branch --show-current`, and `git log -5 --oneline`. The files under `data/` may contain user-generated refresh results; never overwrite or revert them during tests.

## Stack

- Python 3.10+ standard-library HTTP server and data pipeline
- Vanilla HTML, CSS, and JavaScript frontend
- Optional Node.js Excel export using `build_predictions_excel.mjs`
- No required Python package installation for the web GUI or tests

## Main commands

```powershell
python web_gui.py
python -m unittest discover -s tests -v
python -m py_compile competition_data.py world_cup_data.py prediction_service.py web_gui.py WCMP.py
node --check web/app.js
node --check build_predictions_excel.mjs
```

Use an isolated directory for live data verification:

```powershell
python competition_data.py --competition big_five --league eng.1 --data-dir <temporary-directory>
python web_gui.py --data-dir <temporary-directory>
```

## Directory responsibilities

- `competition_data.py`: competition registry, current/previous season fetching, normalization, cache isolation, and status contracts
- `world_cup_data.py`: legacy-compatible World Cup ESPN normalization
- `prediction_service.py`: competition-aware model payloads and exports
- `WCMP.py`: core prediction and tournament engine
- `web_gui.py`: local HTTP API and static-file server
- `web/`: browser UI
- `tests/`: offline tests; all writes must use temporary directories
- `data/`: local user data and generated snapshots

## Project rules

- Keep `competition` optional in APIs; omitted values must default to `world_cup`.
- Use ESPN team IDs as club-team model keys. Display abbreviations are not unique across the Big Five.
- Keep World Cup legacy files at `data/world_cup_*`; keep other caches under `data/competitions/<competition>/`.
- Do not treat unpublished schedules as errors or invent fixtures/participants.
- Domestic and two-leg club fixtures are not neutral by default.
- Do not publish Champions League advancement probabilities until both-leg aggregate context is modeled; finals are neutral.
- Pass the same normalized stage into simulated win probabilities and displayed expected-goal distributions.
- Only World Cup mode may invoke `TournamentSimulator`.
- Preserve last-known-good caches on network failure and write cache files atomically.
- Do not add automatic bulk FotMob scraping. The existing World Cup xG path is legacy/on-demand only.
- Do not commit, push, pull, stash, or stage unless explicitly requested.
- Update `CURRENT_TASK.md` after meaningful stages and before handoff.

---
> Source: [jamespeter20110915/Football-Predictor-v6.0](https://github.com/jamespeter20110915/Football-Predictor-v6.0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
