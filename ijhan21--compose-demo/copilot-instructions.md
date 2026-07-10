## compose-demo

> Learn environment separation (dev/prod) and JWT-based microservices architecture using DRF, FastAPI, and React.

# Multi-Service Architecture Learning Project

## 🎯 Project Goal
Learn environment separation (dev/prod) and JWT-based microservices architecture using DRF, FastAPI, and React.

## 🏗️ Architecture

### Services
- **auth-service** (DRF): JWT token issuer on port 8000
- **blog-service** (FastAPI): Blog API with JWT verification on port 8001  
- **frontend** (React): Vite + Tailwind CSS on port 5173
- **postgres**: Single database, separate schemas (auth_db, blog_db)
- **nginx** (prod only): Reverse proxy on port 80

### Auth Flow
1. React → DRF `/api/token/` → Access + Refresh tokens
2. React → FastAPI with `Authorization: Bearer <token>` header
3. FastAPI verifies token using **same JWT_SECRET_KEY** as DRF
4. Token refresh via DRF `/api/token/refresh/`

## 📦 Tech Stack

### Backend
- Python 3.12
- Django 5.x + DRF 3.x + djangorestframework-simplejwt
- FastAPI 0.110+ + PyJWT + SQLAlchemy 2.x + Alembic
- PostgreSQL 16
- python-dotenv for environment variables

### Frontend
- React 18 + Vite
- Tailwind CSS 3.x
- Axios for HTTP requests

### Infrastructure
- Docker + Docker Compose
- Nginx (production only)
- Gunicorn (DRF) + Uvicorn (FastAPI)

## 📁 Project Structure
```
project-root/
├── auth-service/
│   ├── Dockerfile.dev
│   ├── Dockerfile.prod
│   ├── requirements.txt
│   ├── .env.dev
│   ├── .env.prod
│   └── manage.py
├── blog-service/
│   ├── Dockerfile.dev
│   ├── Dockerfile.prod
│   ├── requirements.txt
│   ├── .env.dev
│   ├── .env.prod
│   └── main.py
├── frontend/
│   ├── Dockerfile.dev
│   ├── Dockerfile.prod
│   ├── package.json
│   ├── .env.dev
│   ├── .env.prod
│   └── src/
├── nginx/
│   ├── nginx.conf
│   └── Dockerfile
├── compose.dev.yml
├── compose.prod.yml
└── .env.example
```

## 🔧 Commands

### Development
```bash
docker compose -f compose.dev.yml up --build
docker compose -f compose.dev.yml exec auth-service python manage.py migrate
docker compose -f compose.dev.yml exec blog-service alembic upgrade head
```

### Production
```bash
docker compose -f compose.prod.yml up --build -d
docker compose -f compose.prod.yml exec auth-service python manage.py migrate
docker compose -f compose.prod.yml exec blog-service alembic upgrade head
```

## 🚨 Critical Rules

### Environment Variables
- **NEVER commit** `.env.dev` or `.env.prod` files
- DRF and FastAPI **MUST share the same JWT_SECRET_KEY**
- Use `.env.example` as template only

### Development vs Production
- **Dev**: All service ports exposed, source code mounted, DEBUG=True
- **Prod**: Only Nginx port 80 exposed, no volume mounts, DEBUG=False, Gunicorn/Uvicorn

### Docker Compose
- Dev: `compose.dev.yml` with volume mounts for hot reload
- Prod: `compose.prod.yml` with built images only
- PostgreSQL uses named volume in both environments

### CORS Settings
- Dev: Allow `localhost:5173` (React dev server)
- Prod: Allow only production domain

### Network
- Dev: Direct port access (8000, 8001, 5173)
- Prod: Nginx routes `/api/auth/*` → auth-service:8000, `/api/blog/*` → blog-service:8001

## 📝 Code Style

### Python (DRF & FastAPI)
- Use `python-dotenv` for loading `.env` files
- Settings in `settings.py` (DRF) or `config.py` (FastAPI)
- Environment-specific settings via `os.getenv()` with defaults

### React
- Environment variables via Vite: `VITE_*` prefix
- Access with `import.meta.env.VITE_*`
- Axios instance with base URL from env

### Docker
- Multi-stage builds for production
- Development Dockerfile mounts source code
- Production Dockerfile copies source code

## ⚠️ Do Not

- Do not hardcode URLs, secrets, or database credentials
- Do not mix development and production configurations
- Do not expose PostgreSQL port in production
- Do not use Django's development server in production
- Do not commit migration files before testing

## 🎓 Learning Objectives

By completing this project, you will understand:
- Environment separation using `.env` files
- JWT token issuing and cross-service verification
- Docker Compose multi-environment setup
- Nginx reverse proxy configuration
- Hot reload in development vs optimized production builds
- Microservices with shared authentication

---
> Source: [ijhan21/compose-demo](https://github.com/ijhan21/compose-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-10 -->
