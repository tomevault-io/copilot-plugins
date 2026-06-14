## memory-server

> > This document is intended for AI agents (Antigravity, Codex, etc.) to quickly orient themselves when working on this repository.

# memory-server — AI Agent Developer Guide

> This document is intended for AI agents (Antigravity, Codex, etc.) to quickly orient themselves when working on this repository.  
> Authoritative design reference: `../云记忆框架/设计文档/` (v1.3)

---

## Repository Overview

`memory-server` is the **sole core business backend** for the YuYi (羽忆) Unified Cloud Memory Infrastructure. It owns all business logic, API routing, memory judgment, recall orchestration, and security control.

- **Language / Framework**: Java 21 + Spring Boot 3.3.5
- **Database**: PostgreSQL 16 + pgvector (vector search) + zhparser (Chinese full-text search)
- **ORM**: MyBatis-Plus 3.5.7
- **Migrations**: Flyway
- **Auth model**: Admin HttpOnly Session Cookie + API Token (dual credential)
- **API docs**: SpringDoc OpenAPI 3.1, available at `/swagger-ui.html` at runtime
- **Build artifact**: `memory-server.jar` (fixed via `archiveFileName` in `build.gradle.kts`)

---

## Package Structure

Root package: `top.zyaire.memory`

| Sub-package | Responsibility |
|-------------|----------------|
| `auth` | Admin login / logout / `me` / bootstrap-status |
| `security` | API Token management (CRUD, revoke), auth filter chain, Spring Security config |
| `gateway` | Global exception handling (`advice/`), CORS config, OpenAPI config, rate-limit filter |
| `identity` | Current user resolution, Scope validation and auto-fill |
| `ingest` | Write Pipeline entry: write stable memories / history records |
| `judge` | Memory Judge: three-stage decision chain (hard guard → LLM → rule fallback) |
| `stable` | Stable Memory Service: CRUD, versioning, state machine, lexical/vector search |
| `history` | History Record Service: append writes, TTL policy, lexical/vector search |
| `recall` | Recall Orchestrator: hybrid retrieval → scoring → dedup → conflict detection → token budget |
| `search` | HybridSearchService: RRF fusion (lexical + vector), shared by recall and search endpoints |
| `embedding` | Embedding Provider management: create / activate / test; runtime embedding generation |
| `llm` | LLM Provider management; HTTP client wrapper for LLM calls in Judge |
| `audit` | Audit event recording and query |
| `dashboard` | Aggregated view APIs for Web Console (no business logic duplication) |
| `control` | Mode switching (manual / assistive / temporary), auto-write toggle |
| `common` | Unified response wrapper, exception hierarchy, enums, validators, config classes |

---

## Key Entry Points

| File | Description |
|------|-------------|
| `MemoryServerApplication.java` | Spring Boot main class, application entry point |
| `security/config/SecurityConfig.java` | Filter chain declaration, CORS config, public path whitelist |
| `gateway/advice/GlobalExceptionHandler.java` | Unified JSON error response for all unhandled exceptions |
| `common/response/ApiResponse.java` | **Unified response wrapper for all endpoints** — format: `{code, message, data, httpStatus, timestamp}` |
| `common/exception/ErrorCode.java` | Full business error code registry — format: `MEM-{category}{number}` |
| `common/config/MemoryProperties.java` | `memory.*` configuration property binding |

---

## Three Core Pipelines and Key Classes

### A. Write Pipeline

```
IngestController → IngestService → MemoryJudge → StableMemoryService / HistoryRecordService
```

| Class | Location | Description |
|-------|----------|-------------|
| `IngestController` | `ingest/controller/` | `POST /v1/memories`, `POST /v1/history-records` |
| `IngestService` | `ingest/service/` | Scope auto-fill, temporary mode check, Judge invocation, storage routing |
| `MemoryJudge` | `judge/service/` | Decision facade: hard guard first, then LLM, then rule fallback |
| `RuleEngine` | `judge/rule/` | L1 deterministic rule set (sensitive content, length, keyword type inference, importance scoring, semantic_key generation) |
| `LlmJudgeService` | `judge/service/` | L2 LLM call (structured JSON output validation, auto-fallback on low confidence) |

**Decision chain execution order** (strictly enforced in code, cannot be bypassed):

1. `RuleEngine.evaluateHardGuards()` — sensitive content / length violations → immediate `REJECT`, **LLM cannot override**
2. `LlmJudgeService.judge()` — optional; confidence must be ≥ `memory.judge.llm-confidence-threshold` (default `0.65`)
3. `RuleEngine.evaluateFallback()` — keyword-based type inference + importance + semantic_key + tags defaults

**Expected rejection scenarios** (400/403 on write):

- `MEM-MEM004` — Judge rejected (noise, too short, too long)
- `MEM-MEM007` — Sensitive content detected (password/key assignment pattern, Bearer token literal, PEM private key block)
- `MEM-MEM006` — Temporary mode is active

### B. Recall Pipeline

```
RecallController → RecallOrchestrator → HybridSearchService → ImportanceRecencyScorer → RecallContextBlock
```

