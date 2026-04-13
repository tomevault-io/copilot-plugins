## kuzueventbus

> Multi-tenant **Kuzu graph database service** with FastAPI and hexagonal architecture. Core mission: Simple, testable, evolvable service following Clean Architecture, **TDD (Test-Driven Development)**, **DDD (Domain-Driven Design)**, and YAGNI principles, failfast and logs.


# Kuzu Event Bus - AI Coding Agent Instructions

## 🎯 Project Overview

Multi-tenant **Kuzu graph database service** with FastAPI and hexagonal architecture. Core mission: Simple, testable, evolvable service following Clean Architecture, **TDD (Test-Driven Development)**, **DDD (Domain-Driven Design)**, and YAGNI principles, failfast and logs.

## 🏗️ Architecture Fundamentals

### Hexagonal Architecture (STRICT)

```
src/
├── domain/              # Pure business logic (CustomerAccount, TenantName)
├── application/         # Use case orchestration (CustomerAccountService)
├── infrastructure/      # Technical adapters (InMemoryTenantRepository)
└── presentation/                # FastAPI controllers (customers, databases, health)
```

**CRITICAL**: Domain never depends on infrastructure. Use Protocol-based ports for dependency inversion.

### YAGNI Strategy

- **Start simple**: Memory-based implementations for MVP
- **Migrate progressively**: PostgreSQL/Redis only when metrics justify
- **No over-engineering**: One feature at a time

### Development Methodologies

- **TDD (Test-Driven Development)**: Red-Green-Refactor cycle mandatory
- **DDD (Domain-Driven Design)**: Business logic drives architecture
- **Fail Fast**: Explicit validation, immediate error detection

## 📝 Development Workflow

### Test-First Development (TDD)

```bash
# Run tests (from backend/)
pytest                          # All tests
pytest tests/unit/             # Unit tests only
pytest tests/integration/      # Integration tests
pytest --cov=src               # With coverage
```

**TDD Cycle**: Red (failing test) → Green (minimal code) → Refactor (improve)
**Test Structure**: `tests/{unit,integration,e2e}/` mirroring `src/` structure

### Development Environment

```bash
# Setup (from backend/)
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Services
docker-compose up -d           # Redis, PostgreSQL, MinIO

# Run API server
uvicorn src.presentation.api.main:app --reload
```

## 🎯 Core Patterns & Conventions

### 1. Protocol-Based Ports (Not ABC)

```python
# ✅ GOOD: Port in domain/shared/ports/
@runtime_checkable
class CustomerAccountRepository(Protocol):
    async def save(self, customer: CustomerAccount) -> str: ...

# ✅ GOOD: Adapter in infrastructure/
class InMemoryCustomerRepository:
    async def save(self, customer: CustomerAccount) -> str:
        # Implementation
```

### 2. Immutable Value Objects

```python
@dataclass(frozen=True)
class TenantName:
    value: str

    def __post_init__(self):
        if len(self.value) < 3:
            raise ValidationError("Must be at least 3 characters")
        if not re.match(r'^[a-z0-9-]+$', self.value):
            raise ValidationError("Invalid characters")
```

### 3. Explicit Exception Handling

```python
# ✅ GOOD: Specific business exceptions
class BusinessRuleViolation(Exception): pass
class ValidationError(Exception): pass

# ✅ GOOD: Fail fast validation
if not tenant_name:
    raise ValidationError("Tenant name required")

# ❌ BAD: Silent failures
if not tenant_name:
    return None
```

### 4. FastAPI Dependency Injection

```python
# ✅ GOOD: Factory functions for YAGNI
def get_customer_service() -> CustomerAccountService:
    return CustomerAccountService(
        account_repository=InMemoryTenantRepository(),
        auth_service=SimpleAuthService(),
    )

# Use in endpoints
@router.post("/register")
async def register(
    request: CustomerRegistrationRequest,
    service: CustomerAccountService = Depends(get_customer_service)
):
```

### 5. API Key Pattern

```python
# ✅ GOOD: Consistent format with prefix
def generate_api_key() -> str:
    return f"kb_{secrets.token_urlsafe(32)}"

# ✅ GOOD: Format validation
if not api_key.startswith("kb_"):
    raise ValidationError("Invalid API key format")
```

## 🛠️ Key Implementation Details

### Multi-Tenant Isolation

- **Customer**: Top-level account entity
- **Tenant**: Isolated workspace within customer
- **Storage**: Tenant-specific folders in MinIO (`/{tenant_name}/databases/`)

### Authentication Middleware

Located in `src/api/middleware/authentication.py` - validates API keys across all endpoints except health checks.

### Current MVP Scope

**Implemented**:

- ✅ Customer registration with API key generation
- ✅ Health checks (`/health/`)
- ✅ Architecture foundation
- ✅ 84+ passing tests

**Next priorities**:

1. API key authentication on endpoints
2. Database management endpoints
3. Query execution basics
4. Migration to persistent storage (only when needed)

## 🎯 Code Generation Guidelines

When generating code:

1. **Type hints mandatory** - MyPy must pass
2. **Async/await** for all I/O operations
3. **Domain language** - Use business vocabulary (CustomerAccount, not User)
4. **Protocol over ABC** - Use `@runtime_checkable` protocols
5. **Frozen dataclasses** - For all value objects
6. **Test-first** - Write failing test before implementation
7. **Repository pattern** - For all data persistence
8. **Dependency injection** - Use FastAPI Depends()

### Critical File Management Rules

- **Explicit file names** - `customer_account_service.py`, not `service.py`
- **Respect hexagonal layers** - Never put domain logic in infrastructure files
- **Modify existing files** - Don't recreate files that already exist, update them
- **Follow existing structure** - Check `src/` layout before creating new files

### Example: Adding New Domain Entity

```python
# 1. Value object
@dataclass(frozen=True)
class DatabaseName:
    value: str
    def __post_init__(self): # validation

# 2. Port (interface)
class DatabaseRepository(Protocol):
    async def save(self, db: Database) -> str: ...

# 3. Entity
@dataclass
class Database:
    name: DatabaseName
    tenant_id: str

# 4. Test first
def test_database_creation():
    db = Database(DatabaseName("test-db"), "tenant-123")
    assert db.name.value == "test-db"

# 5. Service
class DatabaseManagementService:
    def __init__(self, repository: DatabaseRepository): ...
```

## 📁 Key Files to Reference

- `src/domain/shared/ports/` - All protocol definitions
- `src/domain/tenant_management/customer_account.py` - Core entity patterns
- `src/infrastructure/memory/` - YAGNI implementation examples
- `src/api/routers/customers.py` - FastAPI endpoint patterns
- `pyproject.toml` - Test configuration and dependencies

Focus on following existing patterns rather than introducing new approaches. The codebase prioritizes consistency and simplicity over clever solutions.

## ⚠️ Important Constraints

- **NEVER recreate existing files** - Always modify/extend existing implementations
- **Respect hexagonal boundaries** - Domain code stays in `domain/`, infrastructure in `infrastructure/`
- **Use explicit naming** - File names must clearly indicate their purpose and layer
- **Check existing structure first** - Use semantic search to understand current implementation before adding new code
- **NO "Enhanced" prefixes** - Never create files with "Enhanced", "Improved", "Better" or similar prefixes. Instead, merge enhanced functionality directly into existing components or create new files with descriptive names
- **Avoid duplication** - Always merge functionality into existing components rather than creating duplicated files with prefix variations

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/superWorldSavior) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
