## fastapi-clean

> >


# FastAPI Clean Architecture — Pragmático

Este skill define cómo estructurar proyectos **FastAPI** separando lo que vale la pena separar (lógica de negocio, servicios externos, persistencia testeable) sin caer en arquitectura limpia ortodoxa que duplica entidades, mete mappers en cada capa, exige Unit of Work y termina siendo más ceremonia que valor.

**No es arquitectura limpia purista.** Es FastAPI + casos de uso + inyección de dependencias selectiva + capas solo donde pagan. Suficiente para integraciones complejas, multi-disparador (HTTP + Celery + management command) y tests serios, sin volverse engorroso.

## Cuándo usar este skill

Disparalo cuando el usuario pida **crear, refactorizar o explicar**:

- Un endpoint, router o módulo FastAPI nuevo.
- Un caso de uso (`UseCase`) o flujo de aplicación.
- Un repositorio (SQLAlchemy/SQLModel o PyMongo async).
- Una task Celery 5.4 que llame lógica de negocio.
- Dependencias (`Depends`), `lifespan`, settings, schemas Pydantic.
- Auth con Clerk JWT, API keys SHA-256, o ambos.
- Separar lógica que hoy vive mezclada en routers, signals, tasks o handlers.

También aplica cuando el usuario habla de "arquitectura por capas", "ports & adapters", "desacoplar FastAPI", "testear sin tocar la DB", o muestra código con lógica mezclada en routers.

## Filosofía

Cuatro decisiones que definen este enfoque:

1. **FastAPI es la capa de entrega, no el centro.** Routers, `Depends`, `HTTPException` y `response_model` viven en la API layer; la lógica de negocio NO.
2. **Pydantic vive en la frontera HTTP, no en el dominio.** Los schemas validan requests/responses; las entidades de dominio son `@dataclass` puros (cuando paga separarlos).
3. **Los servicios externos (HTTP, colas, terceros) van detrás de un Protocol** — ahí es donde la abstracción paga, porque es lo que duele en tests. ORM con BD de tests muchas veces puede quedar concreto.
4. **Excepciones de dominio, no HTTPException, dentro de use cases.** El mapeo a HTTP vive en `@app.exception_handler`.

Regla mental para decidir si algo va detrás de Protocol: **"¿En los tests me duele tener la implementación real?"**. Si sí (red, secrets, latencia, errores impredecibles) → Protocol + fake. Si no (ORM con factories y BD de tests) → clase concreta, abstraer cuando duela.

## Cuándo aplicar el patrón completo

Aplicá **Domain + Application + Infrastructure + API** cuando se cumple al menos uno:

- **Integración con un servicio externo** (API REST, webhook saliente, cola).
- La operación se dispara desde **múltiples lugares** (router + Celery + command).
- Hay **reglas de negocio no triviales** que vale testear sin levantar FastAPI ni tocar la red.
- Orquesta **varias fuentes** (DB + HTTP + cache + cola) en una sola operación.

**No apliques el patrón** para CRUD plano. Si tu endpoint es "lista, crea, actualiza un modelo SQLModel sin reglas", un router con `SQLModel` directo + `response_model` resuelve más rápido y más claro. Forzar capas vacías sin valor agrega ruido.

## Modelo de capas

| Capa | Responsabilidad | Qué SÍ tiene | Qué NO tiene |
|------|-----------------|--------------|--------------|
| **Domain** | Entidades, value objects, reglas de dominio, excepciones, ports | Python puro, dataclasses, Protocols | `fastapi`, ORM, `HTTPException`, BSON, `Request` |
| **Application** | Use cases, DTOs (commands/queries), errores de aplicación | Depende de Protocols de domain | Implementaciones concretas, `HTTPException`, status codes |
| **Interface / API** | Routers, schemas Pydantic, deps, exception handlers, auth deps | `APIRouter`, `Depends`, `response_model`, `BackgroundTasks` | Lógica de negocio, queries ORM directas |
| **Infrastructure** | Implementación de repos, clientes HTTP, tasks Celery, mongo_indexes | SQLAlchemy/PyMongo, `requests`, `Celery` | Reglas de negocio |
| **Composition root** | `main.py`, `lifespan`, wiring | Instanciación de clientes, registro de handlers | — |

## Estructura recomendada

