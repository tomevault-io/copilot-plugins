## safe-install

> This file provides coding agents with essential information to work effectively in this repository.

# Agent Guidelines for safe-install

This file provides coding agents with essential information to work effectively in this repository.

## Build, Test, and Quality Commands

### Building
```bash
go mod download          # Download dependencies
go build -o safe-install # Build for current platform
go build -o safe-install-linux   # Build for Linux
go build -o safe-install-darwin  # Build for macOS
go build -o safe-install.exe     # Build for Windows
```

### Running Tests
```bash
go test ./...                        # Run all tests
go test ./internal/config/...        # Run tests for specific package
go test ./internal/config/... -run TestLoad_Success/valid_config  # Run single test case
go test ./internal/config/... -v     # Run with verbose output
go test ./internal/config/... -cover # Run with coverage
```

### Code Quality
```bash
go fmt ./...      # Format all Go files
go vet ./...      # Run static analysis
```

## Project Overview

Safe-Install is a cross-platform CLI wrapper enforcing security policies across package managers. Initial scope is pip (Python), designed for npm extension.

**Architecture:** Subcommand pattern with Cobra CLI framework
**Language:** Go 1.25.6+
**Entry Point:** `main.go` → `cmd.Execute()`

## Directory Structure
```
safe-install/
├── cmd/                 # CLI command definitions (Cobra)
│   ├── root.go          # Root command
│   ├── pip.go           # pip subcommand
│   └── npm.go           # npm subcommand (stub)
├── internal/
│   ├── config/          # Config loading & validation
│   └── policy/          # Rules engine (env checks, argument injection)
└── main.go
```

## Code Style Guidelines

### Imports
- Group imports: stdlib, third-party, internal packages
- Sort alphabetically within groups
- Use blank line between groups
- No unused imports

Example:
```go
import (
    "fmt"
    "os"

    "github.com/spf13/cobra"

    "safe-install/internal/config"
)
```

### Formatting
- Use `gofmt` standard formatting (tabs for indentation)
- Maximum line length: not strictly enforced, use common sense
- One blank line between functions
- Exported functions have comments (godoc style)

### Types and Naming
- **Structs:** PascalCase, exported fields use PascalCase, unexported fields use camelCase
- **Functions:** PascalCase for exported, camelCase for unexported
- **Variables:** camelCase for local/struct fields, short names in loops (i, idx)
- **Constants:** PascalCase for exported, camelCase for unexported
- **JSON tags:** snake_case to match config schema

Example:
```go
type PipConfig struct {
    Enabled     bool   `json:"enabled"`
    RequireVenv bool   `json:"require_venv"`
    RegistryURL string `json:"registry_url"`
}

func Load() (*Config, error) { ... }
func getConfigPath() (string, error) { ... }
```

### Error Handling
- Always wrap errors with context using `fmt.Errorf("description: %w", err)`
- Return errors from functions, don't panic in normal flow
- Check errors immediately after operations
- Provide descriptive error messages

Example:
```go
data, err := os.ReadFile(path)
if err != nil {
    return nil, fmt.Errorf("failed to read config file: %w", err)
}
```

### Testing
- Use table-driven tests for multiple scenarios
- Name tests descriptively: `TestFunctionName_Scenario`
- Use `t.Run()` for subtests
- Test both success and error paths
- Clean up resources with `defer`
- Use `t.TempDir()` for temporary files

Example:
```go
func TestLoad_Success(t *testing.T) {
    tests := []struct {
        name       string
        configJSON string
        wantErr    bool
        validate   func(*testing.T, *Config)
    }{
        {
            name: "valid config",
            configJSON: `{"common": {...}}`,
            wantErr: false,
            validate: func(t *testing.T, cfg *Config) {
                if cfg.Common.BlockInteractive != true {
                    t.Errorf("got %v, want true", cfg.Common.BlockInteractive)
                }
            },
        },
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            tmpDir := t.TempDir()
            // Test logic
        })
    }
}
```

### Configuration
- Config is JSON, located at platform-specific paths
- Use `runtime.GOOS` for platform detection
- Fail fast on missing/invalid config
- Struct tags define JSON field names (snake_case)

### CLI Patterns
- Use Cobra for command structure
- Commands register in `init()` functions
- Use `DisableFlagParsing: true` when passing through args to subprocess
- Errors print to stderr and exit with status 1

### Security Considerations
- Validate user arguments before execution
- Block dangerous flags (--index-url, --extra-index-url for pip)
- Enforce virtual environment requirements when configured
- Inject safety flags (--no-input, --index-url override)
- Never log secrets or sensitive configuration

### Package Organization
- `cmd/`: CLI layer only, delegates to internal packages
- `internal/config/`: Config loading, validation, structs
- `internal/policy/`: Business logic, validation, argument building
- Keep packages focused and single-responsibility
- Circular imports are forbidden

### When Making Changes
1. Write tests first if adding new functionality
2. Run `go fmt ./...` before committing
3. Run `go vet ./...` to catch common issues
4. Run `go test ./...` to verify no regressions
5. Test specific package with `go test ./path/to/package/...`
6. Test single test case with `-run TestName/pattern`

### Adding New Package Managers
1. Create `internal/policy/{manager}_policy.go`
2. Create `cmd/{manager}.go` with Cobra command
3. Add config struct in `internal/config/config.go`
4. Add manager to `ManagersConfig` struct
5. Follow existing pip implementation as template

---
> Source: [zarguell/safe-install](https://github.com/zarguell/safe-install) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
