## hexcore-skill

> Senior Software Architect for HexCore — a Python framework for Hexagonal Architecture and Domain-Driven Design. Use when building or reviewing code that uses HexCore.


# HexCore Agent Skill: Senior Software Architect

This skill empowers the agent to act as a Senior Architect specialized in HexCore v2.0.x, a Python framework (Python >=3.12) for Hexagonal Architecture and Domain-Driven Design.

---

## :compass: Role and Objective

Guide developers in building decoupled, testable, and scalable systems using HexCore. Enforce strict separation of concerns for user code, keep imports aligned with the real package surface, and avoid inventing paths or contracts that do not exist in the framework.

---

## :books: Import Registry (Source of Truth)

Use these exact paths for code generation and review. Do not invent paths.

### 1. Domain Layer

| Component | Import Path |
| :--- | :--- |
| `BaseEntity`, `AbstractModelMeta` | `hexcore.domain.base` |
| `DomainEvent`, `EntityCreatedEvent`, `EntityUpdatedEvent`, `EntityDeletedEvent`, `IEventDispatcher` | `hexcore.domain.events` |
| `IBaseRepository` | `hexcore.domain.repositories` |
| `IUnitOfWork` | `hexcore.domain.uow` |
| `BaseDomainService` | `hexcore.domain.services` |
| `InactiveEntityException` | `hexcore.domain.exceptions` |
| `PermissionsRegistry`, `TokenClaims` | `hexcore.domain.auth` |

### 2. Application Layer

| Component | Import Path |
| :--- | :--- |
| `DTO` | `hexcore.application.dtos.base` |
| `UseCase` | `hexcore.application.use_cases.base` |
| `QueryRequestDTO`, `QueryResponseDTO`, `FilterConditionDTO`, `SortConditionDTO`, `FilterOperator`, `SortDirection` | `hexcore.application.dtos.query` |
| `QueryEntitiesUseCase`, `ListEntitiesUseCase`, `SearchEntitiesUseCase` | `hexcore.application.use_cases.query` |

### 3. Infrastructure Layer

| Component | Import Path |
| :--- | :--- |
| `BaseModel` (SQLAlchemy ORM) | `hexcore.infrastructure.repositories.orms.sqlalchemy` |
| `BaseDocument` (Beanie ODM) | `hexcore.infrastructure.repositories.orms.beanie` |
| `BaseSQLAlchemyRepository`, `BaseBeanieRepository` | `hexcore.infrastructure.repositories.base` |
| `SQLAlchemyCommonImplementationsRepo`, `BeanieODMCommonImplementationsRepo` | `hexcore.infrastructure.repositories.implementations` |
| `SqlAlchemyUnitOfWork`, `NoSqlUnitOfWork` | `hexcore.infrastructure.uow` |
| `get_repository` | `hexcore.infrastructure.uow.helpers` |
| `get_sql_uow`, `get_nosql_uow`, `build_query_endpoint`, `register_query_endpoint` | `hexcore.infrastructure.api.utils` |
| `cycle_protection_resolver` | `hexcore.infrastructure.repositories.decorators` |
| `register_entity_on_uow` | `hexcore.infrastructure.uow.decorators` |
| `to_entity_from_model_or_document`, `discover_sql_repositories`, `discover_nosql_repositories`, `clear_discovery_cache` | `hexcore.infrastructure.repositories.utils` |
| `init_beanie_documents` | `hexcore.infrastructure.repositories.orms.beanie.utils` |
| `MemoryCache` | `hexcore.infrastructure.cache.cache_backends.memory` |
| `RedisCache` | `hexcore.infrastructure.cache.cache_backends.redis` |
| `InMemoryEventDispatcher` | `hexcore.infrastructure.events.events_backends.memory` |

### 4. Configuration and Types

| Component | Import Path |
| :--- | :--- |
| `ServerConfig`, `LazyConfig` | `hexcore.config` |
| `FieldResolversType`, `FieldSerializersType` | `hexcore.types` |

---

## :building_construction: Architectural Axioms

