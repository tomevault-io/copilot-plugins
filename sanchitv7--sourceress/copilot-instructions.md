## sourceress

> This document provides a comprehensive overview of the Sourceress project, its architecture, and development conventions.

# Sourceress Project Overview

This document provides a comprehensive overview of the Sourceress project, its architecture, and development conventions.

## Project Purpose

Sourceress is an AI-powered recruiting assistant that automates the process of sourcing candidates from LinkedIn. It takes a job description as input, finds relevant profiles, scores them, and generates outreach pitches. The goal is to streamline the top-of-funnel recruiting workflow and allow recruiters to focus on engaging with candidates.

## Architecture

The application is designed as a pipeline of agents, each responsible for a specific task in the sourcing process. The agents are orchestrated using a manual chaining of agents, with a plan to migrate to CrewAI for more robust workflow management.

The key components of the architecture are:

*   **Agents**: A set of specialized agents responsible for tasks like ingesting job descriptions, sourcing on LinkedIn, scoring candidates, and generating pitches.
*   **Workflows**: The `workflows.py` file defines the sequence of agent execution, either manually or through CrewAI.
*   **LinkedIn Scraping**: The `scraping.py` utility handles the interaction with LinkedIn, including authentication and profile extraction, using Selenium.
*   **LLM Integration**: The application uses a Large Language Model (LLM) via the `llm.py` utility to parse job descriptions and generate content.
*   **Data Models**: Pydantic models in `models.py` define the data structures used throughout the application.

## Building and Running

To build and run the project, follow these steps:

1.  **Install Dependencies**:
    ```bash
    pip install -e .[dev]
    ```

2.  **Run the Application**:
    ```bash
    python -m sourceress.main --jd-file <path_to_jd_file> --output <output_file_path>
    ```

3.  **Run Tests**:
    ```bash
    pytest
    ```

## Development Conventions

*   **Code Style**: The project uses `ruff`, `black`, and `isort` for code formatting and linting. These are enforced through pre-commit hooks.
*   **Type Hinting**: The codebase uses type hints, and `mypy` is used for static type checking.
*   **Logging**: The `loguru` library is used for logging.
*   **Modularity**: The application is well-structured, with clear separation of concerns between agents, utilities, and data models.
*   **Authentication**: LinkedIn authentication is handled by `linkedin_auth.py`, which saves and reuses session cookies.
*   **Error Handling**: The application includes error handling for LLM parsing failures and LinkedIn scraping issues.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sanchitv7)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sanchitv7)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
