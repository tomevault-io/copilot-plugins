## vpsweb

> This document provides context for the Gemini Code Assistant to understand the `vpsweb` project.

# Gemini Code Assistant Context

This document provides context for the Gemini Code Assistant to understand the `vpsweb` project.

## Project Overview

`vpsweb` is a Python application for AI-powered poetry translation. It uses a three-step workflow to produce high-fidelity translations:

1.  **Initial Translation:** An LLM translates the original poem and provides translator's notes.
2.  **Editor Review:** A second LLM reviews the initial translation and provides suggestions for improvement.
3.  **Translator Revision:** A third LLM revises the initial translation based on the editor's feedback.

The project also includes features for:

*   **WeChat Integration:** Generating and publishing articles to WeChat Official Accounts.
*   **Web Interface:** A FastAPI-based web UI for managing poems and translations.
*   **CLI:** A command-line interface for running the translation workflow.

## Technologies

*   **Programming Language:** Python 3.9+
*   **Web Framework:** FastAPI
*   **Database:** SQLAlchemy with SQLite
*   **Data Validation:** Pydantic
*   **Dependency Management:** Poetry
*   **CLI:** Click

## Building and Running

The project uses a set of shell scripts for common tasks.

### Setup

To set up the development environment, run the following command:

```bash
./scripts/setup.sh
```

This will install dependencies, initialize the database, and perform other setup tasks.

### Running the Web Application

To start the web application, run:

```bash
./scripts/start.sh
```

The web interface will be available at `http://127.0.0.1:8000`.

### Running the CLI

The main entry point for the CLI is `vpsweb`. You can use it to run the translation workflow, generate WeChat articles, and publish articles.

**Translate a poem:**

```bash
vpsweb translate -i poem.txt -s English -t Chinese
```

**Generate a WeChat article:**

```bash
vpsweb generate-article -j translation.json
```

**Publish a WeChat article:**

```bash
vpsweb publish-article -d article_directory/
```

## Development Conventions

*   **Code Style:** The project uses `black` for code formatting and `flake8` for linting.
*   **Testing:** The project uses `pytest` for testing. Tests are located in the `tests/` directory. To run the tests, use the `pytest` command.
*   **Type Hinting:** The project uses type hints throughout the codebase, and `mypy` is used for static type checking.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OCboy5)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/OCboy5)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
