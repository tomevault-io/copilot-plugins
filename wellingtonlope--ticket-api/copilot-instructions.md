## ticket-api

> go test -v -covermode=count -coverprofile=coverage.out ./...

# AGENTS.md

## Commands

### Build
```bash
go run cmd/main.go
```

### Test
```bash
# Run all tests
go test ./...

# Run all tests with coverage
go test -v -covermode=count -coverprofile=coverage.out ./...

# Run a single test
go test -v ./path/to/package -run TestFunctionName

# Run tests in a specific package
go test -v ./path/to/package

# Run BDD tests
cd test && go test -v -run ^TestRegisterFeature$
```

### Lint/Format
```bash
# Format code
make format

# Check formatting (dry-run)
make format-check

# Run linters
make lint

# Run tests
make test

# Run format-check, lint, and test
make all

# Install required tools
make install-tools
```

### Docker
```bash
docker-compose up --build
```

## Code Style Guidelines

### Imports
- Standard library imports first, then third-party imports, then project imports
- Each import group separated by a blank line
- No unused imports

```go
import (
    "errors"
    "time"

    "github.com/stretchr/testify/assert"
    "github.com/wellingtonlope/ticket-api/internal/domain"
)
```

### Formatting
- Use `gofumpt` for formatting (stricter than go fmt)
- No line comments unless for special cases
- No trailing whitespace
- Use tabs for indentation (Go standard)

### Types and Naming
- Use PascalCase for exported types, functions, constants
- Use camelCase for unexported types, functions, constants
- Interface names should be simple (e.g., `Get`, `Login`, `Register`)
- Struct fields are PascalCase
- Local variables are camelCase
- Error variables are prefixed with `Err` (e.g., `ErrTicketNotFound`)
- Repository interfaces defined in `internal/app/repository/`

### Error Handling
- Use `errors.New()` for simple errors
- Define package-level error variables
- Return errors from functions, never panic in business logic
- Check errors immediately
- Domain-specific errors defined in `internal/domain/`
- Repository errors defined in `internal/app/repository/`

### Domain Layer (internal/domain/)
- Contains business logic and entities
- Uses value objects (Email, Password)
- Defines package-level errors with `var` blocks
- Factory functions for entity creation (e.g., `OpenTicket()`, `UserRegister()`)
- Methods on structs for business operations (e.g., `ticket.Get()`, `ticket.Close()`)
- Private fields on value objects (e.g., `email` string in Email struct)

### Use Cases (internal/app/usecase/)
- Each use case in its own package (ticket/, user/)
- Interface definition with `Handle()` method
- Private struct implementation with dependencies
- Constructor function `NewXxx()`
- Input structs (e.g., `GetInput`, `LoginInput`)
- Output DTOs in package-level files (e.g., ticket.go, user.go)
- Convert domain objects to output DTOs with helper functions (e.g., `ticketOutputFromTicket()`)

### Repository Pattern
- Interface definition in `internal/app/repository/`
- Multiple implementations (memory, mongo)
- Return errors for not found cases
- Methods: `GetByID()`, `Insert()`, `Update()`, `GetAll()`, `DeleteByID()`, `GetAllByXxx()`

### Testing
- Use testify/assert for assertions
- Test file name: `xxx_test.go`
- Use table-driven tests with `t.Run()` for scenarios
- Use in-memory repositories for testing
- Test both success and error cases
- Assertions: `assert.Nil(t, err)`, `assert.NotNil(t, output)`, `assert.Equal(t, expected, actual)`

### HTTP Layer (internal/infra/http/)
- Echo framework for HTTP server
- Middleware pattern for auth
- Handler function signature: `func(Request) Response`
- Controllers in user.go, ticket.go
- Error responses wrapped in `ErrorResponse` struct
- JSON marshaling with `wrapBody()`, `wrapError()`

### BDD Testing
- Feature files in `test/features/` using Gherkin syntax
- Step definitions in `test/steps/*.go` (e.g., `register.go` for `register.feature`)
- Shared utilities in `test/steps/utils.go`
- Entry point in `test/suite_test.go` with `TestXxxFeature` functions
- Feature suite structs (e.g., `RegisterFeatureSuite`) contain step definition methods
- Step definitions are methods of the feature suite struct (not standalone functions)
- Use `InitializeTestSuite()` and `InitializeScenario()` methods on the suite struct
- Run with: `cd test && go test -v -run ^TestRegisterFeature$`
- Uses godog framework integrated with `go test`

### Architecture
- Clean Architecture with DDD principles
- Three main layers: Domain, Application (usecases), Infrastructure
- Dependency injection through interfaces
- Domain layer has no dependencies on outer layers
- Use dependency inversion principle

### Constants
- Use const blocks at package level
- PascalCase for exported constants
- String-based enums for status types (e.g., `StatusOpen`, `ProfileOperator`)

### Time Handling
- Use `time.Time` from Go standard library
- Pass timestamps from use cases to domain
- Store as pointers in structs for nilability

### Security
- JWT authentication in `internal/infra/jwt/`
- User profile types: `ProfileOperator`, `ProfileClient`
- Security errors in `internal/app/security/`
- Authorization middleware for protected routes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wellingtonlope)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/wellingtonlope)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
