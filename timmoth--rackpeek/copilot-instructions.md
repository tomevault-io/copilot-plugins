## rackpeek

> just build              # dotnet build RackPeek.sln

# RackPeek — Agent Quick Reference

## Commands

**Build & Test**
```bash
just build              # dotnet build RackPeek.sln
just test-all           # CLI + E2E tests (rebuilds Web image)
just ci                 # alias for test-all (matches CI checklist)
just test-cli           # dotnet test Tests/Tests.csproj
just test-e2e           # requires just build-web first
```

**Run Locally**
```bash
just run-docker         # builds and starts Docker container on :8080
just rpk [args]         # run CLI directly from debug build
```

**E2E Setup** (first time only)
```bash
just e2e-setup          # installs Playwright CLI + browsers
```

**Demo**
```bash
just build-cli-demo     # VHS CLI demo (needs: vhs, imagemagick, chrome)
just build-web-demo     # Web UI demo (needs: Chrome, ImageMagick)
```

**Release**
```bash
just docker-push <ver>  # multi-arch Docker push (e.g., just docker-push 1.3.0)
```

## Workflow

1. **CI order**: `format → cli-tests → webui-tests`
2. **PR checklist**:
   - Linked GitHub issue
   - Approach validated with maintainers
   - Small, focused PR
   - CLI tests passing locally
   - E2E tests passing locally
   - YAML migration defined if persisting changes

3. **Draft PR** until:
   - All tests pass locally
   - Scope complete
   - Debug code removed

## Architecture

**Solution structure**:
- `RackPeek/` — CLI application
- `RackPeek.Domain/` — shared domain models
- `RackPeek.Web/` — Web UI (Blazor)
- `RackPeek.Web.Viewer/` — Web UI viewer
- `Shared.Rcl/` — shared Blazor components
- `Tests/` — CLI unit tests
- `Tests.E2e/` — Playwright E2E tests

**Key files**:
- `justfile` — developer workflow commands
- `.github/workflows/test.yml` — CI pipeline
- `RackPeek.sln` — solution root
- `docs/development/` — dev guides (dev-cheat-sheet.md, testing-guidelines.md)

## Gotchas

- **E2E tests require Docker image**: run `just build-web` before `just test-e2e`
- **Playwright browsers** installed via `just e2e-setup` (first time)
- **CI runs on `ubuntu-latest`** (CLI tests) and `ubuntu-24.04` (WebUI)
- **Docker image tag**: `rackpeek:ci` used locally, `aptacode/rackpeek` on registry
- **Format check**: `dotnet format --verify-no-changes` (CI step 1)
- **Debugging E2E**: Set `Headless = false, SlowMo = 500` in `PlaywrightFixture.cs`, revert before commit
- **YAML changes**: Always define migration if modifying persisted schema

## Testing

- **CLI tests**: `dotnet test Tests/Tests.csproj` (fast, no Docker)
- **E2E tests**: `dotnet test Tests.E2e` (requires Docker image, Playwright browsers)
- **Full suite**: `just ci` or `just test-all`

## Docs References

- `docs/development/contribution-guidelines.md` — PR process
- `docs/development/dev-cheat-sheet.md` — build/release details
- `docs/development/testing-guidelines.md` — testing principles
- `README.md` — overview and Docker usage

---
> Source: [Timmoth/RackPeek](https://github.com/Timmoth/RackPeek) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