Feature-modules: organizar por dominio, no por tipo de archivo. Cada módulo es autónomo.

```
app/
├── main.py                              # FastAPI app + register_exception_handlers + lifespan
├── workers/
│   └── celery_app.py                    # composition root de Celery (separado de FastAPI)
├── core/
│   ├── config.py                        # Settings (pydantic-settings) + get_settings()
│   ├── exceptions.py                    # ApplicationError, PermissionDenied, ValidationError
│   └── security.py                      # helpers crypto (api_key hash, compare_digest)
├── api/
│   ├── deps.py                          # providers: get_db, get_<feature>_repository, get_<use_case>
│   ├── exception_handlers.py            # register_exception_handlers(app)
│   ├── auth/
│   │   ├── clerk_auth.py                # get_user_principal (Clerk JWT)
│   │   └── api_key_auth.py              # get_service_principal (API key SHA-256)
│   └── v1/
│       ├── users.py                     # APIRouter
│       └── projects.py
└── modules/
    └── users/
        ├── domain/
        │   ├── entities.py              # User dataclass
        │   ├── value_objects.py
        │   ├── exceptions.py            # UserNotFound, UserAlreadyExists
        │   └── ports.py                 # UserRepository (Protocol)
        ├── application/
        │   ├── dto.py                   # CreateUserCommand, UserResult
        │   ├── use_cases.py             # CreateUserUseCase, GetUserUseCase
        │   └── tests/test_use_cases.py
        ├── infrastructure/
        │   ├── repositories.py          # MongoUserRepository | SqlAlchemyUserRepository
        │   ├── mongo_indexes.py         # ensure_user_indexes(db)
        │   └── celery_tasks.py          # tasks como adapters
        └── schemas.py                   # UserCreateSchema, UserResponse, UserUpdateSchema
```

Para proyectos chicos (3-4 endpoints), todo en `app/main.py` + `app/models.py` puede ser suficiente. La estructura modular paga a partir de ~3 features distintos.

## Dependency Injection

`Depends()` es para **wiring de framework**, no para esconder lógica de negocio.

Usá `Depends` para:

- Conexiones DB / sesiones ORM
- `AuthenticatedPrincipal` (después de verificar Clerk o API key)
- Settings cacheados (`get_settings`)
- Repositorios concretos
- Use cases ya compuestos

`yield` para cleanup de recursos:

```python
async def get_sql_session(request: Request):
    async with request.app.state.session_maker() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
```

`dependency_overrides` en tests (ver `references/testing.md`).

Antipatrón:

```python
# MAL
@router.post("/users")
def create_user(data: UserCreate, session: Session = Depends(get_session)):
    # validación, queries ORM, envío de email, llamada a Clerk... TODO acá
```

Bien:

```python
# BIEN
@router.post("/users")
async def create_user(
    payload: UserCreateSchema,
    use_case: CreateUserUseCase = Depends(get_create_user_use_case),
    principal: AuthenticatedPrincipal = Depends(get_user_principal),
) -> UserResponse:
    result = await use_case.execute(payload.to_command(), principal=principal)
    return UserResponse.model_validate(result)
```

## Capa Domain

La capa más estable. Solo Python puro.

**Sí**:

- Entidades (`@dataclass(frozen=True)`)
- Value objects
- Excepciones de dominio (`UserNotFound`, `InvalidEmail`)
- Servicios de dominio (lógica que no encaja en una entidad)
- Ports / repositorios como `Protocol`

**No**:

- `from fastapi import ...`
- `from sqlalchemy import ...` / `from sqlmodel import ...`
- `from pymongo import ...` / `from bson import ObjectId`
- `from pydantic import BaseModel` (entidades NO son Pydantic models en este patrón)
- `HTTPException`, status codes, `Request`, `Response`

Regla fuerte: **nunca lanzar `HTTPException` desde domain o application**. Lanzar excepciones de dominio y mapearlas en la API layer.

## Pydantic schemas vs entidades

Cinco modelos distintos, cada uno con su rol:

```text
UserCreateSchema      # request HTTP (Pydantic, EmailStr, validation rules)
UserResponse          # response HTTP (Pydantic, model_config from_attributes)
CreateUserCommand     # DTO para application (dataclass)
User                  # entidad de dominio (dataclass frozen, validación de invariantes)
UserORM / UserDoc     # representación de persistencia (SQLAlchemy ORM o dict Mongo)
```

