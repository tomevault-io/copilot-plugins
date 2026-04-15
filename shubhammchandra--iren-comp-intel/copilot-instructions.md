## iren-comp-intel

> Step-by-step guides for common multi-file changes in this project


# Common Multi-File Tasks

## Adding a New Signal Type
1. `config.py` — add to `SIGNAL_TYPES` list and to `SIGNAL_LABELS` and `SIGNAL_DESCRIPTIONS` (for README)
2. `scoring/weights.py` — add entry to `SIGNAL_WEIGHTS` (max_points, base_points, halflife). Ensure all max_points still sum to 100
3. `database/models.py` — add `<name>_score` column to `ProspectScore`
4. `scoring/engine.py` — add the new field to the `ProspectScore(...)` constructor in `score_prospect()`
5. Every page's icon map — add icon to `TYPE_ICONS` in `dashboard.py`, `prospects.py`, `signal_feed.py`
6. `tests/test_scoring.py` — update `test_all_signal_types_have_weights` and `test_max_points_sum_to_100`
7. Run `python scripts/update_readme.py` to refresh the README scoring table, then commit the updated README
8. Run `pytest` to verify

## Adding a New Collector
1. Create `collectors/<name>_collector.py` inheriting from `BaseCollector`
2. Set `collector_name = "<name>"`
3. Implement `collect()` method, ending with `self.finish()`
4. Use `self._create_signal()` for all signal creation (handles exact-title + semantic dedup)
5. Register in `collectors/runner.py` `COLLECTORS` dict
6. Add tests in `tests/test_collectors.py`

## Adding a New Company
- Via UI: Settings page → Manage Companies tab
- Via code: add dict to `PROSPECTS` or `COMPETITORS` in `database/seed.py`
- Required fields: `name`, `industry`, `website`, `description`, `hq_location`

## Re-scoring After Changes
After modifying weights, adding signals, or changing scoring logic:
```bash
python -c "from scoring.engine import score_all_prospects; score_all_prospects()"
```

## Running Competitive Intel Collection
After adding competitors or to refresh competitive events:
```bash
python -m collectors.runner competitive_intel
```
This searches the web for each competitor's recent deals, expansions, pricing changes, and talent moves. Results are stored as `CompetitorEvent` rows.

## Backfilling Embeddings
After adding new signals without Ollama running, or to embed all existing signals:
```bash
python -m ai.embed_backfill           # only signals missing embeddings
python -m ai.embed_backfill --force   # re-embed everything
```
Requires Ollama running with `nomic-embed-text` pulled (`ollama pull nomic-embed-text`).

## README auto-update
When you change signal types, scoring weights in `scoring/weights.py`, or add a collector, run `python scripts/update_readme.py` from the repo root and commit the updated README. The script regenerates the Scoring Model table from `config.py` and `scoring/weights.py`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ShubhamMChandra) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
