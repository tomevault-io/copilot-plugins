## ai-platform-trainer

> This file provides an overview of the `ai-platform-trainer` project, its structure, and common commands to ensure the Gemini assistant can be as effective as possible.

# Gemini Context: ai-platform-trainer

This file provides an overview of the `ai-platform-trainer` project, its structure, and common commands to ensure the Gemini assistant can be as effective as possible.

## 1. Project Summary

This project is a Python-based platform for training and running AI models within a 2D game simulation, with a high-performance C++/CUDA backend for physics calculations.

## 2. Core Technologies

- **Primary Language:** Python 3
- **AI/ML:** Stable Baselines3 (inferred from model structure), PyTorch (likely backend)
- **Game/Simulation:** Pygame (inferred from gameplay logic and rendering files)
- **High-Performance Backend:** C++ with CUDA for GPU-accelerated physics.
- **Python/C++ Bridge:** Pybind11
- **Code Quality:** Pylint, pre-commit hooks

## 3. Architectural Overview

- **`ai_platform_trainer/`**: The main Python source code module.
  - **`ai/`**: Contains all AI-related logic, including model definitions (`models/`), training environments (`training/`), and inference code (`inference/`).
  - **`core/`**: Application-wide components like configuration management and logging.
  - **`cpp/`**: Source code for the high-performance C++/CUDA physics engine. Interfaced with Python via Pybind11 (`pybind/`).
  - **`entities/`**: Defines all game objects (Player, Enemy, Missile) and their behaviors.
  - **`gameplay/`**: Manages the core game loop, rendering, state, and user input.
- **`assets/`**: Contains static game assets like sprites.
- **`data/`**: Stores raw training data.
- **`models/`**: **Output directory** for trained and serialized model files (e.g., `.zip`).
- **`tests/`**: Contains all unit and integration tests.
- **`scripts/`**: Utility and deployment-related shell scripts.
- **`run_game.py`**: The main entry point for running the game in interactive mode.

## 4. Key Workflows & Commands

- **Install Dependencies:**
  ```bash
  pip install -r requirements.txt
  ```
  *(Note: The C++ environment may require separate compilation with CMake and a C++ compiler with CUDA support.)*

- **Run the Game:**
  ```bash
  python run_game.py
  ```

- **Run Tests:**
  ```bash
  pytest
  ```

- **Run Linting & Code Quality Checks:**
  ```bash
  pre-commit run --all-files
  pylint ai_platform_trainer
  ```

- **Train a Model (Example):**
  ```bash
  python -m ai_platform_trainer.ai.training.train_enemy_rl
  ```

## 5. Project Conventions

- **Performance-Critical Code:** Physics and heavy computations should be implemented in the C++/CUDA backend (`cpp/`) and exposed to Python.
- **High-Level Logic:** Game logic, AI agent control, and UI should be implemented in Python.
- **Configuration:** Centralized configuration is managed via `config.json` and accessed through the `ConfigManager` in `ai_platform_trainer/core/`.
- **Testing:** New features should be accompanied by corresponding unit tests in `tests/unit/` or integration tests in `tests/integration/`.
- **Code Style:** Adhere to the rules enforced by `.pylintrc` and the `.pre-commit-config.yaml`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/life423)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/life423)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
