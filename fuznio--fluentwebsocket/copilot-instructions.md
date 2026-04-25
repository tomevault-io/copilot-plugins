## fluentwebsocket

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Fuzn.FluentWebSocket is a .NET library providing a fluent API over `System.ClientWebSocket`. It is published as a NuGet package (`Fuzn.FluentWebSocket`). Targets .NET 10.

## Build & Test Commands

```bash
# Build
dotnet build src

# Run all tests
dotnet run --project src/Tests

# Run a single test by name
dotnet run --project src/Tests -- --filter "FullyQualifiedName~Fuzn.FluentWebSocket.Tests.ClassName.MethodName"

# Pack NuGet package
dotnet pack src/FluentWebSocket/FluentWebSocket.csproj --configuration Release
```

Solution file: `src/FluentWebSocket.slnx`. All dotnet commands use `src` as the path argument.

## Project Structure

- **src/FluentWebSocket/** - Main library (NuGet package: `Fuzn.FluentWebSocket`)
- **src/Tests/** - Integration tests using MSTest + TestFuzn adapter
- **src/TestWsServer/** - ASP.NET Core minimal API WebSocket server for tests

## Architecture

The library provides a fluent builder API over `ClientWebSocket` for configuring and managing WebSocket connections.

**Request flow:** `ClientWebSocket` -> `ClientWebSocketExtensions.Request()`/`.Url()` -> `FluentWebSocketRequest` (builder with chaining) -> `.Connect()` -> `FluentWebSocketConnection`

Key types:
- **FluentWebSocketRequest** - Core builder. Holds `FluentWebSocketRequestData` internally. All `With*()` methods configure the request; `.Connect()` establishes the connection.
- **FluentWebSocketConnection** - Live connection with `SendText`, `SendBinary`, `Send<T>` (JSON), `Receive`, `Listen` (IAsyncEnumerable), `OnMessage`, and `Close`. Implements `IAsyncDisposable`.
- **FluentWebSocketMessage / FluentWebSocketMessage\<T>** - Message wrapper with `ContentAs<T>()` for deserialization. Generic version has lazy `Data` property.
- **FluentWebSocketDefaults** - Static global defaults (`Serializer`, `ReceiveBufferSize`, `MaxMessageSize`, `LoggerFactory`).
- **ISerializerProvider** - Interface for pluggable serialization (default: `SystemTextJsonSerializerProvider`).

Serializer resolution order: per-request `WithSerializer()` > `FluentWebSocketDefaults.Serializer`.

## Coding Standards

- XML documentation comments on all public members; none on private/internal
- File-scoped namespaces
- Private fields prefixed with underscore (`_fieldName`)
- Use `ArgumentNullException.ThrowIfNull()` for null validation
- Only public API surfaces should be public; everything else internal or private
- Beta version: breaking changes are acceptable

## Documentation

Always update `README.md` when adding new features or changing existing functionality.

## Testing Conventions

- Framework: MSTest with TestFuzn adapter (`Fuzn.TestFuzn.Adapters.MSTest`)
- Test classes inherit from `Test` base class
- Use `[TestClass]` on classes and `[Test]` on methods (not `[TestMethod]`)
- Pattern: `Scenario().Step().Run()` from TestFuzn
- Shared state via `SuiteData` static class (provides `WsBaseUrl`)
- Test server endpoints are defined in `SuiteData.cs`

## Test Infrastructure

`SuiteData` spins up an in-process ASP.NET WebSocket server on a random port. Test endpoints:
- `/ws/echo` - echo messages back
- `/ws/json` - parse action field, respond with structured JSON
- `/ws/disconnect-after/{count}` - echo N messages then close
- `/ws/large-message/{sizeKb}` - send a message of specified size
- `/ws/query-echo` - respond with the query string
- `/ws/silent` - accept but never send (timeout tests)
- `/ws/slow-echo/{delayMs}` - echo with delay

All tests connect via `SuiteData.WsBaseUrl`.

## CI/CD

NuGet publish is manual (`workflow_dispatch`). Version is in `src/Directory.Build.props`. The workflow creates a git tag, builds, packs, pushes to NuGet, and creates a GitHub Release.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/FuznIO) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
