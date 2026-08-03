## create-fastapi-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an enterprise-grade FastAPI backend template named "evoai-backend-template" with a clean three-layer architecture (API → Service → Repository → Model). It includes built-in RBAC permission management, user management, file management, audit logging, and other core enterprise features. The project uses UV for package management and focuses on providing a production-ready, scalable backend framework.

**Key Features:**
- 🏗️ **Clean Architecture**: Three-layer separation (API/Service/Repository)
- 🔐 **RBAC Authorization**: Role-based access control with menus and API permissions
- 👤 **User Management**: Complete user lifecycle with JWT authentication
- 📝 **Audit Logging**: Comprehensive activity tracking and monitoring
- 📁 **File Management**: Secure file upload/download with validation
- 🚀 **Performance**: Redis caching, database optimization, rate limiting
- 🐳 **Production Ready**: Docker support, health checks, monitoring
- 📖 **Documentation**: Auto-generated API docs with MkDocs integration

## Common Commands

### Environment Setup
```bash
# Install UV package manager (if not installed)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install project dependencies
uv sync

# Install development dependencies (includes testing, linting tools)
uv sync --dev

# Install documentation dependencies
uv sync --group docs
```

### Development Server
```bash
# Run development server with hot reload (entry point is src/__init__.py)
uv run uvicorn src:app --reload --host 0.0.0.0 --port 8000

# Run production server with multiple workers
uv run uvicorn src:app --host 0.0.0.0 --port 8000 --workers 4

# Alternative: Run via Python module
uv run python -m uvicorn src:app --reload
```

### Database Operations
```bash
# Initialize database (first time setup)
uv run aerich init-db

# Generate migration after model changes
uv run aerich migrate --name "describe_your_changes"

# Apply migrations
uv run aerich upgrade

# View migration history
uv run aerich history
```

### Testing
```bash
# Run all tests
uv run pytest

# Run specific test file
uv run pytest tests/test_users.py

# Run with coverage report
uv run pytest --cov=src --cov-report=html
```

### Code Quality

#### 🔧 Pre-commit Hooks (自动化)
```bash
# hooks 会在 uv sync 时自动安装并配置
# 每次 git commit 时自动运行，确保代码质量

# 手动运行所有检查
uv run pre-commit run --all-files

# 禁用 hooks (如不需要)
uv run pre-commit uninstall

# 跳过单次检查 (紧急提交)
git commit --no-verify -m "urgent fix"
```

#### ⚙️ 手动检查命令
```bash
# 代码检查和自动修复 (替代 black + isort)
uv run ruff check --fix src/

# 代码格式化
uv run ruff format src/

# 类型检查 (可选)
uv run mypy src/
```

📖 **详细配置**: 查看 [docs/pre-commit-guide.md](docs/pre-commit-guide.md)

### Docker Operations
```bash
# Build image
docker build -t backend-template .

# Run container
docker run -p 8000:8000 backend-template
```

### Documentation
```bash
# Install documentation dependencies
uv sync --group docs

# Serve documentation locally
uv run mkdocs serve

# Build documentation
uv run mkdocs build

# Deploy documentation to GitHub Pages
uv run mkdocs gh-deploy
```

## Architecture Overview

The project follows a clean three-layer architecture with clear separation of concerns:

```
┌─────────────────────────────────────────────────────────────┐
│                        API Layer                            │
│  (src/api/v1/) - Routes, parameter validation, responses    │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                      Service Layer                          │
│  (src/services/) - Business logic, permissions, validation  │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                   Repository Layer                          │
│  (src/repositories/) - Data access, CRUD operations         │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                      Model Layer                            │
│  (src/models/) - Tortoise ORM models, database schemas     │
└─────────────────────────────────────────────────────────────┘
```

### Key Design Principles
- **Single Responsibility**: Each layer handles only its own logic
- **Dependency Injection**: Managed through FastAPI's dependency system
- **Type Safety**: Comprehensive Python type annotations throughout
- **Async First**: All I/O operations are asynchronous
- **Security First**: Multiple built-in security mechanisms

### Core Components

- **Authentication**: JWT-based with access tokens (4 hours) and refresh tokens (7 days)
- **Authorization**: RBAC system with roles, menus, and API permissions
- **Rate Limiting**: Built-in request rate limiting using SlowAPI
- **File Management**: Secure file upload/download with type validation and size limits
- **Audit Logging**: HTTP request logging and user activity tracking via middleware
- **Caching**: Redis integration with configurable TTL and smart caching strategies
- **Middleware Stack**: Security headers, request logging, background tasks, audit logging
- **Data Processing**: Built-in sensitive content filtering and data processors

## Development Workflow for New Features

When adding new functionality, follow this standard process:

1. **Define Model** (`src/models/admin.py`) - Create Tortoise ORM model
2. **Create Schema** (`src/schemas/`) - Define Pydantic validation schemas
3. **Implement Repository** (`src/repositories/`) - Add data access layer
4. **Write Service** (`src/services/`) - Implement business logic
5. **Add API Routes** (`src/api/v1/`) - Create endpoint handlers
6. **Generate Migration** - Run `uv run aerich migrate --name "feature_name"`
7. **Write Tests** (`tests/`) - Add test coverage

## Security Considerations

- JWT tokens are configured with HS256 algorithm
- Default admin credentials: username=`admin`, password=`abcd1234` (change immediately!)
- Password requirements: minimum 8 characters with letters and numbers
- File upload restrictions: whitelist validation, size limits, dangerous file detection
- Production checklist:
  - Set `DEBUG=False`
  - Generate strong `SECRET_KEY` with `openssl rand -hex 32`
  - Configure proper `CORS_ORIGINS`
  - Use PostgreSQL instead of SQLite
  - Set strong `SWAGGER_UI_PASSWORD`

