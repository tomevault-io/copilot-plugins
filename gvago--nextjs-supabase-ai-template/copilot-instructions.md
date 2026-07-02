## nextjs-supabase-ai-template

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Next.js + Supabase full-stack template with TypeScript frontend and Python FastAPI backend, featuring an advanced **3-tier Claude Code agentic framework** for intelligent development. The project uses a broker architecture pattern for organizing API endpoints and includes a smart installation system with automatic port detection.

## Smart Installation & Setup

**🚀 One-Command Setup** (Recommended):
```bash
# Intelligent installer with auto port detection
npm run setup
# OR platform-specific:
./install.sh        # Linux/Mac (optimized)
install.bat         # Windows (PowerShell integration)
```

**What the smart installer does:**
- ✅ Detects available ports automatically
- ✅ Configures all services without conflicts
- ✅ Generates environment files
- ✅ Starts development environment
- ✅ Verifies everything works

## Common Commands

### Full-Stack Development
```bash
# Smart development startup (uses detected ports)
npm run dev         # Start everything with auto-detected ports
npm run setup       # Re-run smart installer
npm run status      # Check service status
npm run logs        # View service logs
npm run stop        # Stop all services
npm run clean       # Clean restart
```

### Frontend (Next.js)
```bash
cd front
npm install      # Install dependencies
npm run dev      # Start development server at localhost:3000
npm run build    # Build for production
npm run lint     # Run ESLint
```

### Backend (FastAPI)
```bash
cd back
pip install -r requirements.txt  # Install dependencies
uvicorn main:app --reload       # Start API server at localhost:8000
pytest                          # Run tests
```

### Local Services (Auto-Detected Ports)
```bash
# Services use auto-detected ports (check .dev-ports.json)
# Default ports (may vary based on availability):
# Frontend: http://localhost:3000
# Supabase API Gateway: http://localhost:8000
# FastAPI Backend: http://localhost:8001  
# Email UI (Inbucket): http://localhost:9000
# Database (PostgreSQL): localhost:5432

# Check actual ports:
cat .dev-ports.json
```

## 🤖 3-Tier Claude Code Framework

This project features a sophisticated agentic framework that automatically routes development requests to specialized experts:

```
Tier 1: Main Orchestrator (Your entry point)
├── Tier 2: Front Agent → Frontend domain coordination
│   └── Tier 3: Frontend Specialists (UI, Auth, State, Forms, Testing)
└── Tier 2: Back Agent → Backend domain coordination
    └── Tier 3: Backend Specialists (API, Database, Testing, DevOps)
```

**Usage**: Simply describe what you want to build - the framework automatically routes to appropriate specialists.

**Examples**:
```bash
"Create a user dashboard with real-time analytics and file upload"
"Build a payment system with Stripe integration and comprehensive testing"
"Add a notification system with email preferences and push notifications"
```

➡️ **Complete framework guide**: [FRAMEWORK-GUIDE.md](./FRAMEWORK-GUIDE.md)

## Architecture

### Frontend Structure
- Next.js 14 with App Router
- Supabase client SDK for auth, database, and storage
- Material-UI (MUI) for components
- React Hook Form with Yup validation
- Authentication context with useAuth hook
- Real-time subscriptions and state management

### Backend Structure - Broker Architecture
The backend follows a FastAPI-based broker architecture pattern:

