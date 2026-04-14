## taproot-sdk

> This file provides guidance to Claude Code when working with the taproot-sdk codebase.

# CLAUDE.md

This file provides guidance to Claude Code when working with the taproot-sdk codebase.

## Library Overview

The Taproot SDK (`taproot-sdk`) is a Python library with three pillars:

1. **OpenTelemetry-based instrumentation** -- `@instrument()` decorator with 9 span kinds, 7 LLM auto-instrumentors (OpenAI, Anthropic, Google, Cohere, Vertex AI, Bedrock, Mistral), and a custom JSON OTLP exporter for AWS API Gateway v1 compatibility.
2. **Unified async client** (`TaprootClient`) -- ~120 async methods covering all 6 Taproot platform services (Retrieval-S, Evals-S, Guardrail-S, Prompt-S Management, Prompt-S Serving, ToolBox-S) through a single APIM gateway, with typed frozen-dataclass return models.
3. **CLI** (`taproot-tools`) -- 16+ subcommands for tool push/invoke/discover, credentials, MCP server management, and OpenAPI import.

Published as `taproot-sdk` on PyPI. Version 0.1.0 (beta), Python 3.11+, Apache 2.0, built with Hatchling.

## Development Rules

1. **Deployment Rule**: Published to PyPI. Version bumps go through CI/CD. Never change version in `pyproject.toml` without explicit instruction.
2. **Multi-Cloud Rule**: `TaprootClient` supports APIM mode (all clouds -- AWS API Gateway, Azure APIM, GCP API Gateway) and direct mode (local dev, direct service access). Auth headers differ by mode. Never hardcode cloud-specific logic in client methods.
3. **LLM Agnostic Rule**: Auto-instrumentation supports 7 LLM providers via OpenLLMetry. The `@instrument()` decorator is provider-agnostic -- it wraps any sync or async function. Never couple instrumentation to a specific LLM library.
4. **Terraform Rule**: N/A (SDK library, no infrastructure).
5. **Live Testing Rule**: Test client methods against the live APIM gateway environment. Reference `deployment_env.txt` for base URL and API key. Use `E2E_API_KEY` env var for the raw API key.
6. **No Assumptions Rule**: Ask, don't assume. When requirements are ambiguous, clarify before implementing.
7. **Continuous Improvement Rule**: Keep this CLAUDE.md current. Update it when adding new client methods, models, or changing architecture.

## Feature List

- [x] `@instrument()` decorator with 9 span kinds (workflow, agent, chain, tool, retrieval, embedding, completion, chat, rerank)
- [x] 7 LLM auto-instrumentors (OpenAI, Anthropic, Google, Cohere, Vertex AI, Bedrock, Mistral)
- [x] Custom JSON OTLP exporter (AWS API Gateway v1 binary payload workaround)
- [x] `TaprootClient` with ~120 async methods across 6 services
- [x] Prompt stale-while-revalidate cache (fresh/stale/expired states)
- [x] `assert_eval()` CI/CD quality gate helper
- [x] Pytest plugin (`eval_client`, `eval_run` fixtures)
- [x] `taproot-tools` CLI with 16+ subcommands
- [x] Typed exception hierarchy (7 exception types with HTTP status mapping)
- [x] Frozen dataclass response models (all immutable, `from_api_response()` classmethods)
- [x] Client-side variable rendering for prompts (server stores templates as-is)
- [x] SHA-256 hash verification on prompt responses
- [x] Chat prompt support (PromptType.TEXT / PromptType.CHAT)
- [x] A/B testing via weighted labels
- [x] OpenAI / Anthropic message format converters
- [x] Guardrail check results, analytics, named config management
- [x] Agent Discovery (L1/L2 probing, suggestions, approval workflow)
- [x] ToolBox OAuth flow, credential vault, MCP server registration
- [x] Semantic tool discovery
- [x] OpenAPI import, MCP registry import/export
- [x] Worker-S client methods (create/get/send/approve/reject/delete sessions)

## Build & Development Commands

```bash
# Install for development (from taproot-sdk/)
pip install -e ".[dev,all]"
# Or with uv:
uv pip install -e ".[dev,all]"

# Run all tests
pytest                                     # or: uv run pytest
pytest tests/test_decorators.py -v         # single file
pytest tests/test_decorators.py::test_name # single test

# Type checking (strict mode)
mypy src/

# Linting (line-length 100)
ruff check src/
ruff format --check src/

# Coverage
pytest --cov=src/taproot_sdk --cov-report=term-missing
```

## Architecture

### Module Structure

