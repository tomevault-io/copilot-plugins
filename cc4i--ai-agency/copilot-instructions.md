## ai-agency

> This document provides a comprehensive overview of the AI Agency project, its architecture, and development practices to be used for context in future Gemini conversations.

# Gemini Workspace Context: AI Agency

This document provides a comprehensive overview of the AI Agency project, its architecture, and development practices to be used for context in future Gemini conversations.

## 1. Project Overview

The "AI Agency" is a multi-agent system designed to simulate a creative agency. A user acts as a Creative Director, interacting with an "Executive Producer" (powered by Gemini Live) via a conversational voice interface. The Executive Producer then coordinates a team of specialist AI agents to generate a complete creative campaign, including strategy, visual assets, video, audio, and a web presence.

The system is built with a product-agnostic design, capable of handling any product category.

### Core Technologies

-   **Backend**: Python (3.13+) with FastAPI, using `uv` for package management.
-   **Frontend**: Next.js 14+ (App Router) with React 18 and TypeScript.
-   **AI/ML**: Google AI APIs are used extensively:
    -   **Gemini Pro & Live**: For conversational interaction and strategic planning.
    -   **Imagen**: For art and image generation.
    -   **Veo**: For video production.
    -   **Lyria & Chirp**: For music and audio generation.
    -   **Gemini Code Assist**: For web development.
-   **Infrastructure**: Redis for state management and as an event bus (Pub/Sub), and Celery for asynchronous task execution.

## 2. Architecture

The project is a monorepo with two main components: `frontend` and `backend`.

-   **`backend/`**: A Python FastAPI application that serves the core business logic.
    -   `app/agents/`: Contains the logic for each specialist AI agent (Art Director, Strategist, etc.). The agents follow an "execute-critique-revise" loop to ensure quality.
    -   `app/producer/`: Holds the logic for the Executive Producer, including planning and critique.
    -   `app/services/`: Core services like Redis client, Google AI API clients, agent orchestration, and the event bus.
    -   `app/models/`: Pydantic models for data structures like the Project Brief and Assets.
    -   `main.py`: The FastAPI application entry point.
-   **`frontend/`**: A Next.js application that provides the user interface.
    -   `src/app/`: The main application code using the Next.js App Router.
    -   `src/components/`: Reusable React components for the UI (e.g., `MicrophoneInterface`, `AssetDisplay`).
    -   `src/hooks/`: Custom React hooks for managing WebSockets and microphone input.
    -   `src/lib/stores/`: Zustand stores for global state management.

## 3. Building and Running the System

Follow these steps to get the full application running locally.

### Prerequisites

-   Python 3.13+
-   Node.js 18+
-   Docker (for running Redis)
-   A configured Google Cloud account with the necessary AI APIs enabled.

### Full System Startup

1.  **Start Redis:**
    ```bash
    docker run -d -p 6379:6379 redis:latest --appendonly yes
    ```

2.  **Set up and Run Backend:**
    ```bash
    cd backend

    # Install uv if you haven't already
    curl -LsSf https://astral.sh/uv/install.sh | sh

    # Create virtual environment and install dependencies
    uv venv --python 3.13
    source .venv/bin/activate
    uv sync --dev

    # Configure environment variables
    cp .env.example .env
    # IMPORTANT: Edit .env with your Google Cloud credentials and API keys

    # Run the FastAPI server
    uv run uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
    ```

3.  **Set up and Run Frontend:**
    ```bash
    # In a new terminal
    cd frontend

    # Install dependencies
    npm install

    # Run the development server
    npm run dev
    ```

4.  **Access the Application:**
    -   Open your browser to `http://localhost:3000`.
    -   Click the microphone button to begin the voice session.

## 4. Development Conventions

### Backend (`python`)

-   **Package Management**: Use `uv` for all dependency management.
    -   Add a new dependency: `uv add <package-name>`
    -   Install from lock file: `uv sync`
-   **Testing**: Use `pytest`. Run tests with `uv run pytest -v`.
-   **Formatting**: Use `black`. Format code with `uv run black .`.
-   **Linting**: Use `ruff`. Lint with `uv run ruff check . --fix`.
-   **Type Checking**: Use `mypy`. Check types with `uv run mypy app/`.

### Frontend (`typescript`)

-   **Package Management**: Use `npm`.
-   **Testing**: Use `npm test`.
-   **Linting**: Use `npm run lint`.
-   **Development Server**: Run with `npm run dev`.
-   **Production Build**: Create a build with `npm run build`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cc4i) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
