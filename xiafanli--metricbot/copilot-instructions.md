## metricbot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MetricBot is an intelligent operations monitoring platform powered by large language models. It consists of a Python FastAPI backend and a Vue 3 TypeScript frontend.

## Architecture

### Backend ([`backend/`](backend/))
- **Framework**: FastAPI with Uvicorn ASGI server
- **Structure**: Modular design with [`apps/`](backend/apps/) for application logic and [`common/`](backend/common/) for shared utilities
- **Entry Point**: [`main.py`](backend/main.py) creates the FastAPI app with `/api/v1` prefix and CORS middleware
- **Environment**: Uses Python 3.11+ with virtual environment at [`backend/venv/`](backend/venv/)
- **Dependencies**: Managed via [`pyproject.toml`](backend/pyproject.toml) with key packages: FastAPI, Uvicorn, Pydantic, httpx, python-dotenv

### Frontend ([`frontend/`](frontend/))
- **Framework**: Vue 3 with TypeScript and Vite build tool
- **UI Library**: Element Plus with icons
- **State Management**: Pinia
- **Routing**: Vue Router with routes at [`src/router/`](frontend/src/router/)
- **API Client**: Axios instance at [`src/api/index.ts`](frontend/src/api/index.ts) configured for `/api/v1` base URL
- **Development Proxy**: Vite proxies `/api` requests to `http://localhost:8000`

## Development Commands

### Backend
```bash
cd backend
# Install dependencies using virtual environment
pip install -e .
# Install new packages using Aliyun mirror
pip install -i https://mirrors.aliyun.com/pypi/simple/ <package>
# Start development server
python main.py
```
Backend runs on `http://localhost:8000` with API docs at:
- Swagger UI: `http://localhost:8000/docs`
- ReDoc: `http://localhost:8000/redoc`

### Frontend
```bash
cd frontend
# Install dependencies
npm install
# Start development server
npm run dev
# Build for production
npm run build
# Lint code
npm run lint
```
Frontend runs on `http://localhost:5173`

### Docker
```bash
# Build and start all services
docker-compose up --build
# Stop services
docker-compose down
```

## Key Configuration Files

- **Backend Config**: [`backend/.env.example`](backend/.env.example) - template for environment variables (PROMETHEUS_URL, HOST, PORT)
- **Frontend Proxy**: [`frontend/vite.config.ts`](frontend/vite.config.ts) - API proxy configuration
- **Docker Setup**: [`docker-compose.yaml`](docker-compose.yaml) - orchestrates both services

## Working Conventions

### Backend Development
- Use the virtual environment at `backend/venv/`
- Install Python packages from Aliyun mirror when adding dependencies
- Follow FastAPI patterns with async/await where appropriate
- Pydantic models for data validation
- Modular structure: new features go in `backend/apps/` with appropriate subdirectories

### Frontend Development
- Vue 3 Composition API with TypeScript
- Element Plus components for UI
- API calls go through the centralized Axios client in `src/api/index.ts`
- New views go in `src/views/` and register in `src/router/`
- Use `@` alias for `src/` directory imports

### Important Notes
- The project includes a separate `SQLBot/` directory - this is a separate project, not part of the main MetricBot application
- The `.trae/rules/project_rules.md` contains additional development guidelines, including a requirement to respond in Chinese and be critically analytical
- Frontend API proxy is essential for development - it forwards `/api` requests to the backend
- CORS is configured to allow all origins in development (adjust for production)

---
> Source: [xiafanli/MetricBot](https://github.com/xiafanli/MetricBot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
