## bleue

> `bleue` is a Textual-based terminal user interface (TUI) and CLI tool for workflow management. It serves as an interactive interface for browsing issues, viewing details, and managing workflow status, backed by a Supabase database. The project was formerly/also known as CAPE (CLI, ADW, and Worker), and some code artifacts (like models and logs) still reflect this naming.

# Bleue

## Project Overview

`bleue` is a Textual-based terminal user interface (TUI) and CLI tool for workflow management. It serves as an interactive interface for browsing issues, viewing details, and managing workflow status, backed by a Supabase database. The project was formerly/also known as CAPE (CLI, ADW, and Worker), and some code artifacts (like models and logs) still reflect this naming.

## Architecture

The project is structured as a standard Python package managed by `uv`.

-   **CLI (`src/bleue/cli/`)**: The entry point for the application, built with `typer`.
-   **TUI (`src/bleue/tui/`)**: The main user interface, built with `textual`.
    -   `app.py`: The main `BleuApp` class.
    -   `screens/`: Contains different views like `IssueListScreen`, `IssueDetailScreen`, etc.
    -   `components/`: Reusable TUI widgets.
-   **Core (`src/bleue/core/`)**: Shared infrastructure.
    -   `models.py`: Pydantic models representing the domain (e.g., `CapeIssue`, `CapeComment`).
    -   `database.py`: Supabase database client and interaction logic.
-   **Tests (`tests/`)**: Comprehensive test suite using `pytest`.

### Tech Stack

-   **Language:** Python 3.12+
-   **Package Manager:** `uv`
-   **TUI Framework:** `textual`
-   **CLI Framework:** `typer`
-   **Database:** Supabase (via `supabase` and `postgrest` python clients)
-   **Validation:** `pydantic`

## Building and Running

### Prerequisites

-   Python 3.12+
-   `uv` installed
-   Supabase credentials (`SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`) set in `.env` or environment.

### Commands

**Run the TUI:**
```bash
uv run bleue
```

**Run the CLI (version check):**
```bash
uv run bleue --version
```

**Run Tests:**
```bash
uv run pytest tests/ -v
```

## Development Conventions

-   **Dependency Management:** Always use `uv sync` to install dependencies.
-   **Formatting:** The project uses `black` for code formatting.
    ```bash
    uv run black src/
    ```
-   **Linting:** The project uses `ruff` for linting.
    ```bash
    uv run ruff check src/
    ```
-   **Type Checking:** The project uses `mypy` for static type checking.
    ```bash
    uv run mypy
    ```
-   **Testing:** Write unit tests for new features. Tests are located in `tests/` and use `pytest-asyncio` for async support.

## Project Structure

```
bleue/
├── .cape/                # Logs (historical/CAPE context)
├── src/
│   └── bleue/
│       ├── cli/          # CLI entry points
│       ├── core/         # Data models and database logic
│       └── tui/          # TUI application and components
├── tests/                # Unit and integration tests
├── pyproject.toml        # Project configuration
├── uv.lock               # Lock file
└── README.md             # Project documentation
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bponghneng) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