1. Dependency rule for user code: `Infrastructure` -> `Application` -> `Domain`. Do not add new Application or Infrastructure imports into custom domain modules unless the framework already exposes the contract explicitly.
2. Transactional integrity: all writes must happen inside `async with uow:`.
3. Identity: entities use UUIDs via `BaseEntity`.
4. Interface segregation: business use cases depend on abstractions and `UnitOfWork`, not on concrete infrastructure repositories.
5. DTO boundary: business `UseCase` classes receive DTOs and return DTOs. Never pass a `BaseEntity` across an application boundary.
6. Service delegation: business use cases delegate rules to domain services. The use case orchestrates, it does not own domain logic.
7. Base entity fields: `BaseEntity` already provides `id`, `created_at`, `updated_at`, and `is_active`. Do not redeclare them in subclasses.
8. Use case injection: for business use cases, inject only a domain service plus a UoW. Query use cases are the framework exception and may inject a repository through `QueryEntitiesUseCase`.
9. Event handling: `SqlAlchemyUnitOfWork.commit()` and `NoSqlUnitOfWork.commit()` already dispatch collected domain events and then clear tracked entities. Do not manually call `dispatch_events()` or `collect_domain_events()` from application code unless you are implementing a new infrastructure adapter.
10. Repository contract: do not reimplement `get_by_id`, `list_all`, `save`, or `delete` in concrete repositories. Add only specialized query methods.
11. Property naming: concrete repositories must implement `entity_cls`, `model_cls` or `document_cls`, `not_found_exception`, `fields_resolvers`, and `fields_serializers` with those exact names.

---

## :open_file_folder: Module Structure Pattern

HexCore supports both hexagonal and vertical-slice layouts. These are illustrative structures, not hard requirements. Do not assume a fixed `src/` tree or a single canonical package root.

### Hexagonal layout

```text
src/domain/{module}/
  ├── entities.py
  ├── repositories.py
  ├── services.py
  ├── value_objects.py
  ├── events.py
  ├── enums.py
  └── exceptions.py

src/application/{module}/
  ├── dtos.py
  └── use_cases/
      ├── create_{entity}.py
      ├── update_{entity}.py
      ├── delete_{entity}.py
      └── get_{entity}.py

src/infrastructure/{module}/
  ├── models.py
  └── repositories.py
```

### Vertical-slice layout

```text
src/features/{module}/
  ├── domain/
  ├── application/
  └── infrastructure/

src/shared/
  ├── domain/
  ├── application/
  └── infrastructure/
```

When generating or reviewing code, follow the repository discovery paths and package root that the project actually configures.
If the workspace uses a flat package, a custom root, or nested feature folders, adapt the examples above instead of forcing a `src/`-based structure.

---

## :zap: Use Case Pattern (Mandatory)

Every business operation is a dedicated `UseCase` class. Avoid shared mutation use cases.

### Signature Contract

```python
from hexcore.application.use_cases.base import UseCase
from hexcore.application.dtos.base import DTO

class CreateUserUseCase(UseCase["CreateUserCommand", "UserResponse"]):
    async def execute(self, command: CreateUserCommand) -> UserResponse:
        ...
```

- `UseCase` is generic: `UseCase[T, R]` where `T` is the input DTO and `R` is the output DTO.
- The only public method is `async def execute(self, command: T) -> R`.
- Business use cases must delegate business rules to a domain service.
- The use case orchestrates service calls, persistence, and DTO mapping.

### Full Business Use Case Example

```python
from uuid import UUID

from hexcore.application.dtos.base import DTO
from hexcore.application.use_cases.base import UseCase
from hexcore.infrastructure.uow import SqlAlchemyUnitOfWork


class CreateUserCommand(DTO):
    name: str
    email: str


class UserResponse(DTO):
    id: UUID
    name: str
    email: str


class CreateUserUseCase(UseCase[CreateUserCommand, UserResponse]):
    def __init__(self, service: UserService, uow: SqlAlchemyUnitOfWork) -> None:
        self.service = service
        self.uow = uow

    async def execute(self, command: CreateUserCommand) -> UserResponse:
        async with self.uow:
            user = await self.service.create_user(name=command.name, email=command.email)
            await self.uow.commit()
        return UserResponse(id=user.id, name=user.name, email=user.email)
```

### Query Use Case Pattern

Use this pattern for list, search, filter, sort, and pagination flows.

```python
from hexcore.application.dtos.query import QueryRequestDTO, QueryResponseDTO
from hexcore.application.use_cases.query import QueryEntitiesUseCase


class ListUsersUseCase(QueryEntitiesUseCase[User]):
    async def execute(self, command: QueryRequestDTO) -> QueryResponseDTO:
        return await super().execute(command)
```

Rules for query use cases:
- Prefer `QueryRequestDTO` and `QueryResponseDTO` for read endpoints that need search, filters, sort, or pagination.
- Never invent ad hoc dict payloads for query params when the query DTO already exists.
- Use `build_query_endpoint(...)` for simple FastAPI endpoints.
- Use `register_query_endpoint(...)` when you want to attach the endpoint directly to an `APIRouter`.
- If the repository implements `query_all(...)`, `BaseDomainService.list_entities(...)` should prefer that path.
- If the repository does not implement `query_all(...)`, `BaseDomainService.query_entities(...)` is the fallback.
- Query use cases are the only supported exception to the "do not inject repositories directly" rule.

