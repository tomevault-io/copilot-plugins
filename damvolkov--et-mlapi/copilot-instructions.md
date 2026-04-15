## et-mlapi

> Python coding standards. Typing, imports, control flow, performance.


# Python Standards

Target: Python ≥3.13. Async by default for all I/O.

## Imports

- Absolute only. No relative.
- Top-level only. No function-scope.
- No circular — use TYPE_CHECKING block.
- No __init__.py unless structural.

## Typing (STRICT — everything typed, always)

EVERYTHING is strictly typed. No exceptions. No `Any` unless interfacing with untyped third-party.

### Static & Runtime

- `ty` (Astral) for static analysis. `beartype` for runtime enforcement.
- `@decorate_methods(beartype)` on classes with business logic (stores, pipelines, processors).
- Never `from unittest.mock import Mock/AsyncMock` — breaks type safety.

### Modern Syntax (Python ≥3.13)

- Union: `str | None`, `int | float` — never `Optional[X]` or `Union[X, Y]`.
- Builtins: `list[int]`, `dict[str, Any]`, `tuple[str, ...]` — never `List`, `Dict`, `Tuple`.
- `Self` for return types in classmethods/validators.
- `Annotated[T, metadata]` for semantic markers + validation combined.

### Generics (PEP 695)

- New syntax always: `class Store[D: Node]:` not `class Store(Generic[D]):`.
- `type` statement for aliases: `type FieldType = Literal["tag", "text"]`.
- Bound TypeVars: `[D: Node]`, `[T_in, T_out]`, `[TRaw, TNode: Node]`.
- Each subclass propagates generics: `class KnowledgeStore[D: Node](BaseStore[D]):`.

### TypeVars & ParamSpec

- Single letter or `T`-prefixed, always bound: `D = TypeVar("D", bound=Node)`.
- `ParamSpec("P")` + `TypeVar("T")` for decorator signatures that preserve types.
- Coroutine types: `Callable[[BackgroundTask[Any]], Coroutine[Any, Any, None]]`.

### Annotated Types

- Compose validation + serialization + semantic markers in one declaration.
- `type TagField = Annotated[str, "tag"]` — semantic marker for schema generation.
- `type RedisUrlStr = Annotated[str, AfterValidator(lambda v: str(RedisDsn(v)))]` — validated alias.
- `type RedisList = Annotated[list[str], BeforeValidator(ensure_list), PlainSerializer(ensure_str)]` — bidirectional.

### ClassVar

- `ClassVar[T]` for constants that must NOT be loaded from env (pydantic-settings).
- `ClassVar[re.Pattern[str]]` for pre-compiled regex at class level.
- `ClassVar[dict[str, type]]` for registries.

### Protocol

- `Protocol` with `@runtime_checkable` for structural subtyping — over ABCs when possible.
- Define interface contracts without inheritance: adapters, stores, strategies.

### @overload

- Use `@overload` for functions with different return types based on input types.
- Common: decorators that work with/without args, sync/async dispatch.

### Type Reflection

- `get_origin()`, `get_args()` for runtime type introspection.
- `UnionType` / `Union` handling for schema generation.
- `Annotated` args extraction for metadata-driven code gen.

## Control Flow

### Hard Rules

- **Nested if-else: STRICTLY FORBIDDEN.** Always. No exceptions. Flatten with early returns, guard clauses, or dispatch.
- **>2 branches: NEVER if-else.** The moment there are 3+ options, use one of the dispatch patterns below.
- **Binary (2 options):** simple `if/else`, ternary, or guard clause is acceptable.
- Walrus operator (`:=`) everywhere possible — positive and negative checks.

### Dispatch Hierarchy (choose by complexity)

| Scenario | Pattern | Where to declare |
|----------|---------|-----------------|
| Simple/flat value dispatch, each branch is 1-2 lines | `match-case` | Inline in function |
| Complex logic, multiple entities, many branches | Dict/map dispatcher | Module-level or class-level constant |
| Different input argument types drive different logic | `plum-dispatch` | Typed `@dispatch` overloads |

### 1. `match-case` — simple value/shape dispatch

Default for ≥3 flat branches. Each `case` body is short (1-3 lines).

```python
match query_type:
    case "vector": return VectorQuery(...)
    case "hybrid": return HybridQuery(...)
    case "keyword": return KeywordQuery(...)
```

### 2. Dict/map dispatcher — complex multi-entity dispatch

