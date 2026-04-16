## quantum-internet

> This project is a decentralized AI mesh agent system called **InfraFlow ZeroCloud**. It's designed with quantum-aware logic, real-time orchestration, and an offline-first architecture. The system integrates with both SambaNova and local Large Language Models (LLMs).

## Project Overview

This project is a decentralized AI mesh agent system called **InfraFlow ZeroCloud**. It's designed with quantum-aware logic, real-time orchestration, and an offline-first architecture. The system integrates with both SambaNova and local Large Language Models (LLMs).

The core of the project is a multi-agent system built with the `autogen` framework. It consists of three agents:

*   **Planner:** Creates step-by-step plans from high-level missions.
*   **Strategist:** Executes the technical and strategic tasks.
*   **Commander:** A user proxy agent that initiates and terminates the missions.

The system is exposed as a web service using the `fastapi` framework. A command-line interface (CLI) is provided to interact with the service. The project also includes a `docker-compose.yml` file for containerization, which sets up the main application and an `n8n` workflow automation tool.

## Building and Running

### Using Docker

The most straightforward way to run the project is by using Docker.

1.  **Create an `.env` file:**
    *   Copy the `.env.template` file to `.env`.
    *   Fill in the required environment variables, especially `SAMBANOVA_API_KEY` and `SAMBANOVA_BASE_URL`.

2.  **Run with Docker Compose:**
    ```bash
    docker-compose up -d
    ```

This will start the `agent-server` and the `n8n` service. The agent server will be available at `http://localhost:8000`.

### Running Locally

1.  **Install dependencies:**
    ```bash
    pip install -r requirements.txt
    ```

2.  **Set environment variables:**
    *   Create an `.env` file (as described above) or set the environment variables in your shell.

3.  **Run the server:**
    ```bash
    uvicorn mesh_server:app --host 0.0.0.0 --port 8000
    ```

### Interacting with the service

The `infra_agent.py` script provides a CLI to interact with the running service.

```bash
python infra_agent.py
```

## Development Conventions

*   **API:** The project uses the `fastapi` framework for building the API. The main application is in `mesh_server.py`.
*   **AI Agents:** The `autogen` library is used for the multi-agent system. The agent definitions and logic are in `mesh_server.py`.
*   **Dependencies:** Python dependencies are managed in the `requirements.txt` file.
*   **Containerization:** The project is containerized using Docker. The `dockerfile` and `docker-compose.yml` files are provided.
*   **Configuration:** The application is configured using environment variables. A `.env.template` file is provided as a template.

## Key Files

*   `mesh_server.py`: The main FastAPI application that contains the multi-agent system logic.
*   `infra_agent.py`: A CLI for interacting with the `mesh_server`.
*   `inferencer.py`: A simple client for the SambaNova API. The API key is passed as an argument to the `query` function.
*   `requirements.txt`: The Python dependencies.
*   `docker-compose.yml`: The Docker Compose configuration for running the project.
*   `dockerfile`: The Dockerfile for building the application image.
*   `agent_config.json`: Configuration file for the `infra_agent.py`.
*   `README.md`: The project's README file.

## TODO

*   **Add Tests:** There are no tests in the project. Adding unit and integration tests would improve the code quality and maintainability.
*   **Flesh out n8n integration:** The `docker-compose.yml` file includes an `n8n` service, but there is no information on how it is used. The integration should be documented.
*   **Quantum-aware logic:** The `README.md` mentions "quantum-aware logic", but there is no clear implementation of this in the code. This should be clarified and documented.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Influwealth) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
