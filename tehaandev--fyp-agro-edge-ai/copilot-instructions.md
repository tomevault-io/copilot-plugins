## fyp-agro-edge-ai

> This document provides essential context and instructions for the **Agro Edge AI** project, an edge-based machine learning solution for real-time decision support in small-scale farms, specifically targeting the Sri Lankan agricultural context.

# GEMINI.md - Agro Edge AI Project Context

This document provides essential context and instructions for the **Agro Edge AI** project, an edge-based machine learning solution for real-time decision support in small-scale farms, specifically targeting the Sri Lankan agricultural context.

## 🌟 Project Overview

- **Goal**: Develop lightweight, offline ML models optimized for Raspberry Pi 4B to assist smallholder farmers (<2 hectares) with real-time decision-making.
- **Problem**: 40% of Sri Lanka's agricultural output is lost due to inefficiencies. Existing solutions are often cloud-dependent, expensive, or require high technical literacy.
- **Solution**: A modular edge system that integrates disease detection, irrigation control, and crop suitability recommendation without requiring internet connectivity.

### Key Performance Targets
- **Inference Time**: < 1 second per model (Total system < 200ms achieved).
- **Power Usage**: ≤ 5W (Raspberry Pi 4B target).
- **Model Size**: < 50MB per model.
- **Accuracy**: > 95% across all pipelines.

---

## 🛠 Tech Stack

- **Core**: Python 3.12+, TensorFlow 2.19 (TFLite), Scikit-learn 1.7.2.
- **Computer Vision**: OpenCV, Pillow, MobileNetV2 (Transfer Learning).
- **Data Science**: Pandas, NumPy 2.1.3, Matplotlib, Seaborn.
- **Web UI (Planned/POC)**: Flask, SQLAlchemy.
- **Hardware Target**: Raspberry Pi 4B (ARM Cortex-A72, 4GB RAM).

---

## 🧬 System Architecture

The system follows a **Notebook-first development workflow** and an **Orchestration-based deployment architecture**.

### 1. ML Pipelines
- **Disease Detection**: MobileNetV2 → TFLite. Binary (Healthy/Diseased) and Multiclass support. Optimized for 224x224 RGB inputs.
- **Irrigation Control**: RandomForestClassifier. Uses a 6-hour sliding window of sensor data (37 engineered features including rolling stats and change rates).
- **Crop Suitability**: TFLite model. Evaluates tomato suitability based on Nitrogen, Phosphorus, Potassium, pH, and environmental factors.

### 2. Integration & Fusion (scripts/edge_orchestrator.py)
- **Lazy Loading**: Models are loaded only when needed to save memory.
- **Late Fusion**: Disease detection results influence irrigation (e.g., fungal detection triggers reduced watering to inhibit pathogen spread).
- **Confidence-Aware**: Decision logic includes thresholds to trigger manual verification for low-confidence predictions.

---

## 📂 Project Structure

- `data/`: Contains `raw/` and `processed/` data. *Gitignored.*
- `models/`: Production-ready TFLite and Pickle models. *Gitignored.*
- `notebooks/`: **Primary development environment**.
  - `0_poc.ipynb`: Main integration proof-of-concept.
  - `1_disease_detection.ipynb`: Image classification pipeline.
  - `3_irrigation_preprocessing.ipynb`: Feature engineering (37 features).
  - `5_irrigation_rf_deployment.ipynb`: Production RandomForest training.
- `scripts/`: Utility and deployment scripts.
  - `edge_orchestrator.py`: Main system integration for edge devices.
  - `split_dataset_*.py`: Data preparation utilities.
- `logs/`: JSON-formatted training and execution logs.
- `docs/`: Technical rationales (e.g., `MODEL_SELECTION_RATIONALE.md`).

---

## 🚀 Building and Running

### Environment Setup
```bash
# Create and activate virtual environment
python3 -m venv .venv
source .venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

### Running the Orchestrator (Simulation)
The orchestrator simulates sensor inputs and runs the full ML stack.
```bash
python scripts/edge_orchestrator.py
```

### Development Workflow
1. **Always use `random_state=42`** for data splits and model training.
2. **Notebook-First**: Prototype all ML logic in Jupyter notebooks.
3. **TFLite Mandatory**: Image models must be converted to TFLite for edge compatibility.
4. **Log Everything**: Ensure `logs/` contains the latest training metrics in JSON format.

---

## 📝 Development Conventions

- **Feature Engineering**: Irrigation uses 3h, 6h, and 12h rolling statistics.
- **Safety First**: Prioritize "Production Reliability" over "Theoretical Accuracy". (e.g., Choosing RandomForest 97.89% over DecisionTree 100% due to overfitting).
- **Sri Lankan Context**: Seasonality checks (`is_yala_season`) are integrated into the decision logic.
- **Image Preprocessing**: Models include a `Rescaling(1./255)` layer; do not manually rescale in scripts unless specified.

---

## 🎯 Current Priorities

1. **Integration**: Complete the irrigation model integration in `notebooks/0_poc.ipynb`.
2. **Benchmarking**: Validate combined inference time on actual hardware (Pi 4B).
3. **Documentation**: Maintain `docs/MODEL_SELECTION_RATIONALE.md` for FYP defense preparation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tehaandev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tehaandev)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
