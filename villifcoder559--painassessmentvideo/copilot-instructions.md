## painassessmentvideo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Video-based pain/emotion assessment framework using video foundation models (VideoMAEv2, V-JEPA, MAE-DFER) with Optuna hyperparameter optimization. The pipeline extracts features from video backbones, passes them through attention-based prediction heads, and trains with various loss functions for facial expression recognition on datasets like BioVID, UNBC, CAER, and AgeDB.

## Important: External Codebases

The directories `jepa/`, `MAE_DFER/`, and `VideoMAEv2/` are external codebases used as dependencies. **Always ask before modifying files in these directories.** Custom pipeline code lives in `custom/`.

## Running

### Training
```bash
python3 train_model.py --head ATTENTIVE_JEPA --mt G --lr 0.0001 --ep 7 \
  --csv partA/starting_point/samples.csv \
  --ffsp partA/video/features/VideoMaeV2_S/whole_full_features_BIOVID_S_last143 \
  --global_folder_name history_run_debug \
  --path_video_dataset partA/video/video \
  --k_fold 5 --opt adamw --batch_train 5 --loss sim_loss \
  --n_trials 1 --optuna_sampler grid
```

Key arguments:
- `--mt`: Model type (S/B/G for VideoMAEv2 sizes, or VJEPA2, DFER)
- `--head`: Head type (ATTENTIVE_JEPA, GRU, POOL_MLP, LINEAR)
- `--ffsp`: Path to pre-extracted features (safetensors format)
- `--csv`: Labels CSV file
- `--loss`: Loss function (l1, l2, ce, cdw_ce, huber_ce, sim_loss, contrastive, rnc, coral)
- `--composite_loss` + `--composite_loss_lambda`: Combine multiple losses with weights
- `--embedding_reduction`: spatial, temporal, all, adaptive_pooling_3d, or none
- `--n_trials` / `--optuna_sampler`: Optuna HPO config (tpe, random, auto, grid)

### Feature Extraction
```bash
python3 extract_feature.py --model_type S --saving_after 8700 --emb_red none \
  --path_dataset partA/video/video_frontalized_all_no_interp \
  --path_labels partA/starting_point/samples.csv \
  --saving_folder_path partA/video/features/output_folder
```

### Tests
```bash
python3 -m pytest tests/                    # all tests
python3 -m pytest tests/test_rnc_v2.py      # single test
python3 -m unittest tests.test_rnc_v2       # alternative
```

Tests use unittest and are in `tests/`. They test loss functions and utilities (DisentangledLoss, PHuberCE, RnCLossV2, etc.).

## Architecture

### Data Flow
```
CSV labels + Video files → Feature Extraction (backbone) → .safetensors cache
→ customDatasetWhole (or customDataset) → DataLoader
→ Model_Advanced (backbone features → head → predictions)
→ Loss computation → Optuna objective
```

Features are pre-extracted and cached as safetensors files. Training loads from cache rather than running the backbone each time.

### Key Modules (custom/)

- **model.py** — `Model_Advanced`: orchestrates backbone + head, training loop, evaluation
- **backbone.py** — `VideoBackbone`: loads VideoMAEv2/VJEPA2/DFER, extracts features, supports adapters and freezing
- **head.py** — Prediction heads: `AttentiveHeadJEPA` (cross-attention with learnable queries), `GRUHead`, `PooledHeadMLP`, `LinearHead`. Also contains early stopping classes
- **dataset.py** — `customDataset`: handles frame sampling strategies (uniform, sliding_window, central, random), augmentations, CSV label management. `balancedBatchSampler` for stratified sampling
- **loss.py** — CDW_CE, PHuberCrossEntropy, SimLoss, RESupConLoss, RnCLossV2, DisentangledLoss
- **composite_loss.py** — `CompositeLoss` combines multiple losses; `CCCLoss`, `SupConWeightedLoss`
- **helper.py** — Enums for MODEL_TYPE, HEAD, SAMPLE_FRAME_STRATEGY, CUSTOM_DATASET_TYPE, EMBEDDING_REDUCTION. Global `dict_data` for in-memory feature cache
- **tools.py** — Feature I/O (safetensors), attention visualization, confusion matrices, results logging
- **optimizers.py** — `OptimizerFactory`: Adam/AdamW/SGD with warmup, cosine, step decay, OneCycle schedulers
- **adapters.py** — Temporal depth-wise convolution adapters for backbone fine-tuning

### train_model.py Structure

The main script (~1965 lines) uses Optuna for HPO:
1. `_suggest()` — proposes hyperparameters from search space
2. `objective()` — main training function called by Optuna
3. Inside objective: model init → dataset loading → training loop with early stopping → evaluation
4. Factory functions: `get_optimizer()`, `get_loss()`, `get_composite_loss_module()`, `get_sampling_frame_strategy()`

### Augmentation System

Augmentations defined in `helper.py`: hflip, jitter, rotation, latent_basic, latent_masking, shift, zoom, and combinations (hflip+shift, rotation+jitter, etc.). Enabled via CLI flags like `--hflip 1`, `--rotation 1`.

## Environment

- Features stored in safetensors format
- PyTorch + CUDA required

## Code Style

- **Indentation: 2 spaces** (not 4, not tabs)
- Follow existing patterns in `custom/` for new modules
- Type hints are encouraged but not enforced
- **Every function must begin with a docstring** describing its purpose, arguments (with shapes/types), and return value. Use the following format:
```python
  def forward(self, x: torch.Tensor, mask: Optional[torch.Tensor] = None) -> torch.Tensor:
    """
    Forward pass through the attentive head.

    Args:
      x:    Input feature tensor. Shape: (B, T, D) where B=batch, T=frames, D=embed dim.
      mask: Optional attention mask. Shape: (B, T). True = ignore token.

    Returns:
      Prediction logits. Shape: (B, num_classes).
    """
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/villifCoder559) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
