## ganglia

> Ganglia is a **Java 25** AI Agent framework built on **Vert.x 5.0.6**, designed for high-performance, non-blocking agentic workflows. It follows a **Hexagonal (Ports & Adapters)** architecture with a robust ReAct reasoning loop, pluggable memory via SPI, plan mode with disk persistence, and multi-frontend support (Terminal, WebUI, IM channels).

# AGENTS.md — Ganglia Project Instructions

## 1. Project Overview

Ganglia is a **Java 25** AI Agent framework built on **Vert.x 5.0.6**, designed for high-performance, non-blocking agentic workflows. It follows a **Hexagonal (Ports & Adapters)** architecture with a robust ReAct reasoning loop, pluggable memory via SPI, plan mode with disk persistence, and multi-frontend support (Terminal, WebUI, IM channels).

**Version:** 0.1.7-SNAPSHOT

## 2. Technology Stack

|    Category     |                                   Technology                                    |
|-----------------|---------------------------------------------------------------------------------|
| Runtime         | Java 25-zulu (SDKMAN! managed)                                                  |
| Core            | Vert.x 5.0.6 (Reactive, Non-blocking I/O)                                       |
| LLM Integration | Native OpenAI & Anthropic protocol, Codex (OkHttp SSE)                          |
| Terminal UI     | JLine 3.25.1, CommonMark (ANSI Markdown)                                        |
| Web UI          | React 18 + TypeScript (Vite), Vert.x Web (WebSocket + JSON-RPC 2.0)             |
| Networking      | Vert.x WebClient 5.0.6                                                          |
| Logging         | SLF4J 2.0.16 + Log4j2                                                           |
| Testing         | JUnit 5, Mockito, Vertx-JUnit5, E2E Simulation Harness                          |
| Code Quality    | Spotless (Google Java Format), Checkstyle (google_checks.xml), SpotBugs, JaCoCo |

## 3. Module Structure

```
ganglia-parent (pom.xml)
├── ganglia-harness                  # Core: kernel, ports, infrastructure (no memory impl)
├── ganglia-local-file-memory        # File-based memory SPI implementation
├── ganglia-sqlite-memory            # SQLite-backed memory SPI implementation
├── ganglia-observability            # Independent Trace Studio backend and REST API
├── ganglia-trajectory               # Unified observation event stream and JSONL trace management
├── coding-agent/                    # Directory grouping (NOT a Maven module)
│   ├── ganglia-coding               # Coding agent builder + tools (bash, file-edit, web-fetch)
│   ├── ganglia-coding-web           # WebSocket + JSON-RPC 2.0 web UI backend
│   ├── ganglia-terminal             # JLine 3 terminal UI
│   ├── ganglia-swe-bench            # SWE-bench evaluation with Docker sandboxing
│   └── ganglia-coding-webui         # React multi-page frontend (NOT a Maven module)
├── trading-agent/                   # Directory grouping
│   ├── ganglia-trading              # Multi-agent adversarial trading system
│   ├── ganglia-trading-web          # WebSocket backend for trading dashboard
│   └── ganglia-trading-webui        # React frontend for trading UI (NOT a Maven module)
├── claw-agent/                      # Directory grouping
│   ├── ganglia-claw                 # IM-integrated agent orchestration (channels, ACP, scheduling)
│   ├── ganglia-claw-web             # Management dashboard REST API
│   └── ganglia-claw-app             # Application layer: daemon lifecycle, multi-agent verticles
├── e2e-test                         # E2E tests with real LLM calls (trading/, coding/, channel/, gateway/)
├── integration-test                 # Component integration tests with mock models
└── ganglia-example                  # Demo apps (WebUIDemo)
```

> **Note:** `coding-agent/`, `trading-agent/`, and `claw-agent/` are plain directories, not Maven aggregators. Sub-modules are listed directly in the root `pom.xml`. `*-webui` projects are standalone Vite/React projects, not managed by Maven.

### Dependency Graph

