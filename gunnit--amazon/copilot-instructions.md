## amazon

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Inthezon** is a multi-tenant SaaS platform for managing multiple Amazon Seller/Vendor Central accounts with automated data extraction, analytics, and predictive insights. The platform is designed for account managers, data analysts, and strategists at Libera Brand Building Group.

## Architecture

The system follows a three-tier architecture deployed on Render.com:

```
Frontend (React/Vite) ←→ Backend (FastAPI) ←→ Workers (Celery)
                              ↓
                    PostgreSQL + Redis + S3
```

### Core Services
- **Frontend**: React 18 + TypeScript + Tailwind CSS + shadcn/ui (Render Static Site)
- **Backend**: Python 3.11 + FastAPI + SQLAlchemy 2.0 (Render Web Service)
- **Workers**: Celery + Redis for background job processing (Render Background Worker)
- **Database**: PostgreSQL with time-series optimized schema
- **Cache/Queue**: Redis for Celery broker and caching

### External Integrations
- Amazon SP-API (Seller Central)
- Amazon Vendor Central API
- Amazon Advertising API
- SendGrid for email notifications
- AWS S3/Cloudflare R2 for report storage

## Project Structure

```
inthezon-platform/
├── backend/
│   ├── app/
│   │   ├── main.py              # FastAPI entry point
│   │   ├── config.py            # Settings management
│   │   ├── api/v1/              # API endpoints
│   │   ├── core/amazon/         # Amazon API clients (SP-API, Vendor, Ads)
│   │   ├── models/              # SQLAlchemy models
│   │   ├── schemas/             # Pydantic schemas
│   │   ├── services/            # Business logic
│   │   └── db/                  # Database session and migrations
│   └── workers/
│       ├── celery_app.py        # Celery configuration
│       └── tasks/               # Background tasks
├── frontend/
│   ├── src/
│   │   ├── components/          # React components (incl. shadcn/ui)
│   │   ├── pages/               # Page components
│   │   ├── hooks/               # Custom React hooks
│   │   ├── services/api.ts      # API client
│   │   └── store/               # Zustand state management
│   └── vite.config.ts
└── render.yaml                  # Render deployment blueprint
```

## Development Commands

### Backend
```bash
cd backend
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
alembic upgrade head                    # Run migrations
uvicorn app.main:app --reload           # Start dev server
```

### Frontend
```bash
cd frontend
npm install
npm run dev                             # Start dev server
npm run build                           # Production build
```

### Workers
```bash
cd backend
celery -A workers.celery_app worker --loglevel=info     # Run worker
celery -A workers.celery_app beat --loglevel=info       # Run scheduler
```

### Docker (Local Development)
```bash
docker-compose up -d
```

## Key Technical Decisions

### Database Schema
- Time-series data (sales, inventory, ads) uses date-partitioned tables with composite indexes on `(account_id, date)`
- Amazon credentials stored encrypted using Fernet symmetric encryption
- JSONB used for flexible forecast predictions storage
- Views provided for common aggregations (e.g., `v_account_summary`)

### Authentication
- JWT with short-lived access tokens (30 min) + refresh token rotation
- bcrypt for password hashing (12 rounds)

### Amazon API Integration
- All three Amazon APIs (SP-API, Vendor Central, Advertising) require separate credential sets
- OAuth refresh tokens stored encrypted and auto-refreshed
- Rate limiting implemented to stay within SP-API limits

### State Management
- Frontend uses Zustand for global state
- Redis for server-side session caching and rate limiting

## Deployment

Deployment is on Render.com using the `render.yaml` blueprint:
- Auto-deploy enabled on main branch push to GitHub
- Backend requires environment variables for Amazon APIs, database, Redis, SendGrid
- Frontend served as static site with API calls to backend service

## QA Validation

After deployments, use the `playwright-qa-validator` agent to run comprehensive automated tests:
- Functional correctness
- UI/UX quality
- Error handling
- Console errors/warnings
- API integration verification

---
> Source: [gunnit/amazon](https://github.com/gunnit/amazon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
