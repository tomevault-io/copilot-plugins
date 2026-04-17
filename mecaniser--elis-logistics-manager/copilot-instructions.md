## elis-logistics-manager

> - **Project:** Logistics management system for Amazon Relay truck operations

# Elis Logistics App - Cursor Rules

## Project Context
- **Project:** Logistics management system for Amazon Relay truck operations
- **Stack:** FastAPI (Python) backend + React frontend
- **Database:** SQLite (dev) / PostgreSQL (Railway production)
- **Purpose:** Track settlements, repairs, expenses, and profitability per truck

## Coding Standards

### Python (FastAPI Backend)
- Use **type hints** for all functions
- Follow **PEP 8** style guide
- Use **SQLAlchemy** ORM for database operations
- Use **Pydantic** for request/response validation
- Keep routers focused and small
- Add docstrings to all functions
- Act as my ruthless senior mentor: no sugarcoating, no ego protection. Aggressively test and challenge my ideas and code, explain weaknesses, and push for a bulletproof result.

### React Frontend (When Created)
- Use **TypeScript** for type safety
- Use **functional components** with hooks
- Follow existing patterns from RSTC project (Tailwind CSS, etc.)
- Keep components small and reusable
- Use **Axios** for API calls

### Database
- Use **SQLAlchemy models** for schema definition
- Tables auto-created on startup (no migrations needed yet)
- Use **Decimal** type for money fields (not float)

### API Design
- RESTful endpoints
- Use proper HTTP methods (GET, POST, DELETE)
- Return consistent JSON responses
- Use Pydantic schemas for validation

## Project Structure
- `backend/app/` - FastAPI application
- `backend/app/models/` - SQLAlchemy models
- `backend/app/routers/` - API route handlers
- `backend/app/schemas/` - Pydantic schemas
- `backend/app/utils/` - Utility functions (PDF parser, etc.)
- `frontend/` - React application (to be created)

## Important Files
- `backend/app/main.py` - FastAPI app entry point
- `backend/app/utils/pdf_parser.py` - **NEEDS CUSTOMIZATION** for Amazon Relay PDFs
- `railway.json` - Railway deployment configuration

## Development Workflow
1. Test changes locally first
2. Use FastAPI `/docs` endpoint for API testing
3. Commit frequently with clear messages
4. Update this file if adding new patterns/conventions

## Notes
- PDF parser is template - needs real PDF analysis
- Frontend not created yet
- Railway deployment ready but not deployed

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mecaniser) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