```
ganglia-harness
    ↑
ganglia-local-file-memory, ganglia-sqlite-memory, ganglia-trajectory
    ↑
ganglia-coding, ganglia-trading, ganglia-claw, ganglia-observability
    ↑
ganglia-coding-web, ganglia-trading-web, ganglia-claw-web
    ↑
ganglia-claw-app (combines claw + trading + coding + sqlite-memory)
    ↑
integration-test, ganglia-example, ganglia-swe-bench
```

## 4. Build & Development Commands

|             Command             |                                              Description                                              |
|---------------------------------|-------------------------------------------------------------------------------------------------------|
| `mvn clean install -DskipTests` | Full build (skip tests)                                                                               |
| `just test-backend`             | Unit tests: all Java modules (harness, memory, coding, trading, claw, etc.) |
| `just test-it`                  | Integration tests                                                                                     |
| `just test-it-one <ClassName>`  | Single integration test                                                                               |
| `just frontend`                 | Vite dev server (port 5173)                                                                           |
| `just backend`                  | WebUI backend (port 8080)                                                                             |
| `just obs`                      | Observability Studio backend (port 8081)                                                              |
| `just ui-watch`                 | Frontend watch mode (auto-rebuild dist/)                                                              |
| `just coverage`                 | JaCoCo coverage report                                                                                |
| `just build-all`                | Full production build (UI + Backend JAR)                                                              |
| `just clean`                    | Clean all build artifacts                                                                             |

## 5. Architecture

### 5.1 Hexagonal Layers

- **Kernel** (`ganglia-harness/kernel/`): ReAct reasoning loop, task scheduling, sub-agents
  - `GangliaKernel` — Main orchestrator, late-binding assembly via `AgentEnv`
  - `ReActAgentLoop` — Iterative Thought → Action → Observation cycle
  - `AgentTaskFactory` / `DefaultAgentTaskFactory` — Maps LLM tool calls to executable tasks
  - `DefaultGraphExecutor` — DAG execution with mission context propagation and `inputMapping` wiring
  - `Blackboard` / `InMemoryBlackboard` — Append-only cross-cycle fact store with optimistic concurrency
  - `PersonaRegistry` — Extensible Persona registry with built-in personas (GENERAL, INVESTIGATOR, REFACTORER); agents inject custom personas via `BootstrapOptions.extraPersonas()`
  - `InterceptorPipeline` — Pre/post turn hooks with `executeOnTurnComplete()` for fire-and-forget post-processing
- **Observability** (`ganglia-observability/`): Trace Studio, hierarchical execution tracking
  - `ObservabilityVerticle` — Independent REST API and UI server (Port 8081)
  - `StructuredTraceManager` — Writes JSONL traces with parent-child span relationships
- **Port** (`ganglia-harness/port/`): Domain interfaces and models
  - `chat/` — Message, Turn, SessionContext (immutable records)
  - `internal/` — PromptEngine, ModelGateway, ContextOptimizer, SessionManager, StateEngine, Blackboard
  - `internal/memory/` — MemoryService, MemorySystemProvider, MemorySystem, MemorySystemConfig (SPI)
  - `external/` — ToolSet, ExecutionContext
  - `kernel/subagent/blackboard/` — Fact, FactStatus (domain records for Blackboard)
  - `mcp/` — MCP protocol support
- **Infrastructure** (`ganglia-harness/infrastructure/`): Technical implementations
  - `external/` — Model gateways (OpenAI, Anthropic), ToolsFactory
  - `internal/` — Prompt engine, state persistence, context optimization

### 5.2 Memory SPI Pattern

Memory is pluggable via `java.util.ServiceLoader`:

```
MemorySystemProvider (SPI interface in ganglia-harness)
    ├── FileSystemMemoryProvider (ganglia-local-file-memory)
    └── SqliteMemoryProvider (ganglia-sqlite-memory)
            └── Creates MemorySystem record containing:
                MemoryStore, ObservationCompressor, ContextCompressor,
                TimelineLedger, DailyRecordManager, LongTermMemory,
                MemoryService, MemoryContextSource, KeyFactContextSource,
                SessionStore
```

