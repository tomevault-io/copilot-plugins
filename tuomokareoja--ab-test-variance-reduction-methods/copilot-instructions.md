## ab-test-variance-reduction-methods

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository is a simulation-driven playground for analyzing the behavior of different A/B test variance reduction methods across progressively complex experiments. The project evaluates six different estimators (t-test, change-score, diff-in-diff, CUPED, simple covariate adjustment, and causal-graph IPW adjustment) under various experimental scenarios.

## Key Commands

### Environment Setup
```bash
# Create conda environment
conda env create -f environment.yml

# Activate environment
conda activate ab-test-variance-reduction-methods
```

### Run Pipeline
```bash
# Run the full simulation and evaluation pipeline
dvc repro

# Run specific stages
dvc repro simulate_experiments
dvc repro evaluate_methods
```

### Launch Interactive App
```bash
streamlit run app/main.py
```

### Testing
```bash
# Run all tests
pytest tests/

# Run a specific test file
pytest tests/test_file.py
```

### Code Quality
```bash
# Run linting
ruff check .
```

## Project Structure

The project uses DVC (Data Version Control) to manage the data pipeline with the following stages:
1. `simulate_experiments`: Generates simulated experiment data
2. `evaluate_methods`: Evaluates different statistical methods on the simulated data

The simulation considers different scenarios:
- Post-only measurements (baseline A/B test)
- Pre + Post measurements (utilizing correlation for variance reduction)
- Outcome-only covariate (ANCOVA usefulness)
- Selection-bias covariate (requires causal adjustment)

## Implementation Details

### Key Files
- `simulate.py`: Creates simulated experiments based on parameters
- `evaluate.py`: Applies statistical methods to the simulated experiments
- `plot.py`: Generates visualizations of method performance
- `params.yml`: Configuration parameters for the simulations

### Data Flow
1. Parameters are read from `params.yml`
2. Simulation data is generated and stored in `experiments/` directory
3. Statistical methods are applied to the data with results in `results/` directory
4. Visualizations are created in the `plots/` directory

Note: Heavy artifacts (like `results/`) are excluded via `.gitignore`; use `dvc repro` to regenerate them.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TuomoKareoja) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
