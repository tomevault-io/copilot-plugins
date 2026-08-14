## ezmock

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

EzMock is a CLI-based API mocking tool built with Bun and TypeScript. It allows developers to quickly spin up mock API servers using JSON configuration files or interactive prompts.

## Running the Project

```bash
# Start with interactive setup
bun start

# Start with configuration file
bun start -c example.json
bun start -c configs/my-mock.json

# Development mode (auto-reload on changes)
bun run dev

# Override port or host
bun start -c config.json -p 8080
bun start -c config.json -h 0.0.0.0

# Enable request logging at startup
bun start -c config.json --logging

# Run tests
bun test
```

## Building

```bash
# Compile binary for current platform
bun run build

# Compile for specific platforms
bun run build:macos
bun run build:linux
bun run build:windows
bun run build:all

# Run the compiled binary
./ezmock -c example.json
```

## Architecture

The codebase is organized into three main components:

### 1. Entry Point (`index.ts`)
- CLI argument parsing using Node's `parseArgs`
- Interactive prompts using `@inquirer/prompts`
- Orchestrates the flow: load config → create server → start

### 2. Configuration Layer (`src/config-manager.ts`)
- Searches for config files in order: specified path → `ezmock.json` → `mock.json` → `.ezmockrc.json`
- Validates and merges configurations with defaults
- Handles saving user-created configurations

### 3. Server Layer (`src/mock-server.ts`)
- Built on `Bun.serve()` native HTTP server
- Route matching with parameterized paths (e.g., `/api/users/:id`)
- Supports four response types: `json`, `text`, `file`, `stream`
- Stream responses use `ReadableStream` API with configurable chunk delays
- Two streaming modes: SSE (Server-Sent Events) with `text/event-stream` and raw chunks
- Optional response delays, custom headers, CORS, and request logging

### Type Definitions (`src/types.ts`)
- `MockRoute`: Individual route configuration
- `MockConfig`: Server-level configuration including routes array

## Configuration File Structure

Config files are typically stored in:
- `configs/` directory for user-generated configs (gitignored)
- Root directory for example configs (e.g., `example.json`, `stream-example.json`)

Config files must include:
- `port` and `host` for server binding
- `routes` array with route definitions
- Optional: `cors` (boolean)

Each route requires:
- `method`: HTTP verb (GET, POST, PUT, DELETE, PATCH)
- `path`: Route path, supports params like `/:id`
- `response`: The response content (object, string, or file path)
- `type`: Response type (`json`, `text`, `file`, or `stream`)
- Optional: `statusCode`, `delay` (ms), `headers` (object)
- For streams: `stream` config with `chunks` array, `chunkDelay` (ms), and `mode` (`line` or `chunk`)

## Key Implementation Details

### Route Matching
Routes are matched by exact method comparison and path segment comparison. Path segments starting with `:` are treated as wildcards (e.g., `:id`, `:userId`).

### Response Handling
- JSON responses are automatically stringified
- File responses use `Bun.file()` and check existence before serving
- All responses support custom headers and respect CORS settings
- Delays are implemented with `setTimeout` promises

### CLI Workflow
1. Parse command-line arguments
2. Load config from file or prompt for interactive creation
3. Allow user to save interactively created configs
4. Override config values with CLI arguments (port/host) and apply runtime logging options
5. Initialize and start the MockServer

## Development Notes

- The project uses Bun's native APIs (`Bun.serve`, `Bun.file`, `Bun.write`)
- TypeScript strict mode is enabled
- The CLI is executable via shebang: `#!/usr/bin/env bun`
- Uses ESM modules (`type: "module"` in package.json)

---
> Source: [FinleyGe/ezmock](https://github.com/FinleyGe/ezmock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
