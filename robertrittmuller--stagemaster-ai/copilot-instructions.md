## stagemaster-ai

> This document summarizes the recommended tech stack and deployment approach for this project.

# AGENTS — Tech Stack Overview

This document summarizes the recommended tech stack and deployment approach for this project.

## Overview
- **Backend:** FastAPI (Python)
- **Frontend:** React (recommended: Vite) with Tailwind CSS
- **Deployment:** Containerized (Docker) with optional `docker-compose` for local dev and multi-stage builds for production. Kubernetes or cloud-native services recommended for production scale.

## Backend — FastAPI
- Purpose: REST + WebSocket API server, high-performance Python async framework.
- Minimum recommendations: Python 3.10+, `fastapi`, `uvicorn[standard]`, `pydantic`, ORM (SQLAlchemy or Tortoise), migration tool (Alembic or Aerich).
- Run locally (via Docker):

```bash
# from project root
docker-compose up --build
# Or for development with hot reload:
docker-compose up backend
```

### Backend configuration (example)

Create a `.env.backend` (or `.env.example`) for core settings:

```env
DATABASE_URL=postgresql+psycopg://postgres:example@db:5432/app_db
CORS_ORIGINS=http://localhost:3000
SECRET_KEY=change-me
LOG_LEVEL=info
```

### Example backend Dockerfile (production-ready, multi-stage)

```dockerfile
FROM python:3.12-slim as base
WORKDIR /app
ENV PYTHONUNBUFFERED=1

FROM base as deps
RUN pip install --upgrade pip
COPY requirements.txt /app/
RUN pip install --no-cache-dir -r requirements.txt

FROM base as runner
COPY --from=deps /usr/local/lib/python3.12/site-packages /usr/local/lib/python3.12/site-packages
COPY . /app
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## Frontend — React + Tailwind
- Purpose: Single-page app (SPA) using React and Tailwind for utility-first styling.
- Recommended toolchain: Vite for fast dev server and small production bundles.
- Run locally (via Docker):

```bash
# dev (hot reload)
docker-compose up frontend
# build
docker-compose build frontend
```

### Example frontend Dockerfile (multi-stage)

```dockerfile
FROM node:18-alpine as build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:stable-alpine as runner
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

## Containerized Deployment
- Use separate containers for backend and frontend.
- Use environment variables and secrets for credentials and config.
- For local development, `docker-compose.yml` can start both services and a database.

### Minimal `docker-compose.yml` example

```yaml
version: '3.8'
services:
  backend:
    build: ./backend
    ports:
      - "8000:8000"
    env_file: .env.backend
    depends_on:
      db:
        condition: service_healthy

  frontend:
    build: ./frontend
    ports:
      - "3000:80"

  db:
    image: postgres:15
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: example
      POSTGRES_DB: app_db
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

## Suggested repo layout

```
/
├─ backend/        # FastAPI app, Dockerfile, requirements
├─ frontend/       # React + Tailwind app, Dockerfile
├─ docker-compose.yml
├─ AGENTS.md       # this file
```

## Production notes
- Use multi-stage Docker builds to keep images small.
- Put a reverse proxy (NGINX or Traefik) in front of the services for TLS and routing.
- Consider a process manager (Gunicorn with Uvicorn workers) or directly run Uvicorn with a process supervisor.
- For scaling, use Kubernetes or a managed container service (ECS, GKE, AKS). Use deployments, autoscaling, and managed DB services.

## LLM Integration — liteLLM

- **Primary LLM access:** Use `liteLLM` as the unified client for all LLM access across backend services. `liteLLM` provides a lightweight, local-first API surface and can proxy to remote providers such as OpenRouter when needed.
- **Why:** Simplifies provider-switching, local caching, and consistent configuration across services.

### Example usage (Python)

```python
import os
from litellm import Client

client = Client(provider=os.getenv('LITELLM_PROVIDER', 'openrouter'),
                api_key=os.getenv('OPENROUTER_API_KEY'))

resp = client.generate("Hello from liteLLM + OpenRouter")
print(resp.text)
```

### Example `.env` (OpenRouter)

Create an `.env` (or `.env.example`) containing the OpenRouter key and liteLLM settings:

```env
# OpenRouter API key (create at https://openrouter.ai)
OPENROUTER_API_KEY=sk-xxxxx-your-openrouter-key

# Provider used by liteLLM (default: openrouter)
LITELLM_PROVIDER=openrouter

# Optional: default model to use via OpenRouter
LITELLM_DEFAULT_MODEL=gpt-4o-mini

# Optional: local cache dir for liteLLM
LITELLM_CACHE_DIR=.litellm_cache
```

Notes:
- Set `OPENROUTER_API_KEY` in your deployment environment or `env_file` used by Docker Compose / Kubernetes secrets.
- `liteLLM` supports switching providers without changing application code — only the env vars and provider config need updating.

- Place all generated development plans in the `/plans` folder at the repository root. Track plans as Markdown files with a short metadata header (title, author, date, status) and a link to related issues or tickets.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/robertrittmuller) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
