## sati-go

> Ports package organization and interface design patterns


# Ports Package Organization

## Package Purpose
The ports package defines all interfaces, contracts, and types that the application depends on. It serves as the contract layer between the domain and infrastructure layers.

## File Organization

### Core Files
- **`client.go`** - Client interface definitions for external services
- **`config.go`** - Configuration-related interfaces
- **`domain.go`** - Domain service interfaces
- **`types.go`** - All custom types, parameters, and results
- **`doc.go`** - Package documentation

### File Responsibilities

#### `client.go` - Client Interfaces
```go
// ClientInterface defines the interface for external service clients
type ClientInterface interface {
    // Close closes the client connection
    Close() error
    
    // Agent Management
    AddAgentCallResponse(ctx context.Context, params AddAgentCallResponseParams) (AddAgentCallResponseResult, error)
    GetAgentByID(ctx context.Context, params GetAgentByIDParams) (GetAgentByIDResult, error)
    // ... other client methods
}
```

#### `config.go` - Configuration Interfaces
```go
// ConfigWatcher defines the interface for configuration watching
type ConfigWatcher interface {
    Start(ctx context.Context) error
    Stop() error
    IsWatching() bool
}

// ConfigLoader defines the interface for configuration loading
type ConfigLoader interface {
    LoadConfig(path string) (interface{}, error)
    LoadConfigFromString(configString string) (interface{}, error)
}
```

#### `domain.go` - Domain Service Interfaces
```go
// DomainService defines the interface for domain services
type DomainService interface {
    StartConfigWatcher(ctx context.Context) error
    StartExileClientConfiguration() error
    StartPollEvents() error
    StartStreamJobs() error
    StartHostPlugin() error
    StopAllProcesses() error
    IsRunning() bool
}

// ProcessManager defines the interface for managing domain processes
type ProcessManager interface {
    StartProcess(name string) error
    StopProcess(name string) error
    StopAllProcesses() error
    IsProcessRunning(name string) bool
    GetProcessNames() []string
}
```

#### `types.go` - All Custom Types
```go
// Parameter types for client methods
type AddAgentCallResponseParams struct {
    PartnerAgentID string
    CallSid        int64
    ResponseKey    string
    ResponseValue  string
    AgentSid       int64
}

// Result types for client methods
type AddAgentCallResponseResult struct{}

// Event and Job types
type Event struct {
    Type          string
    Telephony     *ExileTelephonyResult
    AgentCall     *ExileAgentCall
    AgentResponse *ExileAgentResponse
}

type Job struct {
    JobID string
    Type  string
    // ... other fields
}
```

## Interface Design Principles

### Single Responsibility
Each interface should have a single, well-defined responsibility:

```go
// ✅ Good: Single responsibility
type ConfigWatcher interface {
    Start(ctx context.Context) error
    Stop() error
    IsWatching() bool
}

// ❌ Bad: Multiple responsibilities
type ConfigManager interface {
    Start(ctx context.Context) error
    Stop() error
    LoadConfig(path string) (interface{}, error)
    SaveConfig(config interface{}) error
    ValidateConfig(config interface{}) error
}
```

### Interface Segregation
Keep interfaces small and focused:

```go
// ✅ Good: Small, focused interfaces
type EventDispatcher interface {
    DispatchEvents(events []Event)
    DispatchJob(job *Job)
}

type ConfigChangeHandler interface {
    OnConfigChanged(oldConfig, newConfig *GetClientConfigurationResult)
    ShouldRestartProcesses(oldConfig, newConfig *GetClientConfigurationResult) bool
}
```

### Context-Aware Methods
Use `context.Context` for operations that can be cancelled:

```go
// ✅ Good: Context-aware
func GetAgentByID(ctx context.Context, params GetAgentByIDParams) (GetAgentByIDResult, error)

// ❌ Bad: No context
func GetAgentByID(params GetAgentByIDParams) (GetAgentByIDResult, error)
```

## Type Organization

### Parameter Types
- Use `XxxParams` suffix for method parameters
- Group related fields together
- Use descriptive field names
- Include validation tags if needed