### Domain Service Example

```python
from hexcore.domain.services import BaseDomainService


class UserService(BaseDomainService):
    def __init__(self, user_repo: IUserRepository) -> None:
        self._user_repo = user_repo
        super().__init__()

    async def create_user(self, name: str, email: str) -> User:
        user = User(name=name, email=email)
        user.register_event(UserCreatedEvent(entity_id=user.id))
        await self._user_repo.save(user)
        return user
```

---

## :floppy_disk: Repository Setup: SQL

### 1. ORM Model

```python
from hexcore.infrastructure.repositories.orms.sqlalchemy import BaseModel


class UserModel(BaseModel):
    __tablename__ = "users"

    name: str
    email: str
```

### 2. Concrete Repository Using `SQLAlchemyCommonImplementationsRepo`

`SQLAlchemyCommonImplementationsRepo` expects these properties from `HasBasicArgs`:

| Property | Type | Purpose |
| :--- | :--- | :--- |
| `entity_cls` | `type[T]` | Domain entity class |
| `model_cls` | `type[M]` | SQLAlchemy model class |
| `not_found_exception` | `type[Exception]` | Raised when an entity is not found |
| `fields_resolvers` | `FieldResolversType | None` | Async resolvers for model -> entity mapping |
| `fields_serializers` | `FieldSerializersType | None` | Custom serializers for entity -> model mapping |

```python
from hexcore.domain.uow import IUnitOfWork
from hexcore.infrastructure.repositories.implementations import SQLAlchemyCommonImplementationsRepo


class UserRepository(SQLAlchemyCommonImplementationsRepo[User, UserModel], IUserRepository):
    def __init__(self, uow: IUnitOfWork) -> None:
        super().__init__(uow)

    @property
    def entity_cls(self) -> type[User]:
        return User

    @property
    def model_cls(self) -> type[UserModel]:
        return UserModel

    @property
    def not_found_exception(self) -> type[Exception]:
        return UserNotFoundException

    @property
    def fields_resolvers(self) -> FieldResolversType | None:
        return None

    @property
    def fields_serializers(self) -> FieldSerializersType | None:
        return None
```

### 3. Unit of Work Setup for SQLAlchemy

`SqlAlchemyUnitOfWork` is built around an SQLAlchemy `AsyncSession`. Construct the session with `async_sessionmaker` and wire the UoW as a dependency.

```python
from sqlalchemy.ext.asyncio import async_sessionmaker, create_async_engine
from hexcore.config import LazyConfig


config = LazyConfig.get_config()
engine = create_async_engine(config.async_sql_database_url, echo=config.debug)
async_session_factory = async_sessionmaker(engine, expire_on_commit=False)
```

```python
from fastapi import Depends
from hexcore.infrastructure.uow import SqlAlchemyUnitOfWork


async def get_uow() -> AsyncGenerator[SqlAlchemyUnitOfWork, None]:
    async with async_session_factory() as session:
        yield SqlAlchemyUnitOfWork(session=session)
```

```python
from fastapi import APIRouter, Depends


router = APIRouter(prefix="/users", tags=["users"])


@router.post("/", response_model=UserResponse)
async def create_user(
    command: CreateUserCommand,
    use_case: CreateUserUseCase = Depends(get_create_user_use_case),
) -> UserResponse:
    return await use_case.execute(command)
```

---

## :floppy_disk: Repository Setup: Beanie

Use `BeanieODMCommonImplementationsRepo` for MongoDB-backed repositories.

### 1. Document Model

```python
from hexcore.infrastructure.repositories.orms.beanie import BaseDocument


class UserDocument(BaseDocument):
    name: str
    email: str
```

### 2. Concrete Repository

```python
from hexcore.infrastructure.repositories.implementations import BeanieODMCommonImplementationsRepo


class UserRepository(BeanieODMCommonImplementationsRepo[User, UserDocument], IUserRepository):
    def __init__(self, uow: IUnitOfWork) -> None:
        super().__init__(uow)

    @property
    def entity_cls(self) -> type[User]:
        return User

    @property
    def document_cls(self) -> type[UserDocument]:
        return UserDocument

    @property
    def not_found_exception(self) -> type[Exception]:
        return UserNotFoundException

    @property
    def fields_resolvers(self) -> FieldResolversType | None:
        return None

    @property
    def fields_serializers(self) -> FieldSerializersType | None:
        return None
```

### 3. NoSQL Event Tracking

