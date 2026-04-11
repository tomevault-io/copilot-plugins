## ydcc

> This project is a comprehensive system for monitoring and forecasting water quality parameters (specifically Salinity) using Machine Learning. It consists of a data processing and training pipeline built with Python and a frontend/bot interface built with Next.js (TypeScript).

# Water Quality Monitoring & Forecasting System

## Project Overview

This project is a comprehensive system for monitoring and forecasting water quality parameters (specifically Salinity) using Machine Learning. It consists of a data processing and training pipeline built with Python and a frontend/bot interface built with Next.js (TypeScript).

The system predicts salinity levels for future horizons (3 hours, 6 hours, and 12 hours) using historical data and lag features.

## Directory Structure

*   **`data/`**: Stores raw and processed CSV data.
    *   `brisbane_water_quality.csv`: Raw input data.
    *   `processed_data.csv`: Cleaned data with lag features and targets.
*   **`model_training/`**: Contains trained models and evaluation results.
    *   Subdirectories `3h/`, `6h/`, `12h/` hold model artifacts (`best_model.pkl` or `.pth`, `features.json`, `scaler.pkl`) for each forecast horizon.
*   **`nlp/`**: A Next.js application that likely serves as a Telegram bot interface and API for accessing predictions.
    *   `scripts/predict.py`: A standalone script used by the web app to load models and generate predictions.
*   **`preprocessing.py`**: Script for cleaning data, handling missing values, and generating features.
*   **`train_models.py`**: Script for training Random Forest, XGBoost, and LSTM models, evaluating them, and saving the best performers.
*   **`water-quality-monitoring-ml-model.ipynb`**: Jupyter notebook for exploratory data analysis and initial model prototyping.

## Machine Learning Pipeline

### 1. Data Preprocessing (`preprocessing.py`)
*   **Input:** `data/brisbane_water_quality.csv`
*   **Process:**
    *   Parses timestamps and sorts data.
    *   Interpolates missing values and handles outliers (Salinity 0-45).
    *   Generates **Lag Features** (1, 2, 6, 12, 24, 48 steps).
    *   Creates **Target Variables** for 3h, 6h, and 12h future horizons.
*   **Output:** `data/processed_data.csv`

### 2. Model Training (`train_models.py`)
*   **Models:** Random Forest, XGBoost, LSTM (Long Short-Term Memory).
*   **Evaluation:**
    *   Tree models: 10-Fold Cross-Validation.
    *   LSTM: Train/Validation split.
    *   Metric: RMSE (Root Mean Squared Error).
*   **Artifacts:** The script automatically selects the best model for each horizon and saves it to `model_training/{horizon}/`.

## Web Application & Bot (`nlp/`)

The `nlp` directory contains a Next.js application tailored for interaction, possibly via a Telegram bot (`telegraf` dependency).

*   **Tech Stack:** Next.js 14+, React, TypeScript, Tailwind CSS.
*   **Prediction Integration:** The app uses `scripts/predict.py` to invoke the trained Python models and fetch the latest forecast.

### ⚠️ Local Development Note
The Next.js application (`nlp/`) is heavily optimized for a **Dockerized environment**:
1.  **Paths:** `nlp/lib/forecast.ts` hardcodes the Python interpreter path to `/venv/bin/python`, which exists only in the Docker image.
2.  **Directory Structure:** `nlp/scripts/predict.py` expects `data` and `model_training` directories to be siblings of the `scripts` folder (i.e., inside `nlp/` or `/app/` in Docker), whereas locally they are in the project root (`ydcc/`).

**To run the web app locally:**
*   You must modify `nlp/lib/forecast.ts` to point to your local Python interpreter.
*   You may need to adjust `nlp/scripts/predict.py` or symlink the `data` and `model_training` directories into `nlp/`.

## Getting Started

### Prerequisites

**Python Environment:**
Ensure you have Python 3.8+ and the following libraries installed (or use the existing `venv`):
*   `pandas`, `numpy`, `matplotlib`, `seaborn`
*   `scikit-learn`, `xgboost`, `torch`

**Node.js Environment:**
*   Node.js (LTS recommended)
*   npm or yarn

### Usage

1.  **Prepare Data:**
    ```bash
    python preprocessing.py
    ```

2.  **Train Models:**
    ```bash
    python train_models.py
    ```
    *This will populate `model_training/` with the best models for 3h, 6h, and 12h horizons.*

3.  **Run Web App (Docker recommended):**
    The `nlp/Dockerfile.build` handles the complex path and environment setup.
    ```bash
    cd nlp
    docker build -f Dockerfile.build -t water-quality-app .
    docker run -p 3000:3000 water-quality-app
    ```

4.  **Run Web App (Local - requires code changes):**
    ```bash
    cd nlp
    npm install
    npm run dev
    ```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ShayNeeo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ShayNeeo)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
