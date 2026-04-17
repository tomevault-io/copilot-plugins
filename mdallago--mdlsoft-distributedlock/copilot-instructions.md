## mdlsoft-distributedlock

> **MDLSoft.DistributedLock** is a lightweight, high-performance distributed lock library for SQL Server with zero external dependencies. It is designed to work across a wide range of .NET versions, from .NET Framework 4.0 to .NET 8+.

# MDLSoft.DistributedLock Project Context

## Project Overview
**MDLSoft.DistributedLock** is a lightweight, high-performance distributed lock library for SQL Server with zero external dependencies. It is designed to work across a wide range of .NET versions, from .NET Framework 4.0 to .NET 8+.

### Key Features
- **Distributed Locking:** Coordinate access to resources across multiple processes/applications.
- **SQL Server Backend:** Uses a simple SQL table for lock management.
- **Zero Dependencies:** Relies only on `System.Data.SqlClient` (Framework) or `Microsoft.Data.SqlClient` (Core/Standard).
- **Multi-Targeting:** Supports `net40`, `net451`, `net461`, `net48`, and `netstandard2.0`.
- **Sync & Async:** Full support for synchronous and asynchronous operations.
- **User Context:** Optionally store context data (e.g. user, machine) with each lock.

## Architecture & Structure
The solution is organized into the following key projects:

- **`MDLSoft.DistributedLock`**: The core library project. It is multi-targeted to ensure broad compatibility.
- **`MDLSoft.DistributedLock.Tests`**: Unit and integration tests using xUnit and AwesomeAssertions. Targets .NET 8.
- **`MDLSoft.DistributedLock.Example`**: A modern .NET 8 example application demonstrating async usage.
- **`MDLSoft.DistributedLock.Example.Net40`**: A legacy .NET Framework 4.0 example application demonstrating synchronous usage.

## Development Environment

### Prerequisites
- **.NET SDK:** .NET 8.0 SDK is recommended for building and running tests.
- **SQL Server:** A local SQL Server instance (e.g., LocalDB, Express, or Developer edition) is required for running tests and examples.

### Building the Project
Use the standard .NET CLI commands from the root directory:

```bash
# Restore dependencies
dotnet restore

# Build the solution (Release configuration recommended for NuGet packing)
dotnet build --configuration Release
```

### Running Tests
The tests utilize xUnit and require a SQL Server connection. Ensure your local environment is configured or update the connection string in the test project if necessary.

```bash
# Run all tests
dotnet test
```

*Note: The CI pipeline sets up a SQL Server instance automatically. Locally, you may need to adjust connection strings in the test files if not using a standard local setup.*

## Development Conventions

### Coding Style
- **Naming:** Follow standard C# naming conventions (PascalCase for classes/methods, camelCase for local variables).
- **Async:** Asynchronous methods must end with the `Async` suffix.
- **Formatting:** The project uses an `.editorconfig` file to enforce consistent coding styles.
- **Nullability:** The project uses nullable reference types (`<Nullable>enable</Nullable>`) where supported (e.g., .NET 8 contexts).

### Contribution Workflow
1.  **Branching:** Create feature branches from `main` or `develop`.
2.  **Testing:** Ensure all tests pass before submitting a PR. Add new tests for new features.
3.  **Documentation:** Update the `README.md` if public APIs or behaviors change.

### Release Process
The project is set up to automatically pack NuGet packages on build in the Release configuration. The CI/CD pipeline handles publication to NuGet.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mdallago) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
