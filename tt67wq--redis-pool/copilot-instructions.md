## redis-pool

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RedisPool is an Elixir library that provides a Redis connection pool wrapper built on top of NimblePool and Redix. It offers a simple macro-based API for creating Redis connection pools with automatic connection management, health checks, and comprehensive error handling.

## Common Development Commands

### Build and Compilation
```bash
# Initialize development environment (install deps and compile)
make setup

# Compile project
make build
mix compile

# Format code
make fmt
mix format

# Check code formatting
make lint
mix format --check-formatted
```

### Testing
```bash
# Run unit tests
make test
mix test

# Run tests in watch mode
make test.watch
mix test.watch

# Run integration tests (requires Docker)
./test/run_integration_tests.sh standalone    # Redis standalone tests
./test/run_integration_tests.sh network        # Network condition tests
./test/run_integration_tests.sh all           # All integration tests
./test/run_integration_tests.sh full          # Full cycle: start, test, stop

# Manage Docker test environment
./test/run_integration_tests.sh start          # Start test environment
./test/run_integration_tests.sh stop           # Stop test environment
./test/run_integration_tests.sh clean          # Clean test environment
```

### Dependency Management
```bash
# Get dependencies
make deps.get
mix deps.get

# Update dependencies
make deps.update package_name
mix deps.update package_name

# Update all dependencies
make deps.update.all
mix deps.update --all

# Clean unused dependencies
make deps.clean
mix deps.clean --unused

# Show dependency tree
make deps.tree
mix deps.tree
```

### Development Tools
```bash
# Start interactive shell
make repl
iex -S mix

# Type checking
mix dialyzer

# Code quality analysis
mix credo

# Generate documentation
mix docs
```

## Architecture Overview

### Core Module Structure

The library is organized around three main modules:

1. **`RedisPool`** - The main public interface module that provides macros for creating Redis pool modules
2. **`RedisPool.Core`** - The core implementation that handles NimblePool integration and Redis operations  
3. **`RedisPool.Error`** - Comprehensive error handling with structured error types

### Key Design Patterns

**Macro-based Module Generation**: Users create Redis pool modules using the `use RedisPool, otp_app: :my_app` macro, which generates a complete module with supervision capabilities.

**NimblePool Integration**: The library uses NimblePool for efficient connection management, providing automatic checkout/checkin, connection health checks via PING commands, and graceful connection recovery.

**Error Hierarchy**: All errors are structured as `RedisPool.Error` with specific error codes:
- `:connection_error` - Redis connection failures
- `:command_error` - Redis command execution errors
- `:timeout_error` - Operation timeouts
- `:network_error` - Network communication issues
- `:authentication_error` - Redis auth failures
- `:pool_error` - Connection pool management errors
- `:unknown_error` - Unclassified errors

### Configuration System

The library supports both static configuration (via config files) and runtime configuration:

```elixir
# Static configuration
config :my_app, MyApp.Redis,
  url: "redis://:password@localhost:6379/0",
  pool_size: 10

# Runtime configuration  
config = [url: "redis://localhost:6379", pool_size: 5]
Application.put_env(:my_app, MyApp.Redis, config)
```

### API Design

The main API provides two core operations:

```elixir
# Single command execution
{:ok, result} = MyApp.Redis.command(["SET", "key", "value"])

# Pipeline command execution (for multiple commands)
{:ok, results} = MyApp.Redis.pipeline([
  ["SET", "key1", "value1"],
  ["SET", "key2", "value2"]
])
```

### Testing Architecture

**Unit Tests**: Standard ExUnit tests for core functionality
**Integration Tests**: Docker-based test environment with multiple Redis instances:
- Redis 6.2 on port 6379
- Redis 7.0 on port 6380  
- Redis with network simulation on port 6384

**Test Environment Management**: The `test/support/integration/` directory provides comprehensive test environment setup including:
- Docker container management
- Network condition simulation (latency, packet loss)
- Redis failure simulation
- Environment health checks

## Development Guidelines

### When Adding New Features

1. **Error Handling**: Always use the structured error types from `RedisPool.Error` module. Use the provided helper functions like `Error.connection_error/2`, `Error.command_error/2`, etc.

2. **Configuration**: Follow the existing pattern of using NimbleOptions for configuration validation. Add new options to the appropriate schema.

3. **Documentation**: Include comprehensive @moduledoc, @doc, and @spec annotations. All public functions must have typespecs.

4. **Testing**: Add unit tests for new functionality. For features that require Redis interaction, add integration tests using the Docker-based test environment.

### When Debugging Issues

1. **Connection Issues**: Check the Docker test environment is running with `./test/run_integration_tests.sh start`. Use the test helper functions to verify Redis connectivity.

2. **Error Investigation**: Use the structured error messages that include error codes and reasons. The `RedisPool.Error.message/1` function provides detailed error information.

3. **Performance Analysis**: For performance issues, use pipeline operations instead of individual commands for bulk operations. Monitor connection pool utilization.

### When Working with Tests

1. **Integration Tests**: Always start the Docker environment before running integration tests. The test helper automatically verifies environment readiness.

2. **Test Data Management**: Use the `clean_test_data()` functions provided in test modules to ensure test isolation.

3. **Network Simulation**: The test environment supports network condition simulation for testing resilience:
   ```elixir
   RedisPool.Test.Integration.TestHelper.simulate_network_issue(:delay, "100ms 20ms")
   RedisPool.Test.Integration.TestHelper.simulate_network_issue(:loss, "10%")
   ```

## Dependencies and Technologies

### Core Dependencies
- **Redix** `~> 1.5` - Redis client library
- **NimblePool** `~> 1.1` - Connection pool implementation
- **NimbleOptions** `~> 1.1` - Configuration validation

### Development Dependencies  
- **Dialyxir** - Static type checking
- **Credo** - Code quality analysis
- **Styler** - Code formatting
- **ExDoc** - Documentation generation

### Elixir Version Requirements
- Requires Elixir `~> 1.16`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tt67wq) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