Beanie repositories must track entities manually so the UoW can dispatch their domain events.

- Use `uow.collect_entity(entity)` when you save an entity that emitted events.
- Or use the `@register_entity_on_uow` decorator on repository save methods.

---

## :wrench: ServerConfig and LazyConfig

`LazyConfig.get_config()` resolves configuration in this order:

1. `HEXCORE_CONFIG_MODULE`
2. `HEXCORE_CONFIG_MODULES`
3. modules configured with `LazyConfig.set_config_modules(...)`
4. default module `config`

The default discovery target is the root `config.py`. If you use another module path, configure it explicitly.

### Recommended `ServerConfig`

```python
from pathlib import Path

from pydantic import ConfigDict

from hexcore.config import ServerConfig
from hexcore.domain.events import IEventDispatcher
from hexcore.infrastructure.cache import ICache
from hexcore.infrastructure.cache.cache_backends.memory import MemoryCache
from hexcore.infrastructure.events.events_backends.memory import InMemoryEventDispatcher


class ProjectConfig(ServerConfig):
    base_dir: Path = Path(".")
    host: str = "0.0.0.0"
    port: int = 8000
    debug: bool = False
    sql_database_url: str = "sqlite:///./db.sqlite3"
    async_sql_database_url: str = "sqlite+aiosqlite:///./db.sqlite3"
    mongo_database_url: str = "mongodb://localhost:27017"
    async_mongo_database_url: str = "mongodb+async://localhost:27017"
    mongo_db_name: str = "my_db"
    mongo_uri: str = "mongodb://localhost:27017/my_db"
    redis_uri: str = "redis://localhost:6379/0"
    redis_host: str = "localhost"
    redis_port: int = 6379
    redis_db: int = 0
    redis_cache_duration: int = 300
    allow_origins: list[str] = ["https://myapp.com"]
    allow_credentials: bool = True
    allow_methods: list[str] = ["*"]
    allow_headers: list[str] = ["*"]
    cache_backend: ICache = MemoryCache()
    event_dispatcher: IEventDispatcher = InMemoryEventDispatcher()
    model_config = ConfigDict(arbitrary_types_allowed=True)


config = ProjectConfig()
```

For production, swap `MemoryCache` for `RedisCache` and replace `InMemoryEventDispatcher` with a real broker-backed dispatcher.

---

## :hammer_and_wrench: Additional Implementation Guidelines

### Use of Resolvers

When converting models or documents to entities with nested relationships:

- Use `FieldResolversType` for async model/document -> entity attribute mapping.
- Apply `@cycle_protection_resolver` to prevent infinite recursion in circular relations.
- Use `to_entity_from_model_or_document` from `hexcore.infrastructure.repositories.utils` as the central conversion utility.
- Pass `is_nosql=True` when converting Beanie documents.

### Query Stack

- Accept queries as `QueryRequestDTO` at the application boundary.
- Prefer repository pushdown via `query_all(...)` for SQLAlchemy and Beanie repositories.
- Fall back to `BaseDomainService.query_entities(...)` only when the repository does not implement `query_all(...)`.
- Validate invalid filter, sort, and search field names in the API layer and surface them as HTTP 422.
- Keep parser behavior operator-aware: `IN` and `NOT_IN` may split comma-separated values; text operators must preserve the raw string.
- Prefer explicit search fields when available; infer them only as a fallback.

### Folder-Agnostic Architecture

HexCore projects may live in different layouts. Do not assume a fixed `src/` tree or a legacy package path.

- Discover repositories, modules, and configuration from the actual workspace layout.
- Prefer explicit configuration over hard-coded folder assumptions.
- For repository discovery, honor configured discovery paths first.
- Do not introduce legacy fallback behavior unless compatibility is explicitly requested.
- Keep imports aligned with the discovered package root and avoid inventing paths.
- For CLI and bootstrap flows, generate structures that work in both hexagonal and vertical-slice layouts.

### Unit of Work Logic

- `SqlAlchemyUnitOfWork` automatically tracks entities via `session.new`, `session.dirty`, and `session.deleted` once `set_domain_entity()` has been used on the ORM model.
- `NoSqlUnitOfWork` requires explicit entity tracking via `uow.collect_entity(entity)` or `@register_entity_on_uow`.
- UoW commit is the place where events are dispatched; application code should not replicate that workflow.

### Event-Driven Architecture

1. Entities emit `DomainEvent` subclasses through `BaseEntity`.
2. Use cases trigger domain behavior and then call `await uow.commit()`.
3. The UoW dispatches collected events after commit.
4. Domain events must remain infrastructure-agnostic.
5. Configure the dispatcher in `ServerConfig.event_dispatcher`.

