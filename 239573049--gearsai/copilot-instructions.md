## gearsai

> `GearsAI.slnx` is the solution entry point. Shared .NET settings live in `Directory.Build.props`, including multi-targeting for `net8.0`, `net9.0`, and `net10.0`, nullable reference types, implicit usings, and latest C#.

# Repository Guidelines

## Project Structure & Module Organization

`GearsAI.slnx` is the solution entry point. Shared .NET settings live in `Directory.Build.props`, including multi-targeting for `net8.0`, `net9.0`, and `net10.0`, nullable reference types, implicit usings, and latest C#.

Source code is under `src/GearsAI/`. Public abstractions and request/response types sit near the root and in `Abstractions/`, `Requests/`, and `Responses/`. Provider-specific models are in `Models/OpenAi/` and `Models/Claude/`; lower-level HTTP and SSE helpers are in `Internal/`; streaming event contracts are in `Streaming/`.

Tests are under `tests/`. `GearsAI.Tests` contains MSTest unit tests using fake HTTP handlers. `GearsAI.AotSmoke` is a small executable used to validate AOT-compatibility assumptions.

## Build, Test, and Development Commands

- `dotnet restore GearsAI.slnx` restores NuGet packages.
- `dotnet build GearsAI.slnx -c Release` builds the library across configured target frameworks.
- `dotnet test tests/GearsAI.Tests/GearsAI.Tests.csproj -c Release` runs the MSTest suite.
- `dotnet run --project tests/GearsAI.AotSmoke/GearsAI.AotSmoke.csproj -c Release` runs the AOT smoke executable.
- `dotnet format GearsAI.slnx` applies SDK formatting when broad edits are made.

## Coding Style & Naming Conventions

Use C# with file-scoped namespaces, nullable-aware APIs, and explicit public surface design. Indent C# with four spaces. Prefer `PascalCase` for types, methods, and public properties; use `_camelCase` for private fields. Keep async methods suffixed with `Async`.

The library uses `System.Text.Json` models heavily. Prefer typed request/response objects and existing `JsonValues` helpers over manual string construction. Keep provider-specific behavior isolated in the relevant OpenAI or Claude model/client files.

## Testing Guidelines

Use MSTest attributes: `[TestClass]` and `[TestMethod]`. Existing test names use descriptive underscore-separated method names, for example `OpenAi_chat_base_uri_owns_v1_prefix`; follow that pattern for behavioral tests.

Prefer deterministic unit tests with `FakeHttpMessageHandler` over live provider calls. Cover serialization, deserialization, URI construction, streaming event mapping, and unified `AiClient` provider routing when touching protocol behavior.

## Commit & Pull Request Guidelines

This repository currently has no commit history, so use concise imperative commit messages such as `Add Claude streaming usage mapping`. Keep each commit focused.

Pull requests should include a short problem/solution summary, tests run, and any compatibility impact across `net8.0`, `net9.0`, `net10.0`, trimming, or AOT. Link related issues when available and include request/response examples for public API changes.

## Security & Configuration Tips

Do not commit API keys, real provider responses containing customer data, or local IDE caches. Inject secrets through environment variables, user secrets, or test-only configuration outside the repository. Keep networked integration tests opt-in; unit tests should remain offline by default.

---
> Source: [239573049/GearsAI](https://github.com/239573049/GearsAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
