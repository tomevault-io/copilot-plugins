## auto-annotation-tool

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Full-stack image annotation tool with ML-powered auto-annotation. Python/FastAPI backend, SvelteKit frontend, PostgreSQL, MinIO object storage, Docker Compose orchestration.

## Build & Run

```bash
# Full stack (Docker Compose)
docker-compose up --build
# Frontend: localhost:3000, Backend: localhost:8000, Nginx: localhost:80, MinIO Console: localhost:9001

# Backend only
cd backend
pip install -e ".[gpu]"   # or .[dev] for dev dependencies
alembic upgrade head       # run migrations
uvicorn app.main:app --reload

# Frontend only
cd frontend
npm install
npm run dev                # dev server with HMR
npm run build              # production build
npm run check              # svelte-check type checking
```

## Database Migrations

```bash
cd backend
alembic upgrade head       # apply migrations
alembic revision --autogenerate -m "description"  # create new migration
```

## Architecture

**Backend** (`backend/app/`):
- `api/` — FastAPI routers. All routes prefixed `/api/v1`. Key: auth, images, annotations, jobs, exports, projects, workspaces
- `models/` — SQLAlchemy async ORM models (User, Workspace, Project, Image, Annotation, AnnotationJob, ApiKey)
- `schemas/` — Pydantic request/response schemas
- `services/` — Business logic (MinIO storage client)
- `ml/` — ML inference pipelines: detection (SAM3), segmentation (SAM3), OCR (PaddleOCR), classification (Qwen2.5-VL with 4-bit quantization)
- `workers/` — Background annotation job processor
- `config.py` — Settings from env vars
- `database.py` — Async SQLAlchemy session factory

**Frontend** (`frontend/src/`):
- SvelteKit 2 with Svelte 5 runes mode, Tailwind CSS 4
- `lib/api/client.ts` — API client with JWT auth (access + refresh tokens in localStorage)
- `lib/stores/` — Svelte runes-based stores (auth state)
- `routes/` — File-based routing: auth, workspaces, projects, annotation canvas, export, settings

**Key patterns:**
- `ModelManager` singleton manages GPU VRAM budget — loads/evicts models dynamically based on CUDA memory
- Annotation jobs run as FastAPI background tasks with progress tracking (processed count, annotations created)
- Four annotation types: detection, segmentation, OCR, classification — each with its own ML pipeline
- Images stored in MinIO `images/` bucket, masks in `masks/`, exports in `exports/`
- Export formats: COCO JSON, YOLO, VOC XML
- Multi-tenant via Workspaces with member roles
- Rate limiting via slowapi on auth endpoints

## Environment Variables

See `.env.example`. Key vars: `POSTGRES_PASSWORD`, `MINIO_ROOT_USER`, `MINIO_ROOT_PASSWORD`, `JWT_SECRET`, `HF_TOKEN` (Hugging Face for model downloads), `ENVIRONMENT`.

## Frontend Proxy

Vite dev server proxies `/api` requests to `http://localhost:8000` (configured in `vite.config.ts`).

---
> Source: [rubythalib-ai/auto-annotation-tool](https://github.com/rubythalib-ai/auto-annotation-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
