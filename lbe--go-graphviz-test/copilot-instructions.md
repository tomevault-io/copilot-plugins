## go-graphviz-test

> This Go project, `go-graphviz-test`, serves as a minimal reproducible example to investigate and demonstrate a concurrency issue within the `goccy/go-graphviz` library. The core purpose is to showcase that running multiple graph-generation operations in parallel (via goroutines) causes the application to panic.

# Gemini Project Overview: go-graphviz-test

## Project Goal

This Go project, `go-graphviz-test`, serves as a minimal reproducible example to investigate and demonstrate a concurrency issue within the `goccy/go-graphviz` library. The core purpose is to showcase that running multiple graph-generation operations in parallel (via goroutines) causes the application to panic.

## The Core Problem

When attempting to generate multiple graphs concurrently (e.g., 3 or more) using goroutines without a protective mutex, the application panics with a `fatal error: concurrent map writes`. This indicates that the version of the `goccy/go-graphviz` library being used is not thread-safe for this type of parallel execution.

The user has implemented a mutex (`-use_gmutex` flag) as a workaround, which successfully prevents the crash by serializing access to the `go-graphviz` library, at the cost of losing performance benefits from concurrency.

An issue has been filed on the library's GitHub repository: [fatal error: concurrent map writes with concurrent usage #117](https://github.com/goccy/go-graphviz/issues/117).

## Building and Running

1.  **Build the executable:**
    ```bash
    go build
    ```

2.  **Run the program:** The behavior is controlled by command-line flags.

    *   **To run sequentially (works):**
        ```bash
        ./go-graphviz-test -use_goroutines=false -ct 3
        ```

    *   **To run with goroutines and trigger the panic (fails):**
        ```bash
        ./go-graphviz-test -use_goroutines=true -use_gmutex=false -ct 3
        ```

    *   **To run with goroutines and the mutex workaround (works):**
        ```bash
        ./go-graphviz-test -use_goroutines=true -use_gmutex=true -ct 3
        ```

## Key Command-Line Flags

-   `-ct <int>`: The number of graphs to generate and process.
-   `-use_goroutines <bool>`: Whether to use goroutines for concurrent processing. Defaults to `true`.
-   `-use_gmutex <bool>`: Whether to use a mutex to serialize the `createSvg` function calls, preventing the race condition. Defaults to `true`.
-   `-file_type <string>`: The output file type (`svg` or `dot`). Defaults to `svg`.
-   `-minWidth`, `-maxWidth`, `-minDepth`, `-maxDepth`: Control the dimensions of the randomly generated graphs.

## Key Files

-   `main.go`: Contains all the application logic, including graph generation, command-line flag parsing, and the concurrent processing logic that demonstrates the bug.
-   `README.md`: Provides detailed history, context, and the user's own analysis of the problem.
-   `go.mod`: Defines the project dependencies, notably `github.com/goccy/go-graphviz`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lbe) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
