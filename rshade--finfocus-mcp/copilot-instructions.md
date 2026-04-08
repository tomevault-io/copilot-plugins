## finfocus-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PulumiCost MCP Server is a Model Context Protocol implementation that exposes cloud cost analysis capabilities to AI assistants. Built with Goa and Goa-AI, it provides type-safe, design-first development with guaranteed schema consistency.

**Current Status**: Core implementation complete (Phases 1-5). Services implemented with 83.6% test coverage. Phase 6 (observability) in progress.

## CRITICAL: Task Completion Tracking

**MANDATORY**: After completing ANY task from `specs/001-mcp-server/tasks.md`, you MUST immediately mark it as complete by changing `- [ ]` to `- [x]` in the tasks.md file.

**Why this matters**:
- Prevents duplicate work
- Maintains accurate project status
- Enables proper progress tracking
- Supports incremental delivery validation

**How to mark tasks complete**:
1. After successfully completing a task (e.g., T050)
2. Immediately edit `specs/001-mcp-server/tasks.md`
3. Change `- [ ] T050` to `- [x] T050`
4. Commit the tasks.md update with your implementation

**Never skip this step** - it's as important as the implementation itself.

## Quick Reference

### Essential Commands

```bash
# Setup new development environment (first time)
make setup

# Generate Goa code from design (required after design changes)
make generate

# Build server
make build

# Run all tests
make test

# Run single test
go test -v -run TestName ./internal/service

# Run tests with coverage
make test-coverage

# Run linters
make lint

# Run complete validation (lint + test)
make validate

# Run server locally
make run

# Clean generated files and build artifacts
make clean
```

### Key Directories

- `design/` - Goa DSL definitions (source of truth for all APIs)
- `cmd/pulumicost-mcp/` - Server entry point (not yet implemented)
- `internal/service/` - Business logic implementation (not yet implemented)
- `internal/adapter/` - External system adapters (not yet implemented)
- `gen/` - Generated code (never edit directly! Regenerated from design/)
- `examples/pulumi-stacks/` - Example Pulumi projects for testing
- `examples/queries/` - Example MCP queries
- `role-prompts/` - AI assistant role-specific contexts
- `.github/workflows/` - CI/CD pipeline configuration

## CI/CD Pipeline

### Overview

The project uses GitHub Actions for continuous integration and deployment. All workflows are defined in `.github/workflows/`.

### Workflows

#### Main CI Workflow (`.github/workflows/ci.yml`)

Runs on every push and pull request to `main` and `develop` branches.

**Jobs:**

1. **generate-check** - Verifies code generation is up to date
   - Runs `make generate`
   - Fails if generated code differs from committed code
   - Ensures design and generated code stay in sync

2. **test** - Runs test suite with coverage
   - Executes `make test-coverage`
   - Uploads coverage to Codecov
   - Requires 80%+ coverage (target)

3. **lint** - Runs code quality checks
   - Uses golangci-lint v2.6.0
   - Configuration in `.golangci.yml`
   - 5-minute timeout
   - Checks 40+ linters

4. **build** - Builds binaries for all platforms
   - Matrix build: linux, darwin, windows
   - Architectures: amd64, arm64
   - Uploads artifacts for 7 days
   - Verifies cross-compilation works

#### Release Workflow (`.github/workflows/release.yml`)

Triggers on version tags (e.g., `v1.0.0`).

**Steps:**
1. Runs full test suite and linters
2. Builds release artifacts for all platforms
3. Generates SHA256 checksums
4. Creates GitHub release with artifacts
5. Builds and pushes Docker image to ghcr.io
6. Tags Docker image with version and semver patterns

**Docker Tags Generated:**
- `vX.Y.Z` (exact version)
- `X.Y` (major.minor)
- `X` (major)
- `sha-<commit>` (git sha)

#### CodeQL Security Analysis (`.github/workflows/codeql.yml`)

Runs on:
- Push to main/develop
- Pull requests
- Weekly schedule (Mondays at 6:00 AM UTC)
- Manual dispatch

**Features:**
- Scans Go code for security vulnerabilities
- Uses security-and-quality query suite
- Results visible in Security tab
- Integrates with GitHub Security Advisories