## Database Best Practices

- Models inherit from `BaseModel` and `TimestampMixin` for consistency
- Use `select_related()` for foreign key preloading
- Use `prefetch_related()` for many-to-many optimization
- Add indexes on frequently queried fields
- String references for relationships to avoid circular imports: `fields.ForeignKeyField("models.User")`

## Environment Configuration

Key environment variables (configured in `.env`):

### Core Settings
- `APP_ENV`: development/production/testing (default: development)
- `DEBUG`: Enable debug mode (default: True)
- `SECRET_KEY`: JWT signing key (auto-generated if missing, minimum 32 chars)
- `APP_TITLE`: Application title (default: "Vue FastAPI Admin")
- `PROJECT_NAME`: Project name for identification

### Database Configuration
- `DB_ENGINE`: postgres/sqlite (default: postgres for production)
- `DB_HOST`: Database host (default: localhost)
- `DB_PORT`: Database port (default: 5432)
- `DB_USER`: Database username (default: postgres)
- `DB_PASSWORD`: Database password (required for production)
- `DB_NAME`: Database name (default: fastapi_backend)

### Authentication & Security
- `JWT_ACCESS_TOKEN_EXPIRE_MINUTES`: Access token expiry (default: 240)
- `JWT_REFRESH_TOKEN_EXPIRE_DAYS`: Refresh token expiry (default: 7)
- `SWAGGER_UI_USERNAME`: Swagger UI username (default: admin)
- `SWAGGER_UI_PASSWORD`: Swagger UI password (required, minimum 8 chars)

### CORS & External Access
- `CORS_ORIGINS`: Comma-separated allowed origins (default: localhost:3000,localhost:8080)
- `CORS_ALLOW_CREDENTIALS`: Allow credentials in CORS (default: True)

### Caching & Performance
- `REDIS_URL`: Redis connection URL (default: redis://localhost:6379/0)
- `CACHE_TTL`: Default cache TTL in seconds (default: 300)

## Project Structure

```
src/
├── __init__.py                 # FastAPI app entry point
├── api/                        # API Layer
│   └── v1/                    # API version 1
│       ├── apis/              # API management endpoints
│       ├── auditlog/          # Audit logging endpoints
│       ├── base/              # Base/health endpoints
│       ├── depts/             # Department management
│       ├── files/             # File management
│       ├── menus/             # Menu management
│       ├── roles/             # Role management
│       └── users/             # User management
├── core/                      # Core functionality
│   ├── crud.py               # Base CRUD operations
│   ├── dependency.py         # FastAPI dependencies
│   ├── exceptions.py         # Custom exceptions
│   ├── init_app.py          # App initialization
│   └── middlewares.py       # Custom middleware
├── handlers/                  # Data processing
│   ├── data_processor.py    # Data processing utilities
│   └── sensitive_filter.py   # Content filtering
├── models/                    # Data Models (Tortoise ORM)
│   ├── admin.py             # User, Role, API, Menu models
│   ├── base.py              # Base model classes
│   └── enums.py             # Enum definitions
├── repositories/              # Repository Layer
│   ├── api.py               # API data access
│   ├── dept.py              # Department data access
│   ├── menu.py              # Menu data access
│   ├── role.py              # Role data access
│   └── user.py              # User data access
├── schemas/                   # Pydantic Schemas
│   ├── apis.py              # API validation schemas
│   ├── base.py              # Base schemas
│   ├── depts.py             # Department schemas
│   ├── login.py             # Authentication schemas
│   ├── menus.py             # Menu schemas
│   ├── roles.py             # Role schemas
│   └── users.py             # User schemas
├── services/                  # Service Layer (Business Logic)
│   ├── base_service.py      # Base service class
│   ├── file_service.py      # File handling service
│   └── user_service.py      # User business logic
├── settings/                  # Configuration
│   └── config.py            # Application settings
├── log/                       # Logging utilities
│   ├── context.py           # Logging context
│   └── log.py               # Logger configuration
└── utils/                     # Utilities
    ├── cache.py             # Redis caching utilities
    ├── jwt.py               # JWT token utilities
    ├── password.py          # Password hashing utilities
    └── sensitive_word_filter.py  # Content filtering
```

## Important Notes

- **Entry Point**: Application starts from `src/__init__.py` which creates the FastAPI app
- **Async Operations**: All routes and database operations are async - always use `await`
- **Repository Pattern**: Data access through repositories only - avoid direct model queries in services
- **Service Layer**: Business logic and permissions handled in services - keep routes thin
- **Authentication**: Use `current_user: User = Depends(get_current_user)` for auth
- **Admin Endpoints**: Use `current_user: User = Depends(SuperUserRequired)` for admin-only access
- **Database Changes**: Always generate migrations with `uv run aerich migrate --name "description"`
- **Package Management**: Use UV exclusively - avoid pip direct usage
- **Environment**: Copy `.env.example` to `.env` and configure appropriately
- **Testing**: Comprehensive test suite in `tests/` directory with pytest

## API Documentation & Monitoring

After starting the server:
- **Swagger UI**: http://localhost:8000/docs (requires authentication)
- **ReDoc**: http://localhost:8000/redoc (requires authentication)
- **OpenAPI Schema**: http://localhost:8000/openapi.json (requires authentication)
- **Health Check**: http://localhost:8000/api/v1/base/health
- **Version Info**: http://localhost:8000/api/v1/base/version

### Default Credentials
- **Username**: `admin`
- **Password**: `abcd1234`
- **Email**: `admin@admin.com`

⚠️ **Security Warning**: Change default credentials immediately in production!

---
> Source: [JiayuXu0/create-fastapi-app](https://github.com/JiayuXu0/create-fastapi-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
