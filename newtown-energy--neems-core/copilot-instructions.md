## neems-core

> This document contains important instructions for AI agents (like Claude Code) working on the NEEMS Core project.

# AI Agent Instructions for NEEMS Core

This document contains important instructions for AI agents (like Claude Code) working on the NEEMS Core project.

**IMPORTANT: This file should be regularly updated as you learn new patterns, workflows, or project-specific conventions. When you discover something important about how this project works, update this file to capture that knowledge for future sessions.**

## Critical Rules

### Docker Usage

**ALWAYS run commands inside Docker containers, NEVER on the host machine.**

This project uses `../devenv` to coordinate Docker containers. The devenv directory is located one level up from the neems-core directory.

**You MUST use `docker compose exec` from the devenv directory, NOT `docker exec`.**

Key points:
- All commands must be run via `docker compose exec` from `/Users/slifty/Maestral/Code/open-tech-strategies/newtown/devenv`
- Use `cd /Users/slifty/Maestral/Code/open-tech-strategies/newtown/devenv && docker compose exec neems-api <command>`
- The Docker Compose configuration is in `../devenv/docker-compose.yml`
- Never run cargo, tests, or build commands directly on the host
- Check container status with `cd /Users/slifty/Maestral/Code/open-tech-strategies/newtown/devenv && docker compose ps`

**Examples:**

✅ **Correct:**
```bash
cd /Users/slifty/Maestral/Code/open-tech-strategies/newtown/devenv && docker compose exec neems-api /usr/src/app/bin/dosh lint-clippy
cd /Users/slifty/Maestral/Code/open-tech-strategies/newtown/devenv && docker compose exec neems-api /usr/src/app/bin/dosh test
cd /Users/slifty/Maestral/Code/open-tech-strategies/newtown/devenv && docker compose exec neems-api cargo build
```

❌ **Incorrect:**
```bash
docker exec neems-api /usr/src/app/bin/dosh lint-clippy  # Wrong - use docker compose exec
./bin/dosh lint-clippy  # Wrong - DO NOT run on host
cargo test              # Wrong - DO NOT run on host
cargo build             # Wrong - DO NOT run on host
cargo fmt               # Wrong - DO NOT run on host
```

### Docker Tooling Setup

The Docker containers include all necessary Rust development tools:
- `clippy` - for linting
- `rustfmt` - for code formatting
- `rust-src` - for enhanced IDE support and development

These components are installed via `rustup component add clippy rustfmt rust-src` in the Dockerfiles for both `neems-api` and `neems-data` services.

If you need to rebuild the containers to pick up Dockerfile changes:
```bash
cd /Users/slifty/Maestral/Code/open-tech-strategies/newtown/devenv && docker compose build neems-api neems-data
cd /Users/slifty/Maestral/Code/open-tech-strategies/newtown/devenv && docker compose up -d neems-api neems-data
```

## Project Structure

This is a Rust workspace with multiple crates:
- `neems-api` - Main API server
- `neems-admin` - CLI administration tool
- `neems-data` - Data aggregation service (contains the RTAC Modbus integration in `src/rtac/`)
- `neems-rtac-sim` - Simulated RTAC: a Modbus TCP server for exercising the RTAC integration without hardware
- `crates/fixphrase` - Utility crate for GPS coordinate encoding

## Simulated RTAC (`neems-rtac-sim`)

`neems-rtac-sim` is a standalone Modbus TCP server that simulates an RTAC. It
reuses the register map from `neems-data`'s `rtac::protocol` (the single source
of truth), so the simulator and the real client cannot drift apart. Command
registers (target charge, operating mode) drive a simple internal model whose
state of charge and alarm list advance once per tick (1 Hz by default).

- Run it interactively: `cargo run -p neems-rtac-sim` (type `help` for stdin
  control commands like `charge`, `discharge`, `soc 80`, `alarm set 321`).
- In `devenv` it runs as the `neems-rtac-sim` service (with `--no-stdin`,
  binding `0.0.0.0:502`). `neems-data` is pointed at it by default via the
  `RTAC_ADDRESS=neems-rtac-sim:502` env var, read by `RtacConfig::from_env()`.

