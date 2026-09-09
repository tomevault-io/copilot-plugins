## mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Testing
- Run all tests: `clojure -T:build test`
- Tests use `cognitect.test-runner` and are located in the `test/` directory

### Building
- Run CI pipeline (tests + build uberjar): `clojure -T:build ci`
- Built JAR location: `target/mcp-0.1.0-SNAPSHOT.jar`
- Run built JAR: `java -jar target/mcp-0.1.0-SNAPSHOT.jar`

### Running the Application
- Via exec function: `clojure -X:run-x` (default) or `clojure -X:run-x :name '"CustomName"'`
- Via main function: `clojure -M:run-m` (default) or `clojure -M:run-m CustomName`

### MCP Server
- Start MCP server (stdio): `clojure -M:mcp-server`
- Start MCP HTTP server: `clojure -M:mcp-http-server` (default port 3000) or `clojure -M:mcp-http-server 8080` (custom port)
- Both servers expose an `estimate_cost` tool that calculates token costs for text
- HTTP server accepts JSON-RPC requests via POST to the root endpoint

## Project Structure

This is a Clojure application using `tools.build` for build automation and `tools.deps` for dependency management.

### Key Components
- **Main namespace**: `achilles.mcp` - Contains the estimate-cost` function, and `-main` entry point
- **MCP core logic**: `achilles.mcp-core` - Transport-agnostic MCP protocol implementation shared by both servers
- **MCP stdio server**: `achilles.mcp-server` - Model Context Protocol server using stdin/stdout for communication
- **MCP HTTP server**: `achilles.mcp-http-server` - Model Context Protocol server using HTTP for communication
- **Build configuration**: `build.clj` - Defines test and CI pipeline functions using `clojure.tools.build.api`
- **Dependencies**: `deps.edn` - Standard tools.deps configuration with aliases for running, testing, building, and MCP servers

### Architecture
The project follows standard Clojure project conventions:
- Source code in `src/achilles/` 
- Tests in `test/achilles/` mirroring source structure
- Build automation in `build.clj` with separate test and CI functions
- The application can be invoked either as an executable function (`:exec-fn`) or traditional main (`:main-opts`)

### Build System
Uses `clojure.tools.build` with two key build functions:
- `test`: Runs test suite using cognitect test-runner
- `ci`: Full pipeline that runs tests, cleans target, compiles, and builds uberjar

---
> Source: [christianromney/mcp](https://github.com/christianromney/mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