#### Dependabot (`.github/dependabot.yml`)

Automated dependency updates for:
- **Go modules** - Weekly updates on Mondays
- **GitHub Actions** - Weekly version bumps
- **Docker** - Weekly base image updates

**Configuration:**
- Groups related dependencies (Goa, Pulumi, AWS, Azure, etc.)
- Limits to 5 open PRs per ecosystem
- Auto-assigns to @rshade
- Labels PRs with `dependencies` and ecosystem type

### Linter Configuration

File: `.golangci.yml`

**Enabled Linters (40+):**
- **Quality**: errcheck, gosimple, govet, staticcheck, unused
- **Style**: gofmt, gofumpt, goimports, revive, stylecheck
- **Security**: gosec, errorlint
- **Performance**: gocritic
- **Complexity**: gocyclo, gocognit
- **Bug Detection**: bodyclose, rowserrcheck, sqlclosecheck
- **Best Practices**: misspell, whitespace, predeclared

**Special Rules:**
- Skips `gen/` directory (generated code)
- Allows dot imports in `design/` (Goa DSL)
- Relaxed rules for `*_test.go` files
- 5-minute timeout
- All linters enabled except overly strict ones

### Pull Request Template

File: `.github/pull_request_template.md`

**Sections:**
- Description and type of change
- Related issues (auto-closes with "Closes #123")
- Changes made
- Testing evidence
- Code generation checklist
- Documentation updates
- Breaking changes

**Required Checks:**
- [ ] Tests pass (`make test`)
- [ ] Linters pass (`make lint`)
- [ ] Code generation up to date (`make generate`)
- [ ] Documentation updated

### CI/CD Best Practices

1. **Design-First Validation**
   - Always run `make generate` after design changes
   - CI enforces design/generated code sync
   - Commit both design and generated code together

2. **Test Coverage**
   - Target 80%+ coverage
   - Coverage tracked in Codecov
   - Unit tests run on every PR
   - Integration tests run on main branch

3. **Security**
   - CodeQL scans weekly and on PRs
   - Dependabot keeps dependencies current
   - gosec linter catches security issues
   - No secrets in code (enforced by .gitignore)

4. **Release Process**
   ```bash
   # 1. Update version in relevant files
   # 2. Commit changes
   git add .
   git commit -m "chore: bump version to v1.0.0"

   # 3. Create and push tag
   git tag -a v1.0.0 -m "Release v1.0.0"
   git push origin v1.0.0

   # 4. GitHub Actions automatically:
   #    - Runs tests and linters
   #    - Builds multi-platform binaries
   #    - Creates GitHub release
   #    - Builds and pushes Docker image
   ```

5. **Debugging CI Failures**
   - **Generate check fails**: Run `make generate` locally and commit
   - **Tests fail**: Check logs in Actions tab, run `make test` locally
   - **Linter fails**: Run `make lint` locally, fix issues
   - **Build fails**: Verify `go.mod` dependencies, run `make build`

### Status Badges

Add to README.md:

```markdown
[![CI](https://github.com/rshade/pulumicost-mcp/actions/workflows/ci.yml/badge.svg)](https://github.com/rshade/pulumicost-mcp/actions/workflows/ci.yml)
[![CodeQL](https://github.com/rshade/pulumicost-mcp/actions/workflows/codeql.yml/badge.svg)](https://github.com/rshade/pulumicost-mcp/actions/workflows/codeql.yml)
[![codecov](https://codecov.io/gh/rshade/pulumicost-mcp/branch/main/graph/badge.svg)](https://codecov.io/gh/rshade/pulumicost-mcp)
[![Go Report Card](https://goreportcard.com/badge/github.com/rshade/pulumicost-mcp)](https://goreportcard.com/report/github.com/rshade/pulumicost-mcp)
```

### Environment Secrets

Configure in GitHub Settings → Secrets:

- `CODECOV_TOKEN` - For coverage uploads (optional but recommended)
- `GITHUB_TOKEN` - Auto-provided for releases and Docker pushes

### Local CI Testing

While you can't run GitHub Actions locally, you can simulate CI checks:

```bash
# Run the full CI pipeline locally
make generate  # Verify code generation
make lint      # Run all linters
make test      # Run tests
make build     # Build binary

# Or use the combined target
make validate  # Runs lint + test
```