| Class | Location | Description |
|-------|----------|-------------|
| `RecallController` | `recall/controller/` | `POST /v1/recall`, `GET /v1/memories/recent` |
| `RecallOrchestrator` | `recall/service/` | 8-step recall flow: candidates → score → sort → dedup → conflicts → budget → freshness → assemble |
| `HybridSearchService` | `search/service/` | RRF fusion: lexical (zhparser websearch / plainto_tsquery) + vector (pgvector), each can degrade independently |
| `ImportanceRecencyScorer` | `recall/scorer/` | 以归一化 relevance 为主，再叠加 importance / recency，避免高 importance 背景资料压过任务焦点 |

**Token budget allocation** (`RecallOrchestrator.allocateBudget`):

| `prefer` value | stable budget | history budget |
|---|---|---|
| `stable_first` (default) | 70% | 30% |
| `history_first` | 30% | 70% |
| `balanced` | 50% | 50% |
| `stable_only` | 100% | 0% |
| `history_only` | 0% | 100% |

Recall 结果还会额外受条数上限控制，避免在小数据集下退化成“把当前 Agent 的记忆整包拼进上下文块”。

### C. Control Pipeline

| Endpoint | Class | Description |
|----------|-------|-------------|
| `GET /v1/memories` | `StableMemoryController` | Paginated list; supports `projectId` / `status` / `memoryType` / `hasConflict` filters |
| `GET /v1/memories/{id}` | `StableMemoryController` | Single memory detail with ownership check |
| `DELETE /v1/memories/{id}` | `StableMemoryController` | Soft delete |
| `PATCH /v1/memories/{id}/status` | `StableMemoryController` | State transition (invalid transitions return `MEM-MEM003`) |
| `GET/PUT /v1/settings/mode` | `ControlController` | Read/write current operating mode |

**Valid state transitions** (all others are rejected):

```
active  → archived  (reversible)
active  → invalid   (reversible)
active  → deleted   (reversible within tombstone TTL)
archived → active   (restore)
invalid  → active   (restore)
deleted  → active   (only within tombstone TTL)
```

---

## Authentication & Security

### Dual Credential Model

| Credential | Use Case | Filter |
|------------|----------|--------|
| Admin Session Cookie (HttpOnly) | Browser / Web Console | `AdminSessionAuthFilter` |
| API Token (`Authorization: Bearer mk-...`) | CLI / MCP / automation | `ApiTokenAuthFilter` |

Both filters are registered before `UsernamePasswordAuthenticationFilter`; either passing is sufficient.

### Public Paths (no auth required)

```
/v1/health
/v1/auth/login
/v1/auth/bootstrap-status
/v1/api-docs/**
/swagger-ui/**
/actuator/health
OPTIONS /**   (CORS preflight)
```

### Admin-only Endpoints (Session required; API Token not accepted)

```
/v1/auth/**
/v1/admin/**
```

### CORS

In production, allowed origins **must** be explicitly configured via `memory.security.cors-allowed-origin-patterns`. The default allows only `localhost:*`.

---

## Key Enums and Constants

### MemoryType (Stable Memory types)

| Value | Description | Default importance |
|-------|-------------|:-----------------:|
| `preference` | User preference | 0.70 |
| `project_rule` | Project convention | 0.90 |
| `decision` | Technical/business decision | 0.85 |
| `fact` | Confirmed fact | 0.60 |
| `workflow` | Process/method | 0.75 |
| `reference` | Stable reference | 0.60 |
| `summary` | Summarized memory | 0.55 |

Unknown type values fall back to `fact` — `MemoryType.fromValue` never throws.

### RecordKind (History Record types)

`task_summary` / `decision_trace` / `session_excerpt` / `recent_progress` / `incident_context` / `meeting_note`

### MemoryStatus

`active` / `archived` / `invalid` / `deleted`

### ErrorCode (format: `MEM-{category}{number}`)

| Category | Prefix | Examples |
|----------|--------|---------|
| Authentication | `MEM-AUTH` | `MEM-AUTH001` (unauthenticated), `MEM-AUTH005` (wrong password) |
| Memory operations | `MEM-MEM` | `MEM-MEM001` (not found), `MEM-MEM006` (temporary mode) |
| Recall | `MEM-RECALL` | `MEM-RECALL002` (token budget too small) |
| Judge / LLM | `MEM-JUDGE` | `MEM-JUDGE001` (LLM provider not found) |
| Embedding | `MEM-SEARCH` | `MEM-SEARCH006` (no active embedding provider) |
| System / Params | `MEM-SYS` | `MEM-SYS003` (validation error), `MEM-SYS500` (internal error) |

---

## semantic_key Rules (Enforced at Code Level)

Format: `{scope_prefix}:{domain}:{topic}`

- Only lowercase letters, digits, and underscores are allowed
- Each segment must start with a lowercase letter
- Total length ≤ 160 characters
- Validator class: `common/validator/SemanticKeyValidator.java`
- Invalid format → `MEM-MEM005`, write rejected
- Resolution priority: caller-provided > RuleEngine inference > LLM inference > not set (excluded from key-level dedup)

