## task-risk

> **task-risk** is an AI automation risk assessment service that analyzes occupations, tasks, and skills using data from O*NET and the Bureau of Labor Statistics (BLS). It aims to provide actionable insights into how AI technologies (automation, augmentation, etc.) impact various job roles.

# Project Context: task-risk

## Overview
**task-risk** is an AI automation risk assessment service that analyzes occupations, tasks, and skills using data from O*NET and the Bureau of Labor Statistics (BLS). It aims to provide actionable insights into how AI technologies (automation, augmentation, etc.) impact various job roles.

The project is structured as a full-stack application with a Python-based backend (ETL, API, Workers) and a Next.js frontend.

## Architecture

### Backend (Python/FastAPI)
-   **API**: `api/` contains the FastAPI application serving occupation and task data.
-   **ETL**: `batch/` contains scripts for importing and processing O*NET and BLS data.
-   **Workers**: `worker/` handles background tasks, primarily LLM-based scoring of tasks for AI risk.
-   **Database**: PostgreSQL 18 is used as the primary data store.
-   **Configuration**: managed via `.env` files.

### Frontend (Next.js/React)
-   Located in `frontend/`.
-   Built with Next.js (App Router), React, and Tailwind CSS.
-   Consumes the backend API to display occupation details, task lists, and risk scores.

### Infrastructure
-   **Deployment**: Targeted for Ubuntu 24.04 LTS.
-   **Service Management**: Uses Systemd for the API service (`deploy/systemd/`).
-   **Web Server**: Nginx as a reverse proxy (`deploy/nginx/`).
-   **Scheduling**: Cron jobs for periodic updates (`deploy/cron/`).

## Key Workflows & Commands

### Prerequisites
-   Python 3.10+
-   Node.js & npm
-   PostgreSQL 18

### Backend Setup
1.  **Virtual Environment**:
    ```bash
    python3 -m venv .venv
    source .venv/bin/activate
    pip install -r requirements.txt
    ```
2.  **Environment Variables**:
    ```bash
    cp .env.example .env
    # Edit .env to set DATABASE_URL, ONET_DATA_VERSION, etc.
    ```
3.  **Database Initialization**:
    ```bash
    ./scripts/apply_schema.sh
    ```

### Data Pipeline (ETL & Scoring)
The project relies on specific data versions (e.g., `30.1`). Common commands include:

-   **Import O*NET Data**:
    ```bash
    python -m batch.import_onet --sql-dir ./Data/db_30_1_mysql --data-version 30.1
    ```
-   **Set Active Version**:
    ```bash
    python -m batch.set_active_version --data-version 30.1
    ```
-   **Run LLM Scoring** (Mock or Real):
    ```bash
    python -m worker.score_tasks --limit 20 --data-version 30.1
    ```
-   **Aggregate Scores**:
    ```bash
    python -m worker.aggregate_occupations --data-version 30.1
    ```

### Running the API
```bash
uvicorn api.main:app --reload
```
API is typically served at `http://localhost:8000`.

### Frontend Setup & Run
1.  **Install Dependencies**:
    ```bash
    cd frontend
    npm install
    ```
2.  **Run Development Server**:
    ```bash
    npm run dev
    ```
Frontend is served at `http://localhost:3000`.

## Development Conventions

-   **Data Versioning**: All data tables and operations are scoped by `data_version` (e.g., '30.1'). Always specify the version when running batch scripts.
-   **Idempotency**: ETL scripts are designed to be idempotent (truncate & reload).
-   **Mocking**: LLM scoring defaults to `USE_MOCK_LLM=1` in `.env` to avoid API costs during development. Set to `0` for real provider calls.
-   **Code Style**:
    -   Python: Follow PEP 8.
    -   Frontend: Standard React/Next.js patterns, utilizing Tailwind CSS for styling.
-   **Directory Structure**:
    -   `api/`: REST API endpoints.
    -   `batch/`: Data import and processing scripts.
    -   `worker/`: Async tasks and AI scoring logic.
    -   `frontend/`: Web UI.
    -   `Data/`: O*NET source data documentation and SQL snapshots.
    -   `deploy/`: Server configuration files.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brdg-kr)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/brdg-kr)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
