## image2biomass

> This repository is for the **CSIRO - Image2Biomass Prediction** Kaggle competition, a collaboration between CSIRO, Meat & Livestock Australia (MLA), and Google Australia.

# Image2Biomass - CSIRO Kaggle Competition

## Competition Overview

This repository is for the **CSIRO - Image2Biomass Prediction** Kaggle competition, a collaboration between CSIRO, Meat & Livestock Australia (MLA), and Google Australia.

**Objective:** Build a model that estimates pasture biomass from top-view images of Australian pastures.

**Prize Pool:** $75,000 USD

**Timeline:**
- Start: October 28, 2025
- Entry Deadline: January 21, 2026

**Evaluation Metric:** Globally weighted coefficient of determination (R²)

Competition URL: https://www.kaggle.com/competitions/csiro-biomass

## Prediction Task

Predict 5 biomass measurements (in grams) from pasture images:
1. `Dry_Green_g` - Dry green matter
2. `Dry_Dead_g` - Dry dead matter
3. `Dry_Clover_g` - Dry clover matter
4. `GDM_g` - Green dry matter
5. `Dry_Total_g` - Total dry matter (highest weight in loss: 0.5)

**Loss Weights:** `[0.1, 0.1, 0.1, 0.2, 0.5]` - Dry_Total_g is weighted highest

## Project Structure

```
image2biomass/
├── src/
│   ├── train.py           # Main training script with W&B integration
│   ├── models.py          # UnifiedModel class for flexible backbones
│   ├── utils.py           # Model configs and helper functions
│   ├── image_processing.py # BiomassDataset and transforms
│   ├── data_preprocessing.py # Data wrangling utilities
│   ├── submission.py      # Inference script for Kaggle submission
│   └── eda.ipynb          # Exploratory data analysis
├── data/
│   ├── train/             # Training images (357 images, 2000x1000px)
│   ├── test/              # Test images for submission
│   ├── train.csv          # Raw training data (long format)
│   ├── y_train.csv        # Processed targets (wide format)
│   └── sample_submission.csv
├── models/                # Saved model checkpoints (.pt files)
├── scripts/               # Cloud setup scripts
├── results/               # EDA visualizations
├── config.yaml            # Default training configuration
├── sweep.yaml             # W&B hyperparameter sweep config
└── pyproject.toml         # Dependencies (uv/pip)
```

## Quick Start

### Environment Setup

```bash
# Using uv (recommended)
uv sync

# Or pip
pip install -e .
```

**Dependencies:** PyTorch 2.9+, timm 1.0+, wandb, pandas, matplotlib

### Training

```bash
# Standard training with config.yaml
python src/train.py

# Hyperparameter sweep with W&B
python src/train.py --sweep
```

### Inference

The `src/submission.py` script loads a traced model and generates predictions. Update `MODEL_PATH` to point to your trained model.

## Model Architecture

Uses `UnifiedModel` from `src/models.py` - a flexible wrapper around any timm backbone:

```
Input Image (2000x1000)
    -> Crop center (500-1500 px)
    -> Resize to model-specific size (224-518px)
    -> timm backbone (feature extraction)
    -> Regression head (Linear->ReLU->Dropout->Linear->ReLU)
    -> 5 biomass predictions
```

### Supported Backbones (18 total)

Defined in `src/utils.py` MODEL_CONFIGS:
- **ViT:** vit_tiny_patch16_224, vit_small_patch16_224, vit_small_patch32_224
- **DINOv2:** vit_small_patch14_dinov2.lvd142m (518x518 input)
- **ConvNeXt:** convnext_tiny, convnext_nano, convnext_pico
- **Swin:** swin_tiny_patch4_window7_224, swin_s3_tiny_224
- **ResNet:** resnet18, resnet34, resnet50
- **EfficientNet:** efficientnet_b0, b1, b2
- **MobileNet:** mobilenetv3_large_100
- **RegNet:** regnetx_016, regnetx_032

## Configuration

### config.yaml (default training)

```yaml
lr: 0.0001
n_epochs: 1
train_split: 0.8
batch_size: 4
bf16: true           # Mixed precision (CUDA/MPS only)
patience: 5          # Early stopping patience
model_class: UnifiedModel
model_name: vit_tiny_patch16_224
```

### sweep.yaml (hyperparameter search)

Bayesian optimization over model architectures with Hyperband early termination.

## Data Format

**Training data:** 357 unique images with 5 targets each
- Images: 2000x1000 JPG (70cm x 30cm pasture quadrat)
- Metadata: State (NSW/Tas/Vic/WA), Species, Height_Ave_cm, Pre_GSHH_NDVI, Sampling_Date

**Image preprocessing:** Center crop (500-1500px) -> Resize -> Normalize
- ViT models: mean=[0.5, 0.5, 0.5], std=[0.5, 0.5, 0.5]
- ConvNets/DINOv2: ImageNet normalization

## Experiment Tracking

Uses Weights & Biases (wandb):
- Project: `image2biomass`
- Logs: train_loss, val_loss, val_R2 per epoch
- Artifacts: dataset directory, trained models (TorchScript traced)

## Key Files Reference

| File | Purpose |
|------|---------|
| `src/train.py:31` | Main `train()` function |
| `src/models.py:6` | `UnifiedModel` class |
| `src/utils.py:9` | `MODEL_CONFIGS` dict |
| `src/utils.py:55` | `get_model()` factory |
| `src/image_processing.py:102` | `BiomassDataset` class |

## Development Notes

- Models are saved as TorchScript (.pt) for Python 3.14 compatibility
- Train/val split: 80/20 with seed 42
- Early stopping monitors val_loss with configurable patience
- R² computed with loss weights for consistency with competition metric

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/matthieuneau) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
