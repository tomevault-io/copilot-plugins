## pengdows-hangfire

> A high-performance Hangfire storage implementation built on top of the `pengdows.crud` data access layer.

# pengdows.hangfire

A high-performance Hangfire storage implementation built on top of the `pengdows.crud` data access layer.

## Project Overview

`pengdows.hangfire` provides a SQL-first, strongly-typed storage backend for Hangfire. It leverages `pengdows.crud` for efficient database operations, connection management, and multi-dialect support.

### Key Technologies
- **.NET 8.0**
- **Hangfire.Core**: The core Hangfire library.
- **pengdows.crud**: A SQL-first data access layer providing `TableGateway`, `SqlContainer`, and advanced dialect management.
- **SQLite**: Primary target for integration testing (via `Microsoft.Data.Sqlite`).

### Architecture
The project follows a modular gateway-based architecture where each Hangfire storage concern is isolated into its own gateway class:
- `JobGateway`: Manages background jobs and their states.
- `JobQueueGateway`: Handles job queuing and fetching.
- `ServerGateway`: Manages Hangfire server instances and heartbeats.
- `HashGateway`, `SetGateway`, `ListGateway`: Implement Hangfire's primitive storage types.
- `CounterGateway`, `AggregatedCounterGateway`: Handle statistics and counters.

The main entry point is `PengdowsCrudJobStorage`, which orchestrates these gateways and integrates with the Hangfire pipeline.

## Building and Running

### Prerequisites
- .NET 8.0 SDK or later.
- `pengdows.crud` source code must be available in the sibling directory `../pengdows.crud/` as per the project reference in `pengdows.hangfire.csproj`.

### Key Commands
- **Build**: `dotnet build`
- **Test**: `dotnet test` (Runs both unit and integration tests)
- **Pack**: `dotnet pack -c Release`

## Development Conventions

All contributions must strictly adhere to the standards defined in `REVIEW_POLICY.md`.

### SQL & Database
- **No String Interpolation**: Never use string interpolation for SQL values. Use `SqlContainer.AddParameterWithValue` or `AppendParam`.
- **Dialect Awareness**: Use `Context.Dialect` methods (e.g., `AppendPaging`, `WrapObjectName`) to ensure cross-database compatibility.
- **Explicit Identifiers**: Use `WrapObjectName` or `AppendName` for all table and column names in manual SQL.
- **Transaction Safety**: `TransactionScope` is strictly forbidden. Use `Db.BeginTransactionAsync()` or the Hangfire-provided `IStorageTransaction` implementation.

### Code Style
- **Braces Required**: Braces are mandatory for all control flow blocks (`if`, `else`, `foreach`, `while`), even for single-line statements.
- **One Statement Per Line**: Never combine multiple statements on a single line (e.g., `if (condition) return;` must be two lines with braces).
- **Explicit Parentheses**: Use parentheses to make expression intent explicit, even when operator precedence rules would suffice.
- **Async Efficiency**: Use `ValueTask` or `ValueTask<T>` for async methods in hot paths to minimize allocations.

### Testing (TDD)
- **Unit Tests**: Required for all new logic and bug fixes.
- **Integration Tests**: Mandatory for any changes affecting database behavior, SQL generation, or transactions.
- **SQLite Fixture**: Use `SqliteStorageFixture` for integration tests to ensure a clean, isolated database environment for each test collection.

### Resource Management (Lampson Rule)
- Ownership of `DbConnection` and `IDataReader` must be explicit.
- Always use `await using` for `SqlContainer` and other disposable resources to ensure deterministic cleanup.
- Connections and transactions must not escape their intended execution scope.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pengdows) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
