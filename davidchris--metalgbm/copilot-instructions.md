## metalgbm

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MetalGBM is a gradient boosting library designed to run on Apple Silicon GPUs using Metal. The goal is feature parity with XGBoost and LightGBM, since those libraries don't support Apple Silicon GPU acceleration.

Target features:
- Gradient boosted trees with feature parity to XGBoost/LightGBM
- Python scikit-learn interface via PyO3
- Metal GPU acceleration for Apple Silicon

## Build and Test Commands

Build the library:
```bash
cargo build
cargo build --release
```

Run tests:
```bash
cargo test
```

Run a single test:
```bash
cargo test <test_name>
```

Build with Python bindings (optional feature):
```bash
cargo build --features pyo3
```

## Architecture

### Core Components

**Crate Type**: This is both a cdylib (for Python bindings) and rlib (for Rust usage). The PyO3 feature is optional to allow pure Rust development without Python dependencies.

**Histogram (`src/histogram.rs`)**: Core data structure for gradient boosting. Stores:
- `bins`: Feature value bins
- `gradients`: First-order gradients of the loss function
- `hessians`: Second-order derivatives (confusingly commented as "first derivative")

**Test Utilities (`tests/utils.rs`)**: Provides `TestData` struct with synthetic datasets:
- `linear()`: Creates linear relationship data with simulated residuals
- `edge_cases()`: Returns test cases for boundary conditions (all same values, monotonic increase, two clusters)

### Dependencies

- `metal`: Apple's GPU framework for acceleration
- `ndarray`: Multi-dimensional array library (core data structure)
- `pyo3`: Python bindings (optional feature)
- `rayon`: Parallel computation
- `num-traits`: Numeric type abstractions

### GPU Acceleration Strategy

The library uses Metal for GPU operations on Apple Silicon. This is the key differentiator from XGBoost/LightGBM which lack Apple GPU support.

## Development Notes

- Rust edition 2024 is used
- Python bindings target abi3-py310 (stable ABI for Python 3.10+)
- Test data utilities simulate gradient boosting scenarios (residuals from previous trees, various loss function derivatives)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davidchris) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
