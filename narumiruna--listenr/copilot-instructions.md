## listenr

> - `src/main.rs`: Single-binary entrypoint and core pipeline (listener collection, Docker enrichment, rendering).

# Repository Guidelines

## Project Structure & Module Organization
- `src/main.rs`: Single-binary entrypoint and core pipeline (listener collection, Docker enrichment, rendering).
- `docs/CODE_READING_GUIDE.md`: Architecture and safe modification path for parser/join/output logic.
- `README.md`: User-facing install and usage examples.
- `target/`: Build artifacts (generated; do not edit).

Keep core logic in focused functions (for example `parse_lsof_line`, `parse_docker_ps_line`, `build_docker_lookup`) and avoid splitting into extra modules unless complexity clearly requires it.

## Build, Test, and Development Commands
- `cargo run --quiet`: Run the CLI locally.
- `cargo test`: Run unit tests in `src/main.rs`.
- `cargo fmt`: Apply standard Rust formatting.
- `cargo clippy --all-targets --all-features -D warnings`: Enforce lint cleanliness.
- `cargo build --release`: Build optimized binary.
- `cargo install --path .`: Install local development build.

Typical local validation flow:
`cargo fmt && cargo clippy --all-targets --all-features -D warnings && cargo test`

## Coding Style & Naming Conventions
- Follow `rustfmt` defaults (4-space indentation; no manual alignment hacks).
- Use `snake_case` for functions/variables and `CamelCase` for structs.
- Prefer descriptive, behavior-based function names (`collect_*`, `parse_*`, `format_*`).
- Keep error messages explicit and actionable (`lsof failed: ...`, `docker ps failed: ...`).
- Preserve deterministic output behavior (sorted rows, stable formatting).

## Testing Guidelines
- Place unit tests in `#[cfg(test)] mod tests` within `src/main.rs`.
- Name tests as `function_behavior_expected_outcome` (example: `parse_lsof_ipv6_loopback`).
- Add/adjust tests before changing parser or matching behavior.
- Ensure `cargo test` passes before opening a PR.

## Commit & Pull Request Guidelines
- Commit style in history is concise, imperative, and optionally scoped:
  - `docs: add code reading guide`
  - `fix process name`
- Keep commits single-purpose; avoid mixing refactors with behavior changes.
- PRs should include:
  - What changed and why.
  - User-visible impact (sample output if CLI behavior changed).
  - Validation performed (`cargo test`, `cargo clippy`, etc.).
  - Linked issue (if applicable).

## Security & Runtime Notes
- The CLI shells out to `lsof` and `docker`; handle missing/failed commands gracefully.
- Do not assume Docker is available; preserve fallback behavior that still prints local listeners.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/narumiruna)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/narumiruna)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
