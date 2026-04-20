## mops-pred

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Setup

Uses [uv](https://docs.astral.sh/uv/) for environment management:
```bash
uv sync
```

Or with pip:
```bash
pip install -e .
```

## Running Experiments

### Config-based training (DeepLabV3 / SegFormer models)
```bash
python -m mops_pred.training --config configs/deeplabv3_affordance_kitchen.py --mode train
python -m mops_pred.training --config configs/deeplabv3_affordance_kitchen.py --mode debug  # fast_dev_run, no wandb upload
python -m mops_pred.training --config configs/deeplabv3_affordance_kitchen.py --mode train --checkpoint path/to/ckpt.pt
```

### DINOv2 fine-tuning (standalone script)
```bash
python -m mops_pred.train_and_test_dinov2_segmentation
```

### DINOv2 / DINOv3 zero-shot / linear probing
```bash
python -m mops_pred.test_dinov2_segmentation
```

### Linting
```bash
ruff check .
ruff format .
```

Pre-commit hooks run `black`, `isort`, and whitespace fixers — install with `pre-commit install`.

## Architecture

### Registration pattern
Models, backbones, and datasets all use decorator-based registries:
- `@register_model(name="...")` in `mops_pred/models/model_factory.py`
- `@register_backbone(name="...")` in `mops_pred/models/backbones/backbone_factory.py`
- `@register_dataset(name="...")` in `mops_pred/datasets/dataset_factory.py`

New components must be registered before `create_model` / `create_dataloader` can instantiate them by name.

### Config system
Configs use `ml_collections.ConfigDict` (Google's config library). Each config file under `configs/` exports `get_config()`. The main training entrypoint (`mops_pred/training.py`) reads the config via `--config` flag using `ml_collections.config_flags`. Config hierarchy: `default_config.py` → experiment-specific config (e.g., `deeplabv3_affordance_kitchen.py`) → dataset config (under `configs/datasets/`).

### Models
All models are `lightning.LightningModule` subclasses. Two styles:
1. **Config-driven** (`SegmentationModel` via `segmentation.py`): instantiated through `model_factory.create_model(cfg["model"])`, backbone is injected via `backbone_factory`.
2. **Standalone** (`DINOv2SegmentationModel`, `DINOv3SegmentationModel`): directly instantiated in their respective scripts, backbone loaded from `torch.hub` or HuggingFace.

Segmentation tasks are either `semantic` (single-label, CrossEntropy + MulticlassJaccardIndex) or `affordance` (multi-label, BCE/FocalLoss + MultilabelJaccardIndex). The `task` field in the config selects which mask key to read from the batch dict.

### Datasets
All datasets extend `BaseH5Dataset` (`mops_pred/datasets/base_h5.py`), which handles lazy HDF5 file opening and train/test splits via a `metadata/splits` or `labels/splits` key in the HDF5 file.

- `ClutterDataset` (name: `"clutter"`): used for both kitchen and general clutter scenes; accepts a `labels` list (e.g., `["affordance"]`, `["semantic"]`) to select which mask types to load.
- `ObjectCentricDataset`: for object-centric tasks.

Data is expected at `data/mops_data/` (HDF5 files, not in the repo).

### Logging
Config-based training logs to W&B (`WandbLogger`); checkpoints are saved to the W&B run directory. Standalone DINO scripts use Lightning's default TensorBoard logger and save checkpoints to `checkpoints/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LiXiling) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
