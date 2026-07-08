## mlops-portfolio

> > Agent context for the MLOps Learning Portfolio — VAE-Based Crash Severity Pipeline.

# AGENTS.md

> Agent context for the MLOps Learning Portfolio — VAE-Based Crash Severity Pipeline.
> This file complements CLAUDE.md (architecture deep-dive) and tasks.md (execution checklist).

---

## Project Identity

- **Name**: MLOps Learning Portfolio — Crash Severity Use Case
- **Dataset**: CGR Crash Data (Grand Rapids, ~74k rows, 142 cols)
- **Target**: 3-class crash severity — PDO (0), Injury (1), Fatal (2)
- **Constitution**: v3.4.0 | **Architecture**: 10-stage DVC pipeline
- **Macro F1 Gate**: > 0.35 | **Fatal Recall Gate**: > 0.50 (PDO sacrifice accepted)

---

## What Is This Project

A production-grade MLOps reference implementation demonstrating the full toolchain:

- **DVC** — data / model versioning + reproducible pipeline DAG
- **Great Expectations** — data contract validation before any training
- **MLflow** — experiment tracking, model registry, champion alias
- **Optuna** — active hyperparameter search engine (local, continuous space, pruning)
- **Katib** — Kubernetes-native HPO (portfolio reference, retained but not active)
- **Kubeflow Pipelines** — containerised orchestration on Docker Desktop K8s
- **PyTorch + XGBoost** — competing classifiers on VAE-learned latent representations
- **CTGAN/TVAE** — generative augmentation of Fatal-class rows

Pipeline DAG:

```
validate -> ingest -> featurize -> [train_vae || augment] -> encode -> [train_ml || train_dl] -> evaluate -> tune -> register
```

---

## Current State (as of last update)

**Position**: Phase O3.5 (Optuna HPO) + Phase M5 (VAE Fatal Recall Fixes).

**Current metrics**: val recall=52.4% OK, test recall=33.3% FAIL -- 4 more correct Fatal predictions needed on Z_test.

**Next work**: See `specs/002-mlops-portfolio/tasks.md` for the canonical step-by-step execution order. Key open items:

- T129: Wire `OptunaTuner` into `src/tune/run.py` (update `params.yaml` with best params + `dl.input_dim` sync)
- T130: Update `dvc.yaml` tune stage params list
- T131: `uv run dvc repro tune` smoke test
- T132: `uv run dvc repro` full pipeline -- check gates
- Decision: gates pass -> T051 (Register). Gates fail -> M5 fixes in order (Fix A focal loss -> Fix B cyclical KL -> Fix C danger index -> Fix D XGBoost focal -> Fix E supervised latent -> Fix F Tomek links).

---

## Directory Quick Reference

| Path | Purpose |
|------|---------|
| `src/<stage>/` | One package per pipeline stage: `<module>.py` (business-logic class) + `run.py` (thin entry point) |
| `src/config.py` | Typed dataclasses for all `params.yaml` sections; `load_config()` reads `PARAMS_PATH` env var |
| `src/metrics.py` | Shared helpers: `make_eval_dataset`, `per_class_matrix`, `compute_class_weights`, `BalancedFocalLoss` |
| `params.yaml` | Single source of truth for all parameters, column lists, thresholds, HPO search spaces |
| `dvc.yaml` | 10-stage DVC pipeline DAG with deps / outs / params |
| `docs/` | Human-readable artifacts: `data_contract.md`, `evaluation_report.json`, `ab_test_comparison.json` |
| `tests/` | Boundary tests only; real data fixtures; no internal mocking |
| `great_expectations/gx/` | GE v1 file context; suites in `expectations/`; Data Docs in `uncommitted/data_docs/` |
| `k8s/` | Kubernetes manifests: `pvc.yaml`, `katib/vae_experiment.yaml` |
| `pipelines/kubeflow/` | KFP pipeline definition (`pipeline.py`) |
| `docker/` | `Dockerfile` for container-native stage execution |
| `airflow/` | Tutorial DAGs only; not part of active pipeline |
| `.specify/memory/constitution.md` | 18 non-negotiable principles; amendment requires version bump + rationale |
| `CLAUDE.md` | Full architecture reference (GE layer, VAE details, MLflow conventions, skills list) |
| `UBIQUITOUS_LANGUAGE.md` | Canonical domain glossary; must be updated before speckit.plan |

---

## Essential Commands

### Environment
- Use `uv` for all Python operations: `uv add <pkg>`, `uv sync`, `uv run ...`
- Virtualenv is at `.venv/`; Python version pinned in `.python-version`

### Tests (Windows — obey Constitution XVII)
```powershell
uv run python -m pytest tests/ -v
uv run python -m pytest tests/test_optuna_tuner.py -v
```
> Do NOT use `uv run pytest` (console-script form fails on Windows with "Failed to canonicalize script path").

### DVC
```bash
uv run dvc repro              # full pipeline
uv run dvc repro <stage>      # single stage
uv run dvc status             # cache state
uv run dvc push               # sync artifacts to remote
uv run dvc pull               # restore artifacts
```

### MLflow
```bash
uv run mlflow ui              # http://localhost:5000
```

### Katib (portfolio reference)
```bash
kubectl apply -f k8s/katib/vae_experiment.yaml
```

### Airflow (tutorial only)
```powershell
cd airflow
.\setup.ps1                   # one-time
uv run airflow standalone     # http://localhost:8080
```

---

## Development Rules (Constitution Highlights)

These are non-negotiable. Violations block task completion.

