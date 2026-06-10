## mavsim

> These instructions apply to the whole repository. The primary codebase is the

# Repository Instructions

## Scope

These instructions apply to the whole repository. The primary codebase is the
Rust Cargo workspace in `crates/`; `mavsdk-demo/` contains Python helper scripts
for external MAVSDK experiments and should not drive Rust architecture choices.

## Project Shape

- `crates/mavsim-core`: simulator state, physics, world/environment, vehicle
  models, sensors, gimbal, geospatial projection, and public core types.
- `crates/mavsim-mavlink`: MAVLink/PX4 HIL bridge, endpoint routing, runtime
  loop, serial/UDP/TCP transport handling, and dialect bindings generated at
  build time from `mavlink/message_definitions/*.xml`.
- `crates/mavsim-ui`: `eframe`/`egui` desktop UI, keyboard commands, HUD/report
  panels, asset loading, software viewport fallback, and `wgpu` viewport
  callback rendering.
- `crates/mavsim-bin`: `mavsim` CLI compatibility parsing, startup/shutdown
  wiring, logging, and headless vs GUI mode selection.
- Runtime assets live in `models/`, `environment/`, and
  `mavlink/message_definitions/`. Keep paths relative to the repository root
  unless an existing API clearly expects otherwise.

## Rust Quality Bar

- Treat each Rust change as part of a long-lived simulator. Prefer simple,
  readable, maintainable code over clever compactness, and do not stop at "it
  compiles" when behavior, tests, or error handling are still unclear.
- Before changing code, identify the source of truth, existing local patterns,
  failure modes, and invariants that Rust's type system can enforce. When
  requirements are ambiguous, make the smallest safe assumption and document it
  only where future maintainers need the context.
- Prefer clear, typed domain APIs over ad hoc strings or loosely shaped data.
  Public options should flow through `SimConfig` and the typed enums in
  `mavsim-core` where possible.
- Make invalid states unrepresentable where practical: use enums for finite
  states, newtypes for domain-specific values, private fields with validating
  constructors, `NonZero*` types when useful, and `Result<T, E>` for validation
  that can fail.
- Keep simulation behavior deterministic when lockstep is enabled. Time changes
  should go through `Simulator::advance_time`, `tick_fixed`, or
  `tick_fixed_after_external_time_advance` as appropriate; avoid accidental
  double-advancement.
- Preserve PX4/SITL compatibility. Be careful with MAVLink message IDs,
  heartbeat/init semantics, `HIL_ACTUATOR_CONTROLS`, deprecated
  `HIL_CONTROLS`, `HIL_STATE_QUATERNION`, QGC/SDK forwarding, and filtering of
  HIL messages.
- Use `f64` for simulator math where the code already does. Convert deliberately
  at MAVLink, UI, or GPU boundaries and keep units obvious in names or nearby
  context.
- Handle fallible IO and transport setup with `anyhow::Context` in application
  and runtime code. Use typed errors such as `ConfigError` where callers or
  tests need exact error variants.
- Avoid `unwrap`/`expect` in production paths unless failure is truly impossible
  or unrecoverable during startup/build setup. Tests may use them for clarity.
- Do not ignore errors silently. If discarding an error or send failure is
  intentional, keep the scope narrow and leave a short reason nearby.
- Keep lock scopes small around `Arc<Mutex<Simulator>>`. Do not hold a simulator
  lock while performing blocking IO, sleeping, or calling into UI painting.
- Avoid hidden global mutable state, broad catch-all modules, speculative
  generics, excessive macros, and new traits that do not represent a real
  boundary or multiple plausible implementations.
- Reuse existing allocation patterns in hot paths: scratch buffers,
  `*_into(...)` functions, `Vec::with_capacity`, and `clear`/`reserve` are used
  intentionally in viewport and runtime loops.
- Do not edit generated MAVLink output under `target/`. Change XML definitions
  in `mavlink/message_definitions/` or `crates/mavsim-mavlink/build.rs` instead.
- Preserve the current public API smoke tests when changing exports from
  `lib.rs`.

## Python Quality Bar

- Python code currently lives under `mavsdk-demo/` and is a support/demo area
  for MAVSDK and fake PX4 workflows. Keep it isolated from Rust crate
  architecture and avoid moving simulator business logic into Python.
