## mnist-push

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a PyTorch-based MNIST digit classification project consisting of educational exercises in deep learning and statistical computing. The project includes a complete training pipeline, model definition, and inference application.

## Core Architecture

The project follows a three-file pattern for machine learning workflow:

1. **model.py** - Contains the `MNISTNet` class definition (PyTorch nn.Module)
   - 3-layer fully connected network: 784 -> 16 -> 16 -> 10
   - Uses ReLU activations between layers
   - This is the single source of truth for model architecture

2. **main.py** - Training script
   - Imports `MNISTNet` from model.py
   - Trains on MNIST dataset (55k train, 5k validation)
   - Uses SGD optimizer with lr=1e-2 and CrossEntropyLoss
   - Saves trained weights to `my_mnist_brain.pt`
   - Includes SSL verification bypass for dataset downloads

3. **app.py** - Inference script
   - Loads the saved model weights from `my_mnist_brain.pt`
   - Runs predictions on test data
   - Uses matplotlib for visualization

**CRITICAL**: Both main.py and app.py MUST import `MNISTNet` from model.py to ensure architecture consistency. Never define the model architecture inline with nn.Sequential in these files.

## Environment Setup

```bash
# Create and activate virtual environment
python -m venv .venv
source .venv/bin/activate  # On macOS/Linux

# Install dependencies
pip install -r requirements.txt
```

Dependencies: torch, torchvision, matplotlib

## Running the Code

```bash
# Train the model (creates my_mnist_brain.pt)
python main.py

# Run inference on saved model
python app.py

# Verify model architecture
python model.py
```

## Additional Files

- **homework_1.py** - Statistical computing exercises implementing mean, median, mode, range, IQR, and standard deviation both manually and with numpy/pandas. Also includes MNIST training code.
- **math_ops.py** - Contains staged changes (not yet committed)
- **script.py** - Template PyCharm script (not actively used)
- **data/** - MNIST dataset storage (auto-downloaded)

## Development Notes

- The project uses Python 3.14 with PyTorch
- Model weights are saved as state_dict in .pt format
- Input images must be flattened from (28, 28) to (784,) before inference
- SSL verification is disabled globally for MNIST downloads

---
> Source: [philipbergman6-glitch/MNIST-Push](https://github.com/philipbergman6-glitch/MNIST-Push) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