```
src/taproot_sdk/
  __init__.py            Public API surface (~100 symbols in __all__)
  core.py                SDK initialization, TracerProvider setup, OTLP exporter config
  decorators.py          @instrument() decorator for custom function tracing
  auto_instrument.py     Dynamic LLM library instrumentor loading (OpenLLMetry)
  client.py              TaprootClient -- unified async HTTP client (~4585 lines, ~120 methods)
  exceptions.py          Exception hierarchy (TaprootError -> TaprootAPIError -> specific)
  otlp_json_exporter.py  JsonOtlpSpanExporter -- JSON OTLP for AWS API Gateway compat
  evals/
    __init__.py          Eval module exports
    models.py            15+ frozen dataclasses (EvalResult, GoldenDataset, TestConfiguration, etc.)
    assertions.py        assert_eval() CI/CD assertion helper
    exceptions.py        EvalAssertionError
    pytest_plugin.py     eval_client, eval_run fixtures
  guardrails/
    __init__.py          Guardrails module exports
    models.py            GuardrailResponse, ScannerSignal, PolicySegmentResult, RedactionAction
    check_results.py     CheckResult, AnalyticsSummary, TimeseriesBucket
    configs.py           GuardrailConfig, ScannerOverride
  prompts/
    __init__.py          Prompts module exports
    models.py            PromptResponse with render/verify methods, ChatMessage, ToolDefinition
    client.py            PromptClient (DEPRECATED -- use TaprootClient)
    cache.py             PromptCache -- LRU with stale-while-revalidate
    exceptions.py        MissingVariableError
  retrieval/
    __init__.py          Retrieval module exports
    models.py            20+ frozen dataclasses (StoreInfo, QueryResponse, IngestionJob, etc.)
  toolbox/
    __init__.py          ToolBox module exports
    models.py            ToolInfo, InvocationResult, CredentialInfo, MCPServerInfo, etc.
    cli.py               taproot-tools CLI entry point (console_scripts)
  workers/
    __init__.py
    models.py             WorkerSession, SessionCreated, SessionMessage, PendingAction
tests/
  conftest.py            Autouse fixture: reset_sdk() calls shutdown() before/after each test
  test_core.py           TestInit, TestShutdown, TestGetTracer
  test_decorators.py     TestInstrumentDecorator sync/async/error handling
  test_auto_instrument.py Auto-instrumentation tests
  test_client.py         TaprootClient tests
  test_exceptions.py     Exception hierarchy tests
  auto_instrumentation/  Provider-specific tests (test_openai.py, test_anthropic.py)
  evals/                 test_models.py, test_assertions.py
  prompts/               test_client.py, test_models.py, test_cache.py, test_rendering.py, test_ab_testing.py
  retrieval/             test_models.py, test_client_methods.py
  toolbox/               test_models.py, test_client_methods.py, test_cli.py, test_tool_decorator.py, test_oauth_client.py, test_usage_report.py
```

### Key Design Patterns

- **Frozen dataclasses** for all response models (immutable, no mutation)
- **`from_api_response()` / `from_dict()` classmethods** on every model for JSON parsing
- **Tuple** (not list) for collection fields in frozen dataclasses
- **Async-first** client with sync convenience wrappers where needed
- **APIM vs direct mode** routing in TaprootClient (gateway vs local dev)
- **Stale-while-revalidate** cache for prompt serving
- **Typed exception hierarchy** wrapping HTTP errors with contextual info
- **Global state** with atexit handler for automatic span flush and shutdown

### Core Module (core.py)

**Global State** (module-level):
- `_tracer: Tracer | None`
- `_provider: TracerProvider | None`
- `_config: dict[str, Any]`
- `_initialized: bool`

**Functions:**
- `init()` -- Initialize SDK with TracerProvider, exporter, and batch processor. Raises `RuntimeError` if called twice without `shutdown()`.
- `shutdown()` -- Force flush pending spans (10s timeout), shutdown provider, reset global state. Auto-called at exit via `atexit`.
- `get_tracer()` -- Returns global tracer. Raises `RuntimeError` if not initialized.
- `is_initialized()` -- Returns `_initialized` flag.
- `get_config()` -- Returns copy of `_config` dict.

## Instrumentation

### @instrument() Decorator (decorators.py)

Wraps sync or async functions to automatically create OpenTelemetry spans.

```python
@ev.instrument(
    spankind="tool",                   # Span type (see 9 kinds below)
    name="custom-span-name",           # Override function name
    ignore_inputs=False,               # True = skip all; list[str] = skip named params
    ignore_outputs=False,              # True = skip return value capture
    max_attribute_size=65536,          # 64KB default, truncates with ...[TRUNCATED]
)
def my_function(query: str, api_key: str) -> dict:
    ...
```

### 9 Span Kinds

| Span Kind | Description |
|-----------|-------------|
| `workflow` | End-to-end pipeline (default) |
| `agent` | Autonomous agent operations |
| `chain` | Sequential processing |
| `tool` | External tool/function calls |
| `retrieval` | Knowledge base/RAG operations |
| `embedding` | Vector embedding generation |
| `completion` | Text generation (non-chat) |
| `chat` | Conversational LLM calls |
| `rerank` | Result reordering |

### Span Attributes

| Attribute | Description |
|-----------|-------------|
| `ev.type.node` | Span kind string |
| `ev.meta.function` | Function name |
| `ev.meta.module` | Module name |
| `ev.data.inputs` | JSON-serialized inputs |
| `ev.data.outputs` | JSON-serialized outputs |
| `ev.metrics.duration_ms` | Execution time in milliseconds |

**Truncation:** When attribute exceeds `max_attribute_size`, truncates with `...[TRUNCATED]` suffix and adds `{key}_size` (int) and `{key}_truncated` (bool) attributes.

**Serialization:** Custom `_json_default` handles Pydantic v1/v2 models, objects with `__dict__`, numpy arrays (`.tolist()`), falls back to `repr()`. On serialization error, wraps in `{"__repr__": repr(value)}`.

**Error handling:** Records exception via `span.record_exception(e)`, sets status to `StatusCode.ERROR` (truncated to 1000 chars), re-raises. Duration still recorded.

**Graceful degradation:** Works without `ev.init()` -- uses OpenTelemetry's noop tracer (no spans exported, no errors).

**Implementation:** Detects async via `asyncio.iscoroutinefunction(func)`. Pre-computes span name and signature at decoration time. Returns `async_wrapper` or `sync_wrapper`. Uses `@functools.wraps(func)` for metadata preservation.

## Auto-Instrumentation (auto_instrument.py)

Dynamic loading of OpenTelemetry instrumentors for 7 LLM providers via OpenLLMetry packages.

| Library Key | Instrumentor Class | Install Extra |
|-------------|-------------------|---------------|
| `openai` | `OpenAIInstrumentor` | `taproot-sdk[openai]` |
| `anthropic` | `AnthropicInstrumentor` | `taproot-sdk[anthropic]` |
| `google` | `GoogleGenerativeAiInstrumentor` | `taproot-sdk[google]` |
| `cohere` | `CohereInstrumentor` | `taproot-sdk[cohere]` |
| `vertexai` | `VertexAIInstrumentor` | `taproot-sdk[vertexai]` |
| `bedrock` | `BedrockInstrumentor` | `taproot-sdk[bedrock]` |
| `mistral` | `MistralAiInstrumentor` | `taproot-sdk[mistral]` |