## Architecture Principles

### 1. Design-First Development

**CRITICAL**: All APIs are defined in Goa DSL before implementation.

- API contracts live in `design/*.go`
- Running `make generate` produces all transport, validation, and schema code
- Generated code in `gen/` must NEVER be modified directly
- Changes to APIs require updating the design and regenerating

Example workflow:
```
1. Edit design/cost_service.go
2. Run make generate
3. Implement service logic in internal/service/
4. Write tests
5. Commit both design and implementation
```

### 2. Separation of Concerns

```
┌─────────────────────┐
│  Design Layer       │  ← API contracts, types, validation rules
│  (design/*.go)      │
└──────────┬──────────┘
           │ goa gen
┌──────────▼──────────┐
│  Generated Layer    │  ← Transport, encoding, validation
│  (gen/*)            │  ← JSON-RPC, HTTP, MCP protocol
└──────────┬──────────┘
           │ calls
┌──────────▼──────────┐
│  Service Layer      │  ← Business logic
│  (internal/service) │
└──────────┬──────────┘
           │ uses
┌──────────▼──────────┐
│  Adapter Layer      │  ← External integrations
│  (internal/adapter) │  ← pulumicost-core, plugins, spec
└─────────────────────┘
```

### 3. Type Safety

- Use explicit types, never `map[string]interface{}`
- Goa DSL defines all types
- Compiler catches integration errors
- No stringly-typed interfaces

### 4. Error Handling

- Always use context.Context for cancellation
- Return errors, don't panic
- Use Goa's error types for proper HTTP/RPC mapping
- Include actionable error messages

### 5. Testing Strategy

- Unit tests for service layer
- Integration tests for adapters
- End-to-end tests for MCP protocol
- Conformance tests for plugins
- Aim for >85% coverage

## Development Workflow

### Adding a New MCP Tool

1. **Define in Design**
   ```go
   // design/cost_service.go
   Method("new_tool", func() {
       Description("Clear description")
       Payload(RequestType)
       Result(ResponseType)
       Error("error_name", ErrorResult)
       mcp.Tool("tool_name", "AI-focused description")
       JSONRPC(func() {})
   })
   ```

2. **Generate Code**
   ```bash
   make generate
   ```

3. **Implement Service**
   ```go
   // internal/service/cost_service.go
   func (s *costService) NewTool(ctx context.Context,
       req *cost.RequestType) (*cost.ResponseType, error) {
       // Implementation
   }
   ```

4. **Write Tests**
   ```go
   // internal/service/cost_service_test.go
   func TestNewTool(t *testing.T) {
       // Test implementation
   }
   ```

5. **Document**
   - Update README with tool description
   - Add examples to examples/queries/
   - Update role prompts if needed

### Modifying Existing APIs

**DO NOT** modify generated code. Always:

1. Modify the design in `design/*.go`
2. Run `make generate`
3. Update implementation in `internal/`
4. Update tests
5. Verify backward compatibility

### Adding Dependencies

Before adding a new dependency:

1. Check if it's truly needed
2. Verify Go 1.24 compatibility
3. Check license compatibility
4. Consider binary size impact
5. Add to `go.mod` with `go get`
6. Document usage in relevant files

## Common Tasks

### Task: Add Support for New Cloud Provider

1. **Update Design Types**
   ```go
   // design/types.go
   var Provider = Type("Provider", func() {
       Enum("aws", "azure", "gcp", "kubernetes", "new-provider")
   })
   ```

2. **Implement Provider Logic**
   ```go
   // internal/service/cost_service.go
   switch resource.Provider {
   case "new-provider":
       return s.handleNewProvider(ctx, resource)
   }
   ```

3. **Add Tests**
4. **Update Documentation**
5. **Add Examples**

### Task: Implement Caching

1. **Add Cache Configuration**
   ```go
   // internal/config/config.go
   type CacheConfig struct {
       Enabled bool
       TTL     time.Duration
   }
   ```

2. **Implement Cache Layer**
   ```go
   // internal/cache/cache.go
   type Cache interface {
       Get(key string) (interface{}, bool)
       Set(key string, value interface{}, ttl time.Duration)
   }
   ```

