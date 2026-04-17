## egx-stock-api

> This project is a Python-based web application that provides a simple API for fetching the last closing stock prices from the Egyptian Exchange (EGX).

# Gemini Project Context: EGX Stock API

## Project Overview

This project is a Python-based web application that provides a simple API for fetching the last closing stock prices from the Egyptian Exchange (EGX).

**Core Technologies:**
- **Backend:** Python / Flask
- **Data Source:** `yfinance` library (which scrapes Yahoo Finance)
- **Caching:** `cachetools` for in-memory, time-based caching
- **Deployment:** Docker, with a `docker-compose.yml` for local execution and a GitHub Actions workflow for automated builds and pushes to Docker Hub.

**Architecture:**
The application is a single-file Flask app (`app/app.py`) that exposes two main endpoints:
- `/`: Serves an HTML page (`index.html`) with API documentation and usage examples.
- `/EGX/<ticker>`: The main API endpoint that returns the stock price for a given ticker.

A key architectural feature is the caching and lookup logic. For each request to `/EGX/<ticker>`:
1.  The result is stored in a time-based cache (`TTLCache`) for 5 minutes to minimize API calls to the data source.
2.  The application first attempts to find the stock by appending the `.CA` suffix (e.g., `ORWE.CA`).
3.  If that fails, it uses the `yfinance.Search` functionality as a fallback to find the correct ticker symbol on the Cairo Stock Exchange ('CAI').
4.  The API returns the latest closing price as a plain string.

## Building and Running

There are two primary ways to run this project: using Docker (recommended) or running it as a local Python script.

### Using Docker (Recommended)

This method uses `docker-compose` to build the image and run the container.

1.  **Build and Run the Container:**
    ```bash
    docker-compose up --build
    ```
2.  **Access the Application:**
    The API will be available at `http://localhost:5000`.

3.  **(Optional) Use a Custom Port:**
    You can set the `EXTERNAL_PORT` environment variable to run on a different port.
    ```bash
    export EXTERNAL_PORT=8080
    docker-compose up --build
    ```
    The API will then be available at `http://localhost:8080`.


### Local Python Environment

1.  **Install Dependencies:**
    Make sure you are in a virtual environment.
    ```bash
    pip install -r requirements.txt
    ```

2.  **Run the Application:**
    ```bash
    python app/app.py
    ```

3.  **Access the Application:**
    The API will be available at `http://localhost:5000`.

## Development Conventions

*   **Ticker Resolution:** The logic for finding stock data is centralized in the `get_price_for_ticker` function. It employs a two-step process (direct lookup with `.CA` suffix, followed by a `yf.Search` fallback) to ensure reliability.
*   **Caching:** All data-fetching operations are cached for 5 minutes to improve performance and reduce the load on the `yfinance` service. Any new data-fetching logic should be added within the cached `get_price_for_ticker` function.
*   **CI/CD:** The `.github/workflows/deploy_docker.yml` workflow is configured to automatically build and push a Docker image to Docker Hub whenever a new Git tag matching the pattern `v*.*.*` is created. This process requires `DOCKER_USERNAME` and `DOCKER_PASSWORD` to be set as secrets in the GitHub repository.
*   **Testing:** There are currently no automated tests in the project.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rgf2004) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
