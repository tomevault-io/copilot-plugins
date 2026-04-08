## todoapp-go

> This is a command-line Todo application written in Go. It allows users to manage their tasks from the terminal. The application is in the early stages of development, with many features planned but not yet implemented.

# Project Overview

This is a command-line Todo application written in Go. It allows users to manage their tasks from the terminal. The application is in the early stages of development, with many features planned but not yet implemented.

The project follows a clean architecture, with a clear separation of concerns between the command-line interface, business logic, and data storage.

**Key Technologies:**

*   **Language:** Go
*   **Build Tool:** Make
*   **Configuration:** YAML, with support for environment variables

**Architecture:**

*   `cmd/todo/main.go`: The main entry point of the application. It handles command-line argument parsing and initializes the application.
*   `internal/config`: Handles loading and validation of application configuration. It searches for a `config.yaml` file in predefined locations, and if not found, automatically creates a default one in the user's home directory (`~/.todoapp/config.yaml`).
*   `internal/storage`: Defines a `Storage` interface for data persistence and provides a file-based implementation. The interface-based design allows for other storage backends (like a database) to be added in the future.
*   `internal/models`: Defines the data structures for the application, such as `Todo` items.
*   `internal/todo`: Contains the core business logic of the application.
    *   `service.go`: The service layer, which encapsulates the business logic and coordinates with the storage layer.
    *   `handler.go`: The handler layer, which is responsible for handling CLI commands, validating user input, and formatting the output.

# Configuration

The application uses a YAML file for configuration. The configuration file is searched for in the following order:

1.  A path provided via the `-config` command-line flag.
2.  `./config.yaml` (relative to the project root)
3.  `~/.todoapp/config.yaml`
4.  `/etc/todoapp/config.yaml`

If no configuration file is found in any of these locations when the application is run without a custom path, a default `config.yaml` is automatically created at `~/.todoapp/config.yaml`. This file is a copy of the `config.yaml` located in the project root.

Users can also explicitly create this default configuration file by running:
```bash
todo init
```

# Building and Running

The project uses a `Makefile` to automate common development tasks.

*   **Build the application:**
    ```bash
    make build
    ```
*   **Run the application:**
    ```bash
    make run
    ```
*   **Run tests:**
    ```bash
    make test
    ```
*   **Run the linter:**
    ```bash
    make lint
    ```
*   **Clean build artifacts:**
    ```bash
    make clean
    ```
*   **Install dependencies:**
    ```bash
    make install
    ```

# Development Conventions

The codebase is well-structured and follows common Go idioms. The use of an interface for the storage layer is a good practice that promotes modularity and testability.

The extensive use of `TODO` comments throughout the code indicates that the project is under active development. These comments provide a clear roadmap for future work.

The project uses `golangci-lint` for linting, which helps to maintain code quality and consistency.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/elxphaz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/elxphaz)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
