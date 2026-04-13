## diabetes-2025

> This file defines how an automated coding agent (or AI assistant) should work in this repo.

# Agent Instructions

This file defines how an automated coding agent (or AI assistant) should work in this repo.

## Safety rules

- Do **not** upload submissions to Kaggle unless the user explicitly asked you to submit.
- Never print or commit secrets:
  - `kaggle.json` is a credential file and must stay local (it is gitignored).
  - Avoid echoing file contents of credentials or tokens.
- Prefer `--dry-run` flags for submission scripts when verifying behavior.
- Do not make large, speculative refactors. Keep changes minimal and targeted.

## Environment assumptions

- OS: Linux
- Python: use the repo virtualenv in `.venv/`.
- Always prefer `python` *inside* the venv, otherwise use `python3`.

Recommended setup commands:

```bash
cd /path/to/diabetes-2025
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

## Repo conventions

### Data

- `data/train.csv` may be large; prefer chunked reads for quick utilities.
- `data/` is gitignored; scripts should not assume it is tracked in git.

### Outputs

- Submissions go to `submissions/`.
- Manifests are CSVs with columns: `file,recipe`.
- `scratch/` is for local exploration and is gitignored.

### Kaggle

- Competition slug used by default scripts: `playground-series-s5e12`.
- Authentication is via `kaggle.json` (either `KAGGLE_CONFIG_DIR` or `--kaggle-config-dir`).

## How to add a new model or blend

1) Implement a script that outputs a standard Kaggle submission CSV with columns:
   - `id`
   - `diagnosed_diabetes`
2) Save it under `submissions/` with a descriptive filename.
3) Add it to a blend pack (or to a manifest) via:
   - `scripts/prepare_tomorrow_blend_pack.py`
4) Curate best candidates:
   - `scripts/make_best_submit_manifest.py`
5) Upload (only on explicit request):
   - `scripts/submit_from_manifest.py`

## OOF artifacts for learned blending

Some scripts can dump standardized `.npz` prediction artifacts with:
- `train_id`, `y`, `oof`, `test_id`, `test`, `name`

These are consumed by `scripts/oof_weight_blend.py` and the optional OOF-blend path in `scripts/prepare_tomorrow_blend_pack.py`.

## Testing / verification

- For quick syntax checks: `python -m py_compile scripts/<file>.py`
- For “no-op” upload verification: use `--dry-run` where available.
- Prefer small smoke tests when adding new utilities.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mnem0nic7) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
