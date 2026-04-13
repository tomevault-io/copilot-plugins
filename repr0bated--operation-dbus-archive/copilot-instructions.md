## operation-dbus-archive

> - `src/` — Rust 2021 code

# Repository Guidelines

## Project Structure & Module Organization
- `src/` — Rust 2021 code
  - `src/main.rs` — CLI entry (`op-dbus`)
  - `src/state/` — plugin framework (`manager.rs`, `plugin.rs`, `plugins/` with `net.rs`, `systemd.rs`)
  - `src/native/` — direct protocol clients (`ovsdb_jsonrpc.rs`, `rtnetlink_helpers.rs`)
  - `src/blockchain/` — footprint + streaming audit
- Scripts: `build.sh`, `install.sh`, `test-safe.sh`, `test-introspection.sh`
- Config examples: `example-state.json`

## Build, Test, and Development Commands
- Build release: `cargo build --release` (or `./build.sh`) — produce optimized binary.
- Run binary: `target/release/op-dbus ...` — execute CLI locally.
- Unit tests: `cargo test -q` — run fast, focused tests.
- Safe system tests: `sudo ./test-safe.sh` — read‑only checks against host.
- Introspection demo: `./test-introspection.sh` — showcase model + schema.
- Verbose logging: `RUST_LOG=op_dbus=debug op-dbus query` — troubleshoot behavior.

## Coding Style & Naming Conventions
- Format with `cargo fmt`; lint with `cargo clippy -- -D warnings`.
- Names: modules/files `snake_case`; types/enums `CamelCase`; functions/vars `snake_case`; constants `SCREAMING_SNAKE_CASE`.
- JSON state: keys `snake_case`; enum strings may be `kebab-case` (e.g., "type": "ovs-bridge").
- Shell: Bash with `set -e` (prefer `set -euo pipefail`); script filenames `kebab-case`.

## Protocol Usage Rules
- Precedence: prefer D-Bus first. Use `zbus` in code and `busctl` in scripts/tests. Then use OVSDB JSON-RPC (for OVS) and rtnetlink (for kernel networking).
- Do not shell out to network CLIs: avoid `ovs-vsctl`, `nmcli`, `ip`, `ifconfig`, `ovs-ofctl`, `brctl`.
- No NetworkManager or systemd-networkd assumptions; don’t write their configs or call their CLIs.
- D-Bus: `zbus` on the system bus (`/var/run/dbus/system_bus_socket`); `busctl` is the only allowed CLI for D-Bus introspection/verification.
- OVS: OVSDB JSON-RPC via `src/native/ovsdb_jsonrpc.rs` (`/var/run/openvswitch/db.sock`).
- Kernel networking: rtnetlink via `src/native/rtnetlink_helpers.rs`.

## Testing Guidelines
- Prefer unit tests alongside code with `#[cfg(test)]`; use `tests/` for integration.
- Required: all tests pass via `cargo test`.
- Add tests for new plugins, diff logic, and error paths.
- For system-affecting changes, include `op-dbus diff` output before `apply` in PRs.

## Commit & Pull Request Guidelines
- Commits: concise, imperative subject (e.g., "add net diff for ports"); group related changes.
- PRs must include:
  - What/why summary and linked issue (if any)
  - How to test (commands, expected output)
  - Risk notes (systemd/OVS impact) and rollback plan
  - For CLI changes: sample `op-dbus query/diff/apply` output

## Security & Configuration Tips
- Many actions require root and sockets: `/var/run/dbus/system_bus_socket`, `/var/run/openvswitch/db.sock`.
- Always validate with `op-dbus diff /etc/op-dbus/state.json` before `apply`.
- Use a throwaway host/VM for new network/systemd behaviors; avoid enabling the service until manual tests succeed.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/repr0bated) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