3. **Integrate with Service**
   ```go
   // Check cache before expensive operation
   if cached, ok := s.cache.Get(key); ok {
       return cached.(*cost.Response), nil
   }
   ```

### Task: Add Streaming Support

1. **Design with StreamingResult**
   ```go
   Method("analyze_large", func() {
       Payload(Request)
       StreamingResult(ProgressUpdate)
       mcp.Tool("analyze_large_stack", "...")
       JSONRPC(func() {})
   })
   ```

2. **Implement Streaming Handler**
   ```go
   func (s *service) AnalyzeLarge(ctx context.Context,
       req *Request,
       stream AnalyzeLargeServerStream) error {

       for item := range items {
           select {
           case <-ctx.Done():
               return ctx.Err()
           default:
           }

           if err := stream.Send(item); err != nil {
               return err
           }
       }
       return nil
   }
   ```

## Integration Points

### PulumiCost Core

Located in ../pulumicost-core

**Integration**:
- Execute as external binary
- Pass Pulumi JSON via stdin or file
- Parse structured output
- Handle errors and timeouts

**Adapter Location**: `internal/adapter/pulumicost_adapter.go`

### PulumiCost Spec

Located in ../pulumicost-spec

**Integration**:
- Import SDK: `github.com/rshade/pulumicost-spec/sdk/go`
- Use protobuf definitions
- Implement gRPC clients for plugins
- Run conformance tests

**Adapter Location**: `internal/adapter/spec_adapter.go`

### Plugins

Located in ~/.pulumicost/plugins/ or configured directory

**Integration**:
- Discover plugins via filesystem
- Start gRPC connections
- Health check before use
- Circuit breaker for failures
- Timeout management

**Adapter Location**: `internal/adapter/plugin_adapter.go`

### MCP Protocol

**Integration**:
- Goa-AI generates MCP-compliant JSON-RPC
- Tools automatically registered
- SSE for streaming
- Standard error codes

**Generated**: `gen/mcp/`

## Code Patterns

### Context Propagation

Always pass context through call stack:

```go
func (s *service) Method(ctx context.Context, req *Request) (*Response, error) {
    result, err := s.adapter.DoSomething(ctx, req.Param)
    if err != nil {
        return nil, err
    }
    return result, nil
}
```

### Error Handling

Use Goa's error types:

```go
import goa "goa.design/goa/v3/pkg"

// Validation error
if req.ID == "" {
    return nil, goa.MergeErrors(
        goa.MissingFieldError("id", "request"),
    )
}

// Not found
if resource == nil {
    return nil, cost.MakeNotFound(fmt.Errorf("resource %s not found", id))
}

// Internal error
if err != nil {
    return nil, cost.MakeInternalError(err)
}
```

### Adapter Pattern

Isolate external dependencies:

```go
type PulumiCostAdapter interface {
    GetProjectedCost(ctx context.Context, stack string) (*CostResult, error)
    GetActualCost(ctx context.Context, stack string, period TimePeriod) (*CostResult, error)
}

type pulumiCostAdapter struct {
    corePath string
    executor Executor
}

func (a *pulumiCostAdapter) GetProjectedCost(ctx context.Context,
    stack string) (*CostResult, error) {
    // Implementation
}
```

### Dependency Injection

Use constructor functions:

```go
type costService struct {
    adapter adapter.PulumiCostAdapter
    logger  *log.Logger
    cache   cache.Cache
}

func NewCostService(
    adapter adapter.PulumiCostAdapter,
    logger *log.Logger,
    cache cache.Cache,
) *costService {
    return &costService{
        adapter: adapter,
        logger:  logger,
        cache:   cache,
    }
}
```

## Testing Guidelines

### Unit Tests

Test service logic in isolation:

```go
func TestAnalyzeProjected(t *testing.T) {
    // Setup
    mockAdapter := &mockPulumiCostAdapter{
        result: &CostResult{...},
    }
    service := NewCostService(mockAdapter, logger, cache)

    // Execute
    result, err := service.AnalyzeProjected(context.Background(), req)

    // Assert
    require.NoError(t, err)
    assert.Equal(t, expected, result)
}
```