### Caching

- Use the `ICache` interface.
- Swap `MemoryCache` for `RedisCache` without changing application code.
- Configure the active backend in `ServerConfig.cache_backend`.
- Inject cache through dependency injection; do not instantiate it inside domain or application layers.

---

## :no_entry: Blacklist (Hard Prohibitions)

| Prohibition | Reason |
| :--- | :--- |
| Re-declaring `id`, `created_at`, `updated_at`, or `is_active` in entity subclasses | Already provided by `BaseEntity` |
| Injecting repositories directly into business `UseCase` classes | Business use cases should depend on domain services plus a UoW |
| Treating the query use case exception as a rule for mutation use cases | `QueryEntitiesUseCase` is a read-side helper only |
| Calling `session.commit()` outside a `UnitOfWork` | Breaks transactional integrity |
| Calling `repo.save()` outside an `async with uow:` block | Leaves changes untracked and uncommitted |
| Manually calling `dispatch_events()` or `collect_domain_events()` from application code | The UoW already owns commit-time event dispatch |
| Reimplementing `get_by_id`, `list_all`, `save`, or `delete` in a concrete repo | Already implemented by the base class |
| Instantiating domain events manually inside a `UseCase` | Events should be emitted by entities and persisted through the UoW flow |
| Assuming `hexcore.infrastructure.cache.cache_backends` reexports the concrete cache classes | Import the concrete backend modules directly |

---

## :rocket: CLI Commands

```bash
hexcore init                   # Scaffold a new project
hexcore create-domain-module   # Generate 7 standard files for a new domain module
hexcore make-migrations        # Generate Alembic migration scripts
hexcore migrate                # Apply pending database migrations
hexcore test                   # Run pytest suite
```

`hexcore init` supports the `hexagonal` and `vertical-slice` templates.

---

## :gear: Development Workflow

1. Define the entity in `domain/{module}/entities.py`.
2. Define the repository interface in `domain/{module}/repositories.py`.
3. Define the domain service in `domain/{module}/services.py`.
4. Create input and output DTOs in `application/{module}/dtos.py`.
5. Implement one `UseCase` per business operation in `application/{module}/use_cases/`.
6. Use `QueryEntitiesUseCase` for read/list/search/filter/sort/pagination flows.
7. Create `BaseModel` or `BaseDocument` in infrastructure.
8. Implement a concrete repository using `SQLAlchemyCommonImplementationsRepo` or `BeanieODMCommonImplementationsRepo`.
9. Configure `config = ServerConfig(...)` in the root `config.py` or another module selected by `LazyConfig`.
10. Wire dependencies in FastAPI router factories.
11. Run `hexcore make-migrations` and `hexcore migrate` when the schema changes.
12. Write tests for domain behavior, repository discovery, and query flows.
13. Run `hexcore test`.

---

## :mag: Validation Checklist for the Agent

Before providing code, verify:

- [ ] Do all imports match the Import Registry exactly?
- [ ] Is custom domain code free of unnecessary Application or Infrastructure imports?
- [ ] Is every business operation a dedicated `UseCase` class with `execute(command: InputDTO) -> OutputDTO`?
- [ ] Does the business use case inject only a domain service plus a UoW?
- [ ] Does the use case delegate business logic to a domain service?
- [ ] Does the use case return a DTO and not a `BaseEntity`?
- [ ] For query flows, is `QueryRequestDTO` used at the boundary and `QueryEntitiesUseCase` used appropriately?
- [ ] Are UoW writes wrapped in `async with uow:`?
- [ ] Does the UoW commit path own event dispatch rather than the application layer?
- [ ] Does the entity subclass avoid redeclaring base fields?
- [ ] Does the repository avoid reimplementing base CRUD methods?
- [ ] Does the repository declare the exact property names `entity_cls`, `model_cls` or `document_cls`, `not_found_exception`, `fields_resolvers`, and `fields_serializers`?
- [ ] Does the repository inherit from the correct common implementation class for SQL or NoSQL?
- [ ] Is `SqlAlchemyUnitOfWork` built from an `AsyncSession` created by `async_sessionmaker`?
- [ ] If using Beanie, is `is_nosql=True` passed to the entity converter?
- [ ] For NoSQL repositories, are entities tracked with `uow.collect_entity()` or `@register_entity_on_uow`?
- [ ] Is `config = ServerConfig(...)` defined in the default root module or a module resolved by `LazyConfig`?
- [ ] Is the configured event dispatcher appropriate for the target environment?

---
> Source: [Indroic/hexcore-skill](https://github.com/Indroic/hexcore-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
