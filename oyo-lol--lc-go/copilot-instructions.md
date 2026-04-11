## lc-go

> `lc-go` is a collection of Go libraries and utilities designed to simplify common development tasks. It includes packages for dependency injection, language enhancements (iterators, functional helpers), MongoDB object mapping, and Google Cloud Firestore integration.

# lc-go

## Project Overview

`lc-go` is a collection of Go libraries and utilities designed to simplify common development tasks. It includes packages for dependency injection, language enhancements (iterators, functional helpers), MongoDB object mapping, and Google Cloud Firestore integration.

### Key Components

*   **`dij` (Dependency Injection):** A lightweight, reflection-based dependency injection framework. It allows for defining dependencies using struct tags and building instances with resolved dependencies.
*   **`lg` (Language Utilities):** Provides helper functions to make Go code more concise and functional-style.
    *   `Ife`: Ternary operator equivalent (`cond ? trueVal : falseVal`).
    *   Iterators: `MakeIterator`, `IterateFunc`, `IterateFuncInversely`.
    *   Map-Reduce: Functional `Map` and `Reduce` implementations.
    *   Logging: `Printfln` for convenient formatted logging.
*   **`mongobj` (MongoDB Object):** A wrapper around the official MongoDB driver. It simplifies storing and retrieving structs using custom primary keys and handling unique indexes (`InitializeUniqueKey`, `Get`, `Set`, `SetBulk`).
*   **`gcp` (Google Cloud Platform):** Currently provides a wrapper for initializing a Google Cloud Firestore client.
*   **`httpily`:** Contains HTTP-related type definitions (implied).
*   **`io`:** Utilities for memory file handling (implied).

## Building and Running

Since this is a Go library, standard Go commands are used for building and testing.

### Prerequisites

*   Go 1.19 or higher

### Build

To build all packages in the project:

```bash
go build ./...
```

### Test

To run unit tests across all packages:

```bash
go test ./...
```

To run tests with verbose output:

```bash
go test -v ./...
```

## Development Conventions

*   **Project Structure:** Functionality is modularized into top-level directories (packages) based on domain (e.g., `dij` for DI, `lg` for language utils, `mongobj` for database).
*   **Dependency Management:** Dependencies are managed via `go.mod`.
*   **Testing:** Unit tests are co-located with the source code (e.g., `dependency_test.go` next to `dependency.go`). New features should include corresponding tests.
*   **Code Style:** Follow standard Go formatting and idioms (`gofmt`).
*   **Documentation:** Public functions and types should have documentation comments.

### Key Dependencies

*   `github.com/gin-gonic/gin`: Web framework (referenced in `go.mod` and `dij` examples).
*   `go.mongodb.org/mongo-driver`: Official MongoDB driver.
*   `cloud.google.com/go/firestore`: Google Cloud Firestore client.
*   `github.com/joho/godotenv`: Environment variable loader.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OYO-LOL)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/OYO-LOL)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