When each branch involves significant logic or the mapping is reused. Declare the map at **module or class top level**, not inline.

```python
##### DISPATCHERS #####

_PIPELINE_HANDLERS: dict[str, Callable[[PipelineContext], Coroutine]] = {
    "ingest": handle_ingest,
    "search": handle_search,
    "delete": handle_delete,
}

async def dispatch_pipeline(action: str, ctx: PipelineContext) -> Result:
    handler = _PIPELINE_HANDLERS[action]
    return await handler(ctx)
```

### 3. `plum-dispatch` — typed multi-dispatch

When the dispatch dimension is the **type** of the input arguments, not a value. Async-compatible.

```python
from plum import dispatch

@dispatch
async def process(data: TextChunk) -> ProcessedChunk:
    return await _process_text(data)

@dispatch
async def process(data: ImageChunk) -> ProcessedChunk:
    return await _process_image(data)

@dispatch
async def process(data: AudioChunk) -> ProcessedChunk:
    return await _process_audio(data)
```

### Guard Clauses & Walrus

```python
if (result := await redis.get(key)) is not None:
    return process(result)

if (error := validate(data)) is None:
    return save(data)
```

## Conventions

- Enumerable options → `StrEnum` (or `IntEnum` / `Enum` for non-str) with `auto()`. Default choice.
- `Literal` ONLY for: inline dispatchers, small non-reusable option sets, or type narrowing in signatures.
- @functools.cache / lru_cache on pure functions.
- `import orjson as json` — never stdlib `json`.
- Comprehensions, map/filter over raw loops.
- One-liner docstrings. No inline comments. Big O when non-obvious.
- Config: never `os.environ`, `os.getenv`, `python-dotenv`. Always `pydantic-settings`.
- Settings import: ALWAYS `from <package>.settings import settings as st` → access via `st.VAR`. One singleton per package, always aliased `st`. No exceptions.

## Performance & Memory

- Analyze Big O. Prioritize O(1) / O(log n).
- `__slots__` everywhere: `@dataclass(slots=True, frozen=True)` or explicit `__slots__` tuple. Each subclass declares only its OWN slots.
- Iterables: generators (`yield`) and async generators (`async yield`) by default. Never materialize to `list` unless strictly needed.
- Deduplication & membership: `set` / `frozenset` over `list`. `frozenset` when immutable.
- Combine both: generator pipelines feeding into `set`/`frozenset` for memory-efficient, duplicate-free flows.
- Pre-compile regex at module level.
- `contextlib.suppress` over try/except pass. `object.__setattr__()` for frozen/slots models.

## Caching

- `@functools.cache`: pure functions, hashable args, unbounded.
- `@functools.lru_cache(maxsize=N)`: bounded, LRU eviction.
- `@cached_property`: instance-level, computed once.
- `cachetools.TTLCache` / `LRUCache`: shared mutable, time-based.
- Redis GET/SET + EX: cross-process, persistent TTL.

## Stdlib Mastery

Use stdlib to the fullest before reaching for third-party. Advanced usage expected:

- `functools`: `cache`, `lru_cache`, `partial`, `reduce`, `singledispatch`, `wraps`, `cached_property`.
- `itertools`: `chain.from_iterable`, `groupby`, `islice`, `starmap`, `compress`, `takewhile`, `pairwise`, `batched`.
- `operator`: `itemgetter` (multi-key), `attrgetter` (nested), `methodcaller`.
- `contextlib`: `suppress`, `asynccontextmanager`, `AsyncExitStack`.
- `collections`: `deque(maxlen=N)`, `Counter`, `defaultdict`, `ChainMap`.
- `asyncio`: `gather`, `Semaphore`, `TaskGroup`, `timeout`, `Queue`.
- `dataclasses`: `field(default_factory=...)`, `slots=True`, `frozen=True`.
- `pathlib`, `heapq`, `bisect`, `struct` (binary).

## Magic Methods & Protocols

- Implement `__repr__`, `__eq__`, `__hash__`, `__bool__`, `__lt__` (for sorting) on domain objects.
- `__init_subclass__` over metaclasses for auto-registration.
- `__set_name__` for descriptors (validated attrs, cached properties).
- `__class_getitem__` for parametrized types.
- `typing.Protocol` with `@runtime_checkable` for structural subtyping — over ABCs when possible.

## Async Patterns

