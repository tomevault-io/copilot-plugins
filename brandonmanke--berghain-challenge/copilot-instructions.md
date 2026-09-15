## berghain-challenge

> - `src/berghain/`: core logic

# Repository Guidelines

## Project Structure & Module Organization
- `src/berghain/`: core logic
  - `agent.py`: accept/reject policy interface and implementations
  - `scenarios.py`: scenario/constraints definitions and loaders
  - `simulator.py`: environment loop and accounting
  - `metrics.py`: scoring and reporting utilities
- `scripts/`: CLIs and notebooks-to-scripts (e.g., `play.py`, `benchmark.py`)
- `tests/`: pytest suite (e.g., `test_policy.py`, `test_simulator.py`)
- `data/`: optional local inputs (no PII; small files only)

## Build, Test, and Development Commands
- Preferred (uv, Python 3.12):
  - Install Python: `uv python install 3.12`
  - Create venv: `uv venv --python 3.12`
  - Sync deps: `uv sync` (dev tools: `uv sync --extra dev`)
  - Run tests: `uv run pytest -q`
  - Lint/format: `uv run ruff check . && uv run black --check .` (auto-fix with `uv run black .`)
  - Type-check: `uv run mypy src`
- Run agent: `uv run python scripts/play.py --scenario 1 --capacity 1000`
  - JSON logs (NDJSON): by default goes to `logs/run-<policy>-s<scenario>-<timestamp>.ndjson`.
  - You can set a path explicitly via `--log-json`.
- Fallback (pip):
  - `python3 -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt`
  - `PYTHONPATH=src python scripts/play.py --scenario 1 --capacity 1000`
- Config via `.env` (auto-loaded): `BASE_URL=https://berghain.challenges.listenlabs.ai/`, `PLAYER_ID=<uuid>`

### Makefile Shortcuts
- Quick start: `make python venv dev`
- Run agent: `make run SCENARIO=1 CAPACITY=1000 ARGS="--verbose --progress-interval 50"`
- Bench multiple scenarios: `make bench SCENARIOS=1,2,3 VERBOSE=1`
- Bench JSON summary: `make bench SCENARIOS=1,2,3 JSON_OUT=bench.json`

### Policies
- Default: `reserve` — safe minimum-constraints policy.
- `window` — sliding-window relaxed rule (size=500, margin=0.15).
- `ewma` — exponentially-weighted moving average of helpful rate (alpha=0.03, warmup=100).
  - Accept non-helpful when ewma >= S/(R-1) with safety margin.
- `attr-ewma` — per-attribute EWMA rates; accepts non-helpful only if, for every underfilled attribute, expected helpful arrivals in remaining slots cover the deficit with margin.
  - Tunables: `--alpha`, `--risk-margin`, `--warmup` (uses API priors to initialize rates).

### Tuning Tips
- Start safe: `--risk-margin 0.18`, `--warmup 150` (attr‑ewma/ewma).
- Reduce rejections once stable: lower margin to 0.12–0.10; warmup 80–120.
- Use `--alpha 0.05` to adapt quicker when distribution is steady.

### Resume & Error Recovery
- Recommended logging: run with `LOG_INTERVAL=1` and `LOG_JSON=logs/run.ndjson` to capture every step.
- Easiest resume: `make resume-latest POLICY=ewma VERBOSE=1 JSON_LOG=logs/resume.ndjson LOG_INTERVAL=1`
  - Auto-picks the newest `logs/*.ndjson`, rebuilds policy state (constraints, counts, and window/EWMA), and continues the same `gameId` from the next expected `personIndex`.
- Resume from specific log: `make resume RESUME_LOG=logs/run.ndjson ARGS="--policy ewma --verbose"`
- Manual resume: if you see `{"error":"Expected person N, got M"}`
  - Re-send with `--start-index N` (same last decision). Example:
    `make resume GAME_ID=<uuid> START_INDEX=N ARGS="--policy reserve"`
- The runner auto-resyncs when it detects the "Expected person X, got Y" error.
- QA: `make test`, `make lint`, `make fmt`, `make typecheck`
- Install runtime-only deps: `make sync`

## Coding Style & Naming Conventions
- Python 3.11+, 4-space indent, max line length 100.
- Use type hints and docstrings for public functions.
- Names: modules/functions `snake_case`, classes `CapWords`, constants `UPPER_SNAKE`.
- Files: one primary class/module per file when feasible.
- Keep policies pure/deterministic; pass `random.Random(seed)` explicitly.

## Testing Guidelines
- Framework: `pytest` with `pytest-cov` (target ≥90% for policy logic).
- Tests live in `tests/`, named `test_*.py`; functions `test_*`.
- Include scenario fixtures and property-style tests (e.g., constraints never violated).
- Quick checks: `pytest -q -k policy` for selective runs.

## Commit & Pull Request Guidelines
- Use Conventional Commits: `feat: add threshold policy`, `fix(simulator): rounding bug`, `test: add coverage for s2`.
- PRs include: description, scenario(s) touched, reproduction command, before/after metrics (e.g., rejections to fill 1000), and screenshots of summaries if applicable.
- Keep PRs small and focused; add or update tests and docs.

## Security & Configuration Tips
- No secrets in code; keep `.env` out of VCS.
- Keep network calls out of core logic; simulate inputs locally.
- Make randomness/config explicit via CLI flags or `configs/*.yaml`.

---
> Source: [brandonmanke/berghain_challenge](https://github.com/brandonmanke/berghain_challenge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-15 -->