**Functions:**
- `setup_auto_instrumentation(libraries)` -- Loads instrumentors via `importlib`, calls `.instrument()`, tracks in `_initialized_instrumentors` set. If `libraries is None`, instruments all available. Returns list of successfully instrumented. Missing optional deps logged at DEBUG (expected).
- `uninstrument(libraries)` / `uninstrument_all()` -- Calls `.uninstrument()` and removes from tracking set.
- `get_instrumented_libraries()` / `is_instrumented(library)` -- Query state.

**Internal:** `_load_instrumentor(path)` splits on last dot, imports module, gets class, returns instance.

## OTLP JSON Exporter (otlp_json_exporter.py)

`JsonOtlpSpanExporter` -- custom `SpanExporter` that serializes spans as OTLP JSON instead of protobuf.

**Why:** AWS API Gateway REST API v1 does not support binary media types by default and corrupts protobuf payloads.

**Encoding process:**
1. Uses `encode_spans()` from OTel protobuf encoder to build canonical `ExportTraceServiceRequest`
2. Converts to dict via `google.protobuf.json_format.MessageToDict(preserving_proto_field_name=False)`
3. Normalizes trace/span/parent IDs from base64 to lowercase hex (OTLP JSON spec) via recursive `_normalize_ids()`
4. Serializes with `json.dumps(separators=(",", ":"))` for compactness
5. Gzip-compresses the body (enabled by default)
6. Sends via `httpx.post()` with `Content-Type: application/json` and `Content-Encoding: gzip`

**Constructor:**
```python
JsonOtlpSpanExporter(
    endpoint: str,
    headers: dict[str, str] | None = None,
    compress: bool = True,
    timeout: float = 30.0,
)
```

**Returns** `SpanExportResult.SUCCESS` on 2xx, `SpanExportResult.FAILURE` on non-2xx or exception.

## TaprootClient (client.py)

Single async HTTP client for all Taproot platform services. ~4585 lines, ~120 async methods.

### Constructor

```python
client = TaprootClient(
    base_url="https://gateway.taproot.dev",  # APIM gateway URL (required)
    api_key="your-raw-api-key",               # API key (required)
    project_id="my-project",                  # Default project ID
    timeout=30.0,                             # HTTP timeout in seconds
    direct_mode=False,                        # False=APIM, True=direct service
    cache_ttl_seconds=30.0,                   # Prompt cache TTL
    max_stale_seconds=60.0,                   # Prompt cache max staleness
)
```

Falls back to `ev.init()` config for `base_url`, `api_key`, `project_id` if not explicitly provided. Validates `base_url` is set (raises `ValueError` if missing). Strips trailing slash from `base_url`.

### APIM vs Direct Mode

| Aspect | APIM (default) | Direct |
|--------|----------------|--------|
| Auth Header | `x-api-key: {raw_key}` | `X-Api-Key-Id: {key_id}` |
| Retrieval-S | `/api/v1/retrieval/api/v1/...` | `/api/v1/...` |
| Evals-S | `/api/v1/evals/v1/...` | `/v1/...` |
| Guardrail-S | `/api/v1/guardrails/...` | `/...` |
| Prompt-S Mgmt | `/api/v1/prompts/...` | `/...` |
| Prompt-S Serving | `/serve/{pid}/{name}` | `/serve/{pid}/{name}` |

### Retry Logic

- **Retryable status codes:** 429, 500, 502, 503, 504 and connection errors
- **Exponential backoff:** 1s, 2s, 4s, 8s, 10s (capped at `_MAX_RETRY_WAIT=10`)
- **Max retries:** 5 (`_MAX_RETRIES=5`)
- Tracks `_taproot_attempts` and `_taproot_total_wait` on response object, surfaced in `ServerError`

### Error Handling

All non-2xx responses mapped to typed exceptions via `_raise_for_status()`:

| Status | Exception | Details |
|--------|-----------|---------|
| 401/403 | `AuthenticationError` | Includes project_id, actionable hints |
| 404 | `TaprootAPIError` or `PromptNotFoundError` | Prompt-specific subclass for prompt endpoints |
| 409 | `ConflictError` | Duplicate resources |
| 422 | `ValidationError` | Parses FastAPI and Guardrail-S error formats, `.errors` list, caps at 5 |
| 429 | `RateLimitError` | Extracts `Retry-After` header as `.retry_after` |
| 5xx | `ServerError` | Includes `.attempts` and `.total_wait_seconds` |

### Context Manager

```python
async with TaprootClient(...) as client:
    ...  # client is usable; closes and cleans up automatically
```

### Service Method Groups

**Retrieval-S** (Stores, Access, Query, Ingestion, Batch, Documents, Chunks):
- `create_store()`, `list_stores()`, `get_store()`, `get_store_by_name()`, `update_store()`, `delete_store()`, `get_store_stats()`
- `grant_store_access()`, `revoke_store_access()`, `list_store_access()`
- `retrieval_query()` -- vector/hybrid/keyword search with reranking
- `ingest_document()`, `get_ingestion_job()`, `list_ingestion_jobs()`, `cancel_ingestion_job()`
- `batch_ingest()`, `get_batch_status()`, `list_batch_jobs()`, `cancel_batch()`
- `list_documents()`, `get_document()`, `delete_document()`
- `upload_chunks()`, `list_chunks()`, `get_chunk()`, `delete_chunks()`

**Prompt-S Serving** (cached reads):
- `get_prompt(name, version, label)` -- cached via stale-while-revalidate
- `get_prompt_sync()` -- sync convenience wrapper (ThreadPoolExecutor if event loop running)
- `prompt_cache` property for manual invalidation