- Use the Python version pinned by `mavsdk-demo/.python-version` and manage the
  environment with `uv` using the existing `pyproject.toml` and `uv.lock`.
- Prefer clear, typed data containers (`dataclass`, small classes, constants)
  over dictionaries for state that has behavior or protocol meaning.
- Keep MAVLink/PX4 constants named explicitly and close to the code that uses
  them. Do not hide protocol message IDs, mode bits, units, or frame
  conventions behind ambiguous helper names.
- Keep scripts deterministic enough for tests: inject time, connection objects,
  or message streams where practical instead of sleeping or opening sockets in
  unit tests.
- Avoid broad exception swallowing. If a demo script handles an operational
  failure, log or print enough context for a user running the script to diagnose
  the endpoint, message type, or command involved.
- Keep top-level scripts runnable from `mavsdk-demo/` and avoid hard-coded
  absolute paths. If a path to the Rust binary is needed, derive it from the
  repository layout as existing scripts do.
- Do not commit `.venv/`, `.ruff_cache/`, `__pycache__/`, generated logs, or
  captured telemetry unless the user explicitly asks for an artifact.

## Style

- Follow the existing Rust 2024 workspace configuration in `Cargo.toml` and
  keep `rustfmt.toml` aligned with it.
- Run `cargo fmt --all` before finishing Rust edits.
- Use `cargo clippy --workspace --all-targets --all-features` when a change is
  broad, refactor-heavy, or touches shared APIs. Prefer fixing useful Clippy
  findings over silencing them; if a lint is intentionally allowed, keep the
  rationale local.
- For Python edits, use Ruff from the `mavsdk-demo` environment:
  `uv run ruff format .` and `uv run ruff check .`.
- Keep code idiomatic and explicit: `let Some(x) = ... else { ... };`,
  structured matches, typed constants, and small helper functions are preferred
  over clever branching.
- Keep comments sparse and useful. Add comments only for non-obvious simulator,
  MAVLink, concurrency, unsafe, or rendering behavior.
- Public Rust APIs should have useful Rustdoc when behavior, errors, panics, or
  safety requirements are not obvious from the signature.
- Maintain repository conventions for names and log messages when tests assert
  output. Several CLI and MAVLink tests check exact strings.

## Testing And Verification

Use the smallest useful test first, then broaden based on risk.

- Format: `cargo fmt --all --check`
- Lints when warranted: `cargo clippy --workspace --all-targets --all-features`
- Main Rust suite: `cargo test --workspace --all-targets`
- Crate-specific checks:
  - `cargo test -p mavsim-core`
  - `cargo test -p mavsim-mavlink`
  - `cargo test -p mavsim-ui`
  - `cargo test -p mavsim-bin --all-targets`
- Run the CLI manually when changing argument parsing or startup behavior:
  - `cargo run -- -h`
  - `cargo run -- -no-gui -udp 14560 -lockstep`
- Python support scripts from `mavsdk-demo/`:
  - `uv run ruff format --check .`
  - `uv run ruff check .`
  - `uv run python -m unittest`
- Ignored acceptance tests require external conditions and should be run only
  when those conditions are available:
  - PX4 SITL:
    `cargo test -p mavsim-mavlink --test px4_sitl_acceptance -- --ignored`
  - Native GUI desktop session:
    `cargo test -p mavsim-ui --test native_gui_acceptance -- --ignored`

Some binary smoke tests are Unix-specific and use signals or pseudo-terminals.
On Windows, document if those tests were not applicable rather than weakening
the tests.

Prefer behavior-focused tests over implementation tests. Cover domain rules,
edge cases, validation errors, serialization/deserialization, concurrency, and
bug regressions. Keep tests deterministic and isolated; inject time,
connections, or message streams where practical instead of depending on
wall-clock sleeps, external services, or execution order.

## Pragmatic Engineering Guidance

- Keep changes focused, but leave touched code better when it is safe: remove
  dead code, simplify confusing local logic, improve misleading names, tighten
  error handling, or add missing tests. Avoid large unrelated rewrites.
- Avoid duplicating knowledge. Defaults, validation rules, protocol constants,
  error interpretation, and domain assumptions should have one authoritative
  representation even when some syntactic repetition is acceptable.
- Build large features as tracer bullets first: establish the entrypoint,
  domain flow, error visibility, and test seam, then fill in validation,
  edge cases, performance, and polish.
