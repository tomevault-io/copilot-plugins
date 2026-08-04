## vppspy

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project purpose

VppSpy is intended to be a Virtual Power Plant (VPP) metrics importer for Prometheus. The codebase is
currently just the default ASP.NET Core Minimal API template (`net10.0`) — the `/weatherforecast` endpoint
in `src/VppSpy/Program.cs` is template placeholder code, not real functionality. There is no VPP or
Prometheus integration implemented yet.

## Repository layout

- The actual .NET solution lives under `src/` (`src/VppSpy.sln`, `src/compose.yaml`), not the repo root.
- `src/VppSpy/` is the single ASP.NET Core Web API project (`Microsoft.NET.Sdk.Web`).
- All endpoints are currently registered directly in `src/VppSpy/Program.cs` (Minimal API style — no
  Controllers directory exists yet). If/when the project grows real endpoints, follow this same
  minimal-API, top-level-statements pattern unless there's a specific reason to switch to controllers.

## Common commands

Run all commands from `src/` (where the `.sln` file is).

```bash
# Build
dotnet build

# Run locally (http profile, see Properties/launchSettings.json)
dotnet run --project VppSpy

# Run via Docker Compose (builds image from VppSpy/Dockerfile)
docker compose up --build
```

There are no test projects in the solution yet.

- Local run listens on `http://localhost:5225` (see `VppSpy/Properties/launchSettings.json`).
- `VppSpy/VppSpy.http` contains example HTTP requests for use with the JetBrains/VS Code REST client.
- OpenAPI is enabled only in the Development environment (`app.MapOpenApi()` gated on
  `app.Environment.IsDevelopment()` in `Program.cs`).

## Docker

`src/VppSpy/Dockerfile` is a standard multi-stage build (SDK image builds/publishes, `aspnet` image runs).
It expects to be built with `src/` as the context (e.g. `docker compose build` from `src/`, which is what
`compose.yaml` does) because it `COPY`s `VppSpy/VppSpy.csproj` relative to that context.

---
> Source: [joeldroid/vppspy](https://github.com/joeldroid/vppspy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-01 -->
