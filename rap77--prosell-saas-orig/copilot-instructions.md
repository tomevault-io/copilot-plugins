## prosell-saas-orig

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# GEMINI.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## General Workflow
1. Primero, analiza el problema, lee el código base para encontrar los archivos relevantes y escribe un plan en task/todo.md.
2. El plan debe incluir una lista de tareas pendientes que puedas marcar a medida que las completes.
3. Antes de empezar a trabajar, contáctame y verificaré el plan.
4. Luego, empieza a trabajar en las tareas pendientes, marcándolas como completadas a medida que avanzas.
5. Por favor, en cada paso del proceso, simplemente dame una explicación detallada de los cambios que realizaste.
6. Simplifica al máximo cada tarea y cambio de código que realices. Queremos evitar cambios masivos o complejos. Cada cambio debe afectar la menor cantidad de código posible. Todo se basa en la simplicidad.
7. Finalmente, agrega una sección de revisión al archivo [todo.md] con un resumen de los cambios que realizaste y cualquier otra información relevante.


## Project Overview

This is ProSell SaaS
 - a price analysis and scraping platform for marketplaces. The project follows Clean Architecture (Hexagonal Architecture) principles and implements a multi-tenant SaaS application focused on Facebook Marketplace scraping and product analysis.

## Key Technologies

- **Python 3.11+** with **FastAPI** for the backend API
- **SQLAlchemy** for database ORM with support for both SQLite and PostgreSQL
- **Alembic** for database migrations (separate configs for SQLite and PostgreSQL)
- **Playwright** for web scraping
- **Loguru** for structured logging
- **pytest** with **pytest-asyncio** for testing
- **UV** for dependency management

## Common Commands

### Development
```bash
# Install dependencies
uv sync

# Run the application
python -m uvicorn prosell_saas.infrastructure.http.main:app --reload --host 0.0.0.0 --port 8000

# Run tests
pytest

# Run specific test file
pytest tests/test_facebook_endpoint.py

# Run with coverage
pytest --cov=prosell_saas tests/
```

### Database Management
```bash
# Run SQLite migrations
python migrations._proxy.py

# Alternative Windows batch script
migrations.bat

# Create new migration (SQLite)
alembic -c alembic_sqlite/alembic.ini revision --autogenerate -m "description"

# Create new migration (PostgreSQL)
alembic -c alembic_postgres/alembic.ini revision --autogenerate -m "description"
```

## Architecture

### Clean Architecture Structure
```
src/prosell_saas/
├── domain/              # Business entities and repository interfaces
│   ├── entities/        # Core business objects (listing.py)
│   └── repositories/    # Repository interfaces
├── application/         # Use cases and business logic
│   └── use_cases/       # Application-specific business rules
└── infrastructure/      # External concerns and implementations
    ├── config/          # Configuration (database, logger)
    ├── http/           # FastAPI application and endpoints
    ├── repositories/   # Repository implementations
    └── scrapers/       # Web scraping implementations
```

### Database Configuration
The application supports both SQLite and PostgreSQL through environment variables:
- `SQLITE_URL` - for SQLite database connection
- `POSTGRES_URL` - for PostgreSQL connection
- Default is SQLite for development

### Environment Variables Required
- `FACEBOOK_EMAIL` - Facebook account email for scraping
- `FACEBOOK_PASSWORD` - Facebook account password for scraping
- `SQLITE_URL` or `POSTGRES_URL` - Database connection strings

## Key Components

### Main Application Entry Point
- `src/prosell_saas/infrastructure/http/main.py` - FastAPI application with scraping endpoints

### Scraping System
- `ScrapeFacebookUseCase` - Business logic for Facebook scraping
- `FacebookScraper` - Web scraper implementation using Playwright
- Background task processing for scraping jobs

### Database Layer
- Multi-database support (SQLite/PostgreSQL)
- Repository pattern implementation
- Alembic migrations for schema management

