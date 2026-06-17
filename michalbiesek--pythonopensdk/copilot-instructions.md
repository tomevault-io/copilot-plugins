## sdk-generation

> Full SDK generation rules from OpenAPI


# SDK Generation Rules for AI-Driven Generation

> **AI INSTRUCTION:** When asked to generate SDK code from the OpenAPI spec (`openapi.yml`), follow all rules in this file. This is the authoritative source.

Rules and conventions for **AI-driven** generation of the Cribl Python SDK from an OpenAPI specification.

---

## 0. AI Agent Instructions (Read First)

### 0.1 Generation Order

Generate code in this exact order to satisfy dependencies:

1. **types.py** — `Unset`, `UNSET`, `Response`, `File`, `FileTypes`, `RequestFiles`
2. **errors.py** — `UnexpectedStatus`
3. **client.py** — `Client`, `AuthenticatedClient`
4. **models/** — All schemas from `components.schemas` (resolve `$ref` before generating; generate base/referenced schemas first)
5. **api/** — Endpoint modules (each endpoint may depend on models)
6. **api/__init__.py** — Re-exports and client aggregation
7. **__init__.py** — Package exports

### 0.2 Mandatory Rules (NEVER Violate)

- **ALWAYS** use `attrs` for models and clients; **NEVER** use `dataclasses` or raw `__init__`
- **ALWAYS** implement all four endpoint functions: `sync`, `sync_detailed`, `asyncio`, `asyncio_detailed`
- **ALWAYS** use `UNSET` for optional fields; **NEVER** use `None` as "not provided" for JSON-optional fields
- **ALWAYS** include `from __future__ import annotations` at the top of model files
- **ALWAYS** use relative imports (`from ...client`, `from ...models.x import Y`)
- **NEVER** generate placeholder or TODO code; generate complete, runnable code
- **NEVER** invent schema properties; only use properties defined in the OpenAPI spec
- **NEVER** add `__pycache__/` or `*.pyc` binary files to the repository; ensure `.gitignore` excludes them

### 0.3 Schema Name -> File Name Mapping

- PascalCase schema name -> snake_case filename: `HealthServerStatus` -> `health_server_status.py`
- Class name = schema name (PascalCase)

### 0.4 When Parsing OpenAPI

- For each `operationId`, derive the endpoint module name from the path and method (e.g. `get_health` for `GET /health`)
- For each response status code with a schema, add a branch in `_parse_response`
- For `oneOf`/`anyOf` with discriminator: check discriminator field first, then instantiate the correct subclass

### 0.5 Prompt Context for AI

When invoking an AI agent to generate SDK code, provide:

```text
Generate Python SDK code following this Cursor rule.
Source: openapi.yml (OpenAPI 3.1)
Target: [package_name] (e.g. cribl_api_reference_client)
Scope: [all | models only | api only | specific path, e.g. /health | by tag, e.g. auth,health]
```

For incremental generation, specify scope (e.g. `by tag: auth,health`) to avoid regenerating the entire SDK. For large specs (50k+ lines), generate by tag or path prefix in batches.

### 0.6 Post-Generation Validation (AI Self-Check)

Before finishing, verify:

1. All `$ref` in schemas are resolved; no forward references to undefined models
2. Every endpoint has exactly four functions: `sync`, `sync_detailed`, `asyncio`, `asyncio_detailed`
3. `_parse_response` has an `if response.status_code == X` branch for every documented response
4. Optional request/response fields use `UNSET`, not `None`, for "not provided"
5. Python reserved words (`id`, `object`, etc.) are escaped (e.g. `id_` or `object_`) in parameter names
6. No `# TODO` or placeholder implementations
7. No `__pycache__/` or `*.pyc` files in the repository; `.gitignore` excludes them

### 0.7 Common AI Pitfalls to Avoid

| Pitfall | Correct Behavior |
|---------|------------------|
| Using `None` for optional JSON fields | Use `UNSET`; `None` means "explicitly null" in JSON |
| Omitting `asyncio`/`asyncio_detailed` | Always generate all four endpoint functions |
| Hardcoding base URL in endpoints | Use `client.base_url` or accept `server_url` override |
| Inventing response schemas | Only use schemas from `components.schemas` and `responses.*.content.*.schema` |
| Skipping `additional_properties` | If schema has `additionalProperties`, add the dict + accessor methods |
| Wrong import depth | Use `...client` from `api/health/`, `..models` from `api/` |
| Discriminator not checked first | In `from_dict` for unions, check discriminator before instantiating |

---

## 1. Reference & Overview

- **OpenAPI**: 3.1.0, Cribl API Reference
- **Python**: >=3.10, <4
- **Generator-agnostic**: These rules do not depend on gen.yaml or any specific code generator config.

---

## 2. OpenAPI Specification Requirements

### 2.1 Required Vendor Extensions

| Extension | Purpose |
|-----------|---------|
| `x-retries` (or equivalent) | Backoff retry metadata for 429 |
| `x-group` (or equivalent) | Resource grouping (e.g. `auth.tokens`, `nodes`) |
| `x-name-override` (or equivalent) | Friendly method name (e.g. `get`, `list`, `create`) |

### 2.2 Security Schemes

- **bearerAuth**: HTTP Bearer (JWT) for on-prem
- **clientOauth**: OAuth2 client credentials for Cribl.Cloud
- Vendor-specific token endpoint metadata for OAuth audience

### 2.3 Base URL Contexts

Document these in `info.description`:

- Leader: `/api/v1`
- Worker Group / Edge Fleet: `/api/v1/m/{groupName}`
- Host (Worker/Edge Node): `/api/v1/w/{nodeId}`
- Search: `/api/v1/m/default_search`

---

## 3. Project Structure

```text
project/
├── openapi.yml                 # Source OpenAPI spec
├── pyproject.toml              # Poetry (preferred) or setup.py
├── src/
│   └── <package_name>/
│       ├── __init__.py         # Exports SDK class, models
│       ├── <sdk_class>.py      # Main SDK client
│       ├── client.py           # HTTP client(s)
│       ├── errors.py           # Error types
│       ├── types.py            # UNSET, Response, File, etc.
│       ├── py.typed            # PEP 561 marker
│       ├── api/                # Endpoints by tag/group
│       │   ├── auth/
│       │   ├── health/
│       │   ├── sources/
│       │   ├── destinations/
│       │   └── ...
│       └── models/             # One file per schema
│           ├── error.py
│           ├── health_server_status.py
│           └── ...
└── docs/                       # API docs (optional)
```

---

## 4. Client Patterns

### 4.1 Client Classes

**Option A: Client + AuthenticatedClient (OpenAPI Generator style)**

- `Client`: Unauthenticated, uses `httpx.Client` / `httpx.AsyncClient`
- `AuthenticatedClient`: Bearer token via `token`, `prefix`, `auth_header_name`
- Both use `attrs` (`@define`) and `evolve()` for immutable updates
- Lazy creation: `get_httpx_client()` / `get_async_httpx_client()`

**Option B: Single SDK Class (nested resources)**

- `CriblControlPlane` (or equivalent): Single entry point
- Context manager: `with CriblControlPlane(...) as client:`
- Nested resources: `client.sources.list()`, `client.health.get()`

### 4.2 Required Client Attributes

- `base_url` / `server_url`
- `raise_on_unexpected_status: bool = False`
- `timeout`, `verify_ssl`, `headers`, `cookies`
- `get_httpx_client()` / `get_async_httpx_client()`

### 4.3 Cribl-Specific: server_url Parameter

Many operations accept `server_url` to target a specific context (e.g. Worker Group):

```python
# Leader context
client.health.get()

# Worker Group context
client.sources.list(server_url="https://api.example.com/m/my-worker-group")
```

---

## 5. API Endpoint Patterns

### 5.1 Per-Endpoint Module Structure

Each endpoint = one module with **four functions**:

| Function | Purpose |
|----------|---------|
| `sync(client, ...)` | Sync call, returns parsed data or `None` |
| `sync_detailed(client, ...)` | Sync call, returns `Response[T]` |
| `asyncio(client, ...)` | Async call, returns parsed data or `None` |
| `asyncio_detailed(client, ...)` | Async call, returns `Response[T]` |

### 5.2 Internal Helpers

- `_get_kwargs()` -> `dict[str, Any]` with `method`, `url`, `params`, `json`, `files`, etc.
- `_parse_response(client, response)` -> parsed model or `None`; raises `UnexpectedStatus` when `raise_on_unexpected_status` is True
- `_build_response(client, response)` -> `Response[T]`

### 5.3 Request Flow

```python
kwargs = _get_kwargs()
response = client.get_httpx_client().request(**kwargs)
return _build_response(client=client, response=response)
```

### 5.4 Docstrings

- Summary from OpenAPI `summary`
- Description from OpenAPI `description`
- `Raises`: `errors.UnexpectedStatus`, `httpx.TimeoutException`
- `Returns`: Response type

### 5.5 Parameter Handling

- Path params: Required, in URL
- Query params: Optional, in `params`
- Request body: In `json` or `content`
- `server_url`: Override base URL for Cribl context

---

## 6. Model Patterns

### 6.1 Base Structure

- Use `attrs`: `@_attrs_define`, `@_attrs_field`
- `from __future__ import annotations`
- `TypeVar` for `from_dict` return type

### 6.2 Serialization

- `to_dict() -> dict[str, Any]`
- `@classmethod from_dict(cls, src_dict: Mapping[str, Any]) -> T`
- Use `UNSET` for optional/unset fields

### 6.3 Optional Fields

- Type: `str | Unset`, default: `UNSET`
- In `to_dict`: skip if value is `UNSET`
- In `from_dict`: `d.pop("field", UNSET)`

### 6.4 Additional Properties

For schemas with `additionalProperties`:

```python
additional_properties: dict[str, Any] = _attrs_field(init=False, factory=dict)
# __getitem__, __setitem__, __delitem__, __contains__, additional_keys
```

### 6.5 Discriminators (oneOf/anyOf)

- Use discriminator property (e.g. `type`) to select subclass
- In `from_dict`: check discriminator, instantiate correct subclass
- Export union type for response parsing

### 6.6 Enums

- Use `Literal` or `Enum` from `enum`
- Const fields: `Literal["value"]` or `const: true` in schema

### 6.7 Naming

- Schema `Error` -> `error.py`, class `Error`
- Schema `HealthServerStatus` -> `health_server_status.py`
- Snake_case filenames from PascalCase schema names

---

## 7. Types & Utilities

### 7.1 Unset

```python
class Unset:
    def __bool__(self) -> Literal[False]:
        return False

UNSET: Unset = Unset()
```

### 7.2 Response

```python
@define
class Response(Generic[T]):
    status_code: HTTPStatus
    content: bytes
    headers: MutableMapping[str, str]
    parsed: T | None
```

### 7.3 File Uploads

```python
@define
class File:
    payload: BinaryIO
    file_name: str | None = None
    mime_type: str | None = None

    def to_tuple(self) -> FileTypes: ...
```

---

## 8. Error Handling

### 8.1 UnexpectedStatus

```python
class UnexpectedStatus(Exception):
    def __init__(self, status_code: int, content: bytes):
        self.status_code = status_code
        self.content = content
        super().__init__(f"Unexpected status code: {status_code}\n\n{content.decode(errors='ignore')}")
```

### 8.2 When to Raise

- Only when `client.raise_on_unexpected_status` is `True`
- Only for status codes not documented in the OpenAPI spec
- Documented error responses (e.g. 500 with `Error` schema) return parsed model, do not raise

### 8.3 Alternative: Typed Error Subclasses

- Base error class (e.g. `CriblControlPlaneError`)
- Subclasses per status (e.g. `HealthServerStatusError` for 420)
- `ResponseValidationError` for validation failures (if using Pydantic)

---

## 9. Dependencies

### 9.1 Required

```text
httpx >= 0.23.0, < 0.29.0
attrs >= 22.2.0
python-dateutil >= 2.8.0, < 3
```

### 9.2 Optional

- `pydantic` for validation
- `poetry` for package management

---

## 10. AI Validation Checklist

After generating SDK code, the AI agent MUST verify:

- [ ] OpenAPI spec was the single source of truth; no properties or endpoints invented
- [ ] Models: attrs, `from_dict`/`to_dict`, `UNSET` for optional (not `None`)
- [ ] Endpoints: all four functions (sync, sync_detailed, asyncio, asyncio_detailed)
- [ ] `_parse_response` has a branch for every documented response status code
- [ ] `UnexpectedStatus` raised only when `raise_on_unexpected_status` and status is undocumented
- [ ] Docstrings: summary, description, Raises, Returns (from OpenAPI)
- [ ] Type hints on all public functions
- [ ] `py.typed` present for PEP 561
- [ ] Imports: relative (`...client`, `...models`, `...types`); correct depth for module location
- [ ] Cribl `server_url` supported where operations target Worker Group / Edge Fleet context
- [ ] No `__pycache__/` or `*.pyc` files committed; `.gitignore` excludes them

---

## 11. Cribl-Specific Conventions

### 11.1 Resource Grouping

Use a group vendor extension (for example `x-group`) for logical grouping:

- `auth.tokens` -> `client.auth.tokens.get()`
- `nodes` -> `client.nodes.list()`, `client.nodes.count()`
- `sources`, `destinations`, `pipelines`, etc.

### 11.2 Products

Path param `product`: `stream` | `edge` (and possibly `lake` for Cribl.Cloud)

### 11.3 Pack-Scoped Endpoints

Paths like `/p/{pack}/system/inputs` -> pack-scoped operations; include `pack` path param.

### 11.4 JSON Streaming

Endpoints returning `application/x-ndjson` (e.g. `system.capture`): expose as generator/stream, support `with` context manager.

---

## 12. Version Alignment

- SDK version should track OpenAPI `info.version` (e.g. `4.18.0-alpha.xxx`)
- Or use independent versioning (e.g. `0.6.0rc42`) with changelog referencing API version

---
> Source: [michalbiesek/pythonOpenSDK](https://github.com/michalbiesek/pythonOpenSDK) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
