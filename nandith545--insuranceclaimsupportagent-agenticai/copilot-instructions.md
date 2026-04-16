## insuranceclaimsupportagent-agenticai

> - Nandith Reddy Malapati, Senior Software Engineer

# CLAUDE.md — Project Instructions for Claude Code

## WHO I AM
- Nandith Reddy Malapati, Senior Software Engineer
- 9+ years experience in distributed systems, cloud infrastructure, and AI engineering
- Stack: Java, Python, TypeScript, Spring Boot, FastAPI, React, AWS, Terraform, Docker, Kubernetes
- All projects target production-grade quality for resume/portfolio showcase

## PROJECT CONTEXT
- Project Name: [UPDATE PER PROJECT]
- Purpose: [UPDATE PER PROJECT]
- Tech Stack: [UPDATE PER PROJECT]

## CRITICAL RULES — NEVER VIOLATE

### Code Quality
1. Every function must have type hints (Python) or TypeScript types
2. Every public function must have a docstring explaining what it does, its parameters, and return value
3. No function should exceed 30 lines — if it does, refactor into smaller functions
4. No file should exceed 300 lines — if it does, split into modules
5. Use dependency injection everywhere — never hardcode dependencies
6. All configuration must come from environment variables via Pydantic BaseSettings — never hardcode secrets, URLs, or credentials
7. Follow single responsibility principle — one class/module does one thing

### Error Handling
8. Never use bare `except:` — always catch specific exceptions
9. Every API endpoint must have proper error handling with meaningful HTTP status codes
10. Create custom exception classes in src/core/exceptions.py — never raise generic Exception
11. All external service calls (APIs, databases, AI models) must have retry logic with exponential backoff
12. Always use structured logging (not print statements) — use Python's logging module or loguru

### Security
13. Never commit secrets, API keys, or credentials — always use .env files and .env.example templates
14. Always validate and sanitize all user input using Pydantic models
15. Use parameterized queries for all database operations — never string concatenation
16. Add rate limiting to all public API endpoints
17. Set CORS policies explicitly — never use allow_all in production

### Testing
18. Write tests alongside code, not after — every new feature must include tests
19. Minimum 80% code coverage target
20. Use pytest with fixtures in conftest.py
21. Unit tests go in tests/unit/, integration tests in tests/integration/
22. Mock all external dependencies in unit tests
23. Test the happy path, error path, and edge cases for every function

### Performance
24. Use async/await for all I/O operations (database, HTTP, file system)
25. Implement connection pooling for database connections
26. Add caching (Redis or in-memory) for frequently accessed data
27. Use pagination for all list endpoints — never return unbounded results
28. Profile before optimizing — don't prematurely optimize

### Architecture
29. Follow the layered architecture: API routes → Services → Repositories → Database
30. API routes should only handle HTTP concerns (parsing request, returning response)
31. Business logic belongs in the services layer — never in routes or repositories
32. Database queries belong in the repositories layer — never in services or routes
33. Use Pydantic models for all request/response schemas — never return raw dicts from API

### Documentation
34. Update README.md whenever you add a new feature or change setup steps
35. Add inline comments only for "why" — the code should explain "what"
36. Keep architecture.md updated with system design decisions

### Docker & Deployment
37. Use multi-stage Docker builds — never ship dev dependencies to production
38. Pin all dependency versions — never use latest tags in Dockerfiles
39. Use docker-compose.yml for local development with all dependencies (DB, Redis, etc.)
40. All infrastructure must be defined as code in Terraform — no manual AWS console changes

## CODING PATTERNS TO FOLLOW

### FastAPI Application Structure
```python
# src/main.py — always follow this pattern
from fastapi import FastAPI
from contextlib import asynccontextmanager
from src.config.settings import get_settings
from src.api.routes import router
from src.api.middleware.error_handler import error_handler_middleware

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup: initialize connections
    yield
    # Shutdown: close connections

app = FastAPI(
    title="Project Name",
    version="1.0.0",
    lifespan=lifespan,
)
app.include_router(router, prefix="/api/v1")
app.middleware("http")(error_handler_middleware)
```

### Configuration Pattern
```python
# src/config/settings.py — always use this pattern
from pydantic_settings import BaseSettings
from functools import lru_cache

class Settings(BaseSettings):
    app_name: str = "project-name"
    debug: bool = False
    database_url: str
    redis_url: str = "redis://localhost:6379"
    openai_api_key: str = ""

    class Config:
        env_file = ".env"

@lru_cache
def get_settings() -> Settings:
    return Settings()
```

### Service Layer Pattern
```python
# src/services/example_service.py
from src.repositories.example_repo import ExampleRepository
from src.core.exceptions import NotFoundError

class ExampleService:
    def __init__(self, repository: ExampleRepository):
        self.repository = repository

    async def get_by_id(self, item_id: str) -> dict:
        result = await self.repository.find_by_id(item_id)
        if not result:
            raise NotFoundError(f"Item {item_id} not found")
        return result
```

## GIT WORKFLOW
- Branch naming: feature/short-description, fix/short-description, chore/short-description
- Commit messages: use conventional commits (feat:, fix:, chore:, docs:, test:, refactor:)
- Always create a PR — never push directly to main
- Squash merge to main

## COMMANDS
- `make dev` — Start local development server
- `make test` — Run all tests
- `make lint` — Run linter and formatter
- `make build` — Build Docker image
- `make deploy` — Deploy to AWS
- `uv add <package>` — Add a dependency
- `uv run pytest` — Run tests via uv

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Nandith545) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
