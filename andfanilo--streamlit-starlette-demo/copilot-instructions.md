## streamlit-starlette-demo

> This workspace is dedicated to building and experimenting with the integrated Streamlit-Starlette framework (available from Streamlit 1.53+). The goal is to leverage Streamlit for the UI while using Starlette/FastAPI for robust backend features like custom HTTP endpoints, middleware, and advanced security headers.

# Project Context: Streamlit + Starlette

## Overview

This workspace is dedicated to building and experimenting with the integrated Streamlit-Starlette framework (available from Streamlit 1.53+). The goal is to leverage Streamlit for the UI while using Starlette/FastAPI for robust backend features like custom HTTP endpoints, middleware, and advanced security headers.

## Tech Stack

- **Dashboard/UI**: Streamlit (`streamlit.starlette.App`)
- **Backend/Routing**: Starlette or FastAPI
- **Process Management**: Integrated ASGI (uvicorn)

## Core Constraint

**NEVER** suggest running Streamlit and FastAPI/Starlette in separate processes (e.g., using `subprocess`, separate terminals, or Docker Compose with two services). Always run them in a **single unified process** using the `streamlit.starlette.App` wrapper.

## Thinking Process

Before generating any code involving this integration, determine:

1.  **Is Streamlit the Parent or Child?**
    - **Parent (Root at `/`)**: Streamlit serves the main UI, FastAPI is mounted at a sub-path like `/api`.
    - **Child (Mounted at `/dashboard`)**: FastAPI is the main application, Streamlit is a sub-app for visualization.
2.  **Is Shared State Required?**
    - If yes, define a global singleton, use `app.state`, or leverage the `lifespan` context manager.

## Development Workflow

- Entrypoint is an ASGI-compatible file (e.g., `app.py`) that initializes `streamlit.starlette.App`.
- Run locally using: `uv run uvicorn app:app --reload`

## Configuration

Ensure `.streamlit/config.toml` contains:

```toml
[server]
headless = true
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andfanilo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