Regla: **no usar schemas de API ni modelos ORM como entidades de dominio**. Mappers explícitos en el repositorio.

¿Es overkill para CRUD trivial? Sí. Por eso este patrón se aplica **cuando paga** (ver "Cuándo aplicar el patrón completo").

## Repositorios

Application define el Protocol. Infrastructure lo implementa.

```python
# domain/ports.py
from typing import Protocol


class UserRepository(Protocol):
    async def get_by_id(self, user_id: str) -> User | None: ...
    async def get_by_email(self, email: str) -> User | None: ...
    async def save(self, user: User) -> None: ...
```

```python
# infrastructure/repositories.py
class MongoUserRepository:
    def __init__(self, db: AsyncDatabase):
        self._collection = db["users"]

    async def get_by_id(self, user_id: str) -> User | None: ...
```

Pragmatismo: si hay UNA sola implementación y nunca se mockea (ej. SQLAlchemy con BD de tests), está bien empezar concreto. Extraé el Protocol cuando aparezca una segunda implementación o cuando los tests pidan un fake.

Ver `references/pymongo_async.md` para el patrón completo con Mongo.

## Testing

Tres niveles:

```text
Unit tests        -> use cases con fake del Protocol. SIN FastAPI TestClient.
Integration tests -> rutas con TestClient + dependency_overrides + test DB.
Contract tests    -> adaptadores HTTP/colas contra servicio real o WireMock.
```

Regla: **preferí testear use cases directamente**. No todos los tests pasan por HTTP.

```python
# tests del use case: rápidos, sin red, sin DB
class FakeUserRepository:
    def __init__(self):
        self._store: dict[str, User] = {}

    async def get_by_email(self, email: str) -> User | None:
        return next((u for u in self._store.values() if u.email == email), None)

    async def save(self, user: User) -> None:
        self._store[user.id] = user


async def test_create_user_duplicate_raises():
    use_case = CreateUserUseCase(repo=FakeUserRepository())
    await use_case.execute(CreateUserCommand(email="x@y.com", name="X"))

    with pytest.raises(UserAlreadyExists):
        await use_case.execute(CreateUserCommand(email="x@y.com", name="X"))
```

Detalles en `references/testing.md` (fixtures, `dependency_overrides`, test DBs con Mongo/SQL).

## Settings

`pydantic-settings` + `@lru_cache` + acceso vía dependencia.

```python
from functools import lru_cache
from pydantic_settings import BaseSettings


class Settings(BaseSettings):
    mongo_uri: str
    environment: str = "local"


@lru_cache
def get_settings() -> Settings:
    return Settings()
```

En tests, sobrescribí con `app.dependency_overrides[get_settings] = lambda: Settings(...)`.

## Lifespan

Inicialización de recursos costosos (DB clients, JWKS, modelos ML) va en `lifespan`, no en import time.

```python
@asynccontextmanager
async def lifespan(app: FastAPI):
    client = AsyncMongoClient(settings.mongo_uri, ...)
    await client.admin.command("ping")
    app.state.db = client[settings.mongo_db_name]
    await ensure_user_indexes(app.state.db)
    try:
        yield
    finally:
        await client.close()
```

No uses los antiguos `@app.on_event("startup")` — están deprecados.

## Async

Regla:

```text
async def en rutas que llaman I/O awaitable (httpx, pymongo async, sqlalchemy async).
def normal cuando la librería es bloqueante y no soporta await.
NO llamar I/O bloqueante dentro de async def (requests, time.sleep, drivers sync).
```

Para CPU-heavy o long-running: worker process o task queue, no bloqueés el event loop.

## Background tasks vs Celery

`BackgroundTasks` de FastAPI solo para trabajo **chico y no crítico** post-respuesta (logging ligero, email simple).

Para todo lo demás (jobs largos, retries, idempotencia, schedules, multi-host): **Celery**.

```text
Pierdo la tarea sin consecuencia        -> BackgroundTasks
Necesito retry, idempotencia, schedule  -> Celery
Trabajo en otro proceso/host            -> Celery
Trabajo crítico que no puede perderse   -> Celery (con result backend o transactional outbox)
```