### Integration Tests

Test with real dependencies:

```go
func TestPluginIntegration(t *testing.T) {
    // Start test plugin
    plugin := startTestPlugin(t)
    defer plugin.Stop()

    // Create adapter
    adapter := NewPluginAdapter(plugin.Address())

    // Test real communication
    result, err := adapter.GetCost(context.Background(), "resource-id")

    require.NoError(t, err)
    assert.NotNil(t, result)
}
```

### Table-Driven Tests

Use for multiple scenarios:

```go
func TestSupports(t *testing.T) {
    tests := []struct {
        name      string
        provider  string
        resourceType string
        supported bool
    }{
        {"AWS EC2", "aws", "ec2/instance", true},
        {"Azure VM", "azure", "compute/vm", false},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // Test implementation
        })
    }
}
```

## Configuration Management

### Environment Variables

Prefer environment variables for secrets:

```go
// config/config.go
type Config struct {
    APIKey string `env:"API_KEY,required"`
    Port   int    `env:"PORT" envDefault:"8080"`
}
```

### Configuration File

Use YAML for structured config:

```yaml
# config.yaml
server:
  port: 8080
  host: localhost

pulumicost:
  core_path: /usr/local/bin/pulumicost
  plugin_dir: ~/.pulumicost/plugins
```

### Loading Configuration

```go
func LoadConfig(path string) (*Config, error) {
    data, err := os.ReadFile(path)
    if err != nil {
        return nil, fmt.Errorf("read config: %w", err)
    }

    var cfg Config
    if err := yaml.Unmarshal(data, &cfg); err != nil {
        return nil, fmt.Errorf("parse config: %w", err)
    }

    // Override with environment variables
    if apiKey := os.Getenv("API_KEY"); apiKey != "" {
        cfg.APIKey = apiKey
    }

    return &cfg, nil
}
```

## Common Pitfalls

### ❌ Modifying Generated Code

```go
// gen/cost/service.go  ← NEVER EDIT THIS
func (s *Service) Method() { ... }
```

**Solution**: Edit design, regenerate

### ❌ Blocking Operations Without Context

```go
// Bad
func (s *service) Method() error {
    time.Sleep(10 * time.Second)  // No cancellation!
}
```

**Solution**: Use context

```go
// Good
func (s *service) Method(ctx context.Context) error {
    select {
    case <-time.After(10 * time.Second):
    case <-ctx.Done():
        return ctx.Err()
    }
}
```

### ❌ Not Handling Errors

```go
// Bad
result, _ := adapter.GetCost(ctx, id)
return result
```

**Solution**: Always handle errors

```go
// Good
result, err := adapter.GetCost(ctx, id)
if err != nil {
    return nil, fmt.Errorf("get cost: %w", err)
}
return result, nil
```

### ❌ Global State

```go
// Bad
var globalCache = make(map[string]interface{})
```

**Solution**: Use dependency injection

```go
// Good
type service struct {
    cache Cache
}
```

## Debugging Tips

### Enable Verbose Logging

```bash
MCP_LOG_LEVEL=debug ./pulumicost-mcp
```

### Test with cURL

```bash
curl -X POST http://localhost:8080/rpc \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "cost.analyze_projected",
    "params": {...},
    "id": 1
  }'
```

### Test with grpcurl

```bash
# List services
grpcurl -plaintext localhost:50051 list

# Call plugin method
grpcurl -plaintext -d '{"resource_id": "i-123"}' \
  localhost:50051 \
  pulumicost.v1.CostSourceService/GetActualCost
```

### Profile Performance

```bash
# CPU profiling
go test -cpuprofile=cpu.prof -bench=.

# Memory profiling
go test -memprofile=mem.prof -bench=.

# Analyze
go tool pprof cpu.prof
```

## GitHub Issues and Milestones

All implementation work is tracked in GitHub issues organized into 5 phases:

### Phase 1: Foundation (Milestone: 2025-Q4 - Phase 1: Foundation)
- Issues #1-6: CI/CD, linting, testing framework, Makefile, Goa design, code generation

### Phase 2: Core Implementation (Milestone: 2025-Q4 - Phase 2: Core Implementation)
- Issues #7-12: Cost/Plugin/Analysis services, PulumiCost/Plugin/Spec adapters

