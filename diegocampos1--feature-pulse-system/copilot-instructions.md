## feature-pulse-system

> Full-stack feature voting system where users submit, discover, and prioritize product feature requests. Built as a technical assessment demonstrating production-ready architecture.

# FeaturePulse — Feature Voting System

## Project Overview
Full-stack feature voting system where users submit, discover, and prioritize product feature requests. Built as a technical assessment demonstrating production-ready architecture.

## Tech Stack
- **Frontend**: Next.js 15 (App Router) + TypeScript + Tailwind CSS v4 + shadcn/ui
- **Backend**: Django 5 + Django REST Framework + PostgreSQL 16
- **Auth**: JWT (djangorestframework-simplejwt)
- **Containerization**: Docker Compose
- **State Management**: Zustand
- **Forms**: react-hook-form + zod
- **API Client**: Axios + TanStack React Query

## Architecture
- Monorepo: `backend/` (Django) + `frontend/` (Next.js)
- API versioned at `/api/v1/`
- JWT auth: access + refresh tokens
- Mobile-first responsive design

## Project Structure
```
backend/
├── config/         # Django settings, urls, wsgi, asgi
├── core/           # Shared mixins (UUIDMixin, TimestampMixin)
├── apps/
│   ├── accounts/   # User model, auth, JWT endpoints
│   └── features/   # Feature requests + voting
├── manage.py
├── requirements.txt / requirements-dev.txt
├── pyproject.toml / .flake8
└── Dockerfile

frontend/
├── src/
│   ├── app/        # Next.js App Router pages
│   ├── components/ # UI components (shadcn/ui in components/ui/)
│   ├── lib/        # API client, auth helpers, utils
│   ├── hooks/      # Custom React hooks
│   ├── stores/     # Zustand stores
│   └── types/      # TypeScript type definitions
├── package.json
├── components.json # shadcn/ui config
└── Dockerfile
```

## Running the Project
```bash
docker compose up --build
```
- Frontend: http://localhost:3000
- Backend API: http://localhost:8000/api/v1/
- Django Admin: http://localhost:8000/admin/

## Useful Commands
```bash
# Run Django management commands
docker compose exec api python manage.py createsuperuser
docker compose exec api python manage.py makemigrations
docker compose exec api python manage.py migrate

# Run linters manually
docker compose exec api black .
docker compose exec api isort .
docker compose exec api flake8
```

## Coding Standards
- Backend lint: ruff + black + isort + flake8 (auto-runs via Claude Code hook on Edit/Write)
- Frontend lint: eslint
- All model IDs are UUIDs (use UUIDMixin from core/mixins.py)
- All models should use TimestampMixin for created_at/updated_at
- API versioned: /api/v1/
- Forms: react-hook-form + zod
- State management: zustand
- No Redux, no class components
- Prefer functional components with hooks
- Use TypeScript strict mode in frontend
- Mobile-first design approach (min-width breakpoints)
- All UI built with shadcn/ui components

## Prompt Logging
Every time you receive a new instruction or prompt, append it to prompts.txt in the project root with a timestamp (ISO 8601) and a brief summary of what you did in response. Create the file if it doesn't exist. Keep this log updated throughout all sessions.

---
> Source: [DiegoCampos1/feature-pulse-system](https://github.com/DiegoCampos1/feature-pulse-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
