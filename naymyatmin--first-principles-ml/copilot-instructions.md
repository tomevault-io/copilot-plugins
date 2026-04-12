## first-principles-ml

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A from-scratch ML interview prep repository. Every algorithm is implemented in Jupyter notebooks from first principles using NumPy, with PyTorch used in sections 04 (transformers/LLMs) and 05 (training infrastructure). There are no Python modules or package structure — all code lives in `.ipynb` files.

## Setup and Running

```bash
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt
jupyter notebook
```

Dependencies: numpy, torch, matplotlib, seaborn, scikit-learn, pandas, jupyter, tqdm.

## Repository Architecture

Nine numbered sections, each a directory of self-contained notebooks:

- **01-mathematical-foundations/** — Autograd, optimizers, probability, information theory (NumPy)
- **02-classical-ml-from-scratch/** — 14 algorithms from linear regression to HMMs (NumPy). Highest-ROI section for interview prep.
- **03-deep-learning-from-scratch/** — Neural net library built up incrementally in NumPy: autograd → layers → activations → backprop → normalization → CNN → RNN → attention
- **04-transformer-and-llm/** — Transformer architecture and modern LLM components (**PyTorch**, not NumPy)
- **05-training-infrastructure/** — Production training patterns (**PyTorch**)
- **06-evaluation-and-metrics/** — Classification, regression, ranking, NLP metrics, calibration, statistical tests
- **07-feature-engineering-and-data/** — Missing data, encoding, scaling, imbalanced data, augmentation, CV
- **08-ml-system-design/** — System design interview templates and worked examples
- **09-coding-patterns/** — Algorithmic patterns: similarity search, streaming, time series, DP

## Notebook Structure Convention

Each notebook follows a consistent pattern:
1. **Title + interview context** — what interview questions it addresses
2. **Core intuition** — why the algorithm works, key formulas, complexity analysis
3. **From-scratch implementation** — clean class-based NumPy/PyTorch code
4. **Visualizations** — loss landscapes, convergence plots, decision boundaries
5. **Validation against sklearn/PyTorch** — verifying correctness
6. **Interview quick reference table** — common Q&A pairs

## Key Design Principles

- "Implement, don't import" — sklearn is only used for validation, never as the primary implementation
- Classes use sklearn-compatible `.fit()` / `.predict()` API
- Sections 01-03 and 06-09 use NumPy; sections 04-05 use PyTorch
- Notebooks are self-contained: each generates its own synthetic data, no shared utilities

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NayMyatMin)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/NayMyatMin)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
