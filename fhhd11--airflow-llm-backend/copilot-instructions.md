## airflow-llm-backend

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a production-ready FastAPI-based LLM Chat Backend that integrates with Supabase and LiteLLM. The system provides automated user registration, JWT authentication, and pay-per-use LLM access through virtual keys.

## Architecture

### Core Components
- **FastAPI Backend**: Async REST API for LLM interactions with JWT authentication
- **Supabase Integration**: 
  - PostgreSQL database with Row Level Security (RLS)
  - Automatic user registration via Edge Functions and Database Webhooks
  - JWT-based authentication with service-level database access
- **LiteLLM Gateway**: Unified proxy for 100+ LLM providers with automatic billing
- **Deployment**: Railway (backend) + Supabase Cloud + LiteLLM on Railway

### Data Flow
1. Users register through Supabase Auth
2. Edge Function automatically creates user profile and LiteLLM virtual key  
3. Backend validates JWT tokens and retrieves virtual keys from user profiles
4. LiteLLM handles all billing and cost tracking automatically
5. Backend stores chat history without budget restrictions

## Development Commands

### Local Development
```bash
# Install dependencies
pip install -r requirements.txt

# Run development server
python -m uvicorn app.main:app --reload

# Docker development
cd infra && docker-compose up --build
```

### Database Setup
SQL scripts applied via Supabase UI:
1. `db/sql/0001_backend_tables.sql` - Base tables (profiles, chats)
2. `db/sql/0002_rls_policies.sql` - Row Level Security policies  
3. `db/sql/0003_views_and_functions.sql` - Views and functions

### Edge Function Setup
1. Edge Function: `supabase/functions/create-user-profile/index.ts`
2. Database Webhook: Triggers on `auth.users` INSERT
3. Automatic LiteLLM virtual key creation and profile setup

