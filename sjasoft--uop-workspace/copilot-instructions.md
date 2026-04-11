## uop-workspace

> This document provides context for the `uop-workspace` project, a Python monorepo, to help guide future interactions with the Gemini CLI.

# Gemini Project Context: uop-workspace

This document provides context for the `uop-workspace` project, a Python monorepo, to help guide future interactions with the Gemini CLI.

## Project Overview

`uop-workspace` is a modular Python application built and managed using `uv`. It is structured as a monorepo containing several distinct packages, each with a specific purpose. The overall architecture suggests a client-server application with a pluggable database backend, a core business logic layer, and a client application.

### Key Packages

*   **`uop-core`**: This is the central package, containing the core business logic, database interfaces (`db_interface.py`, `db_collection.py`), data querying mechanisms (`query.py`), and application services. It has a command-line entry point defined in its `pyproject.toml`.
*   **`uop-client`**: This package likely represents the client-side of the application, responsible for managing state, loading data, and interacting with the `uop-core` services.
*   **`sjasoft-utils`**: A collection of general-purpose utility modules providing functionality for data structures, date/time manipulation, decorators, and more.
*   **Database Packages (`uop-db-mongo`, `uop-db-sqlite`)**: These packages provide concrete database implementations for the interfaces defined in `uop-core`, allowing for flexibility in the choice of data storage.
*   **`sjasoft-web`**: Suggests a web-facing component, possibly a web server or API.
*   **`uop-app-pyside`**: Indicates a desktop application built with PySide.

## Building and Running

The project uses `uv` for dependency management and workspace organization.

### Dependencies

To install all dependencies for the workspace, run:

```bash
uv pip install -r requirements.txt 
```

### Running Tests

The project is configured to use `pytest`. To run the tests for all packages, you can likely run `pytest` from the root of the workspace:

```bash
pytest
```

To run tests for a specific package, you can specify the package's test directory:

```bash
pytest packages/uop-core/tests/
```

### Running the Application

The `uop-core` package defines a command-line entry point `uop`. To run the application, you may be able to use:

```bash
# This is a placeholder, as the exact command may vary
uop
```

## Development Conventions

*   **Monorepo Structure**: The project is organized as a monorepo, with individual packages located in the `packages/` directory.
*   **Dependency Management**: `uv` is used for dependency management. Dependencies for each package are defined in their respective `pyproject.toml` files.
*   **Testing**: `pytest` is the designated testing framework. Each package appears to have its own `tests` directory.
*   **Namespaces**: The packages use namespace packages, as indicated by the `[tool.setuptools.packages.find]` configuration in the `pyproject.toml` files.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sjasoft)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sjasoft)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
