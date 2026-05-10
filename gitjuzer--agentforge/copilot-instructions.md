## agentforge

> Adaptive multi-model LLM agent system for software engineering project management with task management, testing, debugging, memory management, skill measurement, web UI, and human-in-the-loop collaboration.

# AgentForge - SOTA LLM Agent for SWE Project Management

## Project Overview
Adaptive multi-model LLM agent system for software engineering project management with task management, testing, debugging, memory management, skill measurement, web UI, and human-in-the-loop collaboration.

## Architecture
- **Backend**: Python 3.12+ / FastAPI / SQLAlchemy / ChromaDB
- **Frontend**: React 18 / TypeScript / TailwindCSS / Zustand
- **Database**: PostgreSQL (relational) + ChromaDB (vectors)
- **LLM**: Anthropic Claude API (Opus/Sonnet/Haiku) with adaptive routing

## Key Directories
- `backend/app/` - FastAPI application
  - `router/` - Adaptive model routing (complexity scoring, skill-based selection)
  - `skills/` - Skill detection, benchmarking, calibration
  - `memory/` - Memory management (short-term, long-term, episodic, semantic)
  - `services/` - Business logic (task service, human gateway)
  - `api/` - REST/WebSocket endpoints
  - `models/` - SQLAlchemy models
  - `schemas/` - Pydantic schemas
- `frontend/src/` - React application
  - `pages/` - Dashboard, TaskBoard, Chat, Metrics, Settings
  - `components/` - Reusable UI components
  - `stores/` - Zustand state management

## Development Practices
- TDD: Write tests first
- Agile: Sprint-based with Kanban board
- Adaptive model usage: Haiku for boilerplate, Sonnet for standard dev, Opus for architecture
- All code must have type hints (Python) / TypeScript types (frontend)

## Commands
- Backend: `cd backend && uvicorn app.main:app --reload`
- Frontend: `cd frontend && npm run dev`
- Tests: `cd backend && pytest` / `cd frontend && npm test`
- Docker: `docker-compose up`

---
> Source: [gitjuzer/agentforge](https://github.com/gitjuzer/agentforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
