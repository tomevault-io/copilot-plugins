## bridge

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Bridge is a governance and orchestration layer for AI-driven software development. It coordinates AI agents, enforces policies, and provides audit capabilities across the SDLC.

## Technology Stack

- **Language**: Go 1.23+
- **Architecture**: DDD + event-driven + hexagonal
- **State Management**: felixgeelhaar/statekit
- **Logging**: felixgeelhaar/bolt (zero-allocation, structured)
- **Resilience**: felixgeelhaar/fortify (circuit breaker, retry, rate limiting)
- **MCP**: felixgeelhaar/mcp-go (agent tool bindings)
- **Policy Engine**: OPA (Open Policy Agent) with Rego
- **LLM Providers**: Anthropic, OpenAI, Gemini, Ollama
- **Observability**: OpenTelemetry

## Core Libraries

### felixgeelhaar/bolt - Logging
```go
logger := bolt.New(bolt.NewJSONHandler(os.Stdout))
logger.Info().Str("run_id", id).Msg("Workflow started")
```

### felixgeelhaar/statekit - State Machines
```go
machine, _ := statekit.NewMachine[Context]("workflow").
    WithInitial("pending").
    State("pending").On("START").Target("running").Done().
    Build()
```

### felixgeelhaar/fortify - Resilience
```go
cb := circuitbreaker.New[Response](config)
result, err := cb.Execute(ctx, func(ctx context.Context) (Response, error) {
    return llm.Complete(ctx, request)
})
```

### felixgeelhaar/mcp-go - MCP Protocol
```go
server := mcp.NewServer("bridge-tools", "1.0.0")
server.Tool("file_read").Handler(fileReadHandler)
```

## Architecture

### Bounded Contexts (DDD)

1. **Workflow** - WorkflowDefinition, WorkflowRun, Step, state transitions via statekit
2. **Governance** - PolicyBundle, Approval, AuditEvent
3. **Agents** - Agent, ToolBinding (MCP), Capability model
4. **Analytics** - TelemetryEvent, metrics aggregation

### Execution Flow

```
CLI / CI → Orchestrator → Policy Engine → Workflow Engine → Agent Runners → Audit / Telemetry
```

- Orchestrator schedules steps
- Steps are isolated and idempotent
- State machine controls transitions
- Policies evaluated pre/post execution

## Development Commands

```bash
# Build the CLI
make build

# Run all tests
make test

# Run tests with coverage
make test-coverage

# Run single test by name
go test -v -run TestWorkflowRun ./internal/domain/workflow

# Lint code
make lint

# Format code
make fmt

# Run all checks (lint, fmt, vet, test)
make all

# Generate sqlc code
make sqlc

# Cross-compile for all platforms
make build-all

# Policy validation
make policy-test

# Clean build artifacts
make clean

# Install development tools
make tools
```

### Docker Commands

```bash
# Start development environment (PostgreSQL + RabbitMQ)
docker-compose up -d

# Start demo environment with seeded data
docker-compose -f docker-compose.demo.yml up -d

# Stop all containers
docker-compose down

# View logs
docker-compose logs -f
```

### Release Commands (via relicta)

```bash
# Plan next release
relicta plan --analyze

# Bump version
relicta bump --level auto

# Generate release notes
relicta notes --ai --audience developers

# Validate release requirements
relicta validate

# Publish release
relicta publish
```

## CLI Usage

```bash
# Initialize a new Bridge project
bridge init

# Validate a workflow definition
bridge validate --workflow workflow.yaml

# Run a workflow
bridge run --workflow workflow.yaml

# Run with custom trigger data
bridge run --workflow workflow.yaml --trigger-data '{"key":"value"}'

# Dry run (validate without executing)
bridge run --workflow workflow.yaml --dry-run

# Check workflow run status
bridge status <run-id>

# Approve a pending workflow
bridge approve <run-id>

# Override log level
bridge run --workflow workflow.yaml --log-level debug
```

## Environment Variables

