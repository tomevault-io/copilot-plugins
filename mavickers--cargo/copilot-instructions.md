## cargo

> Cargo is a lightweight Chain of Responsibility pipeline library for .NET.

Cargo is a lightweight Chain of Responsibility pipeline library for .NET.

## Build & Test

- `dotnet build Cargo.sln`
- `dotnet test Cargo.sln`
- `publish.bat -local [version]` — publish to local NuGet feed (`d:\Local Packages`)
- `publish.bat -nuget [version]` — publish to nuget.org
- NuGet package: https://www.nuget.org/packages/LightPath.Cargo

## Project Layout

- `Cargo/` — Main library (namespace: `LightPath.Cargo`, assembly: `LightPath.Cargo`)
- `Cargo.Tests/` — Test project (xUnit, FluentAssertions, Moq, coverlet)
- Both projects multi-target: net472, net48, net6.0, net7.0, net8.0

## Architecture Notes

- Pipeline supports both synchronous (`Go()`) and asynchronous (`GoAsync()`) execution.
- `Go()` throws `InvalidOperationException` if the pipeline contains async stations.
- Stations inherit from `Station<T>` (sync) or `StationAsync<T>` (async), both sharing `StationBase<T>`.

## Test Organization

- `Integration/` — Full pipeline scenario tests
- `Unit/` — Isolated component tests
- Station implementations for tests are nested classes within test files

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mavickers)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mavickers)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
