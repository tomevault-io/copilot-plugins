## tumorimagingbench

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TumorImagingBench is a comprehensive framework for evaluating and comparing foundation model feature extractors for radiomics in medical imaging. It provides a unified interface to extract features from medical images using 10+ state-of-the-art foundation models and benchmark them across 6 different radiomics datasets.

## Package Management

This project uses **uv** for dependency management:

```bash
# Install dependencies
uv sync

# Python version requirement: >=3.13
# Important: pandas requires >=2.1.0 for Python 3.13 compatibility
```

## Repository Architecture

### Core Components

**1. Models (`src/tumorimagingbench/models/`)**
- All models inherit from `BaseModel` (in `base.py`) which defines three abstract methods:
  - `load(weights_path)`: Load model weights
  - `preprocess(x)`: Preprocess input data (expects dict with `image_path`, `coordX`, `coordY`, `coordZ`)
  - `forward(x)`: Run inference
- Models are imported conditionally in `models/__init__.py` with graceful degradation for missing dependencies
- Use `AVAILABLE_EXTRACTORS` dict and `get_extractor(name)` function to check available models at runtime
- Supported models: FMCIB, CT-FM, CT-CLIP-ViT, PASTA, VISTA3D, Voco, SUPREM, Merlin, MedImageInsight, ModelsGen

**2. Feature Extraction (`src/tumorimagingbench/evaluation/`)**
- `base_feature_extractor.py` provides the core feature extraction pipeline:
  - `extract_features_for_model()`: Extracts features for a single model across train/val/test splits
  - `extract_all_features()`: Runs all available models with multiprocessing
  - `save_features()`: Saves extracted features as pickle files
- Dataset-specific extractors (e.g., `nsclc_radiomics_feature_extractor.py`) define:
  - `get_split_data(split)`: Returns pandas DataFrame for a given split
  - `preprocess_row(row)`: Preprocesses a dataset row
  - `extract_features(output_path)`: Main entry point
- Each extractor expects CSV files with columns: `image_path`, `coordX`, `coordY`, `coordZ` (centroid coordinates)

**3. Modeling & Analysis**
- `notebooks/modelling/modelling_utils.py` provides analysis utilities:
  - `train_knn_classifier()`: KNN classifier with Optuna hyperparameter optimization (k=1-50, cosine metric)
  - `evaluate_model()`: ROC AUC evaluation
  - `extract_model_features()`: Concatenates train/val/test features with model-specific aggregation filters
  - `compute_knn_indices()`: Computes k-NN for stability/robustness analysis
  - `compute_overlap_matrix()`: Computes mutual k-NN overlap between models
  - Visualization functions for model comparison and overlap matrices

### Directory Structure

```
src/tumorimagingbench/
├── models/              # Foundation model extractors (10+ models)
├── evaluation/          # Dataset-specific feature extractors
└── __init__.py

notebooks/
├── modelling/           # Dataset-specific modeling notebooks (6 datasets)
├── analysis/            # Cross-model analysis notebooks
└── legacy/              # Deprecated code

scripts/
├── test_inference.py           # Test model inference on dummy volume
└── generate_saliency_maps.py   # Generate occlusion sensitivity maps

data/eval/               # Dataset directory (not tracked in git)
└── [dataset_name]/      # Each dataset has train/val/test CSV splits
```

## Common Development Tasks

### Testing Model Inference

```bash
# Run inference test on a specific model with dummy data
python scripts/test_inference.py --model FMCIBExtractor
```

### Feature Extraction

To extract features for a dataset, use the dataset-specific extractor:

```python
# From src/tumorimagingbench/evaluation/
python nsclc_radiomics_feature_extractor.py --output features/nsclc_radiomics.pkl
```

Or create a custom extractor by:
1. Inheriting from `base_feature_extractor.py` functions
2. Implementing `get_split_data()` and `preprocess_row()` functions
3. Calling `extract_all_features()` with these functions

### Generating Saliency Maps

```bash
python scripts/generate_saliency_maps.py
```

Uses MONAI's `OcclusionSensitivity` with cosine similarity scoring to analyze model attention patterns.

## Key Implementation Details

### Model Input Format

All models expect a dictionary with:
- `image_path`: Path to NIFTI file (.nii.gz)
- `coordX`, `coordY`, `coordZ`: Physical centroid coordinates

The `preprocess()` method transforms this into a tensor using MONAI transforms (defined in `models/utils.py`).

### Model-Specific Aggregation

Different models output different feature shapes. Use `apply_aggregation_filter()` from `modelling_utils.py`:
- `MedImageInsightExtractor`: Mean over axis 0
- `CTClipVitExtractor`: Mean over axes (1,2,3)
- `PASTAExtractor`: Mean over axes (2,3,4)
- Others: No aggregation

### Multiprocessing in Feature Extraction

`base_feature_extractor.py` uses multiprocessing.Pool to parallelize across models. Each model processes all splits (train/val/test) sequentially before returning results.

### Dataset Conventions

All datasets should have:
- CSV files for train/val/test splits
- Columns: At minimum `image_path`, `coordX`, `coordY`, `coordZ`
- Images in NIFTI format with proper spacing/origin metadata

## Analysis Workflows

The typical workflow is:
1. Extract features using dataset-specific extractors → pickle files
2. Load features in modeling notebooks
3. Train KNN classifiers with Optuna hyperparameter search
4. Evaluate with ROC AUC metrics
5. Analyze stability/robustness using k-NN overlap
6. Visualize using Plotly (bar plots, heatmaps)

Analysis notebooks in `notebooks/analysis/`:
- `stability_analysis.ipynb`: Model stability with perturbations
- `robustness_analysis.ipynb`: Robustness to noise/transformations
- `saliency_analysis.ipynb`: Occlusion sensitivity visualization
- `overall_analysis.ipynb`: Aggregate performance comparison

## Important Notes

- Models may have different dependencies; use `AVAILABLE_EXTRACTORS` to check what's available
- GPU is required for feature extraction (models use `.to("cuda")`)
- Feature extraction outputs are large pickle files (not tracked in git)
- All paths in dataset extractors use absolute paths specific to original developer's system - these need updating for new environments
- The package is designed to be extensible: add new models by inheriting from `BaseModel`, add new datasets by following the extractor pattern

---
> Source: [AIM-Harvard/TumorImagingBench](https://github.com/AIM-Harvard/TumorImagingBench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