```bash
# LLM Provider API Keys
ANTHROPIC_API_KEY=...     # Required for Anthropic Claude models
OPENAI_API_KEY=...        # Required for OpenAI GPT models
GEMINI_API_KEY=...        # Required for Google Gemini models
OLLAMA_HOST=...           # Optional: Ollama server URL (default: http://localhost:11434)

# GitHub Integration
GITHUB_TOKEN=...          # Required for GitHub PR operations
GITHUB_WEBHOOK_SECRET=... # Optional: Webhook signature verification

# Database (PostgreSQL)
DATABASE_URL=postgres://bridge:bridge@localhost:5432/bridge?sslmode=disable

# Message Queue (RabbitMQ)
RABBITMQ_URL=amqp://bridge:bridge@localhost:5672/
```

## Project Structure

```
bridge/
├── cmd/bridge/                           # CLI entry point
├── internal/
│   ├── domain/                           # DDD bounded contexts
│   │   ├── workflow/                     # WorkflowDefinition, WorkflowRun, Step
│   │   ├── governance/                   # PolicyBundle, Approval, AuditEvent
│   │   ├── agents/                       # Agent, Runner, Registry
│   │   └── analytics/                    # TelemetryEvent, metrics
│   ├── application/
│   │   └── orchestrator/                 # Workflow orchestration service
│   ├── infrastructure/
│   │   ├── persistence/
│   │   │   ├── memory/                   # In-memory repositories (MVP)
│   │   │   └── postgres/                 # PostgreSQL repositories
│   │   │       └── sqlc/                 # Generated type-safe SQL code
│   │   ├── messaging/
│   │   │   ├── eventbus/                 # In-process event bus (MVP)
│   │   │   └── rabbitmq/                 # RabbitMQ connection, pub/sub
│   │   ├── policy/                       # OPA/Rego integration
│   │   ├── mcp/                          # MCP server + tools
│   │   │   └── tools/                    # File, Git, Shell, GitHub tools
│   │   ├── llm/                          # LLM provider adapters
│   │   ├── github/                       # GitHub API + webhooks
│   │   └── observability/                # Logging setup
│   └── interfaces/cli/                   # CLI commands + output
│       └── commands/                     # Individual CLI commands
├── pkg/
│   ├── config/                           # YAML workflow/policy schemas
│   └── types/                            # Shared types, IDs, errors
├── policies/                             # Default OPA policies
├── demo/seeds/                           # Demo database seeds
├── examples/pr-review/                   # Example PR review workflow
├── .github/
│   └── workflows/                        # CI/CD pipelines
│       ├── ci.yml                        # Lint, test, security, build
│       ├── release.yml                   # Release with governance
│       └── release-prepare.yml           # Prepare release PR
├── Dockerfile                            # Multi-stage production image
├── docker-compose.yml                    # Development environment
├── docker-compose.demo.yml               # Demo with seeded data
├── sqlc.yaml                             # sqlc configuration
├── .relicta.yaml                         # Release governance config
└── install.sh                            # curl installer script
```

## Key Design Decisions

- **Zero-trust security model** - no secrets stored by default, BYO LLM keys
- **Local-first execution** - default deployment is local + CI, hosted control plane is optional
- **Policy-as-code** - all governance via OPA/Rego
- **Signed artifacts** - workflows and policies are signed for integrity
- **Immutable audit logs** - all actions are recorded for compliance

## LLM Provider Configuration

Each LLM provider is wrapped with resilience patterns (circuit breaker, retry, rate limiting):

```go
// Providers support: anthropic, openai, gemini, ollama
registry := llm.NewRegistry()
provider := llm.NewAnthropicProvider(llm.AnthropicConfig{
    ProviderConfig: llm.ProviderConfig{
        APIKey: os.Getenv("ANTHROPIC_API_KEY"),
        Model:  "claude-sonnet-4-20250514",
    },
})
resilient := llm.NewResilientProvider(provider, llm.DefaultResilientConfig())
rateLimited := llm.NewRateLimitedProvider(resilient, logger, llm.DefaultRateLimitConfig())
registry.Register(rateLimited)
```

## GitHub Webhook Integration

Webhooks are verified using HMAC-SHA256 signatures:

```go
handler := github.NewWebhookHandler(logger, os.Getenv("GITHUB_WEBHOOK_SECRET"))
handler.On(github.EventPullRequest, func(ctx context.Context, event github.WebhookEvent, payload any) error {
    data, _ := github.ExtractTriggerData(event, payload)
    // Trigger workflow with extracted data
    return nil
})
```

---
> Source: [felixgeelhaar/bridge](https://github.com/felixgeelhaar/bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