### Logging
- Structured JSON logging with Loguru
- Log files stored in `logs/` directory
- Configured in `src/prosell_saas/infrastructure/config/logger.py`

## Testing

### Test Structure
- Unit tests for all major components
- Integration tests for database operations
- End-to-end tests for API endpoints
- Mock-based testing for external dependencies

### Test Files
- `test_facebook_endpoint.py` - API endpoint tests
- `test_scraper.py` - Web scraper tests
- `test_repository.py` - Database repository tests
- `test_use_case.py` - Business logic tests

### Running Tests
Always run the full test suite before making changes:
```bash
pytest tests/ -v
```

## Important Notes

### Multi-tenancy Support
The architecture includes multi-tenant support as documented in the requirements, though the current implementation is focused on the core scraping functionality.

### Error Handling
- Comprehensive error handling in scraping operations
- Retry mechanisms with Tenacity
- Structured error logging

### Security
- Environment-based configuration for sensitive data
- No hardcoded credentials in source code
- Input validation using Pydantic models

## Development Workflow

1. **Before making changes**: Run `pytest` to ensure all tests pass
2. **Database changes**: Create migrations using Alembic
3. **New features**: Follow the Clean Architecture layers
4. **Testing**: Write tests for new functionality
5. **Environment**: Ensure required environment variables are set

## Documentation

Extensive documentation is available in the `doc/` directory:
- `REQUISITOS_PRODUCTO_ARQUITECTURA.md` - Detailed architecture documentation
- `REQUISITOS_PRODUCTO_MAIN.md` - Main product requirements
- Other requirement and analysis documents

The project aims to be a comprehensive SaaS platform with planned features including price analysis, multi-marketplace support, and advanced analytics capabilities.

## Coding Conventions and Development Rules

### Clean Architecture Principles
- **Fully Typed Code**: All functions, classes, and variables must have explicit type annotations
- **Entity Design**: Use dataclasses or Pydantic models for domain entities
- **Repository Pattern**: Abstract interfaces in domain layer, concrete implementations in infrastructure
- **Use Cases**: Each use case has an `execute()` method with typed input/output dataclasses
- **Dependency Injection**: Use FastAPI's `Depends` for all use case dependencies

### Naming Conventions
- **Variables and functions**: `snake_case`
- **Classes**: `PascalCase`
- **Constants**: `UPPER_CASE`
- **Private methods**: `_underscore_prefix`
- **Repository interfaces**: `I{Entity}Repository`
- **Use cases**: `{Action}{Entity}UseCase`

### Error Handling
- **Domain Exceptions**: All domain exceptions inherit from `DomainException`
- **Centralized Handling**: FastAPI middleware catches and transforms exceptions
- **Structured Logging**: Use Loguru for all logging with structured JSON format
- **Retry Logic**: Implement exponential backoff for external service calls

### UV Package Management Rules
- **Exclusive Use**: NEVER use pip, pip-tools, or poetry directly
- **Environment Management**:
  ```bash
  # Create and activate virtual environment
  uv venv
  source .venv/bin/activate  # Linux/Mac
  .venv\Scripts\activate.bat  # Windows
  ```
- **Dependency Management**:
  ```bash
  # Add dependencies
  uv add package-name
  uv add --dev pytest pytest-asyncio pytest-cov

  # Remove dependencies
  uv remove package-name

  # Sync dependencies
  uv sync

  # Run scripts
  uv run script.py
  uv run pytest
  uv run uvicorn main:app --reload
  ```
- **Development Dependencies**: Always use `--dev` flag for development-only packages
- **Lock Files**: Always commit `uv.lock` to version control