**Prompt-S Management** (CRUD, versioning, labels, approval, examples, optimization, partials, webhooks):
- `create_prompt()`, `list_prompts()`, `get_prompt_metadata()`, `update_prompt()`, `archive_prompt()`, `delete_prompt()`
- `create_version()`, `list_versions()`, `get_version()`, `diff_versions()`
- `set_label()`, `set_weighted_label()`, `list_labels()`, `get_label()`, `delete_label()`
- `approve_version()`, `reject_version()` -- four-eyes rule
- `add_examples()`, `list_examples()`, `get_example_stats()`, `delete_example()`
- `upsert_execution_config()`, `get_execution_config()`, `delete_execution_config()`
- `trigger_optimization()`, `get_optimization_job()`, `list_optimization_jobs()`
- `create_partial()`, `list_partials()`, `get_partial()`, `delete_partial()`
- `create_prompt_webhook()`, `delete_prompt_webhook()`

**Guardrail-S** (content scanning, check results, analytics, configs):
- `check_input(content)` -> `GuardrailResponse`
- `check_output(content, original_input)` -> `GuardrailResponse`
- `list_check_results()`, `get_check_result()`, `delete_check_result()`
- `get_guardrail_analytics(start_date, end_date, granularity)`
- `create_guardrail_config()`, `list_guardrail_configs()`, `get_guardrail_config()`, `update_guardrail_config()`, `delete_guardrail_config()`, `set_default_guardrail_config()`

**Evals-S** (datasets, configs, runs, experiments, alerts, webhooks, discovery, traces, export):
- Golden Datasets: `create_dataset()`, `list_datasets()`, `get_dataset()`, `update_dataset()`, `delete_dataset()`, `add_dataset_item()`, `list_dataset_items()`, `update_dataset_item()`, `delete_dataset_item()`, `bulk_session_assign()`, `promote_trace_to_dataset()`, `create_dataset_version()`, `list_dataset_versions()`
- Test Configs: `create_test_config()`, `list_test_configs()`, `get_test_config()`, `update_test_config()`, `delete_test_config()`
- Test Runs: `trigger_eval_run()`, `list_eval_runs()`, `get_eval_run()`, `cancel_eval_run()`, `get_eval_run_results()`, `compare_eval_runs()`, `wait_for_eval()`
- Experiments: `create_experiment()`, `list_experiments()`, `get_experiment()`, `update_experiment()`, `delete_experiment()`, `compare_experiment_runs()`
- Alerts: `create_alert_rule()`, `list_alert_rules()`, `get_alert_rule()`, `update_alert_rule()`, `delete_alert_rule()`, `get_alert_history()`
- Webhooks: `create_webhook()`, `list_webhooks()`, `get_webhook()`, `update_webhook()`, `delete_webhook()`, `list_webhook_deliveries()`, `get_webhook_delivery()`, `retry_webhook_delivery()`
- Discovery: `start_discovery()`, `list_discovery_sessions()`, `get_discovery_session()`, `get_discovery_probes()`, `get_discovery_suggestions()`, `update_discovery_suggestion()`, `approve_discovery_suggestions()`, `reject_discovery_suggestion()`, `cancel_discovery()`
- Traces: `ingest_traces()`, `list_traces()`, `get_trace()`, `get_trace_stats()`, `delete_trace()`
- Export: `export_eval_results()`, `get_eval_job_status()`

**ToolBox-S** (tools, credentials, MCP, OAuth, import/export):
- `push_tool()`, `register_tool()`, `invoke_tool()`, `list_tools()`, `discover_tools()`, `get_tool()`, `update_tool()`, `delete_tool()`
- `import_openapi()`, `import_mcp_registry()`, `export_mcp_registry()`
- `set_tool_credential()`, `list_credentials()`, `revoke_credential()`
- `start_oauth_flow()`, `complete_oauth_flow()`, `client_credentials_grant()`
- `register_mcp_server()`, `list_mcp_servers()`, `get_mcp_server()`, `delete_mcp_server()`
- `get_usage_report()`, `push_decorated_tools()`

**Worker-S** (sessions, messages, actions):
- `create_worker_session()`, `get_worker_session()`, `send_worker_message()`, `approve_worker_action()`, `reject_worker_action()`, `delete_worker_session()`, `health_workers()`

**Health checks:** `health_retrieval()`, `health_evals()`, `health_evals_basic()`, `health_guardrails()`, `health_prompts()`, `health_toolbox()`

## Prompt Client

### Via TaprootClient (recommended)

```python
prompt = await client.get_prompt("welcome-email", label="production")
rendered = prompt.render(user_name="Alice", plan="Pro")

# Chat prompts
chat_prompt = await client.get_prompt("chat-template")
messages = chat_prompt.render_messages(topic="weather")

# LLM provider convenience
openai_messages = prompt.to_openai_messages(user_name="Alice")
system, messages = prompt.to_anthropic_messages(user_name="Alice")
```

### PromptResponse Model

```python
prompt.schema_version    # int (currently 1)
prompt.name              # str
prompt.version           # int -- resolved version number
prompt.content           # str -- raw template with {{variable}} placeholders
prompt.content_hash      # str -- SHA-256 hex digest
prompt.config            # dict -- arbitrary metadata
prompt.required_variables # tuple[str, ...]
prompt.label             # str | None
prompt.cached_at         # str | None -- ISO-8601 timestamp
prompt.prompt_type       # PromptType.TEXT or PromptType.CHAT
prompt.messages          # tuple[ChatMessage, ...] | None (CHAT only)
prompt.tools             # tuple[ToolDefinition, ...] | None
prompt.ab_test           # bool -- True if served via weighted label
prompt.selected_variant  # int | None -- which variant was selected
```

### Rendering (Client-Side Only)

Variable rendering is **client-side only** (decision D21). The server stores and serves templates with `{{variables}}` intact. Values are never sent to the server.

