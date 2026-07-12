## jikan-net

> Jikan.net is a .NET wrapper for the Jikan REST API. The public package is `JikanDotNet`, and the main goal is to expose strongly typed, asynchronous access to MyAnimeList data returned by Jikan.

# AGENTS.md

## Purpose

Jikan.net is a .NET wrapper for the Jikan REST API. The public package is `JikanDotNet`, and the main goal is to expose strongly typed, asynchronous access to MyAnimeList data returned by Jikan.

## Start Here

- `README.md` explains the project, package installation, supported API areas, and changelog pointers.
- `docs/README.md` is the documentation index and contributor guide for public API docs.
- `JikanDotNet.slnx` contains the library project and the xUnit test project.
- `JikanDotNet/Jikan.cs` contains the endpoint implementations.
- `JikanDotNet/Interfaces/IJikan.cs` defines the public client surface.
- `JikanDotNet/Model/Response`, `JikanDotNet/Model/Base`, `JikanDotNet/Model/Search`, and `JikanDotNet/Enumerations` hold public response models, wrappers, search configs, and enums.
- `JikanDotNet.Test` contains xUnit tests. Many tests call the live public Jikan API and may be network-sensitive.

## Repository Rules

- Preserve public API compatibility unless the task explicitly asks for a breaking change.
- Keep `IJikan` and `Jikan` aligned when adding or changing client methods.
- Validate public inputs with `Guard` before building endpoint routes.
- Prefer `JikanEndpointConsts` for route segments instead of repeating string literals.
- Response models should use `System.Text.Json.Serialization.JsonPropertyName` for JSON field mapping.
- Keep XML documentation on public types and members meaningful because the package emits documentation files.
- In C# code, use braces for `if` and `else` blocks, including single-line branches.
- Follow the style of the file being edited. This repo currently has a mix of block-scoped and file-scoped namespaces; do not reformat unrelated files.

## Docs Rule

Canonical docs for this repo are `README.md`, `docs/README.md`, `docs/API.md`, `docs/Models.md`, `docs/GettingStarted.md`, `docs/RateLimiting.md`, and `docs/Migrating-from-1.x-to-2.0.md`.

If a change touches user-facing behavior, a public API, package setup, rate limiting, migration behavior, or anything already documented, update the relevant canonical doc in the same change.

When adding a new endpoint, update `docs/API.md` with the method description, parameters, return type, example, and model link. When adding or changing public response models, update `docs/Models.md`.

## Verification

Use the smallest verification that matches the change:

```bash
dotnet build JikanDotNet.slnx
```

```bash
dotnet test JikanDotNet.Test/JikanDotNet.Tests.csproj
```

For targeted test runs, prefer a filter, for example:

```bash
dotnet test JikanDotNet.Test/JikanDotNet.Tests.csproj --filter FullyQualifiedName~GetAnimeAsyncTests
```

Before relying on full test results, remember that many tests hit the live Jikan API and can fail because of network, upstream data, or rate limiting rather than local code.

---
> Source: [Ervie/jikan.net](https://github.com/Ervie/jikan.net) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