**SessionStore** provides:
- `saveSession` / `getSession` / `searchSessions` — session record persistence and FTS search
- `saveKeyFacts` / `getKeyFacts` — structured key fact extraction and retrieval (categories: DECISION, FILE_CHANGE, ERROR, USER_PREFERENCE, FACT)
- `saveSessionMemory` / `getSessionMemory` — running session memory persistence (markdown summary)

**Automatic extraction pipeline** (fire-and-forget on turn completion):
- `KeyFactExtractor` — LLM-based structured fact extraction, persisted via `SessionStore.saveKeyFacts()`
- `SessionMemoryExtractor` — periodic session summary extraction (every 3 turns), persisted via `SessionStore.saveSessionMemory()`
- `KeyFactContextSource` — injects previously extracted facts into system prompt
- `LongTermMemory.consolidateUserProfile()` — LLM-based profile deduplication (both SQLite and FileSystem backends)

**SQLite schema migrations**: V1 (core tables) → V2 (graph checkpoints) → V3 (key_facts) → V4 (session_memory)

`GangliaKernel` discovers the provider at startup via SPI.

### 5.3 Prompt Context Layering

`DefaultPromptEngine` + `ContextComposer` stack priority-based layers:

1. **Kernel** — Persona, mandates, plan mode instructions
2. **Process** — Workflow directives (includes planning heuristics and task tracking guidance)
3. **Rule** — Guidelines, tool descriptions
4. **Capability** — Skills injection
5. **Context** — Environment, active plan content, memory index, key facts, todo list

Context sources:
- `ToDoContextSource` — summarized/detailed plan views
- `PlanModeContextSource` — plan mode restrictions + active plan content from disk
- `KeyFactContextSource` — previously extracted structured facts
- `MemoryIndexContextSource` — memory entry index
- `DailyContextSource` — daily journal

Fragments are pruned by priority when token budget is exceeded.

### 5.3.1 Plan Mode & Task Tracking

**Plan mode** provides read-only isolation for planning before execution:
- `enter_plan_mode` — activates plan mode, restricts tools to readOnly via `DefaultToolExecutor` filtering
- `exit_plan_mode` — saves plan to `.ganglia/plans/{name}.md`, triggers user approval (interrupt)
- `read_plan` / `list_plans` — cross-session plan access
- Plan files persist on disk, independent of session lifecycle

**PlanStore** port with `FileSystemPlanStore` implementation.

**Todo tools** for session-scoped task tracking:
- `todo_add` — structured tasks with subject, description, activeForm, blockedBy
- `todo_update` — status changes (in_progress/done/failed/skipped), dependency management
- `todo_get` / `todo_list` — task retrieval
- Completion triggers context compression via `ContextCompressor.summarize()`
- Dependencies enforced: blocked tasks cannot be marked done until blockers complete

**Heuristic guidance** in system prompt ("When to Plan First") lets the LLM self-decide when to plan.

### 5.3.2 Context Compression Pipeline

7-step pipeline in `DefaultContextOptimizer`, executed in fixed insertion order (no priority sorting):

```
Step 1: ToolResultBudgetStep        (non-LLM) — truncate oversized tool results (>50K chars)
Step 2: HardLimitGuardStep          (non-LLM) — fail-fast if hard token limit exceeded
Step 3: SnipStep                    (non-LLM) — remove oldest complete turns (>80% usage)
Step 4: TimeBasedMicrocompactStep   (non-LLM) — clear expired tool results by cache TTL
Step 5: SlimmingStep                (non-LLM) — strip thinking blocks, compact tool args, dedup summaries
Step 6: SessionMemoryCompactStep    (non-LLM) — replace old turns with existing session memory summary
Step 7: CompressionStep             (LLM)     — chunked LLM summarization (last resort)
```

**Design principle**: non-LLM compression first, LLM as last resort.

**Reactive compact**: on `CONTEXT_OVERFLOW` API error, `LoopRecoveryHandler` calls `contextOptimizer.reactiveCompact()` and retries once.

**Observability**: `CompactionReport` records per-step tokensSaved, durationMs, applied status.

**Pressure model**: 4-level `ContextPressure` (NORMAL <60%, WARNING 60-80%, CRITICAL 80-95%, BLOCKING >95%).

### 5.4 Model Gateways

