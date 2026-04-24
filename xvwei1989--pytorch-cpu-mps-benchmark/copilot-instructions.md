## pytorch-cpu-mps-benchmark

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains PyTorch deep learning training code designed to benchmark and compare performance between CPU and MPS (Apple Metal Performance Shaders) devices on Apple Silicon Macs.

## Key Files

- `train_benchmark.py`: Main training script with CPU vs MPS performance comparison
- `requirements.txt`: Python dependencies including PyTorch and related libraries

## Development Commands

### Setup Environment
```bash
# Install Python dependencies
pip install -r requirements.txt

# Verify PyTorch installation and MPS availability
python -c "import torch; print(f'PyTorch: {torch.__version__}'); print(f'MPS Available: {torch.backends.mps.is_available()}')"
```

### Run Training Benchmarks

```bash
# Run full CPU vs MPS comparison (5 epochs, batch size 32)
python train_benchmark.py

# Custom epochs and batch size
python train_benchmark.py --epochs 10 --batch_size 64

# Test only CPU
python train_benchmark.py --cpu_only

# Test only MPS (Apple Silicon required)
python train_benchmark.py --mps_only
```

### Available Command Line Options
- `--epochs`: Number of training epochs (default: 5)
- `--batch_size`: Batch size for training (default: 32)
- `--cpu_only`: Test CPU performance only
- `--mps_only`: Test MPS performance only

## Architecture Overview

### Core Components

1. **SimpleCNN**: Basic convolutional neural network model for benchmarking
   - 3 conv layers with max pooling
   - 2 fully connected layers with dropout
   - Designed for 64x64 RGB images

2. **SyntheticDataset**: Generates synthetic image data for testing
   - Creates random RGB images (64x64x3)
   - Generates random labels for classification
   - Configurable dataset size and number of classes

3. **TrainingBenchmark**: Main training class
   - Handles device-specific model training
   - Tracks training time, loss, and accuracy
   - Supports both CPU and MPS devices

4. **Performance Comparison**: Built-in benchmarking system
   - Compares training times between devices
   - Calculates speedup ratios and time savings
   - Provides detailed performance metrics

### Device Support

- **CPU**: Universal support across all systems
- **MPS**: Apple Silicon Macs (M1, M2, M3 chips) with PyTorch >= 2.0

### Performance Considerations

- Uses synthetic data to eliminate I/O bottlenecks
- Sets random seeds for reproducible results
- Measures wall-clock time for accurate performance comparison
- Provides real-time training progress updates

## System Requirements

- Python 3.8+
- PyTorch 2.0+ (for MPS support)
- Apple Silicon Mac for MPS testing
- Standard requirements for CPU testing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/xvwei1989) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