Detalles de Celery 5.4 en `references/celery_5_4.md` (idempotencia, retries, queues, async dentro de Celery).

## Seguridad

Auth en la API/security layer. Autorización de **negocio** en application/domain.

```text
Verificación JWT / API key        -> dependencia FastAPI (puede HTTPException 401)
Extraer AuthenticatedPrincipal    -> después de verificar
Reglas "X no puede editar Y"      -> use case con principal: AuthenticatedPrincipal
```

Dos referencias separadas:

- `references/clerk_jwt.md` — verificación RS256, JWKS, `AuthenticatedPrincipal`
- `references/api_keys_sha256.md` — generación con `secrets`, hash SHA-256 / HMAC, `hmac.compare_digest`

## Manejo de errores

```python
# application/use_cases.py
class CreateUserUseCase:
    async def execute(self, command: CreateUserCommand):
        if await self.repo.get_by_email(command.email):
            raise UserAlreadyExists(command.email)
        ...
```

```python
# api/exception_handlers.py
@app.exception_handler(UserAlreadyExists)
async def handler(request, exc: UserAlreadyExists):
    return JSONResponse(status_code=409, content={"detail": "User already exists"})
```

**Nunca** `raise HTTPException(...)` dentro de un use case. Detalles en `references/error_handling.md`.

## Persistencia: SQL o Mongo

FastAPI no obliga a usar una DB específica. Elegí según el caso:

| | SQLAlchemy / SQLModel | PyMongo async |
|--|----------------------|---------------|
| Schema rígido | ✅ | parcial |
| Multi-doc transactions | ✅ | ✅ (con replica set) |
| Flexibilidad de schema | menor | mayor |
| FastAPI integration | excelente (SQLModel) | excelente con AsyncDatabase |

Para Mongo: ver `references/pymongo_async.md` (Motor está deprecado, usá `AsyncMongoClient`).

Para SQL: sessions vía `yield`, `response_model` en endpoints, mappers `_to_domain/_to_orm` en el repo cuando hay separación domain/orm.

## Workflow

Dos modos según el pedido:

| Modo | Cuándo usarlo |
|------|----------------|
| **Single-agent** | Agregar un endpoint, ajustar un repo, refactor parcial, explicación, review. El agente lee este SKILL y trabaja secuencialmente. |
| **Multi-subagent** | El usuario pide el **módulo completo** (entities + use cases + repositorio + router + tests + wiring). El orquestador delega a **4 subagentes en paralelo** y sintetiza. |

Si el alcance es ambiguo: **preguntá** antes de paralelizar.

## Workflow multi-subagent

Disparalo cuando el usuario pide generar un módulo **completo**. Para refactors puntuales o agregar una sola pieza, usá single-agent.

### 1. Recolectar requisitos (orquestador)

Antes de delegar, asegurate de tener (preguntá lo que falte):

- [ ] **Paquete Python base** (ej. `app.modules.users`).
- [ ] **Persistencia**: SQL o Mongo. **Una sola** por módulo.
- [ ] **Auth mode**: `clerk_jwt`, `api_key`, mixto, o ninguno (servicio interno).
- [ ] **Nombre de la entidad principal** y campos clave (ej. `User(clerk_user_id, email, name)`).
- [ ] **Use cases requeridos** (ej. `CreateUser`, `GetUserByClerkId`, `UpdateUser`).
- [ ] **¿Hay tasks Celery?** Si sí, cuáles (ej. `send_welcome_email`).
- [ ] **Contrato compartido** acordado por escrito para los 4 subagentes: nombres de clases, paths, excepciones de dominio (lista cerrada), resultado, transacciones.

Sin contrato, los subagentes inventan nombres distintos y la síntesis falla.

### 2. Lanzar 4 subagentes en paralelo

| Subagente | Genera | Path |
|-----------|--------|------|
| **A — Domain** | `entities.py`, `value_objects.py`, `exceptions.py`, `ports.py` | `app/modules/<feature>/domain/` |
| **B — Application** | `dto.py`, `use_cases.py`, `tests/test_use_cases.py` (con fake del Protocol) | `app/modules/<feature>/application/` |
| **C — Infrastructure** | `repositories.py`, `mongo_indexes.py` (si Mongo), `celery_tasks.py` (si aplica) | `app/modules/<feature>/infrastructure/` |
| **D — API** | `schemas.py`, `routers/<feature>.py`, providers en `deps.py`, handlers en `exception_handlers.py` | `app/modules/<feature>/schemas.py` + `app/api/v1/<feature>.py` |

