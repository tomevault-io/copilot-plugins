## taco-bell-drive-thru

> This project is a production-ready, AI-powered drive-thru ordering system for Taco Bell. It uses a voice-based interface to allow customers to place orders. The system is built with Python and FastAPI, and it leverages several Google Cloud services for speech recognition and synthesis, as well as OpenRouter for LLM-based order processing. The application is designed to be deployed using Docker and Kubernetes.

# Taco Bell Drive-Thru Voice AI

## Project Overview

This project is a production-ready, AI-powered drive-thru ordering system for Taco Bell. It uses a voice-based interface to allow customers to place orders. The system is built with Python and FastAPI, and it leverages several Google Cloud services for speech recognition and synthesis, as well as OpenRouter for LLM-based order processing. The application is designed to be deployed using Docker and Kubernetes.

**Key Technologies:**

*   **Backend:** Python, FastAPI
*   **AI Services:**
    *   Google Cloud Speech-to-Text (ASR)
    *   Google Cloud Text-to-Speech (TTS)
    *   OpenRouter (LLM)
*   **Database:** Supabase (PostgreSQL)
*   **Caching:** Redis
*   **Deployment:** Docker, Kubernetes
*   **Monitoring:** Prometheus, Grafana

**Architecture:**

The system follows a microservices-based architecture. The main components are:

*   **TacoBot API:** A FastAPI application that provides the main REST API for the system.
*   **PostgreSQL Database:** Stores order information and other application data.
*   **Redis Cache:** Used for caching frequently accessed data.
*   **Nginx Load Balancer:** Distributes traffic to the API servers.
*   **Prometheus:** Collects metrics from the various services.
*   **Grafana:** Provides dashboards for visualizing metrics.

## Building and Running

The project uses a `Makefile` to automate common development tasks.

**Prerequisites:**

*   Python 3.11+
*   Docker
*   Docker Compose

**Development Setup:**

1.  **Clone the repository:**
    ```bash
    git clone <repository-url>
    cd taco-bell-drivethru
    ```
2.  **Set up the development environment:**
    ```bash
    make setup-dev
    ```
    This command will:
    *   Create a Python virtual environment.
    *   Install all necessary dependencies.
    *   Create a `.env` file from the `.env.example` template.
3.  **Configure the environment:**
    *   Edit the `.env` file to add your API keys and other configuration settings.
4.  **Run the application:**
    *   **Locally:**
        ```bash
        make run
        ```
    *   **With Docker:**
        ```bash
        make run-docker
        ```

**Testing:**

*   **Run all tests:**
    ```bash
    make test
    ```
*   **Run unit tests:**
    ```bash
    make test-unit
    ```
*   **Run integration tests:**
    ```bash
    make test-integration
    ```

## Development Conventions

*   **Code Style:** The project uses `black` for code formatting and `isort` for import sorting.
*   **Linting:** `flake8` is used for linting the code.
*   **Type Checking:** `mypy` is used for static type checking.
*   **Pre-commit Hooks:** The project uses pre-commit hooks to automatically run linters and formatters before each commit.
*   **Testing:** The project uses `pytest` for testing. The tests are organized into `unit`, `integration`, and `e2e` directories.
*   **Documentation:** The project uses `mkdocs` for generating documentation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/404kidwiz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/404kidwiz)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