- `render(**variables)` -- For TEXT prompts. Returns string with `{{placeholders}}` replaced. Raises `MissingVariableError` if required variables missing. Warns about extra variables.
- `render_messages(**variables)` -- For CHAT prompts. Returns new tuple of ChatMessage with variables substituted.
- `verify_hash()` -- SHA-256 integrity check via `hmac.compare_digest` (constant-time).
- `to_openai_messages(**variables)` -- Returns `list[dict[str, str]]` in OpenAI format.
- `to_anthropic_messages(**variables)` -- Returns `tuple[str | None, list[dict[str, str]]]` (system message, messages).
- `to_dict()` -- Serialize to plain dict.

### PromptClient (DEPRECATED)

`PromptClient` in `prompts/client.py` is deprecated. Use `TaprootClient` instead. Issues a `DeprecationWarning` on construction.

### Prompt Cache (prompts/cache.py)

`PromptCache` -- in-memory TTL cache with stale-while-revalidate semantics.

**State machine:**
- **Cache miss** (entry not in store): Call `fetch_fn` immediately, store with `time.monotonic()`, return fresh.
- **Fresh** (age < `ttl_seconds`): Return immediately.
- **Stale** (age between `ttl_seconds` and `ttl_seconds + max_stale_seconds`): Return stale value, schedule background revalidation via `asyncio.ensure_future()`. Track in `_inflight` set to prevent duplicate refreshes.
- **Expired** (age >= `ttl_seconds + max_stale_seconds`): Block on `fetch_fn()`, update cache, return fresh.

**Cache key format:** `"{project_id}:{name}:v={version_str}:l={label_str}"`

**Thread safety:** `asyncio.Lock` for async operations, `threading.Lock` for sync operations.

**Methods:** `get_or_fetch()` (async, full stale-while-revalidate), `get_or_fetch_sync()` (sync, no background revalidation), `invalidate()`, `clear()`.

## Guardrails Client

### check_input() / check_output()

```python
response = await client.check_input(
    "user message content",
    project_id="my-project",       # Optional -- uses client default
    config_name="strict",          # Optional -- named config
    trace_id="abc-123",            # Optional -- auto-generates UUID v4 if omitted
    scanner_overrides={"toxicity": False},  # Check-level: Dict[str, bool]
)

response = await client.check_output(
    "LLM response content",
    original_input="the user's original question",  # Optional -- for context
    scanner_overrides={"prompt_injection": True},
)
```

### GuardrailResponse Model

```python
response.verdict           # "ALLOW", "BLOCK", "ALLOW_WITH_REDACTIONS"
response.is_blocked        # Property: True if verdict == "BLOCK"
response.is_allowed        # Property: True if verdict == "ALLOW"
response.signals           # tuple[ScannerSignal, ...] -- individual scanner results
response.flagged_scanners  # Property -- scanners that flagged content
response.policy_signals    # tuple[PolicySegmentResult, ...] -- LLM judge results
response.violated_policies # Property -- violated policy segments
response.redactions        # tuple[RedactionAction, ...] | None
response.blocked_by        # str | None -- which scanner caused the block
response.block_reasons     # tuple[str, ...] | None
response.total_latency_ms  # float
response.request_id        # str
response.shadow_verdict    # dict[str, Any] | None -- for A/B testing
response.review_id         # str | None -- for HITL
```

### ScannerSignal Model

```python
signal.scanner_id       # "toxicity", "prompt_injection", etc.
signal.scanner_version  # str
signal.is_valid         # True if content passed this scanner
signal.flagged          # Property: not is_valid
signal.score            # float | None
signal.labels           # tuple[str, ...]
signal.reasoning        # str | None (LLM judge)
signal.latency_ms       # float
```

### Scanner Overrides: Two Schemas

- **Check-level** (in `check_input`/`check_output`): `Dict[str, bool]` -- enable/disable scanners per request.
- **Config-level** (in `create_guardrail_config`): `Dict[str, ScannerOverride]` where `ScannerOverride` has `enabled: bool`, `threshold: float | None`, `config: dict[str, Any]`.

### Additional Guardrails Models

- `GuardrailConfig` -- Named config with version, description, scanner_overrides, mode (active/shadow/disabled), is_default.
- `CheckResult` -- Persisted check result with id, project_id, verdict, check_type, config_name, trace_id, blocked_by.
- `AnalyticsSummary` -- Aggregated stats with verdicts dict, scanner_triggers dict, latency percentiles (p50/p95/p99), timeseries.
- `TimeseriesBucket` -- Timestamp + counts (total_checks, total_blocks, total_allows).

## Evals Client

### Golden Datasets

Full CRUD for datasets and items. Multi-turn support via `session_group` and `sequence_order`. Trace-to-dataset promotion. Versioning.

**Key models:** `GoldenDataset` (id, name, description, version, tags, item_count), `GoldenDatasetItem` (input_query, expected_output, expected_tool_calls, expected_contexts, session_group, sequence_order, source_type), `GoldenDatasetVersion` (version, item_count, change_reason).

### Test Configurations

CRUD for test configs that link a dataset to an agent target with specific metrics and pass threshold.

**Key model:** `TestConfiguration` (name, dataset_id, agent_target_id, metrics list, pass_threshold).

### Test Runs

Trigger, monitor, wait, compare, and export eval runs.

```python
handle = await client.trigger_eval_run(test_config_id, tags=["ci"])
result = await client.wait_for_eval(handle.run_id, timeout=300, poll_interval=5)
```

**EvalResult model:**
```python
result.run_id           # str
result.status           # "completed", "failed", "running", etc.
result.total_items      # int
result.completed_items  # int
result.failed_items     # int
result.pass_rate        # Property: (completed - failed) / completed * 100
result.duration_ms      # Property: float | None (ISO-8601 diff)
result.aggregate_scores # dict[str, AggregateScore] -- {metric: {mean, min, max, std_dev, passed, failed}}
result.tags             # list[str]
result.error_message    # str | None
```

### Experiments

A/B run comparison with `compare_experiment_runs()`.

**Key model:** `Experiment` (name, metrics, pass_threshold, dataset_id, agent_target_id, metadata, run_count).

