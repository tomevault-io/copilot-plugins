## k8s-resource-cli

> This file provides guidance for agentic coding agents operating in this repository.

# AGENTS.md

This file provides guidance for agentic coding agents operating in this repository.

## Build Commands

```bash
# Build the binary
go build -o k8s-resource-cli ./cmd/k8s-resource-cli

# Install to PATH
go install ./cmd/k8s-resource-cli

# Build with version info
go build -ldflags="-X main.version=$(git describe --tags --always)" -o k8s-resource-cli ./cmd/k8s-resource-cli
```

## Testing

```bash
# Run all tests
go test ./...

# Run tests with verbose output
go test -v ./...

# Run a single test
go test -v -run TestFunctionName ./...

# Run tests with coverage
go test -cover ./...
```

Note: This project currently has no test files. When adding tests, place them in files named `*_test.go` alongside the code they test.

## Linting and Formatting

```bash
# Format code (use before committing)
gofmt -w .

# Check formatting (non-destructive)
gofmt -d .

# Run go vet
go vet ./...

# View dependency graph
go mod graph
```

## Code Style Guidelines

### Package Structure

- All source files are in the `main` package within `cmd/k8s-resource-cli/`
- Files are organized by responsibility: `cli.go`, `kubernetes.go`, `porter.go`, `output.go`, `types.go`
- Keep functions focused and under 100 lines when possible

### Imports

- Group imports: stdlib first, then external packages
- Use alias for Kubernetes imports: `metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"`
- Example order:
  ```go
  import (
      "context"
      "fmt"
      "net/http"

      metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
      "k8s.io/client-go/kubernetes"
      "k8s.io/metrics/pkg/client/clientset/versioned"
  )
  ```

### Naming Conventions

- **Types**: PascalCase (e.g., `DeploymentMetrics`, `PorterClient`)
- **Variables/Functions**: camelCase (e.g., `getDeploymentMetrics`, `currentReplicas`)
- **Constants**: PascalCase or SCREAMING_SNAKE_CASE (e.g., `OutputTypeUsage`)
- **Private fields**: Leading underscore not used (e.g., `deploymentTargetCache`)
- **Receiver variables**: Single letter (e.g., `c *PorterClient`, `dm DeploymentMetrics`)

### Error Handling

- Use `fmt.Errorf("message: %w", err)` to wrap errors
- Return errors from functions; handle them at the top level
- Exit on fatal errors in CLI entry points: `os.Exit(1)` after printing to `os.Stderr`
- Example pattern:
  ```go
  func getDeploymentMetrics(...) (DeploymentMetrics, error) {
      deployment, err := clientset.AppsV1().Deployments(namespace).Get(ctx, name, metav1.GetOptions{})
      if err != nil {
          return DeploymentMetrics{}, fmt.Errorf("error getting deployment: %w", err)
      }
      // ... rest of function
      return dm, nil
  }
  ```

### Context Usage

- Pass `context.Context` as first parameter to functions that may timeout or be cancelled
- Use `context.Background()` at the CLI entry point
- Use `http.NewRequestWithContext()` for HTTP requests

### Struct Design

- Use simple structs with explicit fields for data transfer objects
- Embedding not used for these types
- Use pointers only when nil checking is needed
- Example:
  ```go
  type DeploymentMetrics struct {
      Name            string
      Namespace       string
      Type            string
      CurrentReplicas int32
      DesiredReplicas int32
      MaxReplicas     int32
      Usage           ResourceMetrics
      Requests        ResourceMetrics
      MaxRequests     ResourceMetrics
  }
  ```

### JSON Struct Tags

- Use `json:"field_name"` for API response structs
- Use `omitempty` for optional fields: `json:"field_name,omitempty"`

### Output Formatting

- Use `text/tabwriter` for aligned table output
- Print data to `os.Stdout`, progress/debug to `os.Stderr`
- Use `fmt.Fprintf()` for writing to specific writers

### Flag Parsing

- Use the standard `flag` package
- Register flags in `runCLI()` with appropriate types and defaults
- Use constants for default values: `OutputTypeRequests`, `OutputTypeUsage`, etc.

### Kubernetes API Patterns

- Use `metav1.ListOptions{}` and `metav1.GetOptions{}` for API calls
- Handle `nil` pointers from Kubernetes APIs (e.g., `cpu.MilliValue()`)
- Use `clientcmd.BuildConfigFromFlags()` for kubeconfig loading

### Critical Files

- `cli.go:20` - Main `runCLI()` entry point
- `kubernetes.go:35` - `getDeploymentMetrics()` core function
- `porter.go:13` - `getPorterApplicationMetrics()` Porter API function
- `output.go:10` - `printResults()` formatting function

### Common Patterns

- Progress spinners use stderr with carriage returns: `\r` and `\033[K` to clear lines
- Cache expensive API lookups in client structs
- Use `for range` when index is unused: `for range collection`

---
> Source: [madeddie/k8s-resource-cli](https://github.com/madeddie/k8s-resource-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