Cada subagente recibe el **contrato compartido** completo + prompt específico (ver `references/multi_subagent_workflow.md`).

### 3. Esperar a que los 4 completen

No sintetices hasta tener las 4 salidas. Si uno falla, relanzá solo ese subagente con el error.

### 4. Síntesis (orquestador, NO un subagente)

Verificá consistencia:

- Las excepciones que el use case captura (B) **existen** en `exceptions.py` (A) con los mismos nombres.
- Los métodos del repositorio que el use case llama (B) **existen** en la implementación concreta (C) con firmas que coinciden.
- El Protocol de A es implementado completamente por C.
- `get_<use_case>` en `deps.py` (D) construye `<Engine><Entity>Repository(db)` con los nombres reales generados (C).
- El router (D) convierte `XSchema → XCommand` y pasa el `Command` al use case (no `Request`, no payload raw).
- Tests (B) usan un fake que implementa el Protocol completo (no `MagicMock` suelto).

Si hay conflicto: corrección inline si es trivial, o re-delegación quirúrgica al subagente que desvió del contrato (citando el conflicto concreto).

### 5. Presentar al usuario

Mini-mapa de archivos tocados/creados + resumen de responsabilidades. Si aplica, linkear `references/multi_subagent_workflow.md` como referencia de protocolo.

## Sub-Agent Prompt Templates

Pegá estos prompts (después del contrato compartido) a cada subagente. Versión completa en `references/multi_subagent_workflow.md`.

### A — Domain

```
Generá la capa Domain del módulo <feature> en app/modules/<feature>/domain/:
- entities.py: dataclass(frozen=True) con validación en __post_init__.
- value_objects.py (si aplica).
- exceptions.py: jerarquía de excepciones de dominio (lista del contrato).
- ports.py: Protocol <Entity>Repository con métodos async que el use case necesita.

NO importes fastapi, sqlalchemy, sqlmodel, pymongo, bson, pydantic.BaseModel.
NO uses HTTPException ni status codes.
Devolvé código completo, sin TODOs.
```

### B — Application

```
Generá la capa Application del módulo <feature> en app/modules/<feature>/application/:
- dto.py: dataclasses para commands/queries (del contrato).
- use_cases.py: clases con __init__(repo: <Entity>Repository) y async def execute(command) -> result.
- tests/test_use_cases.py: tests pytest-asyncio con FakeRepository implementando el Protocol completo.

Importá Protocols de domain, NO implementaciones de infrastructure.
Lanzá excepciones de domain (de exceptions.py), NUNCA HTTPException.
Tests siguen AAA (Arrange/Act/Assert con líneas en blanco entre fases).
```

### C — Infrastructure

```
Generá la capa Infrastructure del módulo <feature> en app/modules/<feature>/infrastructure/:
- repositories.py: <Engine><Entity>Repository implementando el Protocol de domain/ports.py.
- celery_tasks.py (si aplica): tasks como adapters delgados que llaman al use case.
- mongo_indexes.py (solo si Mongo): ensure_<feature>_indexes(db) para correr en lifespan.

Mappers explícitos: _to_domain(doc_o_row) -> Entity, _to_persistence(entity) -> doc_o_row.
ObjectId, columnas, operadores Mongo viven SOLO acá.
En Celery: autoretry_for solo errores transitorios; idempotency_key en tasks críticas; asyncio.run para use cases async.
```

### D — API

```
Generá la capa API del módulo <feature>:
- app/modules/<feature>/schemas.py: <Entity>CreateSchema, <Entity>Response, <Entity>UpdateSchema (Pydantic v2).
- app/api/v1/<feature>.py: APIRouter con endpoints DELGADOS que inyectan use cases via Depends, convierten schema -> command, llaman use_case.execute, retornan Response.
- app/api/deps.py: providers get_<feature>_repository, get_<use_case> con Depends correctos.
- app/api/exception_handlers.py: handlers que mapean excepciones de domain a JSONResponse con status code.

response_model=<Entity>Response en cada endpoint.
Auth: inyectá AuthenticatedPrincipal según el modo del contrato.
Los endpoints NO contienen lógica de negocio.
```