```go
type GetAgentByIDParams struct {
    AgentID int64 `validate:"required,min=1"`
}

type UpdateAgentStatusParams struct {
    AgentID int64  `validate:"required,min=1"`
    Status  string `validate:"required,oneof=available busy offline"`
}
```

### Result Types
- Use `XxxResult` suffix for method results
- Include all relevant response data
- Use pointers for optional fields

```go
type GetAgentByIDResult struct {
    AgentID   int64
    Name      string
    Status    string
    Skills    []string
    CreatedAt time.Time
    UpdatedAt *time.Time // Optional field
}
```

### Event and Job Types
- Use clear, descriptive names
- Include all necessary fields
- Use pointers for optional fields
- Group related fields in embedded structs

```go
type Event struct {
    Type          string
    Timestamp     time.Time
    Telephony     *ExileTelephonyResult
    AgentCall     *ExileAgentCall
    AgentResponse *ExileAgentResponse
}

type Job struct {
    JobID     string
    Type      string
    Priority  int
    Payload   map[string]interface{}
    CreatedAt time.Time
    ExpiresAt *time.Time
}
```

## Documentation Standards

### Interface Documentation
```go
// ClientInterface defines the interface for the TCN Exile Gate service client.
// This interface abstracts the concrete client implementation and allows for
// easier testing and dependency injection following clean architecture principles.
type ClientInterface interface {
    // Close closes the client connection.
    Close() error
    
    // GetAgentByID retrieves an agent by their ID.
    // Returns an error if the agent is not found or if the request fails.
    GetAgentByID(ctx context.Context, params GetAgentByIDParams) (GetAgentByIDResult, error)
}
```

### Type Documentation
```go
// GetAgentByIDParams contains the parameters for retrieving an agent by ID.
type GetAgentByIDParams struct {
    // AgentID is the unique identifier of the agent to retrieve.
    AgentID int64 `validate:"required,min=1"`
}

// GetAgentByIDResult contains the result of retrieving an agent by ID.
type GetAgentByIDResult struct {
    // AgentID is the unique identifier of the agent.
    AgentID int64
    
    // Name is the display name of the agent.
    Name string
    
    // Status is the current status of the agent.
    Status string
}
```

## Import Organization

### Standard Import Order
```go
import (
    // Standard library
    "context"
    "time"
    
    // Third-party packages
    "github.com/go-playground/validator/v10"
)
```

### Import Guidelines
- **No internal package imports** (ports should be self-contained)
- **Minimal external dependencies**
- **Group imports** by type (standard, third-party)
- **Use absolute imports** for external packages

## Best Practices

### Interface Design
- Keep interfaces small and focused
- Use descriptive method names
- Include context for cancellable operations
- Return errors for operations that can fail
- Use consistent parameter/result patterns

### Type Design
- Use clear, descriptive names
- Group related fields together
- Use pointers for optional fields
- Include validation tags when appropriate
- Use embedded structs for common fields

### Documentation
- Document all public interfaces
- Include usage examples where helpful
- Explain the purpose of each interface
- Document parameter and result types
- Keep documentation up to date

### Versioning
- Use versioned interfaces for breaking changes
- Maintain backward compatibility when possible
- Document deprecation notices
- Provide migration guides for breaking changes

## Anti-Patterns to Avoid

### ❌ Large Interfaces
```go
// ❌ Bad: Too many methods
type GodInterface interface {
    // 50+ methods...
}
```

### ❌ Interface in Wrong Package
```go
// ❌ Bad: Interface in domain package
// pkg/domain/interfaces.go
type DomainService interface { ... }
```

### ❌ Inconsistent Naming
```go
// ❌ Bad: Inconsistent naming
type GetAgentParams struct { ... }
type AgentResult struct { ... }
type AgentCallResponse struct { ... }
```

### ❌ Missing Context
```go
// ❌ Bad: No context for cancellable operations
func GetAgent(params GetAgentParams) (GetAgentResult, error)
```

### ❌ Poor Documentation
```go
// ❌ Bad: No documentation
type ClientInterface interface {
    GetAgent(ctx context.Context, params GetAgentParams) (GetAgentResult, error)
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tcncloud)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tcncloud)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
