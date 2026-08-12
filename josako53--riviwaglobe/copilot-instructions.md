## riviwaglobe

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Riviwa is an event-driven microservices platform for Grievance & Feedback Management (GRM). Five independently deployable FastAPI services communicate via Apache Kafka and are fronted by Nginx with SSL termination.

## Common Commands

### Development

```bash
# Start all services (dev mode with hot-reload, single Kafka node)
docker compose up -d

# Start a specific service and its dependencies
docker compose up -d feedback_service

# View logs for a service
docker compose logs -f feedback_service

# Rebuild a service after changing requirements.txt or Dockerfile
docker compose up -d --build feedback_service

# Production (skips override file, uses full Kafka cluster)
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

### Database Migrations (Alembic)

Run inside the service container:

```bash
docker compose exec feedback_service alembic revision --autogenerate -m "description"
docker compose exec feedback_service alembic upgrade head
docker compose exec feedback_service alembic downgrade -1
```

Migration naming convention: `YYYY-MM-DD_HH-MM_<slug>.py` (date-sortable).

Notification service does **not** use Alembic — it creates tables via `SQLModel.metadata.create_all` on startup.

### No Test Suite

There are currently no automated tests or pytest configuration in the codebase.

## Architecture

### Services & Ports

| Service | Port | Directory | Database | Purpose |
|---------|------|-----------|----------|---------|
| **auth** | 8000 | `riviwa_auth_service/` | `riviwa_auth_db` (5433) | Auth, JWT, users, orgs, OTP, fraud detection |
| **payment** | 8040 | `payment/` | `payment_db` (5435) | Payment intents, provider webhooks (AzamPay, Selcom, M-Pesa) |
| **notification** | 8060 | `notification_service/` | `notification_db` (5437) | Centralized delivery (push/SMS/WhatsApp/email/in-app) |
| **stakeholder** | 8070 | `stakeholder_service/` | `stakeholder_db` (5436) | Stakeholder engagement, projects, activities |
| **feedback** | 8090 | `feedback_service/` | `feedback_db` (5434) | Grievances, suggestions, escalations, voice |

### Infrastructure

- **Kafka** (KRaft mode): 3 controller+broker + 1 dedicated broker in prod; single node in dev via `docker-compose.override.yml`
- **PostgreSQL 15**: one database per service (full isolation)
- **Redis 7**: DB 0 for auth (sessions, JTI deny-list, Celery broker), DB 3 for notification (rate limiting, dedup)
- **MinIO**: S3-compatible object storage for voice recordings (`riviwa-voice`) and images (`riviwa-images`)
- **Nginx**: reverse proxy, SSL termination, CORS enforcement (services only enable CORS in dev)

### Kafka Topics & Event Flow

- `riviwa.user.events` — auth publishes user lifecycle (registration, verification, suspension, etc.)
- `riviwa.organisation.events` — auth publishes org/membership/invite/project events
- `riviwa.stakeholder.events` — stakeholder publishes engagement, concerns, communications
- `riviwa.feedback.events` — feedback publishes grievance lifecycle (submitted → acknowledged → escalated → resolved → closed)
- `riviwa.payment.events` — payment publishes payment lifecycle
- `riviwa.notifications` — all services publish notification requests (inbound to notification_service)
- `riviwa.notifications.events` — notification publishes delivery receipts

**Partition key**: `user_id` or `org_id` for ordering guarantees per entity.

### API Routing (Nginx → service)

- `/api/v1/auth`, `/api/v1/users`, `/api/v1/orgs`, `/api/v1/webhooks` → auth
- `/api/v1/projects`, `/api/v1/stakeholders`, `/api/v1/activities`, `/api/v1/communications`, `/api/v1/focal-persons` → stakeholder
- `/api/v1/feedback`, `/api/v1/categories`, `/api/v1/channels`, `/api/v1/committees`, `/api/v1/pap`, `/api/v1/voice`, `/api/v1/reports`, `/api/v1/my`, `/api/v1/escalation-requests` → feedback
- `/api/v1/payments` → payment

### Auth Pattern

- Auth service **signs** JWTs using `SECRET_KEY` / `ALGORITHM`
- All other services **verify** JWTs using `AUTH_SECRET_KEY` / `AUTH_ALGORITHM` (same values, different env var names)
- Password hashing: Argon2id (passlib + argon2-cffi)
- Service-to-service calls use `INTERNAL_SERVICE_KEY` header

### Notification Service: Ignorance Principle

The notification service is decoupled from business logic — it only knows **how** to deliver, not **what** or **why**. Originating services publish to `riviwa.notifications` with `notification_type` (template key), `variables`, `recipient`, and `channels`. The notification service handles template rendering (Jinja2), channel dispatch, retries (APScheduler with exponential backoff), and rate limiting.

## Code Patterns

### Uniform Service Layout

Each service follows the same directory structure:
- `main.py` — FastAPI app, lifespan (DB init, Kafka producer/consumer, seeding)
- `api/` — route handlers
- `models/` — SQLModel ORM models
- `schemas/` — Pydantic request/response schemas
- `services/` — business logic
- `repositories/` — database queries (async)
- `events/` — Kafka producers and consumers
- `core/` — config (pydantic-settings), security, exceptions
- `db/` — async engine/session setup
- `alembic/` + `alembic.ini` — migrations
- `entrypoint.sh` — waits for DB, runs migrations, then exec uvicorn

### Startup Flow (entrypoint.sh)

1. Wait for PostgreSQL (`pg_isready` loop, max 30s)
2. Run `alembic upgrade head` (or `create_all` for notification)
3. `exec` uvicorn (dev: `--reload`; prod: workers)

### Docker Build

Multi-stage Dockerfile: builder installs deps into `/opt/venv` via `uv`, runtime copies venv and app code. Runs as non-root user `josako`. Dev override mounts source as bind mount (preserving `/opt/venv` via anonymous volume).

### Tech Stack

- **FastAPI 0.124**, **Uvicorn 0.38**, **SQLModel 0.0.27** (SQLAlchemy 2.x + Pydantic v2)
- **asyncpg** (async queries), **psycopg** (sync Alembic migrations)
- **aiokafka** (async Kafka producer/consumer), **cramjam** (compression)
- **structlog** (structured logging with stdlib bridge)
- Auth-only: **Celery** (fraud analysis tasks), **redis** (sessions/cache)
- Notification-only: **APScheduler** (scheduled reminders, retry), **firebase-admin**, **africatalking**, **twilio**, **sendgrid**

### Environment & Config

- All config via pydantic-settings (reads `.env` or environment variables)
- `.env` for dev, `.env.production` for prod
- Production disables OpenAPI docs (`/docs`, `/redoc`, `/openapi.json`)
- CORS handled by Nginx in prod; enabled at service level only in dev

---
> Source: [josako53/RiviwaGlobe](https://github.com/josako53/RiviwaGlobe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
