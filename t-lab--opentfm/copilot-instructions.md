## opentfm

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Environment Setup

Always set `PYTHONPATH` to include the implementation directory before running anything:

```bash
export PYTHONPATH=$PWD/implementation
```

## Project Overview

This is a trainable PyTorch implementation of the TabPFN-3 architecture from `technical_report/TabPFN_3_model_report.pdf`. There are two inference paths:

1. **Local trainable model** (`tabpfn3_paper.model.TabPFN3Classifier`, `tabpfn3_paper.regressor.TabPFN3Regressor`) — the architecture from the report, trainable from scratch or continued from local checkpoints.
2. **Official pretrained backend** (`tabpfn3_paper.official_backend`) — wraps the released `tabpfn==8.0.1` package for paper-level accuracy (requires a Prior Labs license token `TABPFN_TOKEN` or local licensed checkpoint `TABPFN_MODEL_PATH`).

## Commands

### Training

DDP training on 2 GPUs (the only supported configuration):

```bash
# Quick smoke test
torchrun --standalone --nproc_per_node=2 -m tabpfn3_paper.train \
  --profile tiny --steps 2 --batch-size 1 --n-context 16 --n-query 8 \
  --n-features 6 --n-classes 3 --out implementation/checkpoints/tiny_ddp.pt

# Resume from checkpoint
torchrun --standalone --nproc_per_node=2 -m tabpfn3_paper.train \
  --profile small --resume implementation/checkpoints/checkpoint.pt \
  --steps 2000 --batch-size 2 --n-context 128 --n-query 64 \
  --n-features 16 --n-classes 4 --out implementation/checkpoints/continued.pt

# Report-style prior with mixed task sampling
torchrun --standalone --nproc_per_node=2 -m tabpfn3_paper.train \
  --profile small --resume implementation/checkpoints/checkpoint.pt \
  --steps 2000 --batch-size 2 \
  --sample-task-shapes --min-context 64 --max-context 160 \
  --min-features 8 --max-features 32 --min-classes 2 --max-classes 12 \
  --prior-style mixed --out implementation/checkpoints/dynamic.pt

# Regressor training
torchrun --standalone --nproc_per_node=2 -m tabpfn3_paper.train_regressor \
  --profile tiny --steps 2 --batch-size 1 --n-context 16 --n-query 8 \
  --n-features 6 --out implementation/checkpoints/regressor.pt
```

### Inference

```bash
# Single checkpoint, dual-GPU inference
python -m tabpfn3_paper.infer \
  --checkpoint implementation/checkpoints/checkpoint.pt \
  --devices cuda:0 cuda:1

# Preprocessing ensemble (parallelizes estimators across GPUs)
python -m tabpfn3_paper.ensemble \
  --checkpoint implementation/checkpoints/checkpoint.pt \
  --n-estimators 12 --max-features 29 \
  --ensemble-weighting validation --devices cuda:0 cuda:1
```

### Evaluation and Verification

```bash
# Synthetic SCM accuracy gate against HistGradientBoosting
python -m tabpfn3_paper.evaluate \
  --checkpoint implementation/checkpoints/checkpoint.pt \
  --max-accuracy-gap 0.05 --devices cuda:0 cuda:1

# Real-data audit on bundled sklearn datasets
python implementation/scripts/benchmark_real_data.py \
  --checkpoint implementation/checkpoints/checkpoint.pt \
  --datasets breast_cancer wine digits iris \
  --n-estimators 12 --max-features 29 --ensemble-weighting validation \
  --max-accuracy-gap 0.05 --devices cuda:0 cuda:1

# Full verification suite
python implementation/scripts/verify_goal.py
python implementation/scripts/verify_goal.py --include-openml  # adds network-dependent OpenML audit
```

### Other scripts

```bash
python implementation/scripts/check_architecture_features.py   # NaN/Inf + many-class verification
python implementation/scripts/check_report_prior.py            # report-style prior verification
python implementation/scripts/check_chunk_equivalence.py       # row-chunking correctness
python implementation/scripts/check_cache_equivalence.py       # fit_cache correctness
python implementation/scripts/check_regressor_dual_gpu_inference.py
python implementation/scripts/smoke_regressor.py
python implementation/scripts/smoke_local.py
python implementation/scripts/accuracy_gate.py                 # official pretrained accuracy gate
```

## Architecture

### Model forward path (`model.py`)

The classifier processes tabular data through this pipeline:

