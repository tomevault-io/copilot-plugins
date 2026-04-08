## lybic-sdk-go

> This project is the official Go SDK for the Lybic API. Lybic provides infrastructure for developing, testing, and deploying GUI-based AI agents. This SDK allows developers to programmatically control the Lybic ecosystem, including managing sandboxes, projects, and interacting with agents.

# GEMINI.md

## Project Overview

This project is the official Go SDK for the Lybic API. Lybic provides infrastructure for developing, testing, and deploying GUI-based AI agents. This SDK allows developers to programmatically control the Lybic ecosystem, including managing sandboxes, projects, and interacting with agents.

The SDK is written in Go and provides a client for interacting with the Lybic REST API. It also includes a client for the Model Context Protocol (MCP) for more advanced agent development.

**Key Technologies:**

*   Go
*   Lybic API
*   Model Context Protocol (MCP)

**Architecture:**

The SDK is composed of two main clients:

*   **Lybic Client:** A standard HTTP client for interacting with the Lybic REST API. It provides methods for managing resources such as sandboxes, projects, and users.
*   **MCP Client:** A client for the Model Context Protocol (MCP), which is used for more advanced agent development, including tool calling.

## Building and Running

### Building

To build the project, you can use the standard `go build` command:

```bash
go build ./...
```

### Testing

The project uses `golangci-lint` for code quality checks. You can run the linter with the following command:

```bash
golangci-lint run
```

The project's CI/CD pipeline, defined in `.github/workflows/quality.yml`, also runs `golangci-lint` on every push and pull request to the `main` branch.

## Development Conventions

### Coding Style

The project follows the standard Go coding style.

### Contribution Guidelines

Contributions from the community are welcome. Please see the [Contributing Guidelines](https://github.com/lybic/lybic-sdk-go/blob/main/CONTRIBUTING.md) for more details on how to get involved.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lybic)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lybic)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
