## chemprop4molalkit

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ChemProp for MolALKit (v1.5.2) - A customized version of ChemProp optimized for molecular active learning. Uses Message Passing Neural Networks (MPNNs) to predict molecular properties from SMILES strings. Utilized by [MolALKit](https://github.com/RekerLab/MolALKit).

## Build and Development Commands

```bash
# Install with conda
conda env create -f environment.yml
conda activate chemprop
pip install -e .

# Run all tests
pytest -v

# Run a single test file
pytest tests/test_integration.py -v

# Run a specific test
pytest tests/test_integration.py::TestTrainAndPredict::test_chemprop_train_single_task_regression -v

# Linting
flake8 . --max-complexity=10 --max-line-length=127
```

## CLI Entry Points

All commands are defined in `setup.py` as console scripts:

- `chemprop_train` - Train models on CSV data
- `chemprop_predict` - Make predictions with trained models
- `chemprop_cv` - K-fold cross-validation (Optuna-based)
- `chemprop_optuna` - Hyperparameter optimization with Optuna
- `chemprop_hyperopt` - Bayesian HPO with hyperopt library
- `chemprop_fingerprint` - Extract molecular fingerprints
- `chemprop_interpret` - Model interpretation
- `sklearn_train/sklearn_predict` - Scikit-learn model wrappers

## Architecture

### Data Flow
```
CSV → get_data() → MoleculeDataset → MoleculeDataLoader → Training
                        ↓
              MoleculeDatapoint (SMILES + targets)
                        ↓
              Featurization (RDKit) → BatchMolGraph
```

### Core Modules

- **`chemprop/args.py`** - All CLI arguments using TAP (Typed Argument Parser). Start here to understand configuration options.
- **`chemprop/data/`** - Data loading: `MoleculeDatapoint`, `MoleculeDataset`, `MoleculeDataLoader`, splitting strategies
- **`chemprop/models/`** - Neural networks: `MoleculeModel` wraps `MPN` (message passing) + FFN layers
- **`chemprop/train/`** - Training loop, cross-validation, prediction, metrics, loss functions
- **`chemprop/features/`** - Feature generators (Morgan fingerprints, RDKit 2D), atom/bond featurization
- **`chemprop/optuna/`** - Optuna-based HPO and cross-validation

### Model Architecture

```
MoleculeModel
├── MPN (Message Passing Network)
│   └── MPNEncoder: atom/bond embeddings → message passing → output projection
│       └── Optional: CBPLinear for continual backpropagation
└── FFN (Feed-Forward Network) → predictions
```

### Key Design Patterns

- **TAP Arguments**: Both CLI and programmatic usage via `args.parse_args([])` or direct attribute setting
- **Aggressive Caching**: Molecules, graphs, and features cached in-memory
- **Multi-task Learning**: Single model predicts multiple targets
- **Loss-Dependent Output**: Output layer size/activation varies by loss function (MSE, cross-entropy, MVE, Evidential, etc.)
- **StandardScaler**: Targets normalized before training, stored in checkpoint

### Recent Custom Features (from git history)

- `--features_columns` - Use CSV columns as features alongside `--features_generator`
- `--cbp` - Continual backpropagation via CBPLinear layers
- ConstantLR scheduler option
- Optuna integration for HPO and CV

## Key Files for Common Tasks

| Task | Key Files |
|------|-----------|
| Add new CLI argument | `chemprop/args.py` |
| Modify training loop | `chemprop/train/train.py`, `chemprop/train/run_training.py` |
| Add new loss function | `chemprop/train/loss_functions.py` |
| Add new metric | `chemprop/train/metrics.py` |
| Modify model architecture | `chemprop/models/model.py`, `chemprop/models/mpn.py` |
| Add feature generator | `chemprop/features/features_generators.py` |
| Modify data loading | `chemprop/data/data.py`, `chemprop/data/utils.py` |

---
> Source: [Xiangyan93/chemprop4molalkit](https://github.com/Xiangyan93/chemprop4molalkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
