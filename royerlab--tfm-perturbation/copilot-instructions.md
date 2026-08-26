## tfm-perturbation

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`tfm_perturbation` benchmarks **tabular foundation models** (TabICL, TabPFN) for
predicting cellular responses to perturbations. The core pipeline matches cells
between a labelled *source* cell type and held-out *target* cell types — using
nearest-neighbour (`nn`) or optimal-transport (`ot`) matching — and predicts the
target response with a tabular foundation model or a CatBoost baseline. It also
includes the PRESAGE model and classical fit-to-data baselines for comparison.

Primary data modality is single-cell transcriptomics (CROP-seq / Perturb-seq);
optional multimodal gene embeddings can be stacked as additional features.

This work is inspired by the scGeneScope paper ([OpenReview](https://openreview.net/forum?id=918POZbZ50)), which introduced a treatment-matched benchmark for evaluating multi-modal perturbation response modeling.

## Development Commands

### Installation
```bash
pip install -e ".[dev]"
```

### Testing
```bash
pytest                                    # Run all tests
pytest tests/test_file.py::test_func     # Run specific test
pytest --cov=tfm_perturbation             # With coverage
```

### Code Quality
```bash
ruff check src/ experiments/             # Lint
mypy src/                                 # Type check
pre-commit run --all-files               # All hooks
```

### Running Experiments
```bash
# Cell-match pipeline (TabICL/TabPFN/CatBoost + NN/OT) via the consolidated CLI
tfm-cellmatch --model tabicl --matching ot --data data.h5ad \
    --source-cell-type "T cells" --target-cell-types "NK cells" "B cells"

# Train PRESAGE model directly via Hydra
python experiments/scripts/train.py --config-name presage

# Override config values
python experiments/scripts/train.py --config-name presage data.split_id=0 model.max_epochs=100
```

Output locations default to `./outputs` and can be redirected with the
`TFM_OUTPUT_ROOT` environment variable.

## Architecture

### Hydra Configuration System
The project uses Hydra for configuration management. Configs are composed from:
- `experiments/configs/config.yaml` - Main config with defaults
- `experiments/configs/data/` - Data paths and split configs
- `experiments/configs/model/` - Model architectures (transcriptomics/)
- `experiments/configs/evaluation/` - Evaluation settings

Models and trainers are instantiated via `hydra.utils.instantiate()` using `_target_` fields in YAML configs.

### Core Components

**Models** (`src/tfm_perturbation/models/`):
- `BaseModel` - Abstract interface requiring `forward()`, `get_representation()`, `compute_loss()`
- `PRESAGEModel` - PRESAGE wrapper for perturbation prediction
- `TabICLMultiviewModel`, `CatBoostMultiviewModel` - Multiview baselines
- `PCAModel`, `LinearModel` - Simple baselines

**Evaluation** (`src/tfm_perturbation/evaluation/`):
- `ClassifierEvaluator` - Logistic regression and k-NN evaluation on learned representations
- `PresageEvaluatorWrapper` - PRESAGE-specific evaluation suite

**Data Flow**:
1. `train.py` instantiates model via Hydra
2. Model loads data, performs splits, preprocesses
3. Model trains and extracts latent representations
4. Evaluator runs metrics on representations

## Code Standards

- **Docstrings**: Numpy-style required for public functions/classes
- **Line length**: 88 characters
- **Type hints**: Required for all functions in `src/`
- **Linting**: ruff with pycodestyle, pyflakes, pydocstyle, isort, pyupgrade
- **Type checking**: mypy in strict mode

## Benchmark Design Principles

- Design evaluation splits that simulate realistic experimental scenarios (replicate-based, experiment-based)
- Test under conditions with laboratory variability
- Compare foundation models against classical fit-to-data baselines
- Enable evaluation of treatment identification and MoA discovery tasks

---
> Source: [royerlab/tfm-perturbation](https://github.com/royerlab/tfm-perturbation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
