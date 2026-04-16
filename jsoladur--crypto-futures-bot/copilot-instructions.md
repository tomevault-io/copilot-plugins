## crypto-futures-bot

> This project is a Python-based cryptocurrency trading bot designed to automate futures trading on the **MEXC** and **Bitget** exchanges. It integrates with **Telegram** for real-time monitoring and manual control. The architecture is modular, leveraging dependency injection, asynchronous programming, and a clean separation of concerns (Domain, Infrastructure, Interfaces). The active exchange is selected via the `FUTURES_EXCHANGE` environment variable.

# Crypto Futures Bot

## Project Overview
This project is a Python-based cryptocurrency trading bot designed to automate futures trading on the **MEXC** and **Bitget** exchanges. It integrates with **Telegram** for real-time monitoring and manual control. The architecture is modular, leveraging dependency injection, asynchronous programming, and a clean separation of concerns (Domain, Infrastructure, Interfaces). The active exchange is selected via the `FUTURES_EXCHANGE` environment variable.

## Key Technologies
*   **Language:** Python 3.13+
*   **Dependency Management:** [uv](https://github.com/astral-sh/uv)
*   **Frameworks & Libraries:**
    *   **Bot:** `aiogram` (Telegram Bot API)
    *   **Database:** `sqlalchemy`, `aiosqlite` (Async SQLite), `alembic` (Migrations)
    *   **Scheduling:** `apscheduler`
    *   **DI:** `dependency-injector`
    *   **Configuration:** `pydantic-settings`
    *   **Exchange Integration:** `ccxt`
    *   **Analysis:** `ta` (Technical Analysis), `pandas`
    *   **Utilities:** `taskipy` (Task runner), `ruff` (Linter/Formatter)

## Architecture
The source code is located in `src/crypto_futures_bot` and follows a layered architecture:
*   **`domain/`**: Core business logic, value objects, and enums.
*   **`infrastructure/`**: Database access, external API adapters (MEXC, Bitget), and background services.
*   **`interfaces/`**: Entry points for interaction, primarily the Telegram bot handlers.
*   **`config/`**: Application configuration and Dependency Injection container setup.

## Setup & Configuration

### Environment Variables
Create a `.env` file (copy from `.env.example`) with the following key settings:
*   **Credentials:** `ROOT_USER`, `ROOT_PASSWORD`
*   **Telegram:** `TELEGRAM_BOT_TOKEN`, `TELEGRAM_BOT_ENABLED`
*   **Database:** `DATABASE_URL` (e.g., `sqlite+aiosqlite:///db.sqlite3`)
*   **Exchange Selection:** `FUTURES_EXCHANGE` (`MEXC` or `BITGET`)
*   **Exchange (MEXC):** `MEXC_API_KEY`, `MEXC_API_SECRET`, `MEXC_WEB_AUTH_TOKEN`
*   **Exchange (Bitget):** `BITGET_API_KEY`, `BITGET_API_SECRET`, `BITGET_API_PASSPHRASE`
*   **Bot Settings:** `JOB_INTERVAL_SECONDS`, `NOTIFY_ENTRY_SIGNALS`, `BACKGROUND_TASKS_ENABLED`

### Installation
1.  **Clone the repo.**
2.  **Install dependencies using `uv`:**
    ```bash
    uv sync
    ```

## Development Workflow

### Key Commands (via `taskipy`)
The project uses `taskipy` to define shortcuts in `pyproject.toml`:

*   **Start Application:**
    ```bash
    task start
    # Equivalent to: python -m crypto_futures_bot.main
    ```
*   **Run Tests:**
    ```bash
    task test
    # Equivalent to: pytest -x --log-cli-level=INFO
    ```
*   **Run Linter & Formatter:**
    ```bash
    task lint
    # Equivalent to: pre-commit install && pre-commit run --all
    ```
*   **Generate Database Migrations:**
    ```bash
    task make-migrations
    # Equivalent to: alembic revision --autogenerate
    ```

### Docker
To run with Docker Compose:
```bash
docker-compose up --build
```
This mounts the SQLite database volume locally to persist data.

## Directory Structure Highlights
*   `src/crypto_futures_bot/main.py`: Application entry point. Initializes DI container, migrations, and starts the bot/scheduler.
*   `src/crypto_futures_bot/config/configuration_properties.py`: Pydantic model defining all configuration options.
*   `tests/`: Pytest suite using `pytest-asyncio`.
    *   **`tests/infrastructure/tasks/`**: Integration tests for background tasks.
        *   *Key File:* `signals_task_service_test.py` - Tests `SignalsTaskService`, demonstrating how to mock external services (Exchange, Technical Analysis) and verify `AsyncIOEventEmitter` events for trading signals.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jsoladur) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