- `AbstractModelGateway` — DRY base with `normalizeEndpoint`, `collectToolCalls`, `buildStreamingResponse`, semaphore (max 5 concurrent calls)
- `OpenAIModelGateway` / `AnthropicModelGateway` — Native protocol implementations
- `CodexModelGateway` — OkHttp-based SSE streaming for Codex/local API servers
- `RetryingModelGateway` — Exponential backoff retry wrapper using `category().isRetryable()`
- `LLMErrorCategory` — Taxonomy-based error classification (AUTH, BILLING, RATE_LIMIT, CONTEXT_OVERFLOW, OVERLOADED, SERVER_ERROR, TIMEOUT, MODEL_NOT_FOUND, FORMAT_ERROR, UNKNOWN) with `isRetryable()` and `shouldSwitchProvider()` strategies
- `FallbackModelGateway` — Error-aware fallback using `category().shouldSwitchProvider()` (switches on AUTH, BILLING, MODEL_NOT_FOUND, OVERLOADED; does not switch on RATE_LIMIT, FORMAT_ERROR)
- `AnthropicModelGateway` now supports **Prompt Caching** — system prompt, tools, and strategic message breakpoints with `cache_control` annotations; `TokenUsage` extended with `cacheCreationInputTokens` and `cacheReadInputTokens`

### 5.5 Coding Agent

`CodingAgentBuilder` (in `ganglia-coding`) assembles:
- Tools: BashFileSystemTools, BashTools, FileEditTools, WebFetchTools
- Context sources: CodingPersonaContextSource, CodingWorkflowSource, CodingGuidelineSource, FileContextSource
- PathMapper support for environment isolation

### 5.6 Trading Agent

`TradingAgentBuilder` (in `ganglia-trading`) assembles a multi-agent adversarial trading system:
- **Pipeline stages:** Perception → Debate → Risk → Execution (DAG-based via GraphBuilder classes)
- **Technical indicators:** SMA, EMA, MACD, RSI, ATR, Bollinger Bands, VWMA, MFI (via `IndicatorCalculator` + `IndicatorService`)
- **Data providers:** `MarketDataProvider` SPI with FMP (primary) and AlphaVantage (fallback), routed via `ProviderRouter` with `ProviderRateLimitException`-triggered failover
  - `FmpStockVendor`, `FmpFundamentalsVendor`, `FmpNewsVendor` — FMP (Financial Modeling Prep) as primary source
  - `AlphaVantageStockVendor`, `AlphaVantageFundamentalsVendor`, `AlphaVantageNewsVendor` — AlphaVantage as fallback (retains news sentiment scoring)
  - `ProviderFactory` registers providers; `TradingConfig.DataProvider` enum (`FMP`, `ALPHA_VANTAGE`) configures priority
- **Cache system:** Three-layer caching to conserve API quotas (esp. FMP FREE tier 250 req/day):
  - `OhlcvCache` — file-based, 5-year rolling window, same-day filename key
  - `FundamentalsCache` — file-based, per-ticker directory, TTL 30d (TTM) / 90d (statements)
  - `NewsCache` — in-memory session-scoped `ConcurrentHashMap`, fresh per `TradingAgentBuilder.bootstrap()`
  - `CacheMode` (`PERMISSIVE`/`BYPASS`/`REFRESH`) controlled via `-De2e.cache` system property
- **Tool sets:** `MarketDataTools`, `FundamentalsTools`, `NewsTools` with `RoleAwareToolFilter`
- **Memory:** `TradingMemoryStore`, `FinancialSituationMemory`, BM25 full-text search
- **Reflection:** `Reflector` for post-trade self-improvement

### 5.7 Claw Agent

