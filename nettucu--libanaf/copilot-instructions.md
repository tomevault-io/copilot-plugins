## libanaf

> This document provides essential context for AI models interacting with this project. Adhering to these guidelines will ensure consistency and maintain code quality.

# GEMINI.MD: AI Collaboration Guide

This document provides essential context for AI models interacting with this project. Adhering to these guidelines will ensure consistency and maintain code quality.

## 1. Project Overview & Purpose

* **Primary Goal:** This project, `libanaf`, is a Python library and Command Line Interface (CLI) designed to interact with the Romanian National Agency for Fiscal Administration (ANAF) e-Factura API. Its core functions include handling OAuth2 authentication, downloading and processing electronic invoices (in UBL XML format), and monitoring message statuses.
* **Business Domain:** The project operates in the Fintech sector, specifically focusing on electronic invoicing and fiscal compliance within Romania.

## 2. Core Technologies & Stack

* **Languages:** Python (>=3.11)
* **Frameworks & Runtimes:** Typer for the CLI application, with Flask used for the local server during the OAuth2 authentication flow.
* **Databases:** The application does not use a traditional database. It interacts directly with the ANAF API and stores configuration and downloaded invoices on the local filesystem.
* **Key Libraries/Dependencies:**
    *   `httpx`: Asynchronous HTTP client for API communication.
    *   `authlib`: Handles the OAuth2 authentication flow.
    *   `pydantic-xml`: For parsing and validating UBL XML invoices.
    *   `lxml`: Used for XML schema validation.
    *   `typer`: Powers the command-line interface.
    *   `rich`: For creating formatted and visually rich CLI output.
* **Package Manager(s):** `uv` is used to manage the packages

## 3. Architectural Patterns

* **Overall Architecture:** The project is a modular Python application with a clear separation of concerns. It exposes its functionality as both a library and a CLI. The architecture is designed around distinct packages for authentication (`auth/`), configuration (`config.py`), CLI commands (`cli/`), and domain-specific logic for invoices (`invoices/`) and UBL standards (`ubl/`).
* **Directory Structure Philosophy:**
    *   `/libanaf`: Contains all primary source code for the library and CLI.
    *   `/libanaf/auth`: OAuth2 flow (`AnafAuthClient`) and local TLS callback server (`OAuthCallbackServer`).
    *   `/libanaf/cli`: Typer CLI entry point (`app.py`) and sub-commands (`auth.py`, `invoices/`).
    *   `/libanaf/cli/invoices`: Typer commands for invoice management (`list`, `show`, `summary`, `prod-summary`, `download`, `process`, `render-pdf`).
    *   `/libanaf/invoices`: Pure library package — fetch, download, process, query, and summarize invoices. No CLI dependencies.
    *   `/libanaf/ubl`: UBL XML deserialization and validation via pydantic-xml.
    *   `/libanaf/failure_tracker.py`: Persistent JSON counter for consecutive sync failures across systemd restarts.
    *   `/libanaf/notifications.py`: Email alert helpers (`send_token_expired_alert`, `send_network_failure_alert`) via stdlib `smtplib`.
    *   `/secrets/.env.example`: Full reference for all environment variables (safe to commit — no real secrets).
    *   `/tests`: Unit and integration tests, with fixtures in `/tests/fixtures`.
    *   `/ai/briefs`: Task specifications for AI agents.
    *   `/docs`: Project documentation and sample files.

## 4. Coding Conventions & Style Guide

* **Formatting:** Code is formatted using `ruff` with a line length of 120 characters, double quotes (`"`) for strings, and space-based indentation. All code must pass `ruff check .`.
* **Naming Conventions:**
    *   `variables`, `functions`: `snake_case` (e.g., `get_config`)
    *   `classes`: `PascalCase` (e.g., `LibANAF_AuthClient`)
    *   `files`: `snake_case` (e.g., `product_summary.py`)
* **API Design:** The internal API is modular. The external interaction is with the ANAF REST/JSON API. The CLI follows a command/subcommand structure (e.g., `libanaf invoices prod-summary`).
* **Error Handling:** The application uses custom exceptions (`AnafRequestError` for HTTP/API errors; `AuthorizationError` → `TokenExpiredError` when the OAuth2 refresh token is expired and a hard re-auth is required) and implements a retry policy with exponential backoff for transient API errors. On sync failure, email alerts are sent via `libanaf/notifications.py` (STARTTLS + Gmail App Password); the consecutive failure count is persisted across runs in a JSON file by `libanaf/failure_tracker.py`.

## 5. Key Files & Entrypoints

* **Main Entrypoint(s):** The primary entrypoint for the CLI application is the `app` object in `libanaf/cli/app.py`, which is exposed as a script via `pyproject.toml` (`libanaf.cli:app`).
* **Configuration:** Configuration is loaded from a `.env` file (default: `secrets/.env`, overridden by `LIBANAF_ENV_FILE` env var) via `pydantic-settings` with env prefix `LIBANAF_` and nested delimiter `__` (e.g. `LIBANAF_AUTH__CLIENT_ID`). The `libanaf/config.py` module manages access via the cached `get_settings()` function. Settings sections: `auth`, `connection`, `efactura`, `storage`, `retry`, `log`, `notification` (email alerts), `state` (failure counter path). See `secrets/.env.example` for a full reference.
* **CI/CD Pipeline:** No CI/CD pipeline configuration file (e.g., `.github/workflows/`) was detected in the project structure.

## 6. Development & Testing Workflow

* **Local Development Environment:** To set up a local development environment, install the project in editable mode with development dependencies using `uv pip install -e .[dev]`.
* **Testing:** Tests are written using `pytest` and executed by running `uv run --with pytest pytest -q`. Test files are located in the `/tests` directory. New code should have corresponding test coverage, aiming for ≥85%.
* **CI/CD Process:** There is no automated CI/CD process defined in the repository. All checks (linting, testing) are expected to be run manually before committing.

## 7. Specific Instructions for AI Collaboration

* **Contribution Guidelines:** The `AGENTS.md` file is the definitive guide. Key rules include:
    *   All code must pass `ruff check .` and all tests (`uv run --with pytest pytest -q`).
    *   New code in a module should achieve at least 85% test coverage.
    *   Functions must have Google or NumPy style docstrings and full type hints.
    *   Use f-strings for string formatting.
    *   Do not duplicate code; refactor for reuse if necessary.
    *   All work should be done in feature branches named `ai/<short-task-name>`.
* **Infrastructure (IaC):** No Infrastructure as Code (IaC) was detected.
* **Security:** Be extremely mindful of security. Do not hardcode secrets or keys. Do not log sensitive data like tokens or invoice contents in plaintext. Live ANAF endpoints must not be called without explicit human approval.
* **Dependencies:** New dependencies should be added to the `[project.dependencies]` or `[project.optional-dependencies]` section in `pyproject.toml`.
* **Commit Messages:** While not explicitly defined, follow the Conventional Commits specification (e.g., `feat:`, `fix:`, `docs:`, `refactor:`) for clear and consistent commit history.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nettucu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
