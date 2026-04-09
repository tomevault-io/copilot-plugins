## nexus

> Rust workspace with 4 crates: nexus-core, nexus-validate, nexus-codegen, nexus-cli.

# nexus-gen

## Project Structure
Rust workspace with 4 crates: nexus-core, nexus-validate, nexus-codegen, nexus-cli.

## Build
- `cargo build` — build all crates
- `cargo test` — run all tests (includes 33+ e2e tests)
- `cargo run -- <args>` — run nexus-cli (binary name: nexus-gen)
- `nix develop` — enter dev shell
- `nix build` — build release binary via flake

## Conventions
- Use `thiserror` for error types in library crates
- Use workspace dependencies (defined in root Cargo.toml)
- All public types in nexus-core must derive Debug, Clone
- Tests go in the same file as the code they test (mod tests)

## Architecture
- **nexus-core:** Types, parsers (.nxs via pest, network.toml via serde). Supports 4 transports: unix_socket, grpc, http, iceoryx.
- **nexus-validate:** Graph validation rules (edges, orphans, contracts, schema consistency).
- **nexus-codegen:** Multi-language code generation via minijinja templates:
  - C headers (all transports)
  - C implementations: impl_unix_socket, impl_grpc (C core API), impl_http (libcurl), impl_iceoryx (POSIX)
  - TypeScript clients (HTTP contracts only via fetch)
  - Nix derivations (transport-aware buildInputs)
- **nexus-cli:** Thin CLI shell using clap. Commands: validate, build, diff (planned).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shmul95)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/shmul95)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
