## nb-wrangler

> `nb-wrangler` is a command-line tool that streamlines the curation of JupyterLab notebooks and their runtime environments. It automates the process of building and testing container images from notebook requirements, ensuring that notebooks have the correct dependencies to run successfully.

# GEMINI.md

## Project Overview

`nb-wrangler` is a command-line tool that streamlines the curation of JupyterLab notebooks and their runtime environments. It automates the process of building and testing container images from notebook requirements, ensuring that notebooks have the correct dependencies to run successfully.

The project is a Python-based command-line tool that uses a `spec.yaml` file to define a notebook environment. It leverages `micromamba` for Conda environment management and `uv` for fast pip package installation.

The core workflow is divided into two phases:

1.  **Curation:** Defining the notebooks, Python packages, and data required for a specific environment. This is done by creating a `nbw-spec.yaml` file.
2.  **Reinstallation:** Creating a new environment from a completed `spec.yaml` file.

## Building and Running

### Installation

The project is designed to be bootstrapped and then run from a controlled environment.

1.  **Bootstrap `nb-wrangler`**:
    ```bash
    curl https://raw.githubusercontent.com/spacetelescope/nb-wrangler/refs/heads/main/nb-wrangler >nb-wrangler
    chmod +x nb-wrangler
    ./nb-wrangler bootstrap
    ```

2.  **Activate the environment**:
    ```bash
    source ./nb-wrangler environment
    ```

### Running the Tool

`nb-wrangler` is executed via the `nb-wrangler` script. The main command takes a `spec.yaml` file as input, along with various options to control its behavior.

**Example Curation Workflow:**

```bash
# Curate the software environment
./nb-wrangler spec.yaml --curate

# Test environment basics rapidly
./nb-wrangler spec.yaml --test-imports

# Curate the data dependencies
./nb-wrangler spec.yaml --data-curate

# Run each notebook headless using papermill
./nb-wrangler spec.yaml --test-notebooks
```

**Example Reinstallation Workflow:**

```bash
# Reinstall the software environment
./nb-wrangler spec.yaml --reinstall

# Reinstall the data
./nb-wrangler spec.yaml --data-reinstall

# Run both import and notebook tests
./nb-wrangler spec.yaml --test-all
```

## Development

For development, you can install the project and its dependencies using the `Makefile`:

*   **`make setup`**: Installs the project and its development dependencies.
*   **`make test`**: Runs the test suite using `pytest`.
*   **`make lint`**: Runs various code quality checks (flake8, mypy, black, bandit).
*   **`make coverage`**: Generates a test coverage report.
*   **`make clean`**: Removes build and test artifacts.

### Development Conventions

*   **Code Style:** The project uses `black` for code formatting and `flake8` for linting.
*   **Type Checking:** `mypy` is used for static type checking.
*   **Testing:** `pytest` is used for unit and functional testing. The `Makefile` provides several targets for running tests, including `test`, `test-functional`, and `local-test`.
*   **Dependency Management:** Project dependencies are managed in `pyproject.toml`. The tool itself manages notebook dependencies through the `spec.yaml` file.
*   **Packaging:** The project is packaged using `hatchling`.

### Logging and returning errors

In general,  WranglerLogger methods like info or error return a boolean value with the sense of "no errors". Consequently,
a returning the value from "info(...)" will return *True*, as does "warning(...)".   while returning the value from
"error(...)" or "exception(...)" will return *False*.  This is unfortunately the *opposite* of how shells tend to return
error status where 0 denotes no errors and a non-zero uint code denotes a specific error.  The convention permits succinct
code such that "if call():" has the meaning "if call() succeeded" and "if not call()" has the meaning "call() failed."

### Installing micromamba

The "nb-wrangler bootstrap" script invocation installs micromamba and nb-wrangler relative to NBW_ROOT and other env vars.
DO NOT abandon use of the standard micromamba install.sh script because of errors initializing the wrangler environment related to shell init.
This script works fine using <<EOF notation to input parameters and is critical to keeping bootstrapping resilient to changes
in micromamba and mamba.

### Avoid Code Duplication

Whenever you spot the same intricate code in multiple spots, *factor it out* into a single, well‑named property or function, parameterize it as needed, and replace all inline copies with calls to that abstraction. This keeps the codebase cleaner, more readable, and easier to maintain.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/spacetelescope)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/spacetelescope)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
