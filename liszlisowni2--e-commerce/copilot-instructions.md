## e-commerce

> Full-stack e-commerce: **Laravel 12** backend + **Vue 3** frontend, deployed via Docker Swarm.

# AGENTS.md

## Project Structure

Full-stack e-commerce: **Laravel 12** backend + **Vue 3** frontend, deployed via Docker Swarm.

## Quick Commands

### Backend (Laravel 12, PHP 8.2+)
```bash
cd backend
php artisan test       # Run Pest tests
php artisan serve      # Local dev server
```

### Frontend (Vue 3, TypeScript, Vite)
```bash
cd frontend
npm run test          # Run Vitest tests
npm run dev           # Vite dev server (port 5173)
npm run build         # Production build
```

### Deployment (Docker Swarm)
scripts/runApp.sh           # Build & deploy production stack
scripts/runAppDev.sh        # Build & deploy dev stack
scripts/updateBackend.sh    # Rebuild backend image & update service
scripts/updateFrontend.sh   # Rebuild frontend image & update service
scripts/restartBackendNginx.sh  # Restart nginx service

## Architecture

### Backend (backend/)
- Framework: Laravel 12 with Sanctum authentication
- Database: PostgreSQL 18 (DB_CONNECTION=pgsql)
- Cache/Queue: Redis
- Storage: MinIO (S3-compatible) for product images
- API Routes: backend/routes/api.php
- Models: User, Product, Address
- Enums: ScopeEnum (user/admin/vendor/support/superadmin), StatusEnum (active/banned/inactive), AddressType (shipping/billing)
### Frontend (frontend/)
- Framework: Vue 3 with Composition API
- Build: Vite 8 + TypeScript 6
- State: Pinia
- Data Fetching: TanStack Vue Query + Axios
- UI: Tailwind CSS 4 + Reka UI + Lucide icons
- Forms: VeeValidate + Zod
- Testing: Vitest + Vue Test Utils
### Infrastructure
- Services: frontend, backend (PHP-FPM), nginx, PostgreSQL, Redis, MinIO
- Dev mode: Frontend runs Vite dev server (port 5173), uses .env.dev
- Prod mode: Frontend built to static files served by nginx
- Local registry: 127.0.0.1:5000 (Docker Swarm required)
### Key Files
- backend/.env.example - Backend environment template (copy to .env)
- backend/.env.dev - Dev environment (used by docker-compose.dev.yml)
- docker-compose.yml - Production stack
- docker-compose.dev.yml - Development stack
- scripts/*.sh - Deployment automation

## Testing

### Backend (Pest)
- Tests in backend/tests/Feature/ and backend/tests/Unit/
- Uses RefreshDatabase trait for database isolation
- Requires running backend and PostgreSQL database on docker.
- Run: cd backend && composer test
### Frontend (Vitest)
- Tests in frontend/src/**/*.test.ts
- Uses jsdom environment with vitest.setup.ts
- Axios is mocked globally in setup
- Run: cd frontend && npm run test
### Development Notes
- Backend uses .env.dev in dev mode (not .env)
- Frontend API URL: VITE_API_URI env var (defaults to http://localhost:8080/api)
- Auth tokens stored in localStorage as token
- User roles: user, admin, vendor, support, superadmin
- Passwords require min 8 characters with confirmation

## Avoid reading below files

- .env
- .env.dev
- *.key
- *.pem

---
> Source: [LisZLisowni2/e-commerce](https://github.com/LisZLisowni2/e-commerce) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
