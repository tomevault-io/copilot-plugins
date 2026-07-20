## exredis

> The OTP application code lives in `lib/`. `lib/exredis.ex` exposes the high-level Redis API that wraps commands via helper macros defined in `lib/exredis/helper.ex`. Supervisor wiring sits in `lib/exredis/application.ex`. Runtime configuration defaults are in `config/config.exs`, with test overrides in `config/test.exs`. ExUnit tests reside in `test/`, mirroring the public API they exercise.

# Repository Guidelines

## Project Structure & Module Organization
The OTP application code lives in `lib/`. `lib/exredis.ex` exposes the high-level Redis API that wraps commands via helper macros defined in `lib/exredis/helper.ex`. Supervisor wiring sits in `lib/exredis/application.ex`. Runtime configuration defaults are in `config/config.exs`, with test overrides in `config/test.exs`. ExUnit tests reside in `test/`, mirroring the public API they exercise.

## Build, Test, and Development Commands
- `mix deps.get` installs or updates dependencies.
- `mix compile` builds the application and validates the configuration.
- `mix test` runs the ExUnit suite (requires Redis at `redis://localhost:6379/15`).
- `mix format` rewrites the codebase using `.formatter.exs`.

Use `iex -S mix` to experiment with the client interactively, and `MIX_ENV=test mix test` when validating environment-specific issues.

## Coding Style & Naming Conventions
Rely on `mix format` for canonical formatting: two-space indentation, pipe-friendly line wrapping, and trailing commas on multiline data structures. Modules follow PascalCase (`Exredis.Runtime`), while functions, variables, and directory names stay snake_case. Define new Redis command wrappers via `defredis` in `lib/exredis.ex`, keeping related commands grouped together and reusing helper reducers (e.g. `&int_reply/1`) for consistent decoding.

## Testing Guidelines
Tests use ExUnit in `test/`. Name new test files `<feature>_test.exs` and describe behaviors with string-based `test` blocks. When adding APIs, include doctests or explicit assertions for both success and error flows. `mix test` presumes a local Redis instance; start one with `docker run --rm -p 6379:6379 redis` before running the suite. Aim for deterministic tests that leave Redis keys cleaned up.

## Commit & Pull Request Guidelines
Recent history uses lowercase Conventional Commit-style prefixes (e.g. `refactor:`, `wip:`). Keep summaries imperative and under 72 characters, elaborating on breaking changes or migration steps in the body. Pull requests should link to relevant issues, outline behavioral impact, document required configuration changes, and attach `mix test` output. Include screenshots only when demonstrating external integrations.

## Configuration & Environment Notes
Tune connection settings through environment variables (`REDIS_URL`, `REDIS_POOL_SIZE`, `REDIS_POOL_OVERFLOW`) or by passing overrides to `Exredis.Runtime.supervisor/1`. Never hardcode credentials in `config/*.exs`; prefer `.envrc` or local secrets stores ignored by git. Surface any non-default Redis requirements (ACLs, TLS) in PR descriptions so operators can replicate them.

---
> Source: [xbinxu/exredis](https://github.com/xbinxu/exredis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
