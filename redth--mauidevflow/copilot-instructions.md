## mauidevflow

> MauiDevFlow provides AI-assisted debugging and automation for .NET MAUI apps (native and Blazor Hybrid). It consists of an in-app agent, a Blazor CDP bridge, a CLI tool, and a driver library.

# Copilot Instructions for MauiDevFlow

## Project Context

MauiDevFlow provides AI-assisted debugging and automation for .NET MAUI apps (native and Blazor Hybrid). It consists of an in-app agent, a Blazor CDP bridge, a CLI tool, and a driver library.

## Code Style

- C# with nullable reference types enabled and implicit usings
- Target frameworks: `net10.0-android`, `net10.0-ios`, `net10.0-maccatalyst`, `net10.0-windows10.0.19041.0` for MAUI libraries; `net9.0` for the CLI tool
- Minimal comments — only clarify non-obvious logic
- Use `record` types for data transfer objects (e.g., `FileLogEntry`)
- Platform-specific code uses `#if ANDROID`, `#if IOS || MACCATALYST`, `#if WINDOWS`, etc.

## Architecture

- **Single port** HTTP server inside the MAUI app (no WebSocket) serves both native and CDP commands
- **Blazor→Agent** wiring uses reflection (no direct package dependency)
- **Chobitsu** (JS CDP implementation) runs in BlazorWebView; commands sent via `EvaluateJavaScriptAsync`
- **Chobitsu fires onMessage asynchronously** — CDP needs two JS evals: send + poll for response
- JS scripts are embedded resources in `Resources/Scripts/`, loaded via `ScriptResources.Load()`

## Building

```bash
dotnet build ci.slnf           # builds everything except sample app
dotnet test ci.slnf             # runs unit/integration tests
dotnet build src/SampleMauiApp -f net10.0-maccatalyst  # sample app
```

## Important Patterns

- Agent/Blazor packages are `#if DEBUG` only — never ship in production
- `.mauidevflow` JSON file configures the port; read by MSBuild targets and CLI
- Mac Catalyst apps need `com.apple.security.network.server` entitlement
- Android needs `adb reverse` for port forwarding
- Windows apps connect directly on localhost (no special setup needed)
- File logging uses rotating JSONL files with compact property names (`t`, `l`, `c`, `m`, `e`, `s`)
- WebView console logs are intercepted in JS, buffered, and drained to native ILogger every 2s

## Testing

Always verify features with the sample app (`src/SampleMauiApp`). Mac Catalyst is the fastest target for local testing. The sample app has both native MAUI and Blazor Hybrid pages via Shell navigation.

## Release Process

1. Update version numbers
2. Commit and push to `main`
3. Create GitHub release with tag `vX.Y.Z` at https://github.com/Redth/MauiDevFlow
4. The `publish.yml` workflow automatically builds, tests, packs, and pushes NuGet packages

---
> Source: [Redth/MauiDevFlow](https://github.com/Redth/MauiDevFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