`ganglia-claw` provides an IM-integrated autonomous agent daemon:
- **Channel layer:** `Channel` interface + `ChannelFactory` registry; `LarkChannel` (Feishu/Lark) with WebSocket and Webhook modes
- **ACP protocol:** Session-based agent control with `AcpServer`, handlers, and pluggable `AcpBackend` implementations (gemini-cli, claude-code, qoder) via `AcpBackendFactory` registry
- **Orchestration:** `ClawOrchestrator` dispatches to `AgentVerticle` subclasses via Vert.x EventBus; `TaskTracker` for thread-based routing; `ObservationBridge`/`ObservationForwarder` for agent-to-channel event flow
- **Scheduling:** `TaskScheduler` with cron, interval, and one-shot schedules; pluggable `TaskStore` persistence
- **Personality:** Loads SOUL.md, IDENTITY.md, USER.md, AGENTS.md from workspace with modification-aware caching
- **Daemon:** `ClawDaemon` + `ComponentSupervisor` with exponential backoff restart; `ClawDaemonConfig` groups all dependencies
- **Skill self-learning:** `SkillDistiller` monitors turns via `onTurnComplete` hook; when tool call count exceeds threshold, uses LLM to extract reusable SKILL.md files. Configured via `SkillLearningConfig` (enabled, minToolCalls, skillsDir).
- **IM memory isolation:** Per-user memory via userId propagation through `AgentRequest` → `SessionContext` (`im.userId` metadata) → `LongTermMemory` (topic `user-profile:{userId}`). `ImUserContextSource` injects user identity into prompt context; `UserProfileModule` reads user-scoped profiles. `KnowledgeBaseTools.remember(fact, target="user")` auto-scopes writes to the current IM user.
- **Progressive plan enablement:** Three-level config controls plan tool availability:
  - Level 1 (global): `PlanConfig(enabled)` in `ClawConfig` (default: false for IM agents)
  - Level 2 (per-agent): `AgentVerticle.enrichContext()` injects `plan_tools_enabled` metadata; subclasses override `isPlanEnabled()`
  - Level 3 (scheduled tasks): `ClawMain` forces `plan_tools_enabled=false` for autonomous tasks (no user to approve)

`ganglia-claw-app` wires together claw + trading + coding agents as Vert.x verticles (`TradingAgentVerticle`, `CodingAgentVerticle`).

### 5.9 Tool Concurrent Execution

- `ToolDefinition` extended with `readOnly` flag marking tools that don't mutate state
- `DefaultToolExecutor.executeBatch()` partitions calls: readOnly tools run concurrently via `CompositeFuture.all()`, mutating tools run sequentially after
- **Plan mode filtering**: When `plan_mode=true` in session metadata, `DefaultToolExecutor.getAvailableTools()` filters out non-readOnly tools
- Built-in readOnly tools: NativeFileSystemTools (list_directory, read_file, read_files), WebFetchTools, RecallMemoryTools, SessionSearchTools, all todo_* tools, all plan mode tools

### 5.10 File Cleanup

**Automatic cleanup** of `.ganglia/` ephemeral directories:
- `FileCleanupService` — core cleanup logic: age-based file/directory deletion with pattern matching
- `ScheduledFileCleanup` — background Vertx timer (1-hour interval), configurable via `CleanupConfig`
- `CleanupConfig(enabled, stateRetentionDays=7, logRetentionDays=30, traceRetentionDays=7, tmpRetentionDays=1)`
- `FileStateEngine.deleteSession()` — deletes session state JSON + tmp directory on session deletion
- E2E tests call `FileCleanupService.cleanAll()` in `@AfterEach` tearDown

### 5.11 Skill Repository

- `SkillRepository` extends `SkillLoader` with `save(SkillManifest)` and `delete(skillId)` write operations
- `FileSystemSkillLoader` implements `SkillRepository` — writes SKILL.md with YAML frontmatter to skill directories
- `SkillService.saveSkill()` / `deleteSkill()` delegate to the first available `SkillRepository` among loaders

### 5.12 Observation System

All system activities flow through `ObservationDispatcher` → Vert.x EventBus. Tools and gateways MUST NOT use `vertx.eventBus()` directly.

## 6. Development Guidelines

### Code Conventions

- **Async:** All async operations use `Vert.x Future` — never block
- **Logger field:** Always named `logger` (not `log`)
- **Java 25 features:** Text blocks for JSON schemas, `switch` expressions, records, pattern matching
- **Immutable domain:** Use Java records for domain models (Message, Turn, SessionContext)
- **Defensive Copying:** Core domain objects MUST use `List.copyOf()` or `Map.copyOf()` in constructors to ensure immutability
- **Sequential task execution:** Within the loop via `AgentTask` interface

