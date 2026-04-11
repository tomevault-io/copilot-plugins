## ulr-experiment

> ULR2SS is a PyTorch-based project designed to perform semantic segmentation on ultra-low-resolution images. It achieves this by jointly training a Super-Resolution (SR) model (based on ESRGAN) and a Semantic Segmentation model (DeepLab). The goal is to recover high-fidelity details from low-resolution inputs to improve segmentation accuracy.

# ULR2SS (Ultra-Low Resolution to Semantic Segmentation)

## Project Overview
ULR2SS is a PyTorch-based project designed to perform semantic segmentation on ultra-low-resolution images. It achieves this by jointly training a Super-Resolution (SR) model (based on ESRGAN) and a Semantic Segmentation model (DeepLab). The goal is to recover high-fidelity details from low-resolution inputs to improve segmentation accuracy.

## Configuration
Configuration is centrally managed in `config.py` using `SimpleNamespace`. You should modify this file to change hyperparameters, file paths, and model settings.

**Key Config Sections:**
*   `training_config`: Hyperparameters for the joint training loop (epochs, learning rates, loss weights).
*   `pretraining_config`: Settings for the SR pre-training phase.
*   `format_config`: Defines image resolutions (Ultra-Low, Low, High).
*   `evaluation_config`: Paths for testing and evaluation.

## Key Files & Structure

*   **`config.py`**: Global configuration file.
*   **`inference.py`**: Script for running inference on images.
*   **`training.py`**: Main script for joint training of Generator, Discriminator, and Segmentor.
*   **`pretraining.py`**: Script for pre-training the SR model.
*   **`esrgan.py`**: Definition of the ESRGAN Generator and Discriminator models.
*   **`modeling/`**: Contains the segmentation model definitions (DeepLab, Backbones).
*   **`utils2/`**: Utility functions for loss, metrics, and saving checkpoints.
*   **`datasets/`**: Directory for storing training and validation datasets.
*   **`checkpoints/`**: Directory where model weights are saved.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IgnasJo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/IgnasJo)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
