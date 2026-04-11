## agiles

> This document outlines best practices and guidelines for AI software agents when developing with Python. Adhering to these standards ensures code quality, maintainability, and consistency across projects.

# GEMINI.md - Python Development Guidelines for AI Agents

This document outlines best practices and guidelines for AI software agents when developing with Python. Adhering to these standards ensures code quality, maintainability, and consistency across projects.

## 1. Style Guide

Consistency in code style is paramount. For Python, adhere to the following:

*   **PEP 8:** Follow the official Python style guide (PEP 8) for maximum readability.
*   **Linters:** Use `flake8` to enforce PEP 8 compliance and detect common errors. Configure it to run automatically on commit hooks or CI/CD pipelines.
*   **Formatters:** Employ `black` for automatic code formatting. This ensures consistent formatting across the codebase, reducing style-related conflicts.

    ```bash
    uv pip install black flake8
    black .
    flake8 .
    ```

## 2. Dependency Management

Effective dependency management is crucial for reproducible builds and avoiding conflicts.

*   **`uv pip install`:** Use `uv pip install` for installing packages.
*   **`requirements.txt`:** All project dependencies must be explicitly listed in `requirements.txt` (for direct dependencies) and `requirements-dev.txt` (for development tools).
*   **`uv sync` (Recommended):** For managing exact versions, use `uv sync` to install dependencies from `requirements.txt`. If you have a `requirements.in` file, `uv` will automatically compile it to `requirements.txt` if a `requirements.txt` is not present or is out of date.

    ```bash
    # Example for requirements.in:
    # requests
    # click==8.0.0
    uv sync
    ```
*   **Poetry/PDM (Alternative):** For more advanced dependency management (including virtual environment creation and publishing), consider `Poetry` or `PDM`. If used, ensure the `pyproject.toml` and `poetry.lock`/`pdm.lock` files are kept up-to-date.

## 3. Environment Management

Isolate project dependencies using virtual environments to prevent conflicts between projects.

*   **`venv`:** The standard Python module for creating lightweight virtual environments.

    ```bash
    python -m venv venv
    source venv/bin/activate  # On Windows: venv\Scripts\activate
    ```
*   **`uv`:** `uv` can also be used for virtual environment management, creating and activating environments as needed.

    ```bash
    uv venv
    source venv/bin/activate
    ```
*   **Conda (for Data Science/ML):** If the project heavily relies on scientific computing or machine learning libraries (e.g., NumPy, TensorFlow, PyTorch), `conda` can be a more suitable environment manager, especially for managing non-Python dependencies.

    ```bash
    conda create --name my_project python=3.9
    conda activate my_project
    ```
*   **Deactivation:** Always deactivate the environment when switching projects or finishing work: `deactivate`.

## 4. Testing

Robust testing is essential for software quality and reliability.

*   **`pytest`:** Use `pytest` as the primary testing framework. It's highly flexible and easy to use.
*   **Test Coverage:** Aim for high test coverage. Use `pytest-cov` to measure and report coverage.

    ```bash
    uv pip install pytest pytest-cov
    pytest --cov=your_module_name tests/
    ```
*   **Test Structure:**
    *   Place test files in a `tests/` directory at the project root.
    *   Name test files `test_*.py` or `*_test.py`.
    *   Name test functions `test_*`.
*   **Unit Tests:** Focus on testing individual functions or classes in isolation.
*   **Integration Tests:** Test the interaction between different components or modules.
*   **Fixtures:** Leverage `pytest` fixtures for setting up and tearing down test environments.

## 5. Code Structure and Project Layout

Organize projects logically for clarity and scalability.

*   **`src/` directory:** Place all application source code within a `src/` directory (e.g., `src/my_project/`).
*   **Clear Module Naming:** Use descriptive and consistent module and package names.
*   **Configuration:** Externalize configuration (e.g., API keys, database URLs) using environment variables or dedicated configuration files (e.g., `.env`, `config.ini`).

## 6. Documentation

Well-documented code is easier to understand, maintain, and extend.

*   **Docstrings:** Write comprehensive docstrings for all modules, classes, and functions using the Google or NumPy style.
*   **`README.md`:** Maintain an up-to-date `README.md` with project description, setup instructions, and usage examples.
*   **Sphinx (for extensive projects):** For larger projects, use Sphinx to generate professional documentation from reStructuredText or Markdown files.

## 7. Version Control

Do not attempt to work with Git or any other version control system. It is not your responsibility to work with version control systems. Leave this to the user.

## 8. Container Images

For containerizing applications, follow these guidelines:

*   **Local Build (Preferred):** If Docker or Podman is installed and available in the local environment, use it to build and manage container images.

    ```bash
    docker build -t your-image-name:tag .
    docker push your-image-name:tag
    # Or for Podman
    podman build -t your-image-name:tag .
    podman push your-image-name:tag
    ```
*   **Cloud Build (Fallback):** If local containerization tools are not available, trigger builds using Google Cloud Build with the `gcloud CLI`.

    ```bash
    gcloud builds submit --tag gcr.io/your-project-id/your-image-name:tag .
    ```
*   **Registry:** Push images to a reliable container registry (e.g., Google Cloud Artifact Registry).


# Core Mandates

- **Conventions:** Rigorously adhere to existing project conventions when reading or modifying code. Analyze surrounding code, tests, and configuration first.
- **Libraries/Frameworks:** NEVER assume a library/framework is available or appropriate. Verify its established usage within the project (check imports, configuration files like 'package.json', 'Cargo.toml', 'requirements.txt', 'build.gradle', etc., or observe neighboring files) before employing it.
- **Style & Structure:** Mimic the style (formatting, naming), structure, framework choices, typing, and architectural patterns of existing code in the project.
- **Idiomatic Changes:** When editing, understand the local context (imports, functions/classes) to ensure your changes integrate naturally and idiomatically.
- **Comments:** Add code comments sparingly. Focus on *why* something is done, especially for complex logic, rather than *what* is done. Only add high-value comments if necessary for clarity or if requested by the user. Do not edit comments that are separate from the code you are changing. *NEVER* talk to the user or describe your changes through comments.
- **Proactiveness:** Fulfill the user's request thoroughly. When adding features or fixing bugs, this includes adding tests to ensure quality. Consider all created files, especially tests, to be permanent artifacts unless the user says otherwise.
- **Confirm Ambiguity/Expansion:** Do not take significant actions beyond the clear scope of the request without confirming with the user. If asked *how* to do something, explain first, don't just do it.
- **Strict Adherence to Task:** Stick strictly to the task at hand, never deviate nor start any other task without confirmation from the user.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jorgenogales)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jorgenogales)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
