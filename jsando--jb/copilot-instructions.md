## jb

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

jb (jay-bee) is a Java build tool written in Go that provides a faster, simpler alternative to Maven, Gradle, and Ant. It uses simple JSON configuration files and is implemented as a static Go binary for fast startup times.

## Common Development Commands

```bash
# Build the jb binary
go build

# Run all tests
go test ./...

# Run tests in a specific package
go test ./builder/...

# Format code
go fmt ./...

# Run static analysis
go vet ./...

# Build and test Java projects in tests/ directory (integration tests)
go test -v ./builder/
```

## Architecture Overview

The codebase follows a clean architecture with clear separation of concerns:

- **main.go**: Entry point that dispatches to command functions
- **builder/**: Core build logic including compilation, JAR creation, and test running
- **project/**: Project and module models representing jb configuration
- **maven/**: Maven repository integration for dependency resolution and publishing
- **java/**: Java-specific tooling (classpath handling, JDK detection)
- **util/**: Shared utilities (file operations, HTTP client, etc.)

Key patterns:
- Command pattern for CLI commands
- Builder pattern for project compilation
- Module system with dependency resolution
- Reuses Maven's repository infrastructure

## Configuration Format

jb uses JSON configuration files:
- `jb-module.json`: Single module configuration
- `jb-project.json`: Multi-module project configuration

Example module configuration:
```json
{
  "name": "my-project",
  "modules": ["module1", "module2"],
  "dependencies": {
    "compile": ["com.example:library:1.0.0"]
  }
}
```

## Testing Approach

- Unit tests use Go's standard testing package with testify assertions
- Integration tests in `builder/builder_test.go` build actual Java projects in `tests/` directory
- Test Java projects are located in `tests/` with various scenarios (single module, multi-module, JUnit tests)

## Key Implementation Areas

When working on features:
- Build logic: `builder/builder.go`
- Dependency resolution: `maven/repo.go`
- Project model: `project/module.go`, `project/project.go`
- JAR creation: `builder/jar.go`
- Test runner: `builder/test_runner.go`

## Current Development Focus

The project has a detailed improvement plan in `docs/plan.md`. Key areas being developed:
- Maven/Gradle project conversion (see `convert/` package)
- Enhanced test runner functionality
- Code coverage reporting
- Dependency upgrade checking

## Development Workflow

- From now on let's make any changes in a branch, and submit a PR to merge it to main

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jsando) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