### Web Scraping Rules
- **Architecture Integration**: All scrapers implement `IScraper[T]` interface
- **Use Case Orchestration**: Scraping logic belongs in use cases, not controllers
- **Tool Selection**:
  - **Playwright**: For JavaScript-heavy sites (Facebook, Amazon)
  - **Requests + BeautifulSoup**: For simple static sites
  - **Scrapy**: For large-scale scraping projects
  - **AgentQL**: For complex known processes (login flows, form submissions)
  - **Multion**: For exploratory or unknown tasks

### Scraping Best Practices
- **Rate Limiting**: Always implement delays between requests
- **Error Handling**: Robust retry mechanisms with exponential backoff
- **Data Validation**: Validate all scraped data before storage
- **Headers Rotation**: Implement User-Agent and header rotation
- **Proxy Support**: Configure proxy rotation for production
- **Ethical Scraping**: Respect robots.txt and terms of service

### Configuration Management
- **Environment Variables**: All sensitive configuration via environment variables
- **Pydantic Settings**: Use Pydantic BaseSettings for configuration classes
- **Multiple Environments**: Support dev/staging/prod configurations
- **Secrets Management**: Never commit secrets to version control

### Testing Standards
- **Comprehensive Coverage**: Aim for >90% test coverage
- **Test Structure**: Unit tests for all layers, integration tests for repositories
- **Scraper Testing**: Mock external requests, use fixture HTML for parsing tests
- **Async Testing**: Use pytest-asyncio for async code testing
- **Database Testing**: Use test database with proper cleanup

### Performance and Monitoring
- **Async Operations**: Use asyncio for I/O bound operations
- **Background Tasks**: Use FastAPI BackgroundTasks for long-running operations
- **Caching**: Implement Redis caching for frequently accessed data
- **Monitoring**: Use Prometheus metrics for scraping performance
- **Profiling**: Regular performance profiling with cProfile

### Code Quality Tools
- **Linting**: Use Ruff for code linting and formatting
- **Type Checking**: Use mypy for static type checking
- **Pre-commit Hooks**: Set up pre-commit hooks for code quality
- **CI/CD**: All code quality checks must pass in CI pipeline

## Environment Setup Details

### Required Environment Variables
```bash
# Database Configuration
SQLITE_URL=sqlite:///./prosell_saas.db
POSTGRES_URL=postgresql://user:password@localhost:5432/prosell_saas

# Scraping Configuration
FACEBOOK_EMAIL=your_email@example.com
FACEBOOK_PASSWORD=your_password

# Optional Configuration
LOG_LEVEL=INFO
REDIS_URL=redis://localhost:6379
```

### Development Setup
```bash
# Clone and setup project
git clone <repository-url>
cd prosell-saas
uv venv
source .venv/bin/activate
uv sync

# Setup database
python migrations._proxy.py

# Run application
uv run uvicorn prosell_saas.infrastructure.http.main:app --reload
```

### Production Setup
```bash
# Use production dependencies
uv sync --no-dev

# Set production environment variables
export POSTGRES_URL=postgresql://prod_user:prod_pass@prod_host:5432/prosell_saas
export LOG_LEVEL=WARNING

# Run with production server
uv run gunicorn prosell_saas.infrastructure.http.main:app -w 4 -k uvicorn.workers.UvicornWorker
```

## Solutions to Common Repository Issues

### Database Migration Issues
- **Issue**: Migration conflicts between SQLite and PostgreSQL
- **Solution**: Use separate Alembic configs and ensure environment-specific migrations

### Scraping Rate Limiting
- **Issue**: Getting blocked by target websites
- **Solution**: Implement exponential backoff and rotate headers/proxies

### Memory Usage in Scraping
- **Issue**: High memory usage with large datasets
- **Solution**: Use batch processing and streaming for large data sets

### Type Checking Errors
- **Issue**: mypy errors with complex generic types
- **Solution**: Use proper type annotations and TYPE_CHECKING imports

### Test Database Cleanup
- **Issue**: Test database state pollution
- **Solution**: Use pytest fixtures with proper setup/teardown

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rap77)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rap77)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
