## ml-mania-2026

> - Goal: build machine learning models for Kaggle's March Machine Learning Mania 2026 competition.

# AGENTS.md

## Project Overview

- Goal: build machine learning models for Kaggle's March Machine Learning Mania 2026 competition.
- The repository currently contains raw competition data in `data/` plus the local data dictionary in `data-description.md`.
- We only care about Stage 2.
- The solution must be LightGBM only, including any pooled, separate-division, single-model, or ensemble variants, and each inclusion must be justified by validation.
- The main prediction target is the submission format shown in `data/SampleSubmissionStage2.csv`.
- Submission rows use `ID=Season_LowerTeamID_HigherTeamID`, and `Pred` is the win probability for the lower TeamID team.
- The competition combines both men's (`M...`) and women's (`W...`) tournaments in one submission. Keep those data flows explicit so features are not mixed accidentally.
- Stage 2 requires predictions for every possible team matchup in 2026, not just the final tournament field.
- The local data description says the current-season files are complete through February 4, 2026. If a task depends on fresher competition data, verify whether the local files have been updated before modeling.
- Competition-host refreshes matter for current-season features: expect one final full refresh on Selection Sunday or the following morning, and treat late-week `DayNum=132` Massey updates as valid pre-deadline inputs when they become available.
- ONLY STOP/END THE TURN WHEN YOU REACH THE STOP CONDITION SET BY THE USER: As we get deeper into research, it's expected that we will have many iterations without progress in our metric. This is completely normal. Don't get discouraged by it and just look for new angles and unexplored approaches. 

## Evaluation And Validation

- Kaggle evaluates submissions with Brier score, which here is mean squared error on binary game outcomes.
- Local model selection should be driven by walk-forward validation by season.
- Always run the full walk-forward validation over the latest 5 available completed seasons.
- Use each year as its own validation fold with training restricted to prior seasons only.
- Validation-year features may be built for that season, but only from information that would have been available by that season's Selection Sunday or equivalent final pre-tournament data refresh.
- Report mean Brier across those 5 folds, and keep the individual fold scores for comparison.
- Any model, feature set, or ensemble must be accepted or rejected based on this validation.
- A candidate is only considered improved if it lowers mean validation Brier and improves at least 4 of the 5 folds.
- If a candidate lowers the average validation Brier but improves fewer than 4 of the 5 folds, discard it as not improved.
- Do not use random row splits for primary validation.
- Do not skip folds to iterate faster. Speed should come from caching and efficient feature generation, not from partial validation.
- Because 2026 Stage 2 submissions score `0.0` before the tournaments are played, do not use the Kaggle leaderboard for model selection before live games begin.
- Play-in games do not affect official scoring, so do not spend meaningful model-selection effort optimizing predictions for those matchups.

## Repository Layout

- `data/`: raw Kaggle CSV inputs. Treat these as immutable source data.
- `data-description.md`: competition data dictionary and submission semantics.
- There is currently no Python package, no test suite, and no build configuration in this repo. Do not invent extra project structure unless the user asks for it.

## Workflow Expectations

- Start with simple checks before deeper changes: schema inspection, season coverage, duplicate game keys, null handling, and row-count sanity checks.
- Prefer Polars `LazyFrame` pipelines for feature engineering and joins. Push work into the Polars engine instead of Python loops.
- Use `duckdb` or lazy CSV scans for large-file inspection. `data/MMasseyOrdinals.csv` is especially large, so avoid eager full-file loads unless necessary, especially when reprocessing updated `DayNum=132` snapshots during submission week.
- The local machine has 32 GB RAM and 4 CPU cores. Favor memory-aware transformations, cached intermediates, and reusable feature tables.
- Keep raw inputs untouched. If new output directories are needed, create user-facing locations such as `artifacts/`, `models/`, or `submissions/` only when required and keep large generated files out of git.
- Use `data/SampleSubmissionStage2.csv` as the canonical 2026 submission target.
- Maintain a root `JOURNAL.md` and append a short entry for each completed run that changes features, params, or ensemble composition.
- Default experiment runs should be validation-only. Do not generate a submission file or dump trained model files for every experiment.
- Accepted experiments should still be submission-capable in design, even when the script only runs validation.
- For any accepted candidate, keep a clear path to build the same feature set and probability mapping for the full 2026 Stage 2 submission universe once the required current-season files are available.
- Only create a full Stage 2 submission file and persist trained model artifacts when the user explicitly asks for them.

## Modeling Rules

