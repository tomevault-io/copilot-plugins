## pokemon-cry-stats-predictor

> Data ingestion, feature extraction, and training scripts live in `scripts/`, with `run_full_pipeline.py` and `run_pipeline.sh` acting as the end-to-end entry points. Streamlit UI code resides in `dashboard/` (tabs in `dashboard/tabs/`, helpers in `dashboard/utils/`). Trained artifacts and evaluation reports belong in `models/` and `results/`; treat `data/` as regenerable working storage and keep large audio files out of commits. Use `notebooks/` only for exploratory analysis.

# Repository Guidelines

## Project Structure & Module Organization
Data ingestion, feature extraction, and training scripts live in `scripts/`, with `run_full_pipeline.py` and `run_pipeline.sh` acting as the end-to-end entry points. Streamlit UI code resides in `dashboard/` (tabs in `dashboard/tabs/`, helpers in `dashboard/utils/`). Trained artifacts and evaluation reports belong in `models/` and `results/`; treat `data/` as regenerable working storage and keep large audio files out of commits. Use `notebooks/` only for exploratory analysis.

## Build, Test, and Development Commands
- `pip install -r requirements.txt` — bring in the Python 3.8+ stack used across scripts and the app.
- `python run_full_pipeline.py --max-pokemon 200` — rebuild datasets, retrain models, and write comparison JSON to `results/`.
- `python scripts/train_model_advanced.py --model-type nn` — focus on one model variant; outputs land in `models/`.
- `streamlit run dashboard/app.py` — exercise the dashboard locally and verify tab behavior.
- `python scripts/predict_advanced.py data/cries/pikachu.ogg` — quick regression check for exported models.

## Coding Style & Naming Conventions
Follow PEP 8: four-space indentation, snake_case for functions and variables, PascalCase only for classes, and uppercase module constants such as `TARGET_STATS`. Mirror existing docstrings on new public functions and keep runtime logging concise. Default to English prose in comments while preserving helpful bilingual notes. Wrap long expressions for readability and keep imports ordered standard-library, third-party, then local.

## Testing Guidelines
No formal unit suite exists yet, so rely on the pipeline scripts for regression checks. After changing data prep or training, rerun `run_full_pipeline.py` (or the targeted training script) and compare the new `results/` metrics. For dashboard changes, start Streamlit, traverse every tab, and watch logs for warnings. When you introduce automated tests, add them under `tests/` and run them with `pytest -q`.

## Commit & Pull Request Guidelines
Use imperative, present-tense subjects under 72 characters (e.g., “Add ID preview to bulk add flow”). Emojis are welcome when consistent with history (`🤖 Auto-train models`). Note regenerated models or datasets in commit bodies. PRs should link issues, describe behavior changes, list verification commands, and include screenshots for dashboard updates so reviewers can reproduce results quickly.

## Data & Security Tips
Respect `.gitignore`: do not commit large cry datasets, generated models, or virtual environments. Share reproducible steps (e.g., run the download script) instead of bundling binaries. Keep API tokens in environment variables or Streamlit secrets, and report any sensitive Nintendo material so it can be removed promptly.

## Executable Specifications (ExecPlans)

This repository follows a rigorous specification-driven development workflow defined in `.agent/PLANS.md`. When creating or implementing features:

1. **Read `.agent/PLANS.md` thoroughly** - It contains comprehensive guidelines for writing executable specifications that enable any contributor to implement features from scratch.

2. **ExecPlans are living documents** - They must contain:
   - `Progress`: Checkbox list with timestamps tracking granular steps
   - `Surprises & Discoveries`: Unexpected behaviors or insights discovered during implementation
   - `Decision Log`: All design decisions with rationale and dates
   - `Outcomes & Retrospective`: Summary of achievements, gaps, and lessons learned

3. **Key principles from PLANS.md:**
   - **Self-contained**: Include all knowledge needed for a novice to succeed
   - **Observable outcomes**: Define what users can do after implementation and how to verify it
   - **Plain language**: Define all technical terms; avoid undefined jargon
   - **Idempotent**: Steps should be safe to run multiple times
   - **Validation-focused**: Include exact test commands and expected outputs

4. **When implementing from an ExecPlan:**
   - Do not prompt for "next steps" - proceed autonomously to the next milestone
   - Keep all sections up to date as you progress
   - Resolve ambiguities independently
   - Commit frequently
   - Update the Decision Log when making significant choices

5. **Format**: ExecPlans are written as single markdown documents with specific sections (see skeleton in PLANS.md). When writing to a file, omit triple backticks; when including inline, use a single fenced code block.

---
> Source: [shiyow5/pokemon-cry-stats-predictor](https://github.com/shiyow5/pokemon-cry-stats-predictor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
