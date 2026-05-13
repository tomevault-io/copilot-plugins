## shannon

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Shannon** is an open-source, enterprise-grade multi-agent AI platform that combines Rust (agent core), Go (orchestration with Temporal), and Python (LLM services) to create a token-efficient, distributed AI system.

## Essential Commands

```bash
# Setup & Run
make setup && vim .env && make dev         # Setup, configure API keys, start all
make smoke                                # E2E smoke tests
make proto                                # Regenerate proto files after .proto changes
make ci                                   # Run CI checks

# Local Docker Builds
# Pattern: docker build --no-cache -f <Dockerfile> -t <image> .
# Then: docker compose -f deploy/compose/docker-compose.yml up -d <service> --no-build
# Dockerfiles: go/orchestrator/Dockerfile, go/orchestrator/cmd/gateway/Dockerfile,
#              python/llm-service/Dockerfile, rust/agent-core/Dockerfile

# Unit Tests
cd rust/agent-core && cargo test
cd go/orchestrator && go test -race ./...
cd python/llm-service && python3 -m pytest
```

## Project Structure

- **`rust/agent-core/`**: Enforcement gateway, WASI sandbox, gRPC server
- **`go/orchestrator/`**: Temporal workflows, budget manager, complexity analyzer
- **`python/llm-service/`**: LLM providers, MCP tools, agent loop
- **`protos/`**: Shared protocol buffer definitions
- **`deploy/compose/`**: Docker Compose configuration
- **`config/`**: Hot-reload configuration files
- **`docs/`**: Architecture and API documentation
- **`scripts/`**: Automation and helper scripts

## API Endpoints (4 Entry Points)

| Endpoint | Orchestrator? | Format | Use Case |
|----------|:---:|--------|----------|
| `POST /v1/chat/completions` | Yes | OpenAI-compatible | Apps using OpenAI SDK -- auto tool selection, deep research, swarm, strategies |
| `POST /v1/completions` | No (Proxy) | OpenAI-compatible | Thin LLM proxy -- single call, no orchestration, caller-supplied tools only |
| `POST /api/v1/tasks` | Yes | Shannon native (sync) | Full orchestrator pipeline, sync response |
| `POST /api/v1/tasks/stream` | Yes | Shannon native (SSE) | Full orchestrator pipeline, streaming SSE events |

`/v1/completions` does NOT go through the orchestrator -- no tool selection, no decomposition, no strategies. Apps that need orchestrated workflows must use `/v1/chat/completions` or `/api/v1/tasks`.

### Tool Execution API

- `GET /api/v1/tools` -- List available tool schemas (excludes dangerous)
- `GET /api/v1/tools/{name}` -- Get tool metadata + parameter schema
- `POST /api/v1/tools/{name}/execute` -- Execute a tool directly

Dangerous tools (`bash_executor`, `file_write`) are blocked at gateway -- they only run inside orchestrated workflows.

## Skip Auth Configuration

For local development and testing, disable authentication in two places:

```bash
# 1. .env
GATEWAY_SKIP_AUTH=1  # 0=prod, 1=dev/test

# 2. config/shannon.yaml
auth:
  skip_auth: true    # false=prod, true=dev/test
```

Recreate (not restart) both services after changes — `restart` does NOT re-read `env_file`:
```bash
docker compose -f deploy/compose/docker-compose.yml down && docker compose -f deploy/compose/docker-compose.yml up -d
```

Gateway handles HTTP auth (from .env), Orchestrator handles gRPC auth (from shannon.yaml). Both must match.

Quick test:
```bash
curl -sS -X POST http://localhost:8080/api/v1/tasks \
  -H "Content-Type: application/json" \
  -d '{"query":"test","session_id":"test-123"}'
```

## Workflow Types & Strategies

### Core Workflows

| Workflow | File | Trigger |
|----------|------|---------|
| **SimpleTaskWorkflow** | `simple_workflow.go` | Complexity < 0.3, single subtask |
| **SupervisorWorkflow** | `supervisor_workflow.go` | Disabled (`case false:` in router) — all multi-task routes to DAG |
| **StreamingWorkflow** | `streaming_workflow.go` | Streaming mode |
| **TemplateWorkflow** | `template_workflow.go` | Template execution |
| **SwarmWorkflow** | `swarm_workflow.go` | `force_swarm: true` context |

### Strategy Workflows (in `strategies/`)

| Strategy | File | Pattern |
|----------|------|---------|
| **DAGWorkflow** | `dag.go` | Default for all multi-task flows (fan-out/fan-in) |
| **ReactWorkflow** | `react.go` | Reasoning loop |
| **ResearchWorkflow** | `research.go` | React + parallel |
| **ExploratoryWorkflow** | `exploratory.go` | Tree-of-Thoughts |
| **ScientificWorkflow** | `scientific.go` | Multi-pattern |
| **BrowserUseWorkflow** | `browser_use.go` | `context.role == "browser_use"` or auto-detected |
| **DomainAnalysisWorkflow** | `domain_analysis_workflow.go` | Domain-specific analysis |

## Critical Implementation Rules

### Temporal Workflows
- Always await activity completion with `.Get(ctx, &result)`
- Use `workflow.Sleep()` in workflows, never `time.Sleep()` in activities
- Maintain workflow determinism for replay testing
- Any new code path MUST be gated with `workflow.GetVersion()`

### Agent Nicknames
Always use `agents.GetAgentName(workflowID, index)` from `agents/names.go`.

### Database
- Tasks table: `task_executions`
- Status values: UPPERCASE for API/DB (`"COMPLETED"` not `"completed"`)
- Empty UUID strings must convert to NULL

### Proto/gRPC
- Rust enums: Use `ExecutionMode::Simple` (not `ExecutionMode::ExecutionModeSimple`)
- Regenerate with `make proto` after any `.proto` changes

### Build Sequence After Proto Changes

`make proto` -> `cd go/orchestrator && go mod tidy` -> `cd rust/agent-core && cargo build`

## Adding/Fixing LLM Providers

Single source of truth: `config/models.yaml`. Touches:
- Go pricing: `pricing/pricing.go`
- Go detection: `models/provider.go`
- Python provider: `llm_provider/{name}_provider.py`
- Python registry: `providers/__init__.py`

## Synthesis Templates

Customize final answer formatting via `config/templates/synthesis/`:

| Method | Context Parameter | Use Case |
|--------|-------------------|----------|
| Named template | `synthesis_template: "my_format"` | Reusable `.tmpl` files |
| Verbatim override | `synthesis_template_override: "..."` | One-time custom format |

## Environment Gotcha

When modifying `.env`, services must be **recreated** (`docker compose down` then `up -d`), not just restarted -- `restart` does NOT re-read `env_file`.

## Documentation

- [Multi-Agent Workflow Architecture](docs/multi-agent-workflow-architecture.md)
- [Pattern Usage Guide](docs/pattern-usage-guide.md)
- [Streaming APIs](docs/streaming-api.md)
- [Token & Budget Tracking](docs/token-budget-tracking.md)
- [Extending Shannon](docs/extending-shannon.md)
- [Skills System](docs/skills-system.md)
- [Session Workspaces](docs/session-workspaces.md)
- [Swarm Agents](docs/swarm-agents.md)

---
> Source: [Kocoro-lab/Shannon](https://github.com/Kocoro-lab/Shannon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