## Checklist final

### Arquitectura

```text
[ ] Domain NO importa fastapi, sqlalchemy, sqlmodel, pymongo, bson, pydantic.BaseModel.
[ ] Routers son DELGADOS: validar + inyectar use case + responder.
[ ] Use cases contienen los workflows; reciben Protocols, no implementaciones.
[ ] Infrastructure implementa los Protocols.
[ ] Schemas API, DTOs application, entities domain, modelos persistencia: cuatro modelos distintos.
[ ] HTTPException solo en API layer (dependencias auth, exception handlers).
```

### FastAPI

```text
[ ] APIRouter para rutas modulares (app/api/v1/<feature>.py).
[ ] Depends para wiring de framework, no para esconder lógica.
[ ] dependency_overrides en tests.
[ ] yield dependencies para cleanup de DB/session.
[ ] lifespan para startup/shutdown (no @app.on_event).
[ ] pydantic-settings + @lru_cache para config.
[ ] response_model en cada endpoint para validación/filtrado de output.
[ ] register_exception_handlers(app) en main.py.
```

### Async / performance

```text
[ ] Rutas async solo llaman I/O awaitable.
[ ] SDKs bloqueantes no se llaman directo en async def (mover a threadpool o worker).
[ ] CPU-heavy va a workers/procesos.
[ ] BackgroundTasks solo para trabajos no críticos.
```

### Testing

```text
[ ] Unit tests de use cases sin HTTP, con fake del Protocol.
[ ] Fakes implementan Protocol completo (no MagicMock suelto).
[ ] Tests siguen AAA.
[ ] Integration tests con TestClient + dependency_overrides + test DB.
[ ] Cleanup de overrides con app.dependency_overrides.clear().
```

### Celery 5.4 (si aplica)

Ver `references/celery_5_4.md` para el checklist completo.

### PyMongo async (si aplica)

Ver `references/pymongo_async.md` para el checklist completo.

### Clerk JWT (si aplica)

Ver `references/clerk_jwt.md` para el checklist completo.

### API keys SHA-256 (si aplica)

Ver `references/api_keys_sha256.md` para el checklist completo.

## Referencias

- `references/multi_subagent_workflow.md` — Workflow multi-subagent + contrato + ejemplo end-to-end
- `references/celery_5_4.md` — Tasks idempotentes, retries, queues, async dentro de Celery
- `references/pymongo_async.md` — AsyncMongoClient, repos por Protocol, índices, transacciones
- `references/clerk_jwt.md` — Verificación RS256, JWKS, `AuthenticatedPrincipal`
- `references/api_keys_sha256.md` — Generación con `secrets`, hash SHA-256/HMAC, `compare_digest`
- `references/testing.md` — `dependency_overrides`, fakes de Protocol, test DBs
- `references/error_handling.md` — Excepciones de dominio → HTTP handlers

Templates en `assets/` listos para copiar y reemplazar `<Feature>`, `<Entity>`, `<feature_snake>`.

## Fuentes principales

- [The Clean Architecture - Robert C. Martin](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [FastAPI - Bigger Applications](https://fastapi.tiangolo.com/tutorial/bigger-applications/)
- [FastAPI - Dependencies](https://fastapi.tiangolo.com/tutorial/dependencies/)
- [FastAPI - Testing Dependencies](https://fastapi.tiangolo.com/advanced/testing-dependencies/)
- [FastAPI - Lifespan Events](https://fastapi.tiangolo.com/advanced/events/)
- [FastAPI - Settings](https://fastapi.tiangolo.com/advanced/settings/)
- [Celery 5.4 - Tasks](https://docs.celeryq.dev/en/v5.4.0/userguide/tasks.html)
- [MongoDB PyMongo Driver](https://www.mongodb.com/docs/languages/python/pymongo-driver/current/)
- [Clerk - Manual JWT verification](https://clerk.com/docs/guides/sessions/manual-jwt-verification)
- [OWASP REST Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html)
- [fastapi-best-practices](https://github.com/zhanymkanov/fastapi-best-practices)

---
> Source: [fulit103/fastapi-clean](https://github.com/fulit103/fastapi-clean) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
