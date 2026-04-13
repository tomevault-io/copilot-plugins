## fleming

> > Favor clarity over cleverness. Explicitness over implicitness.

# Fleming Backend Directives (Go 1.25+)

> Favor clarity over cleverness. Explicitness over implicitness.

## 1. Tooling & Quality
- **Formatting**: `gofumpt -w .` and `goimports`.
- **Linting**: `golangci-lint run --fix` (mandatory in CI).
- **Go Version**: 1.25+ exclusively. Leverage `slog` and `errors.Join`.

## 2. Naming & Structure
- **Vertical Slicing**: Organize by feature in `internal/{feature}/`.
- **Interfaces**: Small and descriptive (`Repository`, not `UserRepositoryer`).
- **Constructors**: MUST use explicit `NewXxx(deps...)` functions. No globals.

## 3. Database & Logic
- **Repository Pattern**: Services depend on interfaces, never `*gorm.DB`.
- **Context**: MUST be the first parameter in all blocking/IO calls. Respect `ctx.Done()`.
- **Entities**: Live in `entity.go` with GORM tags.

## 4. HTTP & Error Handling
- **Thin Handlers**: Handlers translate HTTP ↔ Domain only. No logic.
- **Error Wrapping**: Always use `%w` for context. Never ignore errors.
- **Logging**: Use `log/slog` structured logging with `slog.InfoContext(ctx, ...)`.

## 5. Concurrency
- **No Naked Goroutines**: Tie lifetimes to context or a manager.
- **Prefer `errgroup`**: For parallel tasks with error propagation.
- **Mutexes**: Use `sync.RWMutex` for shared state protection.

## 6. Performance & Efficiency (Go 1.25)
- **Avoid Allocations**: Preallocate slices/maps when sizes are known.
- **Reuse Buffers**: Use `bytes.Buffer` or `strings.Builder` for string assembly.
- **Prefer stdlib helpers**: Use `cmp`, `slices`, `maps` instead of custom loops.
- **Context Cause**: Use `context.WithCancelCause`/`WithTimeoutCause` for cancellation diagnostics.
- **Structured Logging**: Use `slog` fields, avoid `fmt.Sprintf` in hot paths.

## 7. Testing Strategy
- **Table-Driven**: Default for all unit tests.
- **Coverage**: 80%+ on domain/services.
- **Parallel**: Use `t.Parallel()` for independent tests.

## 8. Protocol-First Architecture (CRITICAL)

### Protocol Layer Usage
- **MUST**: Backend services implement protocol interfaces (`timeline.GraphReader`, `timeline.GraphWriter`, etc.)
- **MUST**: Use protocol domain models (`protocol.Event`, `protocol.Grant`) in service layer
- **MUST**: Convert protocol types ↔ entity types at repository boundary only
- **MUST**: Call protocol validation (`event.Validate()`) before persistence
- **MUST NOT**: Create parallel type systems (e.g., `TimelineEvent` without protocol `Event`)
- **MUST NOT**: Bypass protocol validation in handlers

### Conversion Pattern
```go
// ✅ CORRECT: Handler → Protocol → Entity → Storage
func (h *Handler) HandleCreate(c *gin.Context) {
    // 1. Build protocol domain model (validates)
    event, err := timeline.NewEventBuilder().
        WithPatientID(addr).
        WithType(eventType).
        Build()
    if err != nil { return err }
    
    // 2. Service uses protocol type
    if err := h.service.CreateEvent(ctx, event); err != nil { return err }
}

func (s *Service) CreateEvent(ctx context.Context, event *timeline.Event) error {
    // 3. Repository converts protocol → entity at boundary
    return s.repo.CreateEvent(ctx, event)  // repo implements timeline.GraphWriter
}

func (r *Repository) CreateEvent(ctx context.Context, event *timeline.Event) error {
    entity := ToTimelineEvent(event)  // Conversion layer
    return r.db.Create(entity).Error
}
```

```go
// ❌ WRONG: Handler → Entity → Storage (bypasses protocol)
func (h *Handler) HandleCreate(c *gin.Context) {
    entity := &TimelineEvent{  // Direct entity creation
        PatientID: addr,
        Type: eventType,
    }
    h.service.AddEvent(ctx, entity)  // Uses backend type, no validation
}
```

### Interface Implementation
- **MUST**: Repository implements protocol interfaces (`timeline.GraphReader`, `timeline.GraphWriter`)
- **MUST**: Service depends on protocol interfaces, not concrete repository types
- **MUST**: Use protocol types in function signatures, convert to entities only in repository

### Validation Enforcement
- **MUST**: Always call protocol validation before persistence
- **MUST**: Use protocol builders for complex object creation
- **MUST NOT**: Skip validation even if "we know it's valid"

### Example: Correct Timeline Service
```go
// Service depends on protocol interface
type Service struct {
    repo timeline.GraphWriter  // Protocol interface, not Repository
}

func (s *Service) CreateEvent(ctx context.Context, event *timeline.Event) error {
    // Validation already done by builder
    return s.repo.CreateEvent(ctx, event)  // Protocol type
}

// Repository implements protocol interface
type Repository struct {
    db *gorm.DB
}

func (r *Repository) CreateEvent(ctx context.Context, event *timeline.Event) error {
    // Convert protocol → entity at boundary
    entity := ToTimelineEvent(event)
    return r.db.Create(entity).Error
}
```

## 9. Error Handling & Validation

### Error Wrapping
- **MUST**: Always wrap errors with context using `%w`: `return fmt.Errorf("create event: %w", err)`
- **MUST**: Use protocol error types (`types.ValidationError`, `types.ProtocolError`)
- **MUST NOT**: Return raw errors without context

### Validation
- **MUST**: Use protocol validation (`event.Validate()`, `grant.Validate()`)
- **MUST**: Use protocol builders for complex objects (ensures validation)
- **MUST NOT**: Duplicate validation logic in backend

## 10. Type Conversion

### Conversion Layer Pattern
- **MUST**: Create conversion functions: `ToTimelineEvent()`, `ToProtocolEvent()`
- **MUST**: Place conversions in `internal/{feature}/converter.go`
- **MUST**: Convert at repository boundary only (protocol ↔ entity)
- **MUST NOT**: Convert in handlers or services (use protocol types throughout)

### Conversion Example
```go
// internal/timeline/converter.go
func ToTimelineEvent(event *timeline.Event) *TimelineEvent {
    return &TimelineEvent{
        ID:        event.ID.String(),
        PatientID: event.PatientID.String(),
        Type:      event.Type,
        // ... all fields
    }
}

func ToProtocolEvent(entity *TimelineEvent) (*timeline.Event, error) {
    id, err := types.NewID(entity.ID)
    if err != nil { return nil, err }
    
    patientID, err := types.NewWalletAddress(entity.PatientID)
    if err != nil { return nil, err }
    
    return &timeline.Event{
        ID:        id,
        PatientID: patientID,
        Type:      entity.Type,
        // ... all fields
    }, nil
}
```

## 11. Testing Protocol Integration

### Protocol Testing
- **MUST**: Test protocol interfaces are implemented correctly
- **MUST**: Test conversion functions (protocol ↔ entity)
- **MUST**: Test protocol validation is called
- **MUST NOT**: Mock protocol types in tests (use real protocol types)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/itspablomontes)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/itspablomontes)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
