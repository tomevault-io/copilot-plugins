## testcontainers-zig

> Guidelines for AI agents working in the **testcontainers-zig** repository.

# AGENTS.md

Guidelines for AI agents working in the **testcontainers-zig** repository.

## Project Overview

Zig library for managing throwaway Docker containers in tests, inspired by
[testcontainers-go](https://github.com/testcontainers/testcontainers-go).
Single module: `testcontainers` (Docker HTTP client over unix socket, container
lifecycle, wait strategies, and 10 pre-configured modules).

See [ARCHITECTURE.md](ARCHITECTURE.md) for design decisions and component diagrams.
See [PROJECT_SUMMARY.md](PROJECT_SUMMARY.md) for a full feature inventory.

## Build & Test

```bash
zig build                                  # Build all targets
zig build test --summary all               # Unit tests (no Docker required)
zig build integration-test --summary all   # Integration tests (requires Docker)
zig build example                          # Run examples/basic.zig
```

There is no separate formatter or linter. Follow the style rules below and let
`zig fmt` (built into the compiler) handle formatting.

## Code Style

- Zig 0.15.2, Zig Build System (`build.zig` + `build.zig.zon`).
- 4-space indentation, ~120-character line limit.
- `snake_case` for functions, variables, and fields; `PascalCase` for types (structs, unions, enums).
- `SCREAMING_SNAKE_CASE` for comptime constants.
- All public declarations must have `///` doc comments.
- Use `errdefer` to release resources in error paths.
- Propagate errors explicitly — never use `try!` or `catch unreachable` in production paths.
- Every function that allocates memory must accept `std.mem.Allocator`; document ownership of all returned slices.
- Follow the rules in [CONTRIBUTING.md](CONTRIBUTING.md) for commit messages and PR conventions.

## Architecture Rules

- **Single library**: all code lives under `src/`; no external dependencies (uses built-in HTTP/1.1 client over Unix domain socket).
- **Tagged unions over protocols**: `wait.Strategy` is a tagged union, not an interface.
- **Struct-literal configuration**: `ContainerRequest` uses default field values;
  no builder methods or method chaining.
- **Module pattern**: each `src/modules/<name>.zig` exposes `default_image`, `Options`,
  `run()`, `runDefault()`, and appropriate connection helpers (`connectionString`,
  `httpURL`, `brokers`, `endpointURL`).
- **Deterministic cleanup**: always use `defer ctr.terminate() catch {}` and `defer ctr.deinit()`.

See [ARCHITECTURE.md](ARCHITECTURE.md) for diagrams and rationale.

## Key Paths

| Area | Path |
|------|------|
| Build manifest | `build.zig` + `build.zig.zon` |
| Public API | `src/root.zig` |
| Docker HTTP client | `src/docker_client.zig` |
| Container handle | `src/docker_container.zig` |
| Container config types | `src/container.zig` |
| Wait strategies | `src/wait.zig` |
| Network management | `src/network.zig` |
| Docker API JSON types | `src/types.zig` |
| Pre-configured modules | `src/modules/` |
| Integration tests | `src/integration_test.zig` |
| Example | `examples/basic.zig` |
| CI workflow | `.github/workflows/ci.yml` |

## Common Agent Tasks

### Adding a new container module

1. Create `src/modules/<name>.zig` following the existing `src/modules/postgres.zig` pattern.
2. Export it inside `pub const modules = struct { ... }` in `src/root.zig`:
   ```zig
   pub const <name> = @import("modules/<name>.zig");
   ```
3. Pre-configure: `default_image`, `Options` struct with defaults, wait strategy.
4. Expose a connection helper (`connectionString`, `httpURL`, `brokers`, or `endpointURL`) that returns a caller-owned `[]u8`.
5. Add a `test { ... }` block in the module file.
6. Add an end-to-end integration test in `src/integration_test.zig`.
7. Update the module table in `README.md`, `PROJECT_SUMMARY.md`, and `IMPLEMENTATION_GUIDE.md`.

### Adding a new wait strategy

1. Add a new payload struct and a new field to the `Strategy` union in `src/wait.zig`.
2. Add a `pub fn forXxx(...)` constructor in the `wait` namespace.
3. Handle the new tag in the strategy dispatch `switch` in `src/docker_container.zig`.
4. Add a test exercising the new strategy in `src/integration_test.zig`.

### Updating CI/CD

Workflow files live in `.github/workflows/`. The pipeline runs `zig build test` and
`zig build integration-test` on both Linux and macOS.

## Testing Expectations

- Every new public API must have at least one test.
- Tests are plain `test { ... }` blocks using `std.testing`.
- Integration tests that require Docker must check reachability and return `error.SkipZigTest` when Docker is unavailable.
- Always clean up containers with `defer ctr.terminate() catch {}` and `defer ctr.deinit()`.
- Run `zig build test --summary all` before every commit.

## Additional Best Practices

- Never unwrap optional or error types without handling; prefer `orelse` / `catch` with explicit fallback.
- Use `errdefer` to undo partial initialisation in error paths.
- Pass `std.mem.Allocator` explicitly; document which return values are caller-owned.
- Do not hardcode the Docker socket path — use `tc.docker_socket` or honour `DOCKER_HOST`.
- In test code, create a fresh `DockerProvider` per test function, not a shared global.
- When building `ContainerRequest`, prefer struct-literal syntax with named fields over any builder pattern.
- Keep module wrappers thin — they should build a `ContainerRequest` and delegate to `DockerProvider.runContainer`, not reimplement lifecycle logic.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dragosv)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dragosv)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