### Architecture Rules

- **Unified Observation Stream:** All observations go through `ObservationDispatcher` or `ExecutionContext`. Every event MUST include `spanId` and `parentSpanId` for tree visualization.
- **Network Resilience:** LLM requests enforce timeout (default 60s), retries via `RetryingModelGateway` with child span tracking
- **Memory Isolation:** Memory implementations live in `ganglia-local-file-memory`, not in `ganglia-harness`
- **SPI for Extensibility:** Pluggable subsystems use `ServiceLoader` (e.g., `MemorySystemProvider`)

### Code Quality

- **Formatting:** Spotless with Google Java Format (runs at compile phase)
- **Static Analysis:** Checkstyle with google_checks.xml + suppressions in `config/checkstyle-suppressions.xml`
- **Coverage:** JaCoCo reports via `just coverage`
- Run `mvn spotless:apply` to auto-fix formatting before committing

## 7. Testing

- **Unit tests:** JUnit 5 + Mockito + Vertx-JUnit5
- **In-memory FS:** Google Jimfs for filesystem tests
- **Integration tests** (`integration-test/`): `MockModelIT` base class with mock ModelGateway + `@TempDir` isolation; `SqliteModelIT` adds SQLite backend
- **E2E tests** (`e2e-test/`): Real LLM calls via CodexModelGateway; organized by domain (trading/, coding/, channel/, gateway/, comprehensive/)
- **E2E cleanup:** `E2ETestBase.tearDown()` calls `FileCleanupService.cleanAll()` to remove `.ganglia/state/`, `.ganglia/tmp/`, `.ganglia/logs/`, `.ganglia/trace/`
- **Three-tier memory model:** See `docs/MEMORY_ARCHITECTURE.md`

## 8. Key Documentation

All design docs are in `docs/`:

|           Document           |                         Topic                         |
|------------------------------|-------------------------------------------------------|
| ARCHITECTURE.md              | Hexagonal layers, observation stream, memory system   |
| CORE_KERNEL_DESIGN.md        | ReAct loop, Schedulable tasks, kernel/port separation |
| MODULES.md                   | Module decomposition (all 17 modules)                 |
| MEMORY_ARCHITECTURE.md       | Three-tier memory model                               |
| MEMORY_REFACTORING_DESIGN.md | Memory module extraction to SPI                       |
| PROMPT_ENGINE_DECOUPLING.md  | Context composition                                   |
| CONTEXT_ENGINE_DESIGN.md     | Multi-layer context stacking                          |
| SUB_AGENT_DESIGN.md          | Sub-agent delegation                                  |
| SUB_AGENT_GRAPH_DESIGN.md    | DAG-based task graph execution                        |
| SKILLS_DESIGN.md             | Dynamic skill system                                  |
| ROBUSTNESS_DESIGN.md         | Failure handling and resilience                       |
| SESSION_MANAGEMENT_DESIGN.md | Session lifecycle                                     |
| STATE_EVOLUTION_DESIGN.md    | State transitions                                     |
| DAILY_RECORD_DESIGN.md       | Daily journal system                                  |
| EXECUTION_AND_STRUCTURE.md   | Core execution flow and structural relationships      |
| REQUIREMENTS.md              | Functional and non-functional requirements            |
| CORE_GUIDELINES_DESIGN.md    | Core guidelines and GANGLIA.md format                 |
| INTEGRATION_SCENARIOS.md     | E2E test scenario definitions                         |
| PROMPT_ENGINE_REFACTORING_DESIGN.md | Prompt engine API and refactoring              |
| CLAW_ARCHITECTURE.md           | Claw Agent architecture, channels, ACP, skill learning |
| PLAN_MODE_DESIGN.md           | Plan mode, todo tools, and plan persistence             |
| OBSERVATION_DESIGN.md         | Observation system, trajectory, and Trace Studio        |
| FILE_CLEANUP_DESIGN.md        | File cleanup service and retention policies             |

---
> Source: [stream-iori/ganglia](https://github.com/stream-iori/ganglia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
