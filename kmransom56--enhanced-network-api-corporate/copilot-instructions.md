## enhanced-network-api-corporate

> This is a Python-based project called "Enhanced Network API Builder - Corporate Edition". It's designed to help developers build network APIs in corporate environments, with a special focus on handling challenges like SSL interception, proxy restrictions, and air-gapped deployments.

# Project Overview

This is a Python-based project called "Enhanced Network API Builder - Corporate Edition". It's designed to help developers build network APIs in corporate environments, with a special focus on handling challenges like SSL interception, proxy restrictions, and air-gapped deployments.

The project uses FastAPI for the web framework and `uv` for Python package management. It also includes support for Fortinet FortiManager and Cisco Meraki APIs. The application is designed to be deployed using Docker Compose, with services for the main application, a PostgreSQL database, Redis for caching, and optional Nginx and Grafana containers.

## Building and Running

### Prerequisites

- Python 3.8+
- `uv` package manager
- Docker and Docker Compose

### Running the Application

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/your-username/enhanced-network-api-corporate.git
    cd enhanced-network-api-corporate
    ```

2.  **Install dependencies:**
    ```bash
    python3 -m venv .venv
    source .venv/bin/activate
    pip install uv
    uv pip install -r requirements.txt uvicorn
    ```

3.  **Auto-configure for corporate environment:**
    ```bash
    python -m enhanced_network_api.corporate_environment_detector --detect --auto-configure
    ```

4.  **Start the FastAPI server:**
    ```bash
    uvicorn src.enhanced_network_api.platform_web_api_fastapi:app --host 0.0.0.0 --port 8001
    ```

### Running with Docker Compose

1.  **Set up environment variables:**
    Create a `corporate.env` file with the necessary environment variables, such as `DB_PASSWORD` and `GRAFANA_PASSWORD`.

2.  **Start the services:**
    ```bash
    docker-compose -f docker-compose.corporate.yml up -d
    ```

## Development Conventions

- **Formatting:** The project uses `black` for code formatting.
- **Linting:** `ruff` is used for linting to enforce code quality.
- **Type Checking:** `mypy` is used for static type checking.
- **Testing:** The project uses `pytest` for testing. Tests are located in the `tests/` directory.
- **Pre-commit Hooks:** The project uses pre-commit hooks to automatically run formatting and linting before committing code.

### Running Tests

To run the test suite, use the following command:
```bash
pytest tests/
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kmransom56)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kmransom56)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
