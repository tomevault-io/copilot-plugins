## ship-status-dash

> This is a Go-based dashboard application for monitoring component status, availability, and outages. The system integrates with Prometheus for metrics, PostgreSQL for persistence, and Kubernetes/OpenShift for infrastructure monitoring.

# SHIP Status and Availability Dashboard Monitor

## Project Overview
This is a Go-based dashboard application for monitoring component status, availability, and outages. The system integrates with Prometheus for metrics, PostgreSQL for persistence, and Kubernetes/OpenShift for infrastructure monitoring.

## Technology Stack
- **Language**: Go 1.25
- **Web Framework**: gorilla/mux
- **Database**: PostgreSQL with GORM ORM
- **Monitoring**: Prometheus client
- **Infrastructure**: Kubernetes/OpenShift client-go
- **Logging**: logrus
- **Testing**: testify, testcontainers

## Project Structure
- `cmd/`: Command-line applications
- `pkg/types/`: Core data models and types
- `deploy/`: Deployment configurations
- `test/`: Test utilities and fixtures
- `hack/`: Development scripts

## Code Style and Standards

### General Go Best Practices
- Follow standard Go conventions and idioms
- Use `gofmt` for formatting (enforced)
- Organize imports in standard library, external, local order
- Keep functions focused and testable
- Don't do anything too clever, focus on readibility
- Look to reuse logic instead of copying where possible
- Use meaningful variable and function names
- Add godoc comments for exported types and functions

### Comments
- Avoid excessive commenting - let the code speak for itself
- Comments should explain **why**, not **what** the code does
- Use comments to clarify non-obvious business logic or design decisions
- Document edge cases, workarounds, or temporary solutions
- If code needs comments to explain what it does, consider refactoring for clarity

### Error Handling
- Always check and handle errors explicitly
- Use descriptive error messages with context
- Wrap errors with additional context when propagating
- Use `logrus` for logging errors with appropriate levels

### Database Patterns
- Use GORM models with proper struct tags
- Define indexes on frequently queried columns
- Use transactions for multi-step operations
- Handle NULL values with sql.NullTime, sql.NullString, or pointers
- Add `gorm.Model` for standard ID, CreatedAt, UpdatedAt, DeletedAt fields

### Configuration
- Accept configuration via YAML files
- Support command-line flags for runtime options
- Validate all configuration on startup
- Use the `types.Config` struct for component definitions

### HTTP API Design
- Use gorilla/mux for routing
- Return proper HTTP status codes
- Use JSON for request/response bodies
- Add proper error handling and logging for all endpoints
- Include CORS headers where appropriate

### Logging
- Use `logrus` as the structured logger
- Set appropriate log levels (Debug, Info, Warn, Error, Fatal)
- Include context in log messages (component names, IDs, etc.)
- Use `log.WithFields()` for structured logging ALWAYS use the same instance where possible, DO NOT redifine fields all over a single method

## Data Models

### Outage Model
The core model for tracking component outages:
- Includes severity, start/end times, and auto-resolution flags
- Supports confirmation workflow with `ConfirmedBy` and `ConfirmedAt`
- Tracks who discovered, created, and resolved the outage
- Allows triage notes for incident documentation

### Component Configuration
Components are defined in YAML with:
- Name, description, and ownership (SHIP team, Slack channel)
- Subcomponents with management and confirmation flags
- Owner groups (Rover groups or service accounts)

## Development Workflow

### Building
- Use the `Makefile` for build commands
- Ensure all dependencies are in `vendor/` directory
- Module name: `component-monitor`

### Testing
- Write table-driven tests
- Place tests in `*_test.go` files
- Run tests with `go test ./...`

### Database Migrations
- Use the `cmd/migrate` utility for schema changes
- Test migrations with both up and down operations
- Document schema changes in migration files

## Important Considerations

### Security
- Never commit database credentials
- Use environment variables or secure configuration for DSN
- Validate all user inputs
- Use parameterized queries (GORM handles this)

## Dependencies to Know
- `github.com/gorilla/mux`: HTTP routing
- `gorm.io/gorm`: ORM with PostgreSQL driver
- `github.com/sirupsen/logrus`: Structured logging
- `github.com/prometheus/client_golang`: Prometheus metrics
- `k8s.io/client-go`: Kubernetes API client
- `github.com/openshift/client-go`: OpenShift API client

---
> Source: [openshift-eng/ship-status-dash](https://github.com/openshift-eng/ship-status-dash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-08 -->
