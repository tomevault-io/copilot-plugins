## payroll-backend

> Rust sources live under `src/`, with `src/main.rs` serving as the entry point. Keep reusable logic in dedicated modules under `src/` and expose only what callers need via `mod` declarations in `main.rs`. Integration tests belong in `tests/` (create the directory if missing), while shared fixtures can sit under `tests/support`. Configuration files such as `Cargo.toml` and `Cargo.lock` define dependencies and release metadata—update them alongside code changes.

# Repository Guidelines

## Project Structure & Module Organization
Rust sources live under `src/`, with `src/main.rs` serving as the entry point. Keep reusable logic in dedicated modules under `src/` and expose only what callers need via `mod` declarations in `main.rs`. Integration tests belong in `tests/` (create the directory if missing), while shared fixtures can sit under `tests/support`. Configuration files such as `Cargo.toml` and `Cargo.lock` define dependencies and release metadata—update them alongside code changes.

## Application Architecture
- `src/main.rs` bootstraps configuration, networking, and delegates to `server::run`.
- `src/lib.rs` exposes high-level modules: `server`, `routes`, `handlers`, `domain`, `services`, `infrastructure`, `middleware`, `extractors`, `openapi`, and `error`.
- `src/server.rs` owns router construction (`server::router`) and the Axum/Tokio serving loop (`server::run`). The shared `AppState` wires the Organization, Payroll, Division, Job, Bank, Employee, PayrollConcept, PayrollConceptDefinition, EmployeePayrollConcept, PayrollHistory, and PayrollHistoryDetail services so handlers can enforce cross-entity invariants.
- `src/routes/` defines small, composable routers per feature (e.g., `routes::health`) that only wire HTTP paths.
- `src/handlers/` contains request/response logic (`handlers::health::check`) and converts domain data into transport-friendly payloads.
- `src/domain/` hosts pure business types and helpers (`domain::health::Health`, `domain::payroll_history::PayrollHistory`, `domain::payroll_history_detail::PayrollHistoryDetail`) with no Axum dependencies.
- `src/services/`, `src/infrastructure/`, `src/middleware/`, `src/extractors/`, and `src/error.rs` are reserved for orchestration, adapters, tower layers, custom extractors, and shared error mappers respectively as features grow.

### REST resources (current)
- `/health` — basic service metadata probe.
- `/organizations` — CRUD for organizations.
- `/organizations/{organization_id}/payrolls` — CRUD for payrolls inside an organization.
- `/organizations/{organization_id}/payrolls/{payroll_id}/divisions` — CRUD for divisions within a payroll; optional `parent_division_id` enforces same-payroll adjacency.
- `/organizations/{organization_id}/payrolls/{payroll_id}/jobs` — CRUD for jobs inside a payroll (`job_title` and `salary` fields are mandatory).
- `/organizations/{organization_id}/banks` — CRUD for banks tied to an organization (referenced by employees).
- `/organizations/{organization_id}/payrolls/{payroll_id}/divisions/{division_id}/employees` — CRUD for employees scoped to a division with job/bank validations.
- `/organizations/{organization_id}/payrolls/{payroll_id}/concepts` — CRUD for payroll concepts (`code`, `name`, `type`, `scope`).
- `/organizations/{organization_id}/payrolls/{payroll_id}/concepts/{concept_id}/definition` — Manage the formula/condition that calculates a global payroll concept.
- `/organizations/{organization_id}/payrolls/{payroll_id}/divisions/{division_id}/employees/{employee_id}/concepts` — Manage employee-specific payroll concept assignments (individual scope concepts with custom amounts).
- `/organizations/{organization_id}/payrolls/{payroll_id}/history` — CRUD for payroll history entries (status tracking per payroll run).
- `/organizations/{organization_id}/payrolls/{payroll_id}/history/{history_id}/details` — Create/list/get/delete payroll history detail rows tied to a payroll history entry.

## Build, Test, and Development Commands
- `cargo check` — fast validation of the codebase before committing.
- `cargo fmt && cargo clippy` — enforces formatting and lints; both must pass locally.
- `cargo run` — executes the binary for manual testing.
- `cargo test` — runs unit and integration tests; add `-- --nocapture` to inspect stdout.
- `cargo build --release` — produces optimized artifacts for benchmarks or deployment.

## Coding Style & Naming Conventions
Adhere to Rust’s default 4-space indentation and keep functions short with descriptive names. Use `snake_case` for modules, files, and functions; reserve `CamelCase` for types and traits; favor SCREAMING_SNAKE_CASE for constants. Always run `cargo fmt` before pushing to preserve consistent formatting, and treat `cargo clippy` warnings as errors to catch subtle bugs early. Prefer small, focused modules so agents can reason about behavior quickly.

## Testing Guidelines
Unit tests stay close to the code they verify using `#[cfg(test)]` modules, while scenario tests reside in `tests/` with filenames that mirror the behavior under test (e.g., `tests/payroll_totals.rs`). Strive for meaningful assertions instead of snapshot dumps. When adding features, include regression tests demonstrating the expected inputs and outputs; new modules should ship with at least one happy-path test and one edge-case test. Document any required environment variables at the top of each test file.

### Environment variables
The server expects a reachable SurrealDB instance configured via:

- `SURREALDB_URL`
- `SURREALDB_NAMESPACE`
- `SURREALDB_DATABASE`
- `SURREALDB_USERNAME`
- `SURREALDB_PASSWORD`
- `PORT` (optional, defaults to `3000`)

When writing new integration tests, prefer the in-memory repositories under `tests/support` to avoid external DB dependencies.

## Commit & Pull Request Guidelines
Follow Conventional Commits (e.g., `feat: add overtime calculator`) so change logs remain machine-readable. Each commit should bundle related work only—split refactors from feature code. Pull requests must describe the problem, the approach, and testing evidence (`cargo test`, manual steps, screenshots when applicable). Reference relevant issues in the PR description using `Fixes #ID`. Request review only after CI is green and conflicts are resolved.

---
> Source: [GEBNETI/payroll_backend](https://github.com/GEBNETI/payroll_backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
