## webot-chatflow

> - `frontend/src/presentation/routes/index.tsx` - Main routing configuration

# Project Structure Guide

## Frontend Structure
- `frontend/src/presentation/routes/index.tsx` - Main routing configuration
- `frontend/src/services/AuthService.ts` - Authentication service
- `frontend/src/shared-theme/` - Theme and UI components
- `frontend/src/i18n/` - Internationalization files

## Backend Structure
- `backend/app/` - Main application code
  - `domain/` - Domain entities and business logic
  - `infrastructure/` - Database and external service implementations
  - `use_cases/` - Application use cases
  - `tests/` - Test files
    - `integration/` - Integration tests
    - `conftest.py` - Test configurations and fixtures

## Key Components
- Authentication: OAuth2/JWT based authentication
- WebSocket: Real-time email communication
- Database: PostgreSQL for persistence
- Cache: Redis for caching and event streaming

## Testing
- Backend: pytest with 90% coverage requirement
- Frontend: Jest + React Testing Library
- Integration tests in `backend/app/tests/integration/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RenatoOliveiraS) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