- **brokers/api/** - FastAPI route handlers organized by feature
- **documents/** - DocumentBase classes for all database operations
- **apis/SupabaseClient.py** - Supabase client singleton wrapper
- **models/** - Type definitions (firestore_types.py, function_types.py) 
- **services/** - Business logic orchestration
- **util/** - Authentication helpers, CORS, logging
- **exceptions/** - Custom error classes

### Critical Backend Rules
1. **All database operations must go through DocumentBase classes** - Never access Supabase directly
2. **Use SupabaseClient singleton** located in `src/apis/SupabaseClient.py` for database access
3. **Define types in models/** - Database documents in `firestore_types.py`, API types in `function_types.py`
4. **Test-first development** - Always start with integration tests that verify the complete user flow
5. **Factory pattern** - Use factories for complex document creation (e.g., from CSV files)

### Testing Philosophy & Framework Integration
- **Integration tests over unit tests** - Test actual FastAPI endpoints via HTTP calls
- **Test real database operations** - Verify records are correctly created/updated in Supabase
- **Test error scenarios first** - Missing params, invalid data, auth failures
- **Use local Supabase stack** - Never mock database operations
- **End-to-end workflows** - Test complete user journeys from start to finish
- **Framework-coordinated testing** - Backend and frontend test agents ensure comprehensive coverage
- **Automated test generation** - Framework specialists create tests for their domain expertise

## Development Workflow

### Framework-Enhanced Development
**With 3-Tier Framework**:
1. **Describe the feature** in natural language - let the framework coordinate specialists
2. **Framework routes automatically** to appropriate backend/frontend experts
3. **Specialists implement** following established architecture patterns
4. **Cross-domain coordination** ensures frontend/backend integration
5. **Built-in quality assurance** through code review and comprehensive testing

**Traditional Backend Development**:
1. Start with tests - create/modify test files in `tests/integration/`
2. Write failing tests that define the expected behavior
3. Implement FastAPI route handlers following the architecture patterns
4. Add routes to main.py router includes
5. Run tests until all pass: `pytest`
6. Never deploy without explicit user request

### Key Patterns
- Route naming: `/api/{resource}` for REST endpoints, `/auth/{action}` for auth
- DocumentBase classes handle all database operations for their table
- Pydantic models for validation and type safety
- JWT-based authentication with Supabase

## Environment Setup

### Environment Files (Auto-Generated)
The smart installer automatically generates environment files with detected ports:

### Frontend Environment (front/.env.local)
```env
# Auto-generated with detected ports
NEXT_PUBLIC_SUPABASE_URL=http://localhost:8000  # Or alternative if 8000 busy
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_anon_key
NEXT_PUBLIC_API_BASE_URL=http://localhost:8001  # Or alternative if 8001 busy
FRONTEND_PORT=3000  # Reference for detected port
```

### Backend Environment (back/.env)
```env
# Auto-generated with detected ports
SUPABASE_URL=http://localhost:8000  # Matches frontend configuration
SUPABASE_SERVICE_ROLE_KEY=your_service_role_key
SUPABASE_ANON_KEY=your_anon_key
ENV=development
API_PORT=8001        # Reference for detected port
DATABASE_PORT=5432   # Reference for detected port
```

### Backend Dependencies
- Python 3.11+ with virtual environment
- Docker and Docker Compose for local Supabase
- All dependencies in `requirements.txt`

## Local Development Stack
**Default ports** (automatically adjusted if busy):
- **Next.js Frontend**: port 3000
- **Supabase API Gateway**: port 8000 (Kong proxy)
- **FastAPI Backend**: port 8001 
- **PostgreSQL Database**: port 5432
- **Email UI (Inbucket)**: port 9000

**Smart Development Setup**:
```bash
# Recommended: Use smart installer
npm run setup    # Detects ports, configures everything, starts services

# Manual Docker (if needed)
docker-compose up -d    # Uses generated docker-compose.override.yml
```

## Documentation & Guides

### Quick Navigation
- **🚀 [QUICK-START.md](./QUICK-START.md)** - Get running in 5 minutes
- **🤖 [FRAMEWORK-GUIDE.md](./FRAMEWORK-GUIDE.md)** - Master the 3-tier framework
- **🛠️ [SETUP-GUIDE.md](./SETUP-GUIDE.md)** - Comprehensive setup options
- **🆘 [TROUBLESHOOTING.md](./TROUBLESHOOTING.md)** - Solutions to common issues
- **⚙️ [INSTALL-GUIDE.md](./INSTALL-GUIDE.md)** - Technical implementation details

## Additional Claude Code Rules

This project includes detailed Claude Code workflow rules in the `.claude/rules/` directories:
- **Root level** (`.claude/rules/`): Project-wide Claude Code workflow and principles  
- **Backend** (`back/.claude/rules/`): FastAPI + Supabase development patterns
- **Frontend** (`front/.claude/rules/`): Next.js + Material-UI + Supabase patterns

These rules provide specific guidance on:
- 3-tier framework coordination and specialist routing
- TodoWrite tool usage for tracking progress
- Architecture patterns and code quality standards
- Testing philosophy and debugging approaches
- Integration between frontend, backend, and database layers

---
> Source: [gvago/nextjs-supabase-ai-template](https://github.com/gvago/nextjs-supabase-ai-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
