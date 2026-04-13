## mnist-ai

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Research project investigating PAC learning theory - generalization bounds, VC dimension, growth functions, and the relationship between hypothesis complexity and learnability. Uses MNIST as a testbed for experiments. Designed for rapid prototyping on Google Colab (A100 recommended).

## Project Structure

```
MNIST_AI/
├── shared/
│   └── utils/          # Reusable utilities (import into experiments)
│       ├── data.py     # Dataset loading, clustering, subsetting
│       ├── models.py   # Model architectures (SmallCNN, TinyCNN, MLP)
│       ├── training.py # Training loops (single and parallel)
│       └── evaluation.py # Accuracy, cross-evaluation, statistics
└── experiments/
    ├── digit_correlation/         # Digit-wise correlation analysis
    │   ├── digit_correlation_experiment.ipynb
    │   ├── dashboard/             # Interactive D3.js visualization
    │   └── results/               # Experiment outputs
    └── latent_clustering/         # Latent space evolution
        ├── latent_space_clustering.ipynb
        └── results/               # Experiment outputs
```

## Colab Workflow

```python
# Clone and import
!git clone https://github.com/Caleb-Briggs/MNIST_AI.git
%cd MNIST_AI

import sys
sys.path.append('/content/MNIST_AI')

from shared.utils.data import load_mnist, create_clusters, get_device
from shared.utils.models import SmallCNN, create_model
from shared.utils.training import train_model, train_parallel
from shared.utils.evaluation import evaluate, cross_evaluate, compute_stats
```

## Key Utilities

### Data (`shared/utils/data.py`)
- `load_mnist(device, train=True)` - Returns (images, labels) tensors on device
- `create_clusters(num_samples, sizes, seed)` - Create disjoint clusters
- `create_digit_subset(images, labels, digits)` - Filter to specific digits

### Models (`shared/utils/models.py`)
- `SmallCNN(num_classes=10)` - ~100k params, 3 conv layers
- `TinyCNN(num_classes=10)` - ~10k params, 2 conv layers
- `MLP(hidden_sizes, num_classes)` - Configurable fully-connected
- `create_model(name, device, **kwargs)` - Factory function

### Training (`shared/utils/training.py`)
- `train_model(model, images, labels, indices, epochs, lr, ...)` - Single model
- `train_parallel(models, images, labels, list_of_indices, ...)` - Multiple models

### Evaluation (`shared/utils/evaluation.py`)
- `evaluate(model, images, labels, indices)` - Single accuracy
- `cross_evaluate(models, images, labels, clusters, train_indices)` - MxM matrix
- `compute_stats(matrix)` - Mean, std, percentiles from accuracy matrix

## Experiments

Each experiment is self-contained in its own folder under `experiments/`:

- **digit_correlation**: Trains 50 models on small balanced samples, analyzes digit-by-digit accuracy correlations, clustering of models, and exports data for interactive dashboard
- **latent_clustering**: Trains single model, captures layer activations during training, visualizes how representations evolve from random to structured digit clusters

## Design Principles

1. **Each experiment is isolated** - Self-contained folder with notebook(s), results/, and optional experiment-specific code (e.g., dashboard)
2. **Shared utilities are minimal** - Only truly reusable code goes in `shared/utils/`
3. **Notebooks are primary** - All experiment logic lives in notebooks
4. **Per-experiment results** - Each experiment has its own `results/` directory to avoid clutter
5. **Random seeds vary** - Use different seeds across experiments (not always 42!) to avoid bias

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Caleb-Briggs)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Caleb-Briggs)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