### Discovery

Agent Discovery with L1/L2 probing and suggestion approval workflow.

**Key models:** `DiscoverySession` (status state machine: pending -> probing_l1 -> probing_l2 -> analyzing -> completed|failed|cancelled), `DiscoverySuggestion` (status: pending/approved/rejected/promoted, input_query, expected_output, reasoning, confidence).

### Alerts & Webhooks

Alert rules with condition types (metric_threshold, error_rate, pass_rate) and webhook delivery with retry tracking.

**Key models:** `AlertRule` (condition_type, condition_config, notification_channel, cooldown_minutes), `AlertHistory`, `Webhook` (events list), `WebhookDelivery` (status, attempt_count, response_status_code).

### assert_eval() CI/CD Helper

```python
from taproot_sdk import assert_eval

assert_eval(
    result,
    min_pass_rate=80,                    # Percentage (0-100)
    min_score={"answer_relevancy": 0.7}, # Per-metric mean thresholds
    max_duration_ms=60000,               # Max run duration
)
```

**Checks:**
1. Status == "completed" (raises if not)
2. `pass_rate >= min_pass_rate`
3. For each metric in `min_score`: `aggregate_scores[metric].mean >= threshold`
4. `duration_ms <= max_duration_ms`

**Raises:** `EvalAssertionError` (subclass of `AssertionError`) with `.result` attribute for context.

### Pytest Plugin

Register in conftest.py:
```python
pytest_plugins = ["taproot_sdk.evals.pytest_plugin"]
```

**Fixtures:**
- `eval_client` -- `TaprootClient` configured from env vars (`TAPROOT_EVAL_URL`, `TAPROOT_API_KEY_ID`, `TAPROOT_PROJECT_ID`). Timeout: 300s. Skips test if any env var missing.
- `eval_run` -- Factory fixture returning async function `_run(test_config_id, *, timeout=300, poll_interval=5, tags=None, description=None) -> EvalResult`. Triggers run and waits for completion.

## Retrieval Client

### Stores

`create_store(name, embedding_provider, embedding_model, ...)`, `list_stores()`, `get_store()`, `get_store_by_name()`, `update_store()`, `delete_store()`, `get_store_stats()`.

**Key models:** `StoreInfo` (id, name, embedding_provider, embedding_model, dimensions, index_type), `StoreList` (paginated), `StoreStatistics` (chunk_count, doc_count, size_bytes, query_count, ingestion_count).

### Access Control

`grant_store_access(store_id, api_key_id, access_level)`, `revoke_store_access()`, `list_store_access()`.

**Key models:** `AccessGrant` (api_key_id, store_id, access_level), `AccessGranted`, `AccessList`, `AccessRevoked`.

### Query

`retrieval_query(store_name, query, search_type="vector", top_k=10, filters=None, rerank_config=None)` -- Supports vector, hybrid (vector + BM25 via RRF), and keyword search with optional reranking.

**Key models:** `QueryHit` (chunk_id, score, text, metadata), `QueryResponse` (hits tuple, count, query_metadata, latency).

### Ingestion

`ingest_document(store_name, document, metadata)`, `get_ingestion_job()`, `list_ingestion_jobs()`, `cancel_ingestion_job()`.

### Batch Ingestion

`batch_ingest(store_name, documents)`, `get_batch_status()`, `list_batch_jobs()`, `cancel_batch()`.

### Documents & Chunks

- Documents: `list_documents()`, `get_document()`, `delete_document()`
- Chunks: `upload_chunks()`, `list_chunks()`, `get_chunk()`, `delete_chunks()`

**Key models:** `IndexParams` (HNSW/IVFFlat config), `IngestionJob`, `BatchCreated`, `BatchStatus`, `DocumentInfo`, `DocumentDetail`, `ChunkInfo`, `ChunkList`.

## Agent Identification

`TaprootClient` accepts an optional `agent_id: str` constructor parameter. When set, it is propagated as the `X-Agent-Id` header on every outbound request. Downstream services bind this value into their structlog context, so all log lines and audit events produced during that agent's request include the agent identifier automatically.

```python
client = TaprootClient(
    base_url="https://...",
    api_key="...",
    project_id="my-project",
    agent_id="my-agent-v2",   # propagated as X-Agent-Id on all requests
)
```

## ToolBox Client

### Tool Management

- `push_tool(name, source_code, entry_point, ...)` -- Push hosted Python function (FaaS).
- `register_tool(name, endpoint_url, ...)` -- Register external HTTP endpoint.
- `invoke_tool(tool_name, input_data)` -- Execute a tool and get result.
- `list_tools(tags, tool_type, status)` -- Filter by type (hosted/external/mcp) and status.
- `discover_tools(query, top_k)` -- Semantic search for tools.
- `get_tool()`, `update_tool()`, `delete_tool()`

**Key models:** `ToolInfo` (id, name, tool_type, status, input_schema; hosted fields: entry_point, requirements, runtime, timeout_ms, memory_mb; external fields: endpoint_url, http_method, auth_type), `InvocationResult` (invocation_id, success, result, error, duration_ms), `ToolList`.

### Credentials

`set_tool_credential(tool_id, credential_type, name, payload, expires_at)`, `list_credentials(tool_id)`, `revoke_credential(credential_id, version)`.

**Key model:** `CredentialInfo` (credential_type, name, status, expires_at, version). Property: `is_active`.

### OAuth Flow

`start_oauth_flow(tool_id, scope, redirect_uri)`, `complete_oauth_flow(tool_id, code, state)`, `client_credentials_grant(tool_id, client_id, client_secret, scope)`.

**Key models:** `OAuthFlowResponse`, `OAuthConnectionInfo`.

### MCP Server Registration

`register_mcp_server(name, transport, url)`, `list_mcp_servers()`, `get_mcp_server()`, `delete_mcp_server()`.

**Key model:** `MCPServerInfo` (transport: "sse"/"stdio"/"streamable_http", status: "registered"/"online"/"offline"/"error").