### Phase 3: MCP Integration (Milestone: 2025-Q4 - Phase 3: MCP Integration)
- Issues #13-14: MCP server, Claude Desktop integration

### Phase 4: Testing & Documentation (Milestone: 2025-Q4 - Phase 4: Testing & Documentation)
- Issues #15-16: E2E tests, comprehensive documentation

### Phase 5: Polish & Beta Release (Milestone: 2025-Q4 - Phase 5: Polish & Beta Release)
- Issues #17-19: Performance optimization, observability, release preparation

**Priority Labels**: P0 (critical), P1 (high), P2 (medium)
**Type Labels**: infrastructure, service, adapter, testing, documentation, design

## Resources

### Documentation
- [Goa Documentation](https://goa.design)
- [Goa-AI Guide](https://goa.design/goa-ai)
- [MCP Specification](https://modelcontextprotocol.io)
- [Implementation Plan](IMPLEMENTATION_PLAN.md)
- [Contributing Guide](CONTRIBUTING.md)

### Related Projects
- [pulumicost-core](../pulumicost-core/README.md) - Cost analysis orchestration
- [pulumicost-spec](../pulumicost-spec/README.md) - gRPC plugin specification

### Role Prompts (AI Assistant Context)
- [Senior Architect](role-prompts/senior-architect.md)
- [Product Manager](role-prompts/product-manager.md)
- [DevOps Engineer](role-prompts/devops-engineer.md)
- [Plugin Developer](role-prompts/plugin-developer.md)
- [Cost Analyst](role-prompts/cost-analyst.md)

## Project Conventions

### File Naming
- Design files: `design/*.go`
- Service implementations: `internal/service/*_service.go`
- Tests: `*_test.go` (same package)
- Adapters: `internal/adapter/*_adapter.go`

### Package Names
- Single word, lowercase
- Match directory name
- No underscores or camelCase

### Error Messages
- Lowercase, no trailing punctuation
- Include context: `"failed to load plugin %s: %w"`
- Use `%w` for error wrapping

### Comments
- Exported items must have doc comments
- Start with item name: `// CostService provides...`
- Full sentences with proper punctuation

### Commit Messages
```
type(scope): subject

body

footer
```

Types: feat, fix, docs, style, refactor, test, chore

Examples:
- `feat(cost): add projected cost streaming`
- `fix(plugin): handle connection timeout`
- `docs(readme): update installation instructions`

## Emergency Procedures

### Server Won't Start

1. Check generated code is up to date:
   ```bash
   make clean && make generate
   ```

2. Verify configuration:
   ```bash
   cat config.yaml
   env | grep MCP_
   ```

3. Check for port conflicts:
   ```bash
   lsof -i :8080
   ```

### Tests Failing After Design Change

1. Regenerate code:
   ```bash
   make generate
   ```

2. Update service implementation to match new types

3. Update tests to use new types

4. Run tests:
   ```bash
   make test
   ```

### Plugin Communication Failing

1. Test plugin directly:
   ```bash
   grpcurl -plaintext localhost:50051 list
   ```

2. Check plugin logs

3. Verify plugin path in configuration

4. Test with minimal example

## Testing with Example Stacks

Example Pulumi projects are available in `examples/pulumi-stacks/`:

- `simple-aws/` - Basic AWS stack (EC2, RDS, S3) for testing cost analysis
- Each example includes a `queries.md` with 25+ sample cost analysis queries
- Use these to test MCP tools once implemented

**To use**:
```bash
cd examples/pulumi-stacks/simple-aws
pulumi preview --json > preview.json
# Use preview.json to test get_projected_cost tool
```

---

**Critical Rules**:
1. **Design First**: All API changes start in `design/*.go`, then `make generate`
2. **Never Edit Generated Code**: Files in `gen/` are regenerated and will be overwritten
3. **Always Run Tests**: Aim for >80% coverage, run `make validate` before commits
4. **Context Propagation**: Always pass `context.Context` through the call stack

**Last Updated**: 2025-01-04
**Project Status**: Initial Setup Complete - Ready for Phase 1 Implementation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rshade)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rshade)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