- `asyncio.TaskGroup` (3.11+) for structured concurrency with error propagation.
- `asyncio.Semaphore` for bounded concurrent access.
- `asyncio.timeout` (3.11+) for async timeouts.
- `asyncio.Queue` for producer/consumer patterns.
- Async generators for streaming pipelines. `async for` everywhere.

## App Composition — Explicit Registration

When building applications (API, CLI, plugins), ALWAYS use **explicit registration** — import the component and register it programmatically in a central entrypoint. NEVER use the decorator approach that forces importing the app instance into every module.

**Why:** Decorator-based registration (`@app.command`, `@app.get`) creates an implicit dependency on the app singleton, pollutes every module with the app import, and causes circular imports. Explicit registration keeps modules independent and the dependency graph clean.

### The Pattern

Components (routers, commands, handlers, plugins, adapters) are defined in their own modules with no knowledge of the app instance. The entrypoint imports and registers them:

```python
# commands/users.py — knows NOTHING about the app
async def create(name: str) -> None: ...
async def delete(user_id: str) -> None: ...
async def list_users() -> None: ...

# main.py — the ONLY place that knows about the app
from cyclopts import App
from myapp.commands.users import create, delete, list_users

app = App()
user_app = App(name="user")

user_app.command(create)
user_app.command(delete)
user_app.command(list_users, name="list")

app.command(user_app)
```

### Applies to ALL frameworks

```python
# FastAPI — explicit router inclusion, NOT @app.get decorators
from fastapi import APIRouter, FastAPI
from app.api.router.health import router as health_router
from app.api.router.entities import router as entities_router

app = FastAPI()
app.include_router(health_router, prefix="/health")
app.include_router(entities_router, prefix="/entities")

# Typer — explicit command addition
from typer import Typer
from myapp.commands.sync import sync_command

app = Typer()
app.command(name="sync")(sync_command)

# Plugin/adapter registration — explicit registry
from app.adapters.nlp import NLPRuntimeAdapter
from app.adapters.searxng import SearXNGAdapter

ADAPTERS: dict[str, type[Adapter]] = {
    "nlp": NLPRuntimeAdapter,
    "searxng": SearXNGAdapter,
}
```

**Rules:**
- The app instance is created and lives ONLY in `main.py` (or the entrypoint module).
- No module outside `main.py` imports the app instance.
- Components (routers, commands, handlers) are plain functions or classes — no decorators that bind them to an app.
- Registration happens in ONE place: the entrypoint. This is the single source of truth for what the app exposes.

## Prohibited Patterns (STRICTLY FORBIDDEN)

### No `__init__.py` unless structurally vital

Never create `__init__.py` files. Python ≥3.3 namespace packages work without them. Only exception: when a framework or build tool explicitly requires it and there is no workaround.

### No imports inside functions — EVER

All imports at module top-level. No lazy imports hidden inside function bodies. Use `TYPE_CHECKING` block for circular dependency prevention.

```python
# FORBIDDEN — import inside function
def load_agents_config(config_path: Path) -> AgentsConfig:
    from e_agents.models.agent import AgentsConfig  # NEVER DO THIS
    return AgentsConfig.from_yaml(config_path)

# CORRECT — top-level import
from e_agents.models.agent import AgentsConfig

def load_agents_config(config_path: Path) -> AgentsConfig:
    return AgentsConfig.from_yaml(config_path)

# CORRECT — TYPE_CHECKING for circular deps only
from __future__ import annotations
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from e_agents.models.agent import AgentsConfig
```

### No useless wrapper functions

If a function only imports + delegates to a classmethod/staticmethod, it adds zero value. Call the method directly at the call site.

```python
# FORBIDDEN — pointless wrapper that exists only to hide an import
def load_agents_config(config_path: Path) -> AgentsConfig:
    from e_agents.models.agent import AgentsConfig
    return AgentsConfig.from_yaml(config_path)

# CORRECT — call directly where needed
config = AgentsConfig.from_yaml(config_path)
```

### No ad-hoc helper proliferation — EVER

**STRICTLY FORBIDDEN.** Creating private functions, helpers, or utility snippets on the fly inside route handlers, command modules, or any business logic file without prior domain analysis (N1-taxonomy.md). This is the single most common AI code generation failure and it is NEVER acceptable.

**What is forbidden:**

