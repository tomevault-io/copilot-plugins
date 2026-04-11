## lazykube

> This document provides guidance for the Gemini agent to effectively assist with the development of the `lazykube` project.

# Gemini Project Guide for lazykube

This document provides guidance for the Gemini agent to effectively assist with the development of the `lazykube` project.

## Project Overview

`lazykube` is a terminal-based UI for managing Kubernetes clusters. It provides a user-friendly interface for viewing and interacting with Kubernetes resources, such as pods, deployments, and namespaces.

## Architecture

The project follows a clean architecture pattern, with a clear separation of concerns between the different layers:

-   **`main.go`**: The entry point of the application.
-   **`controllers`**: Contains the controllers that handle user input and orchestrate the interactions between the other layers.
-   **`entities`**: Defines the core domain entities of the application, such as `Pod` and `Deployment`.
-   **`gateways`**: Provides an abstraction layer for accessing external services, such as the Kubernetes API.
-   **`infrastructure`**: Contains the implementation of the gateways, as well as the TUI (Text-based User Interface).
-   **`registries`**: Acts as a cache for the Kubernetes resources.
-   **`usecases`**: Contains the business logic of the application.

## Key Concepts

-   **Resource Dictionary (`resourceDict`)**: A central data structure that holds the state of the TUI, including the current view, the selected resource, and the pages.
-   **Pages (`tview.Pages`)**: The `tview` library uses a `Pages` widget to manage different views or screens of the application.

## Development Workflow

### Building the project

To build the project, run the following command:

```sh
go build
```

### Running the tests

To run the tests, run the following command:

```sh
go test ./...
```

### Running the application

To run the application, run the following command:

```sh
go run ./...

```

## Coding Conventions

-   Follow the standard Go coding conventions.
-   Use the `gofmt` tool to format the code.
-   Use the `govet` tool to check for suspicious constructs.

## Important Files

-   `main.go`: The entry point of the application.
-   `infrastructure/tui/app.go`: The main file for the TUI.
-   `infrastructure/tui/table_resource.go`: The file that implements the table view for the Kubernetes resources.
-   `controllers/PodController.go`: The controller for handling pod-related actions.
-   `gateways/PodGateway.go`: The gateway for accessing pod data from the Kubernetes API.

## Preferences

### Programming Languages and Frameworks

-   **Preferred Language:** Go
-   **Frameworks:** I generally do not use frameworks.

### Coding Style

I follow the official Go conventions and best practices as outlined in:
-   [Effective Go](https://golang.org/doc/effective_go.html)
-   [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)

This includes:
-   Writing clear and concise comments.
-   Using short, descriptive names for variables and functions.
-   Keeping functions small and focused.
-   Handling errors explicitly.

### Tools and IDEs

-   **Editor:** nvim

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andytechcastro)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/andytechcastro)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
