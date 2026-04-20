## python-fastapi-backend

> Enforce the use of FastAPI and its ecosystem for all Python API backends


# Python FastAPI Backend Rules

## Framework and Core Dependencies

For all Python API backend development, use **FastAPI** and its ecosystem:

### Required Core Stack
- **FastAPI** (>=0.104.0) - Modern, fast web framework for building APIs
- **Uvicorn** (>=0.24.0) - ASGI server with `[standard]` extras for production features
- **Pydantic** (>=2.0.0) - Data validation using Python type annotations
- **Pydantic Settings** (>=2.0.0) - Settings management using Pydantic models

### Database and ORM
- **SQLAlchemy** (>=2.0.0) - Modern ORM with async support
- **Alembic** (>=1.12.0) - Database migration tool
- **psycopg2-binary** (>=2.9.9) - PostgreSQL adapter (or appropriate database driver)

### Development Tools
- **pytest** (>=7.4.0) - Testing framework
- **pytest-asyncio** (>=0.21.0) - Async test support
- **httpx** (>=0.25.0) - HTTP client for testing
- **black** (>=23.0.0) - Code formatter
- **isort** (>=5.12.0) - Import sorter
- **mypy** (>=1.6.0) - Static type checker

## Code Structure and Patterns

### Application Structure
```
backend/
├── app/
│   ├── __init__.py
│   ├── main.py              # FastAPI app instance
│   ├── config.py            # Pydantic Settings
│   ├── database.py          # SQLAlchemy setup
│   ├── api/                 # API routes
│   │   ├── __init__.py
│   │   └── *.py
│   ├── models/              # SQLAlchemy models
│   │   ├── __init__.py
│   │   └── *.py
│   ├── schemas/             # Pydantic schemas
│   │   ├── __init__.py
│   │   └── *.py
│   ├── crud/                # Database operations
│   │   ├── __init__.py
│   │   └── *.py
│   └── utils/               # Utility functions
│       ├── __init__.py
│       └── *.py
├── alembic/                 # Database migrations
├── tests/                   # Test files
├── pyproject.toml          # Project dependencies
└── alembic.ini             # Alembic configuration
```

### FastAPI Application Setup

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from .config import settings

app = FastAPI(
    title=settings.PROJECT_NAME,
    version="0.1.0",
    description="API Description",
    openapi_url=f"{settings.API_V1_PREFIX}/openapi.json",
    docs_url=f"{settings.API_V1_PREFIX}/docs",
    redoc_url=f"{settings.API_V1_PREFIX}/redoc",
)

# CORS middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.ALLOWED_ORIGINS,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### Configuration Management

Use **Pydantic Settings** for configuration:

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    PROJECT_NAME: str
    API_V1_PREFIX: str = "/api/v1"
    DATABASE_URL: str
    ALLOWED_ORIGINS: list[str] = ["*"]

    class Config:
        env_file = ".env"
        case_sensitive = True

settings = Settings()
```

### Database Setup

Use **SQLAlchemy 2.0** with async support:

```python
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine
from sqlalchemy.orm import declarative_base, sessionmaker

engine = create_async_engine(DATABASE_URL, echo=True)
AsyncSessionLocal = sessionmaker(
    engine, class_=AsyncSession, expire_on_commit=False
)
Base = declarative_base()
```

### Dependency Injection

Use FastAPI's dependency injection for database sessions:

```python
from fastapi import Depends
from sqlalchemy.ext.asyncio import AsyncSession

async def get_db() -> AsyncSession:
    async with AsyncSessionLocal() as session:
        yield session

@app.get("/items/")
async def read_items(db: AsyncSession = Depends(get_db)):
    # Use db session
    pass
```

### API Routes

Organize routes using APIRouter:

```python
from fastapi import APIRouter

router = APIRouter()

@router.get("/items/")
async def list_items():
    pass

@router.post("/items/")
async def create_item():
    pass
```

### Pydantic Schemas

Use Pydantic v2 models for request/response validation:

```python
from pydantic import BaseModel, Field

class ItemCreate(BaseModel):
    name: str = Field(..., min_length=1, max_length=100)
    description: str | None = None

class ItemResponse(BaseModel):
    id: int
    name: str
    description: str | None

    class Config:
        from_attributes = True  # Pydantic v2
```

### CRUD Operations

Separate database operations into CRUD modules:

```python
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select

async def create_item(db: AsyncSession, item_data: ItemCreate):
    db_item = Item(**item_data.model_dump())
    db.add(db_item)
    await db.commit()
    await db.refresh(db_item)
    return db_item
```

### Testing

Use **pytest** with **httpx** for API testing:

```python
import pytest
from httpx import AsyncClient
from app.main import app

@pytest.mark.asyncio
async def test_create_item():
    async with AsyncClient(app=app, base_url="http://test") as client:
        response = await client.post("/api/v1/items/", json={"name": "Test"})
        assert response.status_code == 201
```

## Best Practices

1. **Type Hints**: Always use type hints for function parameters and return types
2. **Async/Await**: Use async/await for all database operations and I/O-bound tasks
3. **Error Handling**: Use FastAPI's HTTPException for API errors
4. **Validation**: Leverage Pydantic for automatic request/response validation
5. **Documentation**: FastAPI auto-generates OpenAPI docs - use docstrings and response models
6. **Migrations**: Always use Alembic for database schema changes
7. **Testing**: Write tests for all API endpoints using pytest
8. **Code Quality**: Use black (line-length=100) and isort for formatting
9. **Type Checking**: Use mypy for static type checking

## Prohibited Patterns

- ❌ Do NOT use Flask, Django REST Framework, or other web frameworks
- ❌ Do NOT use synchronous database operations when async is available
- ❌ Do NOT bypass Pydantic validation
- ❌ Do NOT manually manage database migrations
- ❌ Do NOT use `@app.on_event("startup")` for database table creation (use Alembic)

## Version Requirements

- Python: >=3.11
- FastAPI: >=0.104.0
- SQLAlchemy: >=2.0.0 (use async patterns)
- Pydantic: >=2.0.0 (use v2 syntax)

---
> Source: [tyrchen/geektime-bootcamp-ai](https://github.com/tyrchen/geektime-bootcamp-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
