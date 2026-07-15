## pkg

> This project is a Go module (`github.com/presbrey/pkg`) that provides a collection of utility packages. The packages cover a range of functionalities, including:

# Project Overview

This project is a Go module (`github.com/presbrey/pkg`) that provides a collection of utility packages. The packages cover a range of functionalities, including:

*   **`cdns`**: Utilities for interacting with Content Delivery Network (CDN) providers like Cloudflare and Fly.io.
*   **`echovalidator`**: Integration of the `go-playground/validator/v10` library with the Echo web framework.
*   **`hooks`**: A flexible hook registration and execution system with priority support.
*   **`slugs`**: A package for generating URL-safe slugs.
*   **`syncmap`**: An extension of `sync.Map` that synchronizes its values from a remote JSON URL.
*   **`syncthing`**: A more advanced, generic version of `syncmap` that provides type-safe access to map values.

The project is written in Go and uses Go modules for dependency management.

# Building and Running

To use the packages in this project, you can import them into your own Go code. For example, to use the `syncmap` package, you would add the following import to your Go file:

```go
import "github.com/presbrey/pkg/syncmap"
```

Then, you can run your Go application as usual.

There is no main application to run in this project, as it is a collection of library packages.

## Testing

To run the tests for all packages, you can use the following command:

```bash
go test ./...
```

# Development Conventions

*   The project follows standard Go coding conventions.
*   Each package is located in its own subdirectory.
*   Each package has its own set of tests.
*   The project uses Go modules for dependency management. The `go.mod` file lists the project's dependencies.

---
> Source: [presbrey/pkg](https://github.com/presbrey/pkg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
