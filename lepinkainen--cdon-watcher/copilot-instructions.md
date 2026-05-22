## cdon-watcher

> This project is a Python-based price tracking system for Blu-ray and 4K Blu-ray movies on CDON.fi. It uses a hybrid scraping approach, combining the power of Playwright for dynamic, JavaScript-heavy listing pages with the efficiency of `requests` and `BeautifulSoup` for parsing individual product pages.

# Gemini Code Assistant Context

## Project Overview

This project is a Python-based price tracking system for Blu-ray and 4K Blu-ray movies on CDON.fi. It uses a hybrid scraping approach, combining the power of Playwright for dynamic, JavaScript-heavy listing pages with the efficiency of `requests` and `BeautifulSoup` for parsing individual product pages.

The system is designed to be deployed with Docker, providing a web dashboard to monitor prices, a background service to check for price drops, and a one-time crawler to populate the database.

## Key Technologies

- **Backend:** Python 3.11, FastAPI, SQLModel
- **Scraping:** Playwright, Requests, BeautifulSoup
- **Database:** SQLite
- **Deployment:** Docker, Docker Compose (or Podman, podman-compose)
- **Dependency Management:** uv
- **Linting/Formatting:** ruff, mypy

## Architecture

The application utilizes a hybrid scraping architecture to efficiently gather data.

- **`listing_crawler.py`**: A Playwright-based crawler responsible for navigating category pages that require JavaScript execution to load and render product listings. It extracts product URLs.
- **`product_parser.py`**: A lightweight and efficient parser using `requests` and `BeautifulSoup`. It takes a product URL, fetches the static HTML, and parses it to extract details like title, price, and movie format. This avoids the overhead of a full browser for simple pages.
- **`cdon_scraper.py`**: The orchestrator that combines the two components above. It gets URLs from the `listing_crawler` and passes them to the `product_parser` for processing, then stores the results in the database.
- **`monitor.py`**: The main entry point for the application, which can be run in three modes:
  - `web`: Starts the FastAPI web server for the user-facing dashboard.
  - `monitor`: Runs the background price monitoring service.
  - `crawl`: Executes the one-time crawl to populate the database.

### Service Architecture (Docker)

The `docker-compose.yml` defines three main services:

- **`web`**: Runs the FastAPI web application, serving the dashboard on port 8080.
- **`monitor`**: Runs the background price checking service, which periodically checks for updates.
- **`crawler`**: A one-time service profile for running the main scraper (`cdon_scraper.py`) to populate the database.

### Data Flow

1. **Crawl**: The `crawler` service is run manually. `listing_crawler.py` (Playwright) scrapes category pages for product URLs.
2. **Parse**: For each URL, `product_parser.py` (Requests + BeautifulSoup) fetches and parses the product page to extract movie details.
3. **Store**: The extracted data is saved to the SQLite database (`data/cdon_movies.db`).
4. **Monitor**: The `monitor` service runs in the background, periodically checking prices of items in the database and sending alerts for price drops.
5. **View**: The `web` service provides a FastAPI-based dashboard to view the collected data, watchlist, and alerts.

## Development Workflows

This project uses `uv` for managing dependencies and running scripts.

### Environment Setup

1. **Install dependencies:**

   ```bash
   uv sync --extra test
   ```

2. **Install Playwright browser:**
   ```bash
   uv run playwright install chromium
   ```

### Essential Commands

- **Run Tests:**
  - Run all tests: `uv run pytest tests/`
  - Run unit tests: `uv run pytest tests/unit/`
  - Run integration tests: `uv run pytest tests/integration/`

- **Linting and Formatting:**
  - Check for linting errors: `uv run ruff check .`
  - Format code: `uv run ruff format .`
  - Run static type checking: `uv run mypy src`

- **Running the Application (Docker/Podman):**
  - **Build the image:** `./scripts/build.sh`
  - **Run in development (macOS/Podman):** `./scripts/run-dev.sh`
  - **Run in production (Linux/Docker):** `./scripts/run-prod.sh`
  - **Run the crawler to populate the database:** `podman-compose run --rm crawler` (or `docker-compose run --rm crawler`)
  - **View logs:** `podman-compose logs -f`

## Project Conventions

- **Dependency Management:** All Python dependencies are managed in `pyproject.toml`. Use `uv sync` to install them.
- **Configuration:** Application configuration is handled via environment variables, documented in `.env.example`. The database path is set with `DB_PATH`.
- **Database:** The SQLite database is located at `data/cdon_movies.db` and is persisted via a Docker volume.
- **Source Code:** All main application code resides in the `src/cdon_watcher` directory.
- **AI Guidelines:** This project includes a `llm-shared` submodule. Refer to the `README.md` and other files in that directory for general AI development conventions.

---
> Source: [lepinkainen/cdon-watcher](https://github.com/lepinkainen/cdon-watcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
