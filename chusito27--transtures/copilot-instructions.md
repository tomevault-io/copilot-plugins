## transtures

> This repository contains two main projects: a Python-based multi-agent coding assistant and a React-based landing page.

# Transtures Monorepo

This repository contains two main projects: a Python-based multi-agent coding assistant and a React-based landing page.

## `transtures-landing`

This project is a React-based landing page for the Transtures application.

### Project Overview

- **Technology:** React
- **Purpose:** To provide a web presence for the Transtures project.

### Building and Running

To run the landing page locally, navigate to the `transtures-landing` directory and run the following commands:

- `npm install` to install dependencies.
- `npm start` to start the development server.
- `npm test` to run tests.
- `npm run build` to build the application for production.

## `Transtures`

This project is a Python-based multi-agent coding assistant. It uses a simple HTTP server to expose an API for handling commands.

### Project Overview

- **Technology:** Python
- **Architecture:** The system consists of a `main.py` file that starts an HTTP server. The server handles POST requests to the `/command` endpoint, which are then passed to an `OrchestratorAgent`. The orchestrator, in turn, delegates tasks to other agents, such as the `FileSystemAgent` and `CodeGenerationAgent`.
- **Purpose:** To provide a multi-agent system for code generation and file system operations.

### Building and Running

To run the Python application, navigate to the `Transtures` directory and run the following command:

- `python src/main.py`

This will start the server on port 8000. You can interact with the application by sending POST requests to `http://localhost:8000/command`. The application also serves a simple UI at `http://localhost:8000/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chusito27)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/chusito27)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