### Configuration
Required environment variables in `.env`:
- `SUPABASE_URL`: Your Supabase project URL
- `SUPABASE_ANON_KEY`: Supabase anonymous key  
- `SUPABASE_SERVICE_KEY`: Supabase service role key (bypasses RLS)
- `LITELLM_URL`: LiteLLM Railway deployment URL
- `LITELLM_MASTER_KEY`: LiteLLM master key for user management
- `SECRET_KEY`: JWT signing secret (minimum 32 characters)
- `REDIS_URL`: Redis connection (for Docker: redis://redis:6379)
- `CORS_ORIGINS`: Comma-separated list of allowed CORS origins (default: "*" for all origins)

## Key Database Tables

### Backend Tables
- **profiles**: User data with LiteLLM virtual key integration
- **chats**: Chat history with JSONB message storage

### LiteLLM Auto-Tables
- **LiteLLM_UserTable**: Virtual key management
- **LiteLLM_SpendLogs**: Automatic cost tracking

## API Patterns

### REST Endpoints (Backend)
- `POST /api/chat` - Send messages, supports streaming responses
- `POST /api/chat/stream` - Send messages with streaming SSE responses
- `GET /api/chats` - List user chats with pagination and filtering
- `DELETE /api/chats/{chat_id}` - Delete specific chat (owner only)
- `GET /api/models` - Available LLM models from LiteLLM
- `GET /api/models/{model_name}` - Detailed model information
- `POST /api/users/register` - User registration/profile retrieval
- `GET /api/users/me` - Current user information (JWT-based)
- `PATCH /api/users/{user_id}/budget` - Update user budget
- `POST /api/users/{user_id}/sync` - Sync user spending from LiteLLM
- `GET /health` - Service health check

### Authentication Flow
1. Frontend authenticates with Supabase Auth
2. JWT token included in all API requests
3. Backend validates JWT and extracts user_id
4. Backend uses service_client to bypass RLS for database operations
5. Virtual key retrieved from user profile for LiteLLM requests

## Security Model
- JWT authentication via Supabase Auth for all endpoints
- Row Level Security (RLS) with service-level bypass for backend operations
- LiteLLM virtual keys for pay-per-use model
- No budget checking in backend (delegated to LiteLLM)
- Virtual keys stored as actual keys (not encrypted) for simplicity

## Development Guidelines

### Code Style
- Python 3.11+ with async/await patterns
- Pydantic models for request/response validation
- Black formatting (line-length 100)
- Ruff linting

### Project Structure
```
app/
├── api/
│   ├── dependencies/auth.py    # JWT authentication
│   └── routers/                # API route handlers
│       ├── chat.py            # Chat endpoints (send, stream, list, delete)
│       ├── models.py          # Models endpoints (list, info)
│       └── user.py            # User endpoints (register, info, budget)
├── clients/                    # External service clients
│   ├── litellm_client.py      # LiteLLM API integration
│   └── supabase_client.py     # Supabase database client
├── core/                      # Core configuration
│   ├── config.py              # Settings and environment
│   └── logging.py             # Structured logging
├── schemas/                   # Pydantic models
│   ├── requests.py            # Request schemas
│   └── responses.py           # Response schemas
├── services/                  # Business logic layer
│   ├── chat_service.py        # Chat management
│   ├── models_service.py      # Model information
│   └── user_service.py        # User management
└── main.py                   # FastAPI application entry point
```

### Error Handling
- 400: Invalid input validation
- 401: Authentication required
- 402: Insufficient budget (from LiteLLM)
- 429: Rate limit exceeded
- 500: Server errors

## Deployment

### Railway Deployment
- Backend: Deploy from `infra/Dockerfile`
- Environment variables configured in Railway dashboard
- Automatic deployment on git push

### Required Railway Services
1. **FastAPI Backend**: Main application
2. **Redis**: For caching (Railway addon)

### External Services
- **Supabase**: Database and authentication
- **LiteLLM**: Separate Railway deployment for LLM proxy

## Testing

### Available Models (Gemini)
Current supported models:
- `gemini/gemini-1.5-flash` - Fast, efficient model for general tasks
- `gemini/gemini-1.5-pro` - Advanced model for complex tasks

### Test Commands
```bash
# Test chat with Gemini
curl -X POST "http://localhost:8000/api/chat" \
  -H "Authorization: Bearer <jwt-token>" \
  -H "Content-Type: application/json" \
  -d '{"message": "Hello", "model": "gemini/gemini-1.5-flash"}'

# Test streaming chat
curl -X POST "http://localhost:8000/api/chat/stream" \
  -H "Authorization: Bearer <jwt-token>" \
  -H "Content-Type: application/json" \
  -d '{"message": "Tell me a story", "model": "gemini/gemini-1.5-flash"}'

# Get available models
curl -H "Authorization: Bearer <jwt-token>" \
  http://localhost:8000/api/models

# Get user chats
curl -H "Authorization: Bearer <jwt-token>" \
  "http://localhost:8000/api/chats?limit=10&offset=0"

# Get user info
curl -H "Authorization: Bearer <jwt-token>" \
  http://localhost:8000/api/users/me

# Health check
curl http://localhost:8000/health
```

## Production Checklist
- [x] Environment variables configured
- [x] Supabase RLS policies applied
- [x] Edge Function deployed and webhook configured
- [x] LiteLLM service running and accessible (Gemini models)
- [x] All API endpoints implemented and tested
- [x] JWT authentication working correctly
- [x] Chat management (create, list, delete) functional
- [x] Model integration with LiteLLM Admin API
- [x] User management with virtual keys
- [x] SSL/TLS certificates configured (Railway)
- [x] Structured logging implemented
- [ ] Redis service connected (optional for caching)
- [ ] Monitoring and alerting setup
- [ ] Performance optimization
- [ ] Rate limiting implementation

## Current Status: ✅ PRODUCTION READY
The system is fully functional with all core features implemented and tested.

---
> Source: [fhhd11/airflow-llm-backend](https://github.com/fhhd11/airflow-llm-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
