## macos-optimizer

> This document provides a comprehensive overview of the "ML Performance Optimizer" project to prime the Gemini Code Assistant for effective collaboration.

# Gemini Code Assistant Context

This document provides a comprehensive overview of the "ML Performance Optimizer" project to prime the Gemini Code Assistant for effective collaboration.

## Project Overview

The "ML Performance Optimizer" is a sophisticated, self-learning background agent specifically designed for macOS. Its primary function is to monitor, predict, and autonomously optimize system performance, with a special focus on Python and AI/ML inference workflows.

The agent operates in two distinct phases:
-   **Phase 1 (Initial Learning):** For the first 3 weeks, the agent collects system metrics (RAM, CPU, swap per process) every 5 seconds. It uses this data to train two scikit-learn models: a regressor to predict future RAM usage and a classifier to predict memory spikes.
-   **Phase 2 (Reinforcement Learning):** After accumulating sufficient historical data, the agent transitions to a PyTorch-based Reinforcement Learning (RL) policy. This policy is bootstrapped using behavioral cloning from Phase 1 decisions and then continuously improves online using the REINFORCE algorithm to discover optimal, user-specific scheduling strategies.

The system is designed to be highly autonomous, capable of sending notifications, renicing processes, and dropping model caches. More critical actions, like killing idle processes, require user confirmation via a macOS dialog.

### Key Technologies

-   **Backend:** Python 3.11+
-   **Machine Learning:**
    -   scikit-learn (GradientBoosting for initial models)
    -   PyTorch (for the Phase 2 RL policy)
-   **Data Storage:** SQLite for time-series metrics
-   **Core Libraries:** `psutil` for system metric collection
-   **Deployment:** Can be run as a foreground script or a persistent background daemon via macOS `launchd`.

### Architecture

The project follows a modular structure:
-   `macos_optimizer/cli.py`: Main entry point for the agent daemon and CLI commands.
-   `macos_optimizer/core/collector.py`: Responsible for sampling system metrics using `psutil` and storing them in a local SQLite database.
-   `macos_optimizer/core/features.py`: Feature engineering pipeline to process raw metrics into features for the ML models.
-   `macos_optimizer/core/models/`: Contains the machine learning models.
    -   `phase1_sklearn.py`: A GradientBoosting regressor and classifier.
    -   `phase2_rl.py`: The PyTorch-based RL policy.
-   `macos_optimizer/core/decisions.py`: Maps model predictions to concrete system actions.
-   `macos_optimizer/core/actions.py`: Implements the system-level actions (e.g., `kill`, `renice`).
-   `macos_optimizer/core/autonomy.py`: A gating mechanism that determines whether an action is executed automatically, requires user permission ("Ask"), or is disabled.
-   `config.yaml`: A central configuration file for all tunable parameters, which supports hot-reloading.

## Building and Running

### Installation via Homebrew (Recommended)

1.  **Tap the Homebrew repository:**
    ```bash
    brew tap gabrielvfonseca/macos-optimizer git@github.com:gabrielvfonseca/macos-optimizer.git
    ```
2.  **Install the package:**
    ```bash
    brew install macos-optimizer
    ```

After installation, follow the instructions provided by `brew info macos-optimizer` to set up the launchd service.

### Manual Setup (If not using Homebrew)

1.  **Clone the repository:**
    ```bash
    git clone git@github.com:gabrielvfonseca/macos-optimizer.git
    cd ml_optimizer
    ```
2.  **Install dependencies:**
    ```bash
    pip install -r requirements.txt
    ```
3.  **(Optional) Install PyTorch for Phase 2 RL:**
    ```bash
    pip install torch
    ```

### Running the Agent

-   **Foreground Mode (for debugging):**
    ```bash
    python macos_optimizer/cli.py
    ```
-   **Background Daemon:**
    ```bash
    python macos_optimizer/cli.py --daemon
    ```
-   **Stop the Daemon:**
    ```bash
    python macos_optimizer/cli.py --stop
    ```

### Running as a macOS Service (`launchd`)

1.  **Configure the service file:** Edit the paths in `com.mloptimizer.agent.plist` to match your environment.
2.  **Install and start the service:**
    ```bash
    cp com.mloptimizer.agent.plist ~/Library/LaunchAgents/
    launchctl load ~/Library/LaunchAgents/com.mloptimizer.agent.plist
    launchctl start com.mloptimizer.agent
    ```
3.  **Monitor logs:**
    ```bash
    tail -f /Users/gabrielfonseca/Desktop/macos_optimizer/logs/agent.log
    ```

### CLI Commands

-   **Check Status:** Get a snapshot of system state and model metrics.
    ```bash
    python macos_optimizer/cli.py --status
    ```
-   **Force Retrain:** Manually trigger the retraining of the machine learning models.
    ```bash
    python macos_optimizer/cli.py --train
    ```

## Development Conventions

### Configuration
-   All core settings are managed in `config.yaml`.
-   The configuration is hot-reloaded every 60 seconds, allowing for dynamic tuning without restarting the agent.
-   Key tunable parameters include `thresholds`, `autonomy` levels for actions, process `whitelist`, and action `cooldowns`.

### Autonomy Model
The agent's actions are governed by an autonomy model defined in `config.yaml`:
-   **Auto:** Actions that are always performed automatically (e.g., sending notifications).
-   **Ask:** Actions that require explicit user confirmation via a system dialog (e.g., killing a process).
-   **Never:** Actions that are disabled.

### Logging
-   The agent maintains a rotating log file at `logs/agent.log`.
-   Use `tail -f logs/agent.log` to monitor the agent's activity in real-time.

### Contribution
-   Before contributing, please review the `CONTRIBUTING.md` and `CODE_OF_CONDUCT.md` files.
-   Follow existing code style and patterns.
-   Ensure any new functionality is accompanied by appropriate documentation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gabrielvfonseca)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gabrielvfonseca)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