- Avoid target leakage. Validation splits should be season-based and should mirror the information available at each season's Selection Sunday or equivalent final pre-tournament data refresh.
- When converting historical games into supervised rows, normalize the representation carefully so winner/loser columns do not leak directly into the target.
- Canonicalize matchup rows to lower TeamID and higher TeamID when training submission-aligned models. Keep labels and feature orientation consistent so predictions mean `P(lower TeamID wins)`.
- Men and women may be modeled together or separately. Pooled models, division-specific models, and hybrids are all allowed if they improve validation, preserve clear feature semantics, and remain LightGBM-only.
- Prioritize feature engineering and LightGBM modeling that capture team strength, recent form, schedule strength, matchup deltas, seed-derived context when available, and ranking features that respect timing.
- The final solution may be a single LightGBM model or a LightGBM-only ensemble, and every inclusion in that solution must earn its place through the mandatory 5-fold walk-forward validation.
- Current-season tournament seeds and other final pre-tournament fields are allowed for final submission and for historical validation folds when they would have been available by that season's Selection Sunday-style refresh.
- Current-season Massey Ordinals are allowed using the latest pre-deadline snapshot available for that stage of the workflow. Historical validation should mirror the best Selection Sunday-style cutoff available for that season, while final 2026 submission experiments may also use later submission-week `DayNum=132` updates.
- Submission generation must cover every possible same-division matchup required by Stage 2, not just seeded tournament teams.
- Do not accept feature pipelines that only work on historical tournament-result rows unless there is a clear, documented way to construct the same features for all 2026 submission matchups.
- When a candidate is accepted, verify that its feature definitions, matchup orientation, and calibration approach can be reused for final-train and final-predict inference without changing the modeling idea.
- When using external team data, map names through `MTeamSpellings.csv` or `WTeamSpellings.csv` before joining.

## Experiment Tracking

- Use `JOURNAL.md` at the repository root to log experiments.
- Log every completed run that changes features, params, or ensemble composition with date, hypothesis, fold seasons, feature set summary, LightGBM configuration summary, mean Brier, fold-level Brier scores, artifact paths, and the next action.
- If a run fails or is aborted, log the failure reason briefly so the same dead end is not retried blindly.
- Prefer short, append-only journal entries over rewriting prior conclusions.
- For routine rejected experiments, a validation summary is enough. Do not require model dumps or submission artifacts to consider the run complete.

## Useful Commands

The repo does not yet contain a runnable pipeline. Prefer lightweight research and data inspection commands such as:

- `duckdb -c "describe select * from read_csv_auto('data/MTeams.csv')"`
- `duckdb -c "select count(*) from read_csv_auto('data/MRegularSeasonCompactResults.csv')"`

If a command depends on files or configuration that do not exist yet, say that clearly instead of pretending the project already has that tooling.

## Code Style Guidelines

- Preserve existing formatting, imports, and file organization.
- Favor small, incremental edits. Do not refactor unrelated code.
- Use descriptive names and readable code.
- Put all library imports at the top of Python files.
- Use `pathlib` for filesystem work, `orjson` for JSON, `httpx` for HTTP, `joblib` for serialization, and `seaborn` for plotting.
- Use timezone-aware datetimes such as `datetime.datetime.now(datetime.timezone.utc)`.
- Do not use pandas in this repo. Prefer Polars, ideally lazy APIs.

## Polars Notes

- Prefer `scan_csv`, lazy joins, lazy aggregations, and parquet caching only when justified.
- Use `.collect_schema().names()` to inspect lazy column names.
- Do not allow object dtypes to leak into lazy plans when reading nested or irregular data.
- For `join_asof`, pass a real `datetime.timedelta` or plain numeric tolerance, not a Polars duration expression.
- Align datetime time units explicitly before as-of joins.

## Security And Data Handling

- Never commit Kaggle credentials, tokens, cookies, or local secrets.
- Never overwrite the raw CSV files in `data/`.
- Be explicit about the provenance of any external data source or handcrafted ranking file.
- Keep large intermediate files, trained models, notebook outputs, and cached artifacts out of git unless the user explicitly asks otherwise.

## Commit And Review Notes

- Commit after each completed run that changes features, params, or ensemble composition.
- Commit messages should use a concise imperative subject and optional bullet points for important details.
- Do not put email addresses in commit messages.
- Do not use backticks inside `git commit -m` arguments.
- After creating or amending a commit, verify the message with `git show -s --format=%B HEAD`.
- In reviews and summaries, call out leakage risks, validation assumptions, data version dependencies, and anything not yet verified.

---
> Source: [ledmaster/ml-mania-2026](https://github.com/ledmaster/ml-mania-2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