- Prototype deliberately when behavior is uncertain, especially around
  third-party crates, runtime/concurrency behavior, serialization formats, or
  performance. Clean up exploratory code before treating it as production.
- Do not over-engineer for imagined futures. Add traits, generics, async
  machinery, caching, or new dependencies only when they solve a current,
  concrete problem.
- Think end-to-end at boundaries: input validation, failure behavior, logging,
  cancellation/shutdown, retries, compatibility, and user impact all matter for
  simulator, MAVLink, CLI, and UI changes.
- Protect sensitive data. Do not log secrets, tokens, private keys, credentials,
  personal data, or full request/response dumps that might contain them. Use
  redaction wrappers or custom `Debug` where needed.
- Make important failures observable with wrapped errors, clear startup
  failures, and useful logs that are diagnostic without being noisy.
- Measure before optimizing. Prefer efficient simplicity, avoid unnecessary
  allocations/clones/boxing/dynamic dispatch/lock contention in hot paths, and
  use benchmarks, profiles, traces, or logs before making clarity tradeoffs.
- Be conservative with dependencies. Prefer the standard library or existing
  workspace crates for small utilities; when a risky dependency is needed, hide
  it behind a narrow module or trait where practical.

## Change Guidance By Area

### Core Simulator

- Keep `SimConfig::validate` as the gate for invalid runtime combinations.
- When reading environment variables, isolate tests with a lock/guard pattern as
  existing config tests do.
- Sensor, wind, gimbal, and manual vehicle-control changes should mark sensor
  reset state when they affect readings.
- Add focused unit tests for physics, timing, parameters, and snapshots. Use
  tolerances for floating-point comparisons unless exact values are intentional.

### MAVLink Runtime

- Preserve nonblocking receive behavior and bounded drain loops; do not let one
  endpoint starve the simulator tick.
- Keep MAVLink v1 compatibility and `allow_recv_any_version` behavior unless a
  change explicitly targets protocol negotiation.
- Avoid forwarding HIL traffic to QGC/SDK or from QGC/SDK back to the
  autopilot. `MavlinkRouter::HIL_MESSAGE_IDS` is the shared boundary.
- When adding supported messages, update event conversion, outbound conversion,
  debug names, router/filter behavior if needed, and tests that exercise actual
  packet exchange.
- Use ephemeral UDP ports in new tests where possible. If a fixed port is
  necessary, avoid existing fixed ports used in the suite.

### UI And Rendering

- Keep the first screen the usable simulator UI: viewport, HUD/log, optional
  report/parameter panels.
- Preserve keyboard command behavior documented in `README.md` and CLI help.
  If a command changes, update implementation, help text, README, and tests
  together.
- UI work should remain immediate-mode `egui` style. Avoid long blocking work in
  the UI update path.
- For viewport changes, keep both the `wgpu` path and software painter fallback
  coherent. Geometry helpers should remain testable without opening a window.
- Keep GPU vertex types `#[repr(C)]` and compatible with `bytemuck::Pod` /
  `Zeroable`.

### CLI

- Preserve compatibility with the single-dash legacy flags listed in `README.md`
  and CLI help.
- Route parsing into `SimConfig`, then call `validate`. Do not duplicate config
  validation in the binary unless it is CLI-specific syntax validation.
- Be careful with stdout/stderr: tests assert that startup/status messages and
  validation errors appear on the expected stream.

### Assets

- Keep OBJ/MTL model orientation compatible with existing models.
- Avoid committing generated logs, `target/`, caches, or large derived assets.
- If adding runtime assets, make loading failures graceful in the UI unless the
  asset is required for headless correctness.

## Dependency Guidance

- Prefer workspace dependencies in the root `Cargo.toml`; do not pin separate
  versions in crate manifests without a strong reason.
- Avoid new dependencies for small utilities that the standard library or
  existing crates already cover.
- For domain logic with protocol or rendering implications, prefer established
  crates already in use (`mavlink`, `nalgebra`, `eframe`/`egui`, `wgpu`) over
  hand-rolled replacements.

## Before Finishing

- Check `git diff` and make sure changes are scoped to the request.
- Run formatting and the most relevant tests you can reasonably run.
- Report any skipped tests, especially GUI, PX4 SITL, serial, or Unix-only
  acceptance coverage.

---
> Source: [RustedBytes/mavsim](https://github.com/RustedBytes/mavsim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->
