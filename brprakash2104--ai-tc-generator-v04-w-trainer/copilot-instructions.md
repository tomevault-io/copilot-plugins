## ai-tc-generator-v04-w-trainer

> This document provides context for the Gemini Code Assistant to understand the `ai-tc-generator` project.

# Gemini Code Assistant Context

This document provides context for the Gemini Code Assistant to understand the `ai-tc-generator` project.

## Project Overview

`ai-tc-generator` is a Python command-line tool that generates test cases from REQIFZ files. It uses large language models (LLMs) via Ollama to understand the requirements and generate relevant test cases. The tool is designed to be modular and extensible, with a clear separation of concerns between data extraction, generation, and formatting.

The project is built with Python 3.14+ and uses modern tooling like `pyproject.toml` for dependency management, `hatchling` for building, `pytest` for testing, and `ruff` for linting and formatting.

### Key Technologies

*   **Python 3.14+**
*   **Ollama:** For interacting with local LLMs, including vision models like `llama3.2-vision:11b`.
*   **Click:** For the command-line interface.
*   **Rich:** for rich text and beautiful formatting in the terminal.
*   **Pydantic:** For data validation and settings management.
*   **aiohttp:** For asynchronous HTTP requests in high-performance mode.
*   **Pandas & Openpyxl:** For writing test cases to Excel files.
*   **Pytest:** For running tests.
*   **Ruff:** For linting and formatting.
*   **Hatchling:** For building the project.
*   **torch & transformers:** For model training and fine-tuning.

### Architecture

The project follows a modular architecture, with the core logic organized into the following components:

*   **`src/core`:** Contains the core components for extraction, generation, formatting, and interaction with Ollama. This now includes a vision-capable client for handling image-based requirements.
*   **`src/processors`:** Contains the high-level processors that orchestrate the test case generation workflow. There are two processors: a standard processor and a high-performance asynchronous processor.
*   **`src/prompts`:** Contains the prompt templates used for generating test cases.
*   **`src/training`:** Implements the training and fine-tuning pipelines for the models, including progressive curriculum learning and vision-aware RAFT training.
*   **`main.py`:** The main entry point for the CLI application.

### Vision Analysis

The project now includes a hybrid AI model strategy that intelligently switches between a text-based model and a vision-capable model. This approach optimizes for both speed and accuracy:

*   **Text-only requirements:** Processed by a fast text-based model (e.g., `llama3.1:8b`).
*   **Requirements with images:** Processed by a more powerful vision-capable model (e.g., `llama3.2-vision:11b`) for higher-quality analysis.

This hybrid strategy is enabled by default. You can configure the vision model settings in `config/cli_config.yaml` or disable it entirely by setting `enable_vision: false`.

## Building and Running

### Installation

The project uses `pyproject.toml` to manage dependencies. To install the project and its dependencies, use one of the following commands:

*   **For production use:**
    ```bash
    pip install -e .
    ```
*   **For development (includes dev tools like pytest, ruff, mypy):**
    ```bash
    pip install -e .[dev]
    ```
*   **For training features (includes torch, transformers):**
    ```bash
    pip install -e .[training]
    ```
*   **For all dependencies:**
    ```bash
    pip install -e .[all]
    ```

### Running the Application

The application can be run using the `ai-tc-generator` or `ai-tc-gen` command.

**Basic Usage:**

```bash
ai-tc-generator <input_path>
```

*   `<input_path>`: Path to a REQIFZ file or a directory containing REQIFZ files.

**Example:**

```bash
ai-tc-generator input/sample.reqifz
```

**High-Performance Mode:**

To enable high-performance mode, use the `--hp` flag:

```bash
ai-tc-generator --hp input/sample.reqifz
```

### Running Tests

The project uses `pytest` for testing. To run the tests, use the following command:

```bash
pytest
```

## Development Conventions

### Linting and Formatting

The project uses `ruff` for linting and formatting. To check for linting errors, run:

```bash
ruff check .
```

To format the code, run:
```bash
ruff format .
```

### Commits

Commit messages should follow the Conventional Commits specification.

### Branching

The project uses the following branching strategy:

*   **`main`:** The main branch, which contains the latest stable release.
*   **`develop`:** The development branch, where new features are integrated.
*   **`feature/...`:** Feature branches, where new features are developed.
*   **`bugfix/...`:** Bugfix branches, where bugs are fixed.
*   **`release/...`:** Release branches, which are used to prepare new releases.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BRPRAKASH2104)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/BRPRAKASH2104)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
