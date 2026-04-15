## rust-recognition-text

> This project is a Neural Network implementation in Rust, designed for text recognition tasks. It utilizes a workspace structure separating the core training logic, a web server for the UI, and a planned WebAssembly module.

# text.recognition.localhost

## Project Overview

This project is a Neural Network implementation in Rust, designed for text recognition tasks. It utilizes a workspace structure separating the core training logic, a web server for the UI, and a planned WebAssembly module.

The core machine learning logic is built using the `candle` framework (by Hugging Face).

## Architecture

The project is organized as a Rust Workspace with the following members:

*   **`packages/core`**: The main binary for training the neural network.
    *   **ML Framework**: Uses `candle-core` and `candle-nn`.
    *   **Data Loading**: Reads CSV data from `datasets/`.
    *   **Persistence**: Saves/loads model weights to/from JSON files in `datasets/`.
*   **`packages/webserver`**: A web application serving the frontend.
    *   **Framework**: Rocket.
    *   **Deployment**: Dockerized with `cargo watch` for hot reloading during development.
*   **`packages/wasm`**: (Planned) WebAssembly module for client-side inference.

## Prerequisites

*   **Rust**: Stable toolchain.
*   **Docker** & **Docker Compose**: For running the webserver service.

## Setup & Usage

### 1. Data Preparation

Before training, you must ensure the dataset is available. The project expects CSV files in the `datasets/` directory.

*   Extract the provided archives:
    ```bash
    cd datasets
    tar -xzf train.tar.gz
    tar -xzf test.tar.gz
    # Ensure train.csv and test.csv are now present in datasets/
    ```

### 2. Training the Model (`core`)

To train the neural network, run the `core` package.

```bash
cargo run -p core
```

**What happens:**
1.  Loads data from `datasets/train.csv`.
2.  Initializes or loads an existing model from `datasets/model.json`.
3.  Trains the network for a configured number of iterations.
4.  Saves training metrics (loss/accuracy plots) to `datasets/metrics/`.
5.  Saves the trained model to `datasets/model.json` and a timestamped backup.

### 3. Running the Webserver

You can run the webserver using Docker Compose or directly with Cargo.

**Using Docker (Recommended for dev):**
```bash
docker compose up
```
*   Access the app at: `http://localhost:57489` (mapped to container port 8080).
*   Uses `cargo watch` for auto-reloading on code changes.

**Using Cargo:**
```bash
cargo run -p webserver
```

## Key Configuration & Files

*   **`packages/core/src/main.rs`**: Entry point for the training process. Configures learning rate (`LEARNING_RATE`), iterations (`NB_ITER`), and layers.
*   **`packages/core/src/internal/dataset.rs`**: Handles CSV parsing and data normalization (0-255 to 0-1).
*   **`packages/core/src/internal/memory.rs`**: Manages model persistence (JSON format).
*   **`datasets/`**: Stores data, trained models (`model.json`), and metrics.

## Development Notes

*   **One-Hot Encoding**: The current implementation supports up to 100 classes (hardcoded `max = 100` in `dataset.rs`).
*   **Model Format**: Custom JSON structure storing layer weights, biases, and activation functions.
*   **Logging**: Custom logger implementation in `packages/core/src/logger.rs`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PitouResk)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/PitouResk)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
