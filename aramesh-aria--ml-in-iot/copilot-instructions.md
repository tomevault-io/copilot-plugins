## ml-in-iot

> This project implements an end-to-end pipeline for energy optimization in LoRaWAN networks using Machine Learning. The core idea is to predict the Signal-to-Noise Ratio (SNR) using various ML models, and then apply a Transmission Power Control (TPC) logic to dynamically adjust the Spreading Factor (SF) and Transmit Power (TP) of LoRaWAN devices. This adjustment aims to minimize energy consumption while maintaining a sufficient link margin. The project uses a proxy metric for energy consumption to evaluate the effectiveness of the TPC decisions.

# Project: LoRaWAN Energy Optimization with Machine Learning (ML_TPC_Lorawan)

## Project Overview
This project implements an end-to-end pipeline for energy optimization in LoRaWAN networks using Machine Learning. The core idea is to predict the Signal-to-Noise Ratio (SNR) using various ML models, and then apply a Transmission Power Control (TPC) logic to dynamically adjust the Spreading Factor (SF) and Transmit Power (TP) of LoRaWAN devices. This adjustment aims to minimize energy consumption while maintaining a sufficient link margin. The project uses a proxy metric for energy consumption to evaluate the effectiveness of the TPC decisions.

## Main Technologies
*   **Programming Language:** Python
*   **Data Manipulation:** Pandas, NumPy
*   **Machine Learning:** scikit-learn
*   **Model Persistence:** Joblib
*   **Plotting:** Matplotlib

## Architecture
The project follows a modular structure with clear separation of concerns:
*   **`data/`**: Stores raw (`subsampled_data.csv`) datasets.

*   **`models_trained/`**: Stores models trained by `train_baselines.py` (e.g., `ridge.joblib`, `rf.joblib`, `svr.joblib`).
*   **`outputs/`**: Stores predictions, TPC decisions, and generated figures.
    *   `outputs/predictions/`: Numeric outputs like `model_metrics.csv`, `snr_predictions.csv`, `tpc_decisions.csv`.
    *   `outputs/figures/`: Visual outputs like `snr_true_vs_pred.png`, `sf_distribution.png`, etc.
*   **`src/`**: Contains the core logic and scripts:
    *   **`config.py`**: Centralized configuration for all project parameters, including file paths, experiment settings (`RANDOM_STATE`, `TEST_SIZE`), target/feature selection (`TARGET_COL`, `DROP_COLS`), selected ML model (`SELECTED_TRAINED_MODEL`), and LoRaWAN/TPC specific parameters (`SF_MIN`, `TP_MIN`, `LINK_MARGIN_DB`, `SNR_LIMIT_BY_SF`, `BASELINE_SF`, `BASELINE_TP`).
    *   **`io_utils.py`**: Utility functions for common I/O operations and data preprocessing, such as ensuring directory existence, loading datasets, detecting target columns, splitting data into X/y, loading models, aligning features, converting features to numeric, and saving CSVs.
    *   **`tpc.py`**: Implements the core Transmission Power Control logic, which decides the optimal SF and TP based on predicted SNR and link margin.
    *   **`energy.py`**: Defines a simplified proxy metric for relative energy consumption (`relative_energy`) and a normalized version (`normalized_energy`) for comparative evaluation.
    *   **`train_baselines.py`**: Script responsible for training and evaluating several baseline Machine Learning models (Ridge, RandomForest, SVR) for SNR prediction. It saves the trained models and their performance metrics (RMSE, R²).
    *   **`run_pipeline.py`**: Orchestrates the entire end-to-end process. It loads the dataset, loads a selected trained ML model, predicts SNR, applies the TPC decision logic, calculates normalized energy, saves the predictions and TPC decisions to CSV files, and generates various plots to visualize the results.
    *   **`summarize_results.py`**: Script for reading the TPC decisions and calculating key performance indicators (KPIs) such as average energy reduction, SF/TP distribution, and link margin statistics.

## Building and Running

### 1. Install Dependencies
The project dependencies are listed in `requirements.txt`.
```bash
pip install -r requirements.txt
```

### 2. (Optional) Sanity Check Dataset
Run this script to perform a quick check on the health of the dataset (e.g., shape, columns, NaN values).
```bash
python -m src.sanity_check
```

### 3. Train Models and Select Best Model
This script trains baseline ML models (Ridge, RandomForest, SVR) for SNR prediction, evaluates their performance, and saves the trained models as `.joblib` files in the `models_trained/` directory. The evaluation metrics (RMSE, R²) are saved to `outputs/predictions/model_metrics.csv`.
The `SELECTED_TRAINED_MODEL` in `src/config.py` should be updated to reflect the best performing model.
```bash
python -m src.train_baselines
```

### 4. Run Main Pipeline
This is the core script that executes the complete end-to-end pipeline:
1.  Loads the dataset.
2.  Loads the ML model specified in `config.SELECTED_TRAINED_MODEL`.
3.  Predicts SNR for all data samples.
4.  Applies the TPC logic to decide new SF and TP values.
5.  Calculates the normalized energy proxy.
6.  Saves SNR predictions (`outputs/predictions/snr_predictions.csv`) and TPC decisions (`outputs/predictions/tpc_decisions.csv`).
7.  Generates several visualization plots (`.png` files) in `outputs/figures/`.
```bash
python -m src.run_pipeline
```

### 5. Summarize KPIs
After running the main pipeline, this script reads the TPC decisions and calculates key performance indicators (KPIs) such as mean/median energy reduction, percentage of energy below certain thresholds, SF/TP distribution, and margin statistics (`Me`). The results are printed to the console.
```bash
python -m src.summarize_results
```

## Development Conventions

*   **Centralized Configuration:** All critical parameters, paths, and settings are defined in `src/config.py` for easy modification and maintenance.
*   **Modularity:** The codebase is organized into distinct Python modules (`io_utils.py`, `tpc.py`, `energy.py`) to promote code reusability and maintainability.
*   **Reproducibility:** The `RANDOM_STATE` variable in `src/config.py` is used to ensure that experiments (e.g., train-test splits, model training) are reproducible.
*   **Robust Data Handling:** Utility functions in `src/io_utils.py` are designed to safely load, preprocess, and align data for machine learning models, handling potential issues like missing columns or non-numeric data.
*   **Energy Proxy:** The energy consumption metric is a simplified proxy, intended for comparative analysis and presentation, rather than a precise physical simulation of hardware power draw.
*   **Code Documentation:** Python files and key functions include docstrings to explain their purpose, inputs, and outputs, facilitating understanding and collaboration.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Aramesh-Aria)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Aramesh-Aria)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