1. **TDD for all `src/` code** — red -> green -> refactor; vertical slices only; tests written BEFORE implementation.
2. **Real data fixtures only** — boundary tests must use real pipeline artifacts (e.g. `data/processed/raw.csv`, `Z_train_augmented.npy`). No `np.random.randn` or hardcoded dummy frames as primary fixtures.
3. **No ad-hoc data quality assertions** — all data quality checks belong in the GE suite (`params.yaml validation.columns`). Stage code and tests must not contain manual `assert df.notna().all()` style checks.
4. **ASCII-only terminal output** — no emoji, no Unicode arrows, no bullets outside ASCII (U+0000–U+007F) in any `src/` print/log. Use `[OK]`, `[ERROR]`, `[WARN]` instead.
5. **Deep module architecture** — each stage = one class with a small public interface (constructor + 1 primary method). `run.py` is the ONLY file that reads files, env vars, and logs to MLflow.
6. **Parameterize everything** — column lists, split ratios, thresholds, architecture dims all live in `params.yaml`. No magic numbers in code.
7. **3-way split sanctity** — train fits weights; val is for early stopping / HPO fitness; test is for final A/B evaluation ONLY. Never train or search on test data.
8. **Feature leakage prevention** — no post-crash columns as model inputs. Forbidden list is in Constitution I and `docs/data_contract.md`.
9. **Class imbalance** — at most three mechanisms: (1) runtime class weights, (2) CTGAN/TVAE on X_train Fatal rows only in `augment` stage, (3) KL annealing in VAE. No SMOTE/ADASYN.
10. **Update `UBIQUITOUS_LANGUAGE.md`** with new terms before starting any task that introduces domain language (see plan.md Pre-Tasks Actions Required).
11. **Verify Constitution gates** via `plan.md`'s Constitution Check table before generating implementation plans.

Full constitution with 18 principles, quality gates, and platform notes: `.specify/memory/constitution.md`.

---

## Stage Execution Order

| Order | Stage | Parallel with | Blocks |
|-------|-------|---------------|--------|
| 1 | validate | — | ingest |
| 2 | ingest | — | featurize |
| 3 | featurize | — | train_vae, augment |
| 4a | train_vae | augment | encode |
| 4b | augment | train_vae | encode |
| 5 | encode | — | train_ml, train_dl |
| 6a | train_ml | train_dl | evaluate |
| 6b | train_dl | train_ml | evaluate |
| 7 | evaluate | — | tune, register |
| 8 | tune | — | writes best params -> invalidates train_vae downstream |
| 9 | register | — | — |

---

## Key Configuration Files

- **`params.yaml`** — all hyperparameters, column definitions, thresholds, and Optuna search space bounds.
- **`dvc.yaml`** — pipeline DAG. When `params.yaml` changes, DVC invalidates downstream stages automatically.
- **`src/config.py`** — `load_config()` converts `params.yaml` into typed dataclasses (`VAEConfig`, `DLConfig`, `ModelConfig`, `OptunaConfig`, etc.).

> When Optuna writes best params back to `params.yaml`, it must also sync `dl.input_dim = latent_dim`.

---

## Decision Gates

### After `evaluate` (T132)
```
eout_fatal_recall > 0.50  AND  eout_macro_f1 > 0.35
    YES  ->  go to STEP 9 (Register / T051)
    NO   ->  go to STEP 5 (Model Fixes / M5)
```

### M5 Fix Order (stop as soon as gates pass)
1. **Fix A** — MLP Balanced Focal Loss (cheapest; no upstream cascade)
2. **Fix B** — Cyclical KL Annealing (VAE cascade: train_vae -> encode -> ...)
3. **Fix C** — Danger Index Features (full 6-stage cascade from featurize)
4. **Fix D** — XGBoost Focal Loss (last automated resort)
5. **Fix E** — Supervised Latent Loss (BLOCKED — Constitution II amendment required)
6. **Fix F** — Tomek Links (BLOCKED — Constitution III amendment required)

Detailed instructions and task numbers are in `specs/002-mlops-portfolio/tasks.md`.

---

## MLflow Conventions

- **Tracking URI**: set via `params.yaml mlflow.tracking_uri`; every stage calls `mlflow.set_tracking_uri(...)` first.
- **Experiments**: `crash-severity-vae`, `crash-severity-ml`, `crash-severity-dl`, `crash-severity-tune`
- **Mandatory metrics per run**: `ein_macro_f1`, `eout_macro_f1`, `generalisation_gap`, `eout_fatal_recall`
- **VAE per-epoch metrics**: `vae_elbo`, `vae_reconstruction_loss`, `vae_kl_loss`, `kl_beta`
- **Autolog disabled** — all metrics logged explicitly.
- **Champion alias**: `models:/crash-severity@champion`

---

## Skills Available

Project-level (`load skill` when task matches):
- `data-scientist`, `mlflow`, `mlops-engineer`, `airflow-dag-patterns`, `grill-me`, `ubiquitous-language`, `great-expectations`, `caveman-review`
- `tdd` — when doing red-green-refactor
- `diagnose` — when debugging bugs / performance regressions
- `improve-codebase-architecture` — when refactoring / deepening modules

---

## Useful References

- `CLAUDE.md` — exhaustive architecture docs, class descriptions, GE layer diagram, conventions.
- `specs/002-mlops-portfolio/tasks.md` — canonical execution checklist (T001–T138) with done/pending status.
- `UBIQUITOUS_LANGUAGE.md` — canonical glossary. Update before introducing new domain terms.
- `.specify/memory/constitution.md` — 18 non-negotiable principles and quality gates.
- `docs/data_contract.md` — human-readable column contracts (dtype, ranges, nulls, sentinels).

---
> Source: [lorenzo1285/mlops-portfolio](https://github.com/lorenzo1285/mlops-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
