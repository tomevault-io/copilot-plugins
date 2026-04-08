## titleparser

> This document provides essential guidelines for AI agents working on the `titleparser` codebase.

# Gemini Development Guide for titleparser

This document provides essential guidelines for AI agents working on the `titleparser` codebase.

## Core Architecture

`titleparser` is a Go-based AWS Lambda function designed to extract titles from URLs. It employs a handler-based, plug-in style architecture.

- **Execution Flow**: The primary entry point is `lambda/main.go`, which receives a URL. It then iterates through a series of registered handlers, attempting to find one that matches the URL's domain. If no specific handler matches, it falls back to a default handler that extracts the title from OpenGraph or HTML `<title>` tags.
- **Handler-based Design**: Each supported website (e.g., Reddit, YouTube, HackerNews) has its own handler in the `handler/` directory. These handlers are self-registering using Go's `init()` function. For example, `handler/reddit.go` contains the logic for parsing Reddit URLs and registers itself with the main application. This design makes it easy to add support for new websites without modifying the core application logic.
- **Configuration**: Key configuration, such as the HTTP client's User-Agent, is centralized in `handler/config.go`.
- **Caching**: The application uses DynamoDB for caching results, but this is disabled when running in local testing mode.

## Developer Workflow

The project uses `Taskfile.yml` for managing common development tasks.

- **Building**:
  - For AWS Lambda deployment: `task build` (creates a zip file in `build/`)
  - For local testing: `task build-local`
- **Testing**:
  - Run all tests: `task test`
  - Run a specific test: `go test -v ./handler -run TestSpecificFunctionName`
- **Linting**: `task lint`
- **Local Development**:
  1. Build for local execution: `task build-local`
  2. Run the local server: `RUNMODE=local ./titleparser`
  3. Send requests to `http://localhost:8081/title`.

## Code Conventions

- **Formatting**: Always run `goimports -w .` to format code and manage imports.
- **Testing**: Use table-driven tests and run them in parallel with `t.Parallel()`.
- **Error Handling**: Return descriptive errors from functions. Avoid using `log.Fatal` within handlers.
- **Logging**: Use the `logrus` library for structured logging (e.g., `log.Infof`, `log.Warnf`).
- **Handlers**: New handlers should be placed in the `handler/` directory, following the existing pattern of a domain-matching regex and a parsing function.

## Shared Standards

This project incorporates shared standards from the `llm-shared` submodule. Refer to these files for broader guidelines:

- **Go best practices**: `llm-shared/languages/go.md`
- **General project standards**: `llm-shared/project_tech_stack.md`
- **GitHub workflow**: `llm-shared/GITHUB.md`

Before completing any task, ensure your changes pass the build by running `task build`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lepinkainen)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lepinkainen)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