---

## Configuration Reference (`application.yml` + environment variables)

| Environment Variable | Default | Description |
|----------------------|---------|-------------|
| `MEMORY_BOOTSTRAP_ADMIN_USERNAME` | `` | Initial admin username |
| `MEMORY_BOOTSTRAP_ADMIN_PASSWORD` | `` | Initial admin password |
| `MEMORY_JUDGE_LLM_ENABLED` | `false` | Enable LLM Judge (stage 2) |
| `MEMORY_JUDGE_LLM_CONFIDENCE_THRESHOLD` | `0.65` | Minimum confidence for LLM Judge result to be accepted |
| `MEMORY_EMBEDDING_ENABLED` | `false` | Enable vector search |
| `MEMORY_EMBEDDING_MASTER_KEY` | `` | Master encryption key for embedding provider secrets |
| `MEMORY_LLM_ENABLED` | `false` | Enable console-managed LLM provider |
| `MEMORY_LLM_MASTER_KEY` | `` | Master encryption key for LLM provider secrets |
| `SERVER_PORT` | `8080` | HTTP server port |
| `SPRING_PROFILES_ACTIVE` | `dev` | Active Spring profile (`dev` / `prod`) |

Recall tuning (`memory.recall.*` in `application.yml`, not overridable via env vars):

- `default-max-tokens: 4000`
- `freshness-warn-days: 30` / `freshness-caution-days: 90` / `freshness-stale-days: 365`

---

## HybridSearchService Degradation Behavior

Vector search (pgvector) and lexical search (zhparser) **degrade independently** — neither failure brings down the full pipeline:

```
No active embedding provider  → skip vector candidates; only lexical results feed RRF
zhparser websearch unavailable → auto-downgrade to plainto_tsquery
Both fail                      → empty candidate set; Recall returns an empty block (no 500)
```

---

## Development Conventions and Key Decisions

### Hard Constraints (enforced in code)

1. **All endpoints return `ApiResponse<T>`** with the fixed format `{code, message, data, httpStatus, timestamp}`.
2. **L1 rule results cannot be overridden by LLM** — in `MemoryJudge.judge()`, hard guards run before the LLM call.
3. **Sensitive content detection scope**: only detects value-assignment patterns (`password = xxx`), Bearer token literals, and PEM private key blocks. Conceptual descriptions (e.g., "discussing token-based authentication") must not be blocked.
4. **Stable memories with importance ≥ 0.9 are never cut by the token budget** (hard-guaranteed in `applyTokenBudget`).
5. **Admin bootstrap** is done via `MEMORY_BOOTSTRAP_ADMIN_*` environment variables; `/v1/auth/bootstrap-status` is used by the frontend to check initialization state.
6. **API Token plaintext is returned only once** at creation time; subsequent list APIs return only the prefix and metadata.

### Module Boundaries

- `dashboard/` provides only aggregated view APIs for the Web Console — **do not** copy business logic here.
- Access-layer clients (CLI / MCP) must call via HTTP API — **do not** replicate Judge rules on the client side.
- `search/HybridSearchService` is the sole retrieval entry point for both recall and search — this ensures "anything the search UI can find, the recall pipeline can also consider."

### LLM Judge Current Status

- Disabled by default (`MEMORY_JUDGE_LLM_ENABLED=false`).
- `NEEDS_CONFIRMATION` decision type is **not yet wired** — `LlmJudgeService` throws immediately on this value, triggering rule fallback.
- Only `writeMemory` flows through LLM Judge; history record writes (`writeHistory`) still use rule-only judgment.

---

## Common Commands

```bash
# Start locally (requires a local PostgreSQL instance)
./gradlew bootRun

# Build JAR
./gradlew bootJar

# Run tests
./gradlew test

# MVP acceptance script (requires service running on port 8080)
bash verify-mvp.sh
```

---

## Database Migrations

Migration scripts are in `src/main/resources/db/migration/` and are applied automatically by Flyway.

Naming convention for new scripts: `V{version}__{description}.sql` — version numbers must be monotonically increasing. **Never modify a migration script that has already been applied.**

---

## Quick Navigation

| Task | Entry point |
|------|-------------|
| Write a stable memory | `IngestController#writeMemory` → `IngestService` → `MemoryJudge` → `StableMemoryService` |
| Recall task context | `RecallController#recall` → `RecallOrchestrator#recall` (8-step flow) |
| Understand the judgment chain | `MemoryJudge` → `RuleEngine.evaluateHardGuards` → `LlmJudgeService.judge` → `RuleEngine.evaluateFallback` |
| Add a new Judge rule | `judge/rule/RuleEngine.java` + `judge/model/JudgeRuleConfigDO.java` + DB migration |
| Modify a standard error code | `common/exception/ErrorCode.java` |
| Change security public paths | `security/config/SecurityConfig.java`, `PUBLIC_PATHS` constant |
| Tune recall freshness thresholds | `memory.recall.freshness-*-days` in `application.yml` |

---
> Source: [PlumeMemory/memory-server](https://github.com/PlumeMemory/memory-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-14 -->