### Import/Export

- `import_openapi(namespace, spec_url, description, auth_type)` -> `ImportResult`
- `import_mcp_registry(registry_url, namespace)` -> `MCPRegistryImportResult`
- `export_mcp_registry()` -> dict

### Usage & Batch Push

- `get_usage_report(start_date, end_date)` -> `UsageReport`
- `push_decorated_tools()` -- Batch push functions decorated with `@client.tool()`

### CLI (taproot-tools)

Entry point: `taproot-tools = "taproot_sdk.toolbox.cli:main"` (console_scripts).

**Environment variables** (required): `TAPROOT_BASE_URL`, `TAPROOT_API_KEY`, `TAPROOT_PROJECT_ID`. Optional: `TAPROOT_DIRECT_MODE`.

**Subcommands:**
```bash
taproot-tools push <file> --name <name> --entry-point <func>
taproot-tools register <name> --endpoint-url <url>
taproot-tools invoke <name> --input '{"key": "value"}'
taproot-tools search "find weather tools"
taproot-tools list [--tags <tags>] [--type hosted|external|mcp]
taproot-tools get <tool-id>
taproot-tools delete <tool-id>
taproot-tools import-openapi --namespace <ns> --spec-url <url>
taproot-tools import-mcp-registry --registry-url <url>
taproot-tools export-mcp-registry [--output <file>]
taproot-tools set-credential --tool-id <id> --type api_key --name <name> --payload '{...}'
taproot-tools list-credentials [--tool-id <id>]
taproot-tools revoke-credential --credential-id <id> --version <v>
taproot-tools mcp-register --name <name> --transport sse --url <url>
taproot-tools mcp-list
taproot-tools mcp-get --id <id>
taproot-tools mcp-delete --id <id>

# All commands support --json for machine-readable output
```

## Exception Hierarchy

```
TaprootError (base Exception)
  TaprootAPIError (HTTP errors -- has status_code, detail, service, request_url, body)
    PromptNotFoundError (404 for prompts -- has prompt_name, prompt_project_id, prompt_version, prompt_label)
    AuthenticationError (401/403 -- has auth_project_id, auto-generates actionable hints)
    ConflictError (409 -- duplicate resources)
    RateLimitError (429 -- has retry_after from Retry-After header)
    ServerError (5xx -- has attempts, total_wait_seconds)
    ValidationError (422 -- has errors list, parses FastAPI + Guardrail-S formats, caps at 5)
  MissingVariableError (prompt rendering -- has variable_name, required_variables)

EvalAssertionError (subclass of AssertionError, NOT TaprootError -- has result attribute)
```

**TaprootAPIError message format:** `"HTTP {status} [{service}] -- {detail} (URL: {request_url})"`

**AuthenticationError hints:**
- 403: "Access denied for project '{pid}'. Verify your API key has access..."
- 401: "Authentication failed. Verify your API key is valid..."

**ServerError message:** Appends "(after {attempts} attempts over ~{total_wait_seconds:.0f}s). This is a server-side issue..."

**ValidationError parsing:** Handles two error formats -- FastAPI (`{"detail": [{loc: [...], msg: "..."}]}`) and Guardrail-S (`{"errors": [{field: "...", message: "..."}]}`).

## Configuration

### init() Parameters

```python
ev.init(
    project_id="my-project",           # Required
    api_url="https://gateway.taproot.dev",  # Required -- APIM gateway URL
    api_key="sk-...",                  # Optional -- raw API key
    auto_instrument=["openai"],        # Optional -- LLM libraries to auto-instrument
    redact_by_default=True,            # Optional (default True)
    sampling_rate=1.0,                 # Optional (default 1.0) -- 0.0 to 1.0
    batch_size=512,                    # Optional -- spans per export batch
    flush_interval_ms=5000,            # Optional -- max time between flushes
    service_name="my-service",         # Optional -- defaults to "taproot-{project_id}"
    service_version="1.0.0",           # Optional
)
```

**Internal details:**
- Creates `TracerProvider` with `Resource` containing `service.name` and `taproot.project_id`
- Sampler: `ParentBased(root=TraceIdRatioBased(sampling_rate))`
- Exporter: `JsonOtlpSpanExporter` (JSON, not protobuf)
- Endpoint: `{api_url}/api/v1/evals/v1/traces`
- Auth header: `x-api-key: {api_key}`
- Batch processor: `max_queue_size=batch_size*4`, `export_timeout_millis=30000`
- Registers `atexit` shutdown handler
- Raises `RuntimeError` if called twice without `shutdown()` first

### Environment Variables

| Variable | Used By | Description |
|----------|---------|-------------|
| `TAPROOT_EVAL_URL` | pytest_plugin | Base URL for eval_client fixture |
| `TAPROOT_API_KEY_ID` | pytest_plugin | API key for eval_client fixture |
| `TAPROOT_PROJECT_ID` | pytest_plugin, CLI | Project ID |
| `TAPROOT_BASE_URL` | CLI | API gateway URL |
| `TAPROOT_API_KEY` | CLI | API key |
| `TAPROOT_DIRECT_MODE` | CLI | "true" for direct mode |

### Installation

```bash
# Core SDK (tracing + client)
pip install taproot-sdk

# With specific LLM auto-instrumentation
pip install taproot-sdk[openai]
pip install taproot-sdk[anthropic]
pip install taproot-sdk[google]
pip install taproot-sdk[cohere]
pip install taproot-sdk[bedrock]
pip install taproot-sdk[vertexai]
pip install taproot-sdk[mistral]

# All LLM providers
pip install taproot-sdk[all]

# Development
pip install taproot-sdk[dev,all]
```

### Core Dependencies

- `opentelemetry-api>=1.21.0`
- `opentelemetry-sdk>=1.21.0`
- `opentelemetry-exporter-otlp-proto-http>=1.21.0`
- `httpx>=0.25.0`

### Dev Dependencies