1. **Cell embedding**: Features are grouped into circular triplets (indices `[i, i+1, i+3] % n_features`). Each cell is `[values, missing_indicators]` concatenated, projected through `cell_embed` linear.
2. **Label embedding**: Train-side cell embeddings are summed with orthogonal label embeddings (`cell_label_embed`), and later rows are summed with orthogonal ICL label embeddings (`icl_label_embed`).
3. **Feature distribution embedder** (`FeatureDistributionEmbedder`): Induced-set attention — learned inducing points attend to feature-level rows and vice versa, repeated for `dist_embed_num_blocks` layers. Input shape `(B, G, R, E)` where G = n_features.
4. **Feature aggregation** (`FeatureAggregator`): Per-row self-attention over feature groups with learned CLS tokens + RoPE. Output shape `(B, R, feat_agg_num_cls_tokens * embed_dim)`.
5. **ICL blocks** (`ICLBlock`): Each block applies train self-attention then test-to-train cross-attention (with GQA — `icl_num_kv_heads_test=1` for test queries). Repeated for `nlayers` blocks.
6. **Many-class decoder** (`ManyClassDecoder`): Retrieval-based — attention from test rows to train rows, weighted one-hot aggregation over train labels. Returns log-probabilities.

Key details:
- All attention uses **QASS** (query-aware softmax scaling) — a small MLP predicts per-head scaling from query norm and key length.
- Normalization is **RMSNorm** throughout (no LayerNorm).
- The model handles NaN/Inf natively via the missing indicator in cell embeddings and robust standardization.
- `fit_cache()` freezes train-side states (standardizer, inducing states, ICL train states, decoder embeddings) for fast repeated prediction without re-encoding the train set.
- `predict_proba()` chunks test rows via `inference_row_chunk_size` to manage GPU memory.

### Regressor (`regressor.py`)

Same encoder stack as the classifier, but:
- Label embeddings are replaced with continuous target embeddings (`cell_target_embed`, `icl_target_embed` — linear from scalar).
- Decoder is a 2-layer MLP bucket decoder predicting `num_regression_buckets` logits, converted to an expected value via softmax-weighted bucket centers.
- Target values are normalized per-task with `TargetCache` (mean/std).

### Config profiles (`config.py`)

| Profile | embed_dim | nlayers | max_num_classes | Purpose |
|---------|-----------|---------|-----------------|---------|
| `paper` | 128 | 24 | 160 | Report-level architecture (needs >8T tokens pretraining) |
| `small` | 64 | 6 | 64 | Local experimentation on commodity GPUs |
| `tiny` | 32 | 2 | 16 | Smoke tests |

### Synthetic prior (`prior.py`)

`SyntheticSCMPrior` generates synthetic tabular tasks with two styles:
- **classic**: Random DAG-like structural equations with nonlinear mixing, sine oscillators, and balanced bucketized labels.
- **report**: Adds categorical variables, high-frequency oscillators, grid spatial patterns, temporal structure, OOD query shifts, missing values, Inf values, prototype-class labels, categorical prototypes, and low-dimensional boundary labels.

### Preprocessing ensemble (`preprocessing.py` + `ensemble.py`)

The ensemble applies per-estimator preprocessing configs:
- **Preprocessing kinds**: `standard` (StandardScaler), `robust_clip` (RobustScaler + tanh clipping), `quantile` (QuantileTransformer to normal).
- **Feature selection**: Round-robin feature subsampling below the threshold; Gini importance (ExtraTrees) above `gini_threshold_rows`.
- **SVD augmentation**: Every `add_svd_every`-th estimator appends PCA components.
- **Validation weighting**: Optional — fits on a validation split, weights estimators by accuracy.

### Dual-GPU pattern

Both training and inference use an explicit 2-GPU model:
- **Training**: `torchrun --nproc_per_node=2` with DDP. Each rank generates independent synthetic tasks and synchronizes gradients.
- **Inference**: `ThreadPoolExecutor` with one worker per GPU. The model state dict is copied to CPU, then each GPU receives a replica. Test rows are split via `torch.chunk` across devices.

## Key Conventions

- All training and inference produces JSON-line logging. Parse stdout lines starting with `{`.
- Checkpoints are dicts: `{"model": state_dict, "config": config.to_dict(), "args": args_dict}`.
- The model standardizer uses NaN-aware statistics (finite-mean, finite-std with `clamp_min`).
- `weights_only=True` is used for all `torch.load` calls.

---
> Source: [T-Lab/OpenTFM](https://github.com/T-Lab/OpenTFM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