### RTAC collector (`rtac::runner`)

`neems-data monitor` starts the RTAC collector alongside the source poller (see
`DataAggregator::start_aggregation`). `rtac::runner::run_rtac_collector` wires
the `ModbusWorker` to the storage and alarm tasks, polls the RTAC at 10 Hz, and
persists SoC readings at 1 Hz to a dedicated `charging_state` source named
`rtac` (created `active = false` so the generic poller doesn't also write it).
Those readings surface through the existing `GET /Sites/<id>/SocHistory`
endpoint and the React dashboard.

- The collector is **closed-loop**: alongside reading status it drives the RTAC
  from the site's schedule. A background poller (`rtac::schedule_http`) fetches
  the active command from neems-api's `GET /Sites/<id>/ActiveCommand` endpoint,
  and `ControlLogicTask` (via `HttpScheduleProvider`) turns it into RTAC commands
  with reactive SoC/alarm safety overrides. Credentials come from
  `NEEMS_API_URL` / `NEEMS_API_EMAIL` / `NEEMS_API_PASSWORD` (falling back to
  `NEEMS_DEFAULT_EMAIL` / `NEEMS_DEFAULT_PASSWORD`); without them the collector
  stays read-only and logs a warning.
- It runs on its own thread with a current-thread runtime because the Modbus
  client context is not `Send`.
- Disabled by default so the system doesn't follow a (possibly absent) RTAC
  connection. Enable it with `RTAC_ENABLED=1` (or `true`/`yes`/`on`).

## The shared `target/` volume

`neems-api`, `neems-data`, and `neems-rtac-sim` all mount the same
`target-cache` volume at `/usr/src/app/target`. Cargo never garbage-collects
`target/`, so without help that volume grows without bound — it reached 173 GB
and filled the Docker VM's disk (issue #94).

**Check it with `./bin/dosh disk`. Prune it with `./bin/dosh sweep`.** The
neems-api container does the same sweep on start and every 6 hours after.

Reference numbers, so a future change can be judged against them: one clean
build of everything — every bin, every test binary, single fingerprint, no
incremental — is about **5.7 GB**. The default ceiling is 10 GB.

Four things hold that line, and each is easy to undo by accident:

- **Never set `RUSTFLAGS` for a one-off command.** Debug info is capped in
  `[profile.dev]` in the workspace `Cargo.toml`. `RUSTFLAGS` is part of a
  build's fingerprint, so setting it for some invocations and not others makes
  cargo build and keep a second complete set of artifacts for the same code.
  Two `libneems_api` rlibs, 300 MB each, is what that looked like.
- **`cargo sweep --maxsize` enforces the ceiling**, oldest artifacts first. A
  ceiling rather than an age cutoff, because only a ceiling answers "this will
  never exceed N".
- **`target/debug/incremental` is bounded separately, and first.** cargo-sweep
  will not touch that directory but does count it toward the total, so a large
  incremental cache makes cargo-sweep delete everything it *can* reach to
  compensate — a full rebuild for nothing. Keep the incremental prune ahead of
  the sweep.
- `fast_test_rocket()` sweeps per-test database copies older than an hour, and
  `bin/create-golden-db.sh` removes superseded golden databases.

Tunable via environment (defaults in `neems-api/docker-entrypoint.sh`, set in
`devenv/docker-compose.yml`): `CARGO_TARGET_MAXSIZE`,
`CARGO_TARGET_SWEEP_INTERVAL`, `CARGO_INCREMENTAL_MAXSIZE_MB`.

## Development Workflow

### Linting

Run lints inside docker (from devenv directory):
```bash
cd /Users/slifty/Maestral/Code/open-tech-strategies/newtown/devenv && docker compose exec neems-api /usr/src/app/bin/dosh lint
cd /Users/slifty/Maestral/Code/open-tech-strategies/newtown/devenv && docker compose exec neems-api /usr/src/app/bin/dosh lint-clippy
cd /Users/slifty/Maestral/Code/open-tech-strategies/newtown/devenv && docker compose exec neems-api /usr/src/app/bin/dosh lint-format
```

### Testing

Run tests inside docker (from devenv directory):
```bash
cd /Users/slifty/Maestral/Code/open-tech-strategies/newtown/devenv && docker compose exec neems-api /usr/src/app/bin/dosh test
cd /Users/slifty/Maestral/Code/open-tech-strategies/newtown/devenv && docker compose exec neems-api /usr/src/app/bin/dosh nextest
```

### Building

Build inside docker (from devenv directory):
```bash
cd /Users/slifty/Maestral/Code/open-tech-strategies/newtown/devenv && docker compose exec neems-api /usr/src/app/bin/dosh build
```

## Temporarily Allowed Clippy Lints

The following clippy lints are temporarily allowed via command-line flags in `bin/dosh`. These should be addressed incrementally:

- `clippy::collapsible_if`
- `clippy::empty_line_after_doc_comments`
- `clippy::expect_fun_call`
- `clippy::if_same_then_else`
- `clippy::items_after_test_module`
- `clippy::len_zero`
- `clippy::match_ref_pats`
- `clippy::too_many_arguments`
- `clippy::useless_vec`

To re-enable a lint, simply remove the corresponding `-A` flag from the `lint-clippy()` function in `bin/dosh`.

## CI/CD

The project uses GitHub Actions for CI/CD:
- `.github/workflows/ci.yml` - Main CI workflow
- `.github/workflows/test.yml` - Test workflow (called by ci.yml)
- `.github/workflows/lint.yml` - Lint workflow (called by ci.yml)
- `.github/workflows/publish-types.yml` - Builds `@newtown-energy/types` on PRs (dry-run), publishes on push to main

## npm Types Package (`@newtown-energy/types`)

TypeScript types are auto-generated from Rust structs via `ts-rs`. On merge to `main`, the `.github/workflows/publish-types.yml` workflow publishes them to GitHub Packages (`npm.pkg.github.com`) as `@newtown-energy/types`.

### How it works

The workflow has two jobs: **check-version** (determines if a new version needs publishing) and **build-and-publish** (runs only when the version has changed). Template files live in `npm/`:
- `npm/package.template.json` — package.json template (version placeholder replaced at build time)
- `npm/tsconfig.json` — TypeScript compiler config

The build job generates types via `cargo test`, scaffolds the package from templates, generates a barrel `index.ts`, and compiles with `tsc`. On PRs this serves as a dry-run to catch build failures before merge.

Publishing uses the `GITHUB_TOKEN` which is automatically available in GitHub Actions. The package is published as a **public package** to GitHub Packages, so no authentication is required to install it.

### Version bumping

The npm package version comes from `neems-api/Cargo.toml`. When changing types:
- **Bump the version in `neems-api/Cargo.toml`** in the same PR that changes types
- **Patch** (0.1.4 → 0.1.5): compatible additions (new optional fields, new types)
- **Minor** (0.1.x → 0.2.0): new types or endpoints that don't break existing consumers
- **Major** (0.x → 1.0): breaking changes (renamed/removed fields, changed type shapes)

### Local development workflow

When developing backend + frontend simultaneously, the published npm package will be out of date. The Docker dev environment handles this automatically:

- On startup, `docker-entrypoint.sh` generates TypeScript types and builds a local `@newtown-energy/types` package in `local-types/` (at the project root)
- `cargo watch` regenerates types whenever Rust source files change, then rebuilds the local package via `bin/build-local-types-package.sh`
- The neems-react container uses `bun link` to symlink `node_modules/@newtown-energy/types` to the shared `local-types/` directory, so imports resolve to the local build automatically
- No manual `npm link` or other steps are needed

## Code Style

- Format code: `cd /Users/slifty/Maestral/Code/open-tech-strategies/newtown/devenv && docker compose exec neems-api cargo fmt`
- Run clippy for linting: `cd /Users/slifty/Maestral/Code/open-tech-strategies/newtown/devenv && docker compose exec neems-api /usr/src/app/bin/dosh lint-clippy`
- Run all linting: `cd /Users/slifty/Maestral/Code/open-tech-strategies/newtown/devenv && docker compose exec neems-api /usr/src/app/bin/dosh lint`
- Follow Rust standard naming conventions

---
> Source: [Newtown-Energy/neems-core](https://github.com/Newtown-Energy/neems-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