- `pytest>=7.0.0`, `pytest-asyncio>=0.21.0`, `pytest-cov>=4.0.0`
- `mypy>=1.0.0` (strict mode)
- `ruff>=0.1.0` (line-length 100)
- `respx>=0.20.0` (HTTP mocking for tests)

## Testing

### Conventions

- **pytest** with `asyncio_mode = "auto"` (pyproject.toml)
- **Autouse fixture** `reset_sdk()` in `tests/conftest.py` calls `shutdown()` before and after each test to ensure clean global state
- All response models have `from_api_response()` / `from_dict()` tests
- Client methods tested with `respx` for HTTP mocking
- Decorator tests verify both sync and async functions, error recording, input/output capture, and truncation
- No live network calls in unit tests

### Running Tests

```bash
pytest                                           # All tests
pytest tests/ -v --tb=short                      # Verbose with short traceback
pytest --cov=src/taproot_sdk --cov-report=term-missing  # With coverage
pytest tests/test_decorators.py -v               # Single file
pytest tests/test_decorators.py::test_name -v    # Single test
```

### Linting & Type Checking

```bash
ruff check src/                    # Lint (line-length 100)
ruff format --check src/           # Format check
mypy src/                          # Strict mode type checking
```

### Test Directory Map

| Directory | What it tests |
|-----------|---------------|
| `tests/test_core.py` | init(), shutdown(), get_tracer() |
| `tests/test_decorators.py` | @instrument() sync/async/error/truncation |
| `tests/test_auto_instrument.py` | Auto-instrumentation setup/teardown |
| `tests/test_client.py` | TaprootClient constructor, routing, retries |
| `tests/test_exceptions.py` | Exception hierarchy, message formatting |
| `tests/auto_instrumentation/` | Per-provider instrumentor tests |
| `tests/evals/` | EvalResult, assert_eval(), pytest fixtures |
| `tests/prompts/` | PromptResponse, cache, rendering, A/B testing |
| `tests/retrieval/` | Retrieval models, client methods |
| `tests/toolbox/` | ToolInfo, CLI, tool decorator, OAuth, usage |

## Integration Patterns

### End-User Application

```python
import taproot_sdk as ev

# 1. Initialize tracing
ev.init(project_id="my-app", api_url="https://gw.taproot.dev", api_key="sk-...")

# 2. Auto-trace LLM calls
import openai
client = openai.OpenAI()  # Automatically instrumented

# 3. Trace custom functions
@ev.instrument(spankind="retrieval")
def search(query: str) -> list:
    ...

# 4. Use platform services
async with ev.TaprootClient() as tc:
    prompt = await tc.get_prompt("welcome")
    guard = await tc.check_input(user_input)
    if guard.is_blocked:
        return "Content blocked"
    results = await tc.retrieval_query("my-store", user_input)
```

### CI/CD Pipeline

```python
import taproot_sdk as ev

async def test_agent_quality():
    async with ev.TaprootClient(
        base_url="https://gw.taproot.dev",
        api_key="ci-key",
        project_id="my-project",
    ) as client:
        handle = await client.trigger_eval_run("test-config-id")
        result = await client.wait_for_eval(handle.run_id, timeout=300)
        ev.assert_eval(result, min_pass_rate=80)
```

### Pytest Plugin Workflow

```python
# conftest.py
pytest_plugins = ["taproot_sdk.evals.pytest_plugin"]

# Set env vars: TAPROOT_EVAL_URL, TAPROOT_API_KEY_ID, TAPROOT_PROJECT_ID

# tests/test_quality.py
@pytest.mark.asyncio
async def test_agent_quality(eval_run):
    result = await eval_run("test-config-uuid", tags=["ci"])
    assert_eval(result, min_pass_rate=80)
```

## Key Files

| File | Lines | Purpose |
|------|-------|---------|
| `src/taproot_sdk/__init__.py` | 188 | Public API surface (~100 exports in `__all__`) |
| `src/taproot_sdk/core.py` | 251 | SDK init, TracerProvider, OTLP exporter setup |
| `src/taproot_sdk/decorators.py` | 294 | @instrument() decorator with 9 span kinds |
| `src/taproot_sdk/auto_instrument.py` | 200 | Dynamic LLM instrumentor loading |
| `src/taproot_sdk/client.py` | ~4585 | TaprootClient -- ~120 async methods for all services |
| `src/taproot_sdk/exceptions.py` | 240 | Typed exception hierarchy |
| `src/taproot_sdk/otlp_json_exporter.py` | 147 | JSON OTLP exporter for API Gateway compat |
| `src/taproot_sdk/evals/models.py` | 721 | 15+ eval-related frozen dataclasses |
| `src/taproot_sdk/evals/assertions.py` | 68 | assert_eval() CI/CD helper |
| `src/taproot_sdk/evals/pytest_plugin.py` | 82 | eval_client, eval_run fixtures |
| `src/taproot_sdk/guardrails/models.py` | 152 | GuardrailResponse, ScannerSignal, etc. |
| `src/taproot_sdk/guardrails/check_results.py` | 90 | CheckResult, AnalyticsSummary |
| `src/taproot_sdk/guardrails/configs.py` | 67 | GuardrailConfig, ScannerOverride |
| `src/taproot_sdk/prompts/models.py` | 319 | PromptResponse with render/verify methods |
| `src/taproot_sdk/prompts/cache.py` | 269 | Stale-while-revalidate prompt cache |
| `src/taproot_sdk/prompts/client.py` | 370 | DEPRECATED PromptClient |
| `src/taproot_sdk/retrieval/models.py` | 757 | 20+ retrieval frozen dataclasses |
| `src/taproot_sdk/toolbox/models.py` | 390 | ToolInfo, InvocationResult, MCPServerInfo, etc. |
| `src/taproot_sdk/toolbox/cli.py` | 866 | taproot-tools CLI with 16+ subcommands |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Prohan21)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Prohan21)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
