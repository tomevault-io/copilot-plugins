## submux

> - AGENTS.md - Development workflows, commands, conventions (START HERE)

# Cursor Rules for submux Project

## 📚 Primary Documentation - READ THESE FIRST
- AGENTS.md - Development workflows, commands, conventions (START HERE)
- DESIGN.md - Architecture, design patterns, technical decisions
- README.md - User-facing documentation
- TODO.md - Planned work and roadmap
- CHANGELOG.md - Version history

## 🔐 CRITICAL: Thread Safety - Lock Ordering
When acquiring multiple locks, ALWAYS follow this order (DESIGN.md):
1. SubMux.mu
2. topologyMonitor.mu
3. topologyState.mu
4. pubSubPool.mu
5. pubSubMetadata.mu

Never acquire locks in reverse order - this causes deadlocks!

## 🎯 Code Quality Rules

### Error Handling
- Return errors, don't panic
- Wrap errors with context: fmt.Errorf("operation failed: %w", err)
- Use structured logging with slog for error context

### Logging
- Use slog.Logger with structured fields
- Log levels: Debug (verbose), Info (normal), Warn (issues), Error (failures)
- Never log sensitive data

### Concurrency
- Prefer sync.RWMutex for read-heavy operations
- Use channels for goroutine communication
- Always defer Unlock() immediately after Lock()

### Testing
- Maintain >65% unit test coverage
- Write integration tests for critical paths
- Use table-driven tests
- Use retry utilities for timing-dependent tests (see integration/shared_cluster_test.go)

## 🏗️ Architecture Patterns

### Single Event Loop Pattern
- Each Redis connection managed by ONE goroutine
- All I/O operations handled sequentially
- Use channel-based command queue (cmdCh)
- See eventloop.go for reference

### State Machines
- Subscriptions: Pending → Active → Closed/Failed
- Document state transitions
- Use atomic operations where appropriate

## 🔧 Development Workflow

### Before Making Changes
1. Read AGENTS.md for context
2. Check DESIGN.md for architecture
3. Read relevant code files
4. Use gopls MCP tools for Go code analysis (if available)

### After Making Changes
1. Run: go test ./...
2. Check: go test -race ./...
3. Format: go fmt ./...
4. Update DESIGN.md if architecture changed
5. Update CHANGELOG.md with version number
6. Update README.md if user-facing API changed

## 🧪 Testing Requirements

### Unit Tests
- Use table-driven tests
- Mock external dependencies (use testutil/mockclient.go)
- Keep tests fast (<10ms each)
- Test error paths, not just happy paths

### Integration Tests
- Run in integration/ directory
- Use retry utilities for flaky operations
- Clean up resources in t.Cleanup()
- Test topology changes, failures, migrations

## 📝 Documentation Maintenance

When making changes, update docs in this order:
1. DESIGN.md FIRST (document the design)
2. Implement the change
3. Update AGENTS.md (if workflows change)
4. Update README.md (if user-facing API changes)
5. Update CHANGELOG.md (ALWAYS, with version number)

## 🎨 Code Style

### Naming
- Exported: PascalCase
- Unexported: camelCase
- Interfaces: -er suffix when possible

### Comments
- GoDoc comments on all exported symbols
- Comment non-obvious logic
- TODO comments: // TODO: description

### File Organization
- One primary type per file
- Group related functionality
- Order: imports, constants, types, constructors, methods, helpers

## 🔍 gopls MCP Tools (if available)

Prefer gopls tools over grep/search for Go code:
- SearchSymbol - Find symbols by name
- FindReferences - Find all usages before refactoring
- GoToDefinition - Navigate to definitions
- Hover - Get type information and docs
- GetDiagnostics - Check for errors after editing
- FormatCode / OrganizeImports - Clean up code

## 🚨 Common Gotchas

### Hashslot Calculation
- Use CRC16(channel) % 16384 (matches Redis Cluster)
- Support hashtag syntax: {tag}

### Topology Changes
- Always handle MOVED and ASK redirects
- Implement timeout for migration detection (30s)
- Use stall detection (2s)

### Callback Execution
- Always recover from panics
- Use worker pool to limit concurrency
- Implement queue backpressure

## 📊 Metrics (OpenTelemetry)

- All metrics MUST be optional (build tag: nometrics)
- Zero overhead when disabled
- Use interface abstraction (metricsRecorder)
- See metrics.go and metrics_otel.go

## 🎯 Git Commits

Follow conventional commits:
- feat: New feature
- fix: Bug fix
- docs: Documentation only
- test: Test improvements
- refactor: Code restructuring
- perf: Performance improvement
- chore: Maintenance

Example: "feat(pool): add connection reuse for hashslots"

## 🔗 Before Submitting Changes

- [ ] All tests pass (go test ./...)
- [ ] No race conditions (go test -race ./...)
- [ ] Code formatted (go fmt ./...)
- [ ] Exported symbols have GoDoc
- [ ] Lock ordering followed
- [ ] Error handling comprehensive
- [ ] DESIGN.md updated (if needed)
- [ ] CHANGELOG.md updated (ALWAYS)
- [ ] Integration tests added/updated (if applicable)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lalloni) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
