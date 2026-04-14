## goes-vfi

> Project rules and convetions

# GOES-VFI Project Rules & Conventions

This document provides context for AI assistants working on the GOES-VFI project.

## 1. Project Overview

*   **Goal:** GOES-VFI (GOES Video Frame Interpolation) is a Python application with a PyQt6 GUI designed to create smooth timelapse videos from sequences of GOES satellite images.
*   **Core Functionality:**
    *   Accepts a directory of sequentially named PNG images.
    *   Uses the RIFE AI model for frame interpolation to generate intermediate frames.
    *   Optionally uses FFmpeg's `minterpolate` filter for additional motion smoothing.
    *   Optionally uses FFmpeg's `unsharp` filter for sharpening.
    *   Encodes the final sequence into an MP4 video using FFmpeg with various codec options (software and hardware).
    *   Provides a GUI for configuring interpolation parameters, filter settings, and output quality.
    *   Includes features like image preview, cropping, and model management (currently basic).

## 2. Core Technologies

*   **Language:** Python (>= 3.10 recommended)
*   **GUI:** PyQt6
*   **Core Processing:**
    *   FFmpeg (expected to be installed system-wide or accessible in PATH)
    *   RIFE AI Model (specifically `rife-cli`, located in `goesvfi/bin/`)
*   **Key Libraries:**
    *   `pathlib` for path manipulation.
    *   `NumPy` (likely used by dependencies or potentially future image processing).
    *   `QSettings` for saving UI state/preferences (e.g., crop rectangle).
    *   `argparse` for command-line arguments (e.g., `--debug`).
    *   `importlib.resources` for accessing package data (like the RIFE executable).
    *   `logging` (via `goesvfi.utils.log`) for application logging.
    *   `colorlog` for colored console logging.

## 3. Directory Structure

*   `goesvfi/`: Main application source code.
    *   `gui.py`: Contains the main `MainWindow`, worker thread (`VfiWorker`), custom widgets (`ClickableLabel`, `ZoomDialog`, `CropDialog`), and UI setup logic.
    *   `pipeline/`: Modules related to the core VFI and encoding process.
        *   `run_vfi.py`: Handles the RIFE model execution and intermediate file generation.
        *   `encode.py`: Handles the final FFmpeg encoding step.
    *   `utils/`: Utility modules.
        *   `config.py`: Manages configuration settings (e.g., output directory).
        *   `log.py`: Configures logging for the application.
    *   `bin/`: Contains external executables, primarily `rife-cli`.
    *   `models/`: Likely intended for storing AI model checkpoints (though currently relies on the packaged `rife-cli`).
*   `.github/workflows/`: Continuous integration/deployment configurations (if any).
*   `.venv/`: Python virtual environment (standard practice).
*   `tests/`: Directory for automated tests (currently seems empty or unused, but should ideally contain `pytest` tests).
*   `docs/`: Project documentation.
*   `requirements.txt`: Lists Python package dependencies.
*   `pyproject.toml`: Project metadata and build configuration (e.g., for `setuptools` or `poetry`).

## 4. Key Files
*   `goesvfi/gui.py`: Central file for the UI logic and main window structure. Most UI changes happen here.
*   `goesvfi/pipeline/run_vfi.py`: Handles the core AI interpolation logic.
*   `goesvfi/pipeline/encode.py`: Handles FFmpeg commands for final video output.
*   `requirements.txt` / `pyproject.toml`: Define dependencies.

## 5. Coding Style & Conventions

*   **Style Guide:** Adhere strictly to PEP 8. Use linters like `flake8` or `ruff` if possible.
*   **Type Hinting:** Use Python's standard `typing` module extensively for all functions and methods. Aim for full type coverage.
*   **Logging:** Use the configured `LOGGER` instance obtained via `goesvfi.utils.log.get_logger(__name__)` within modules. Log informative messages, warnings, and errors appropriately.
*   **Naming:**
    *   Functions and variables: `snake_case`.
    *   Classes: `CamelCase`.
    *   Constants: `UPPER_SNAKE_CASE`.
    *   UI Elements: Store UI widgets as instance attributes, e.g., `self.start_btn`, `self.fps_spin`. Use descriptive names.
*   **Imports:** Use absolute imports where possible (`from goesvfi.utils import log`).
*   **Error Handling:** Use `try...except` blocks for operations that might fail (file I/O, external process calls). Log exceptions using `LOGGER.exception()`. Raise specific exceptions when appropriate.
*   **External Processes:** Use the `subprocess` module to run external commands like `ffmpeg` and `rife-cli`. Capture output/errors and check return codes carefully. The `_run_ffmpeg_command` helper in `encode.py` is a good example.
*   **Concurrency:** Long-running tasks (VFI, encoding) are handled in a separate `QThread` (`VfiWorker`) to keep the GUI responsive. Use signals (`pyqtSignal`) to communicate progress, results, and errors back to the main thread.

## 6. Workflow & Testing

*   **Development:** Edit code -> Run GUI (`python -m goesvfi.gui`) -> Test functionality manually.
*   **Testing:**
    *   Automated tests using `pytest` should be added to the `tests/` directory.
    *   Focus on testing the `pipeline` modules and utility functions.
    *   GUI testing is complex; prioritize testing the underlying logic triggered by UI actions.
    *   Follow the **Testing and Validation** steps outlined in the custom instructions (run tests after changes, analyze failures, add new tests for uncovered issues).
*   **Debugging:** Use the `--debug` flag (`python -m goesvfi.gui --debug`) to prevent cleanup of intermediate files and potentially enable more verbose logging.

## 7. Dependencies

*   Managed primarily via `requirements.txt`. Ensure it's kept up-to-date.
*   `pyproject.toml` might also contain dependency information if using modern packaging tools.
*   External non-Python dependencies: FFmpeg (assumed in PATH), RIFE (`goesvfi/bin/rife-cli`).

## 8. GUI Structure

*   Organized into tabs (`QTabWidget`): "Interpolate", "Interpolation Filters", "Models", "FFmpeg Quality".
*   Uses standard PyQt6 layouts (`QVBoxLayout`, `QHBoxLayout`, `QGridLayout`).
*   Uses custom widgets (`ClickableLabel`, `ZoomDialog`, `CropDialog`) for specific interactions.
*   Long operations are offloaded to `VfiWorker` (`QThread`) to prevent UI freezing.
*   State changes (e.g., enabling/disabling widgets based on other selections) are handled by connecting signals (`.connect()`) to dedicated update slots (e.g., `_update_scd_thresh_state`).

## 9. Custom Instructions Compliance

*   **Strictly follow the user's custom instructions provided in the conversation history.** This includes:
    *   **Analysis & Decomposition:** Break down tasks, ask clarifying questions.
    *   **Planning:** Create detailed, sequential plans before coding.
    *   **Incremental Changes:** Implement changes in small, logical steps. Use checkpoints/commits.
    *   **Testing & Validation:** Run tests after each increment, analyze results, fix issues, add tests if necessary.
    *   **Feedback:** Provide regular updates on progress and test results.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jtn0123)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jtn0123)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