- `##### HELPERS #####` sections in router/endpoint files that grow organically with random private functions.
- Trivial wrappers (1-5 lines) that just instantiate a class, call a constructor, or set a header.
- Private `_scan_*`, `_parse_*`, `_build_*`, `_json_*` functions scattered throughout a module that each solve one micro-problem in isolation.
- Creating ANY function without first asking: what entity does this belong to? What semantic field? Where does it live architecturally?

**Why this is catastrophic:**

Each helper looks "fine" alone. But 6 months later the module has 15 private functions with inconsistent names, no shared vocabulary, no domain structure, and no one knows which are reusable, which are dead, or which duplicate logic elsewhere. It is the exact opposite of the N1 taxonomy approach.

```python
# ❌ FORBIDDEN — ad-hoc helpers scattered in a router file
##### HELPERS #####

def _dir_size_mb(path: Path) -> float:
    return sum(f.stat().st_size for f in path.rglob("*") if f.is_file()) / (1024 * 1024)

def _scan_stt_models() -> list[ModelEntry]:
    stt_dir = st.MODELS_PATH / "stt"
    ...

def _scan_tts_models() -> list[ModelEntry]:
    tts_dir = st.MODELS_PATH / "tts"
    ...

def _json_response(status_code: int, body: str) -> Response:
    return Response(status_code=status_code, headers={"content-type": "application/json"}, description=body)

def _json_error(status_code: int, error: str, detail: str | None = None) -> Response:
    return _json_response(status_code, ErrorResponse(error=error, detail=detail).model_dump_json())
```

**What to do instead — decision tree:**

1. **Is it trivial (1-3 lines)?** → **Inline it** at the call site. No function needed.
   ```python
   # FORBIDDEN — trivial wrapper
   def _json_response(status_code: int, body: str) -> Response:
       return Response(status_code=status_code, headers=..., description=body)

   # CORRECT — inline at call site
   return Response(status_code=200, headers={"content-type": "application/json"}, description=body)
   ```

2. **Is it reusable across multiple modules?** → Move to the proper architectural location:
   - Audio processing → `core/audio.py` (class with `@staticmethod` / `@classmethod`).
   - Generic response builders → `core/helpers.py`.
   - Domain operations → `operations/` or `operational/`.

3. **Is it complex but module-specific?** → It may be a private method on a **class** (with domain suffix per N3). Never a loose private function at module level.

4. **Does it belong to a scan/discovery family?** → Do the N1 taxonomy first. Identify the entity (`Model`), the verb (`scan`), and create a proper class: `ModelScanner.scan_stt()`, `ModelScanner.scan_tts()`.

**The rule is absolute:** No function is created without first answering: (a) what entity does this serve? (b) what verb from the vocabulary? (c) where does it live architecturally? If you can't answer all three, you don't write the function.

### No relative imports

Always absolute. `from .module import X` is forbidden. `from package.module import X` always.

## Logging

Emoji + UPPERCASE_EVENT as message, context in extra={}.

## Class & Method Design

### Method Classification (evaluate in order)

Before defining any class method, decide its category:

1. **Reusable outside the class?** → public method or module-level function.
2. **Operates on class state without instance?** → `@classmethod`.
3. **Pure utility, no access to `self` or `cls`?** → `@staticmethod`.
4. **Internal non-reusable business logic?** → private method with domain suffix (see Naming Conventions).

### Private Methods — Justified Uses Only

Private methods are permitted exclusively when they:

- Encapsulate class-specific business logic that is **not reusable** outside the class.
- Decouple internal steps of a complex public method for readability.
- Are cacheable (`@functools.cache`) or recursive helpers.

**Forbidden uses:**

- As a default — if no concrete reason exists, it must be public.
- To hide poorly typed code or avoid type-checker errors.
- For logic that should be a module-level function or `@staticmethod`.
- For trivial logic (1-3 lines) — inline it instead.
- When two methods share nearly identical logic — consolidate into one with a parameter.
- As a dumping ground for micro-logic that has not been through N1 taxonomy analysis.
- For ad-hoc "parsers", "scanners", "builders" that exist only because the AI decided to extract 3 lines into a function with a cryptic name instead of thinking about domain structure first.

### No `type: ignore`

`# type: ignore` is strictly forbidden. Fix the type annotation instead. The type system is a contract — uphold it.

### Naming Clarity

Method and variable names must be self-explanatory. Obfuscation is a bug. If a name requires a comment to be understood, the name is wrong. Verb-first for methods, noun-based for classes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/damvolkov) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
