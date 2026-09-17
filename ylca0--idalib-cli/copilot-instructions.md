## idalib-cli

> Guidance for code agents working in this repository.

# AGENTS.md

Guidance for code agents working in this repository.

## Build & verify

The real build requires the IDA Pro 9.1 SDK (see README). Without the SDK, use
the dev-only stub:

```sh
# Type-check / build / test against the stub (no SDK needed)
cargo check --no-default-features --features stub-idalib
cargo build --no-default-features --features stub-idalib
cargo test  --no-default-features --features stub-idalib

# Formatting
cargo fmt --all
```

With the SDK available:

```sh
export IDASDKDIR=$HOME/idasdk91
export IDADIR="/Applications/IDA Professional 9.1.app/Contents/MacOS"
cargo build --release
```

## Conventions

- Versioning: tool version x.y.z (current 0.9.1); the compatible IDA version
  lives in `[package.metadata.ida]` (currently 9.1), not in the crate version.
- `idalib` is pinned to `=0.6.1` (the 9.1-matching idalib-rs release).
- All commands output JSON by default; `--json` is a legacy/consistency flag.
- `stubs/idalib` is a DEV-ONLY API stub for `idalib`. It must never be compiled
  into release builds (it is gated behind the `stub-idalib` feature).
- Stateless interface: every command takes `-d/--db <PATH>` (an IDB file or a
  binary). There is no session registry - the IDB file is the only state.
- `src/ffi_ext.rs` holds hand-written `extern "C"` declarations for IDA kernel
  symbols that `idalib-sys` 0.6.1 does not generate (set_name, bin_search,
  xrefblk_t_first_from/next_from). All stub out under `stub-idalib`. When
  bumping the IDA version, re-verify these ABIs against the SDK headers.
  Known upstream bug worked around: the 0.6.1 cxx bridge for
  `xrefblk_t_first_from` returns incoming refs (we call the kernel directly).
- When adding a new IDALib capability, mirror the shape in `src/ops/metadata.rs`
  (or a new op file), add a clap variant in `src/cli.rs`, dispatch it in
  `src/ops/top.rs` (`run_db_op`), and extend the stub `stubs/idalib/src/*.rs`
  so code still type-checks without the SDK.

## Testing

Unit tests live alongside modules (`#[cfg(test)]`) and in `tests/`. Run with the
stub feature as shown above. Integration behaviour (session persistence,
batch, parallel) is exercised against the stub binary via `IDALIB_CLI_HOME`.
Real builds require the IDA SDK + a licensed IDA installation (see README).

## Skills

A single agent skill lives at `skills/idalib-cli/SKILL.md` and describes the
tool's analysis workflows. Keep it in sync with any CLI changes.

## Branching & releases

Source-only repository: no CI, no binaries, no SDK. Users install locally via
`cargo install` with `IDADIR` + `IDASDKDIR` set (build-time SDK requirement).

Tool versions are `x.y.z` (current: `0.9.1`). Two branches per tool minor,
plus `main`:

- `main`              - latest development (merge target of `v*_dev`)
- `v0.9_dev`          - development branch (tool 0.9.x)
- `v0.9_release`      - stable branch (fixes only; same for `v1.0_release`, ...)
- tags `v0.9.1`       - release points

The compatible IDA version is declared in `[package.metadata.ida]`
(`compatible = "9.1"`) in Cargo.toml. Supporting a new IDA version = bump the
`idalib` dependency, update the metadata, run compatibility checks, and open a
new `v0.10_dev` / `v0.10_release` branch line.

Never commit: plaintext IDA SDK, SDK archives (encrypted or not), or binaries.

---
> Source: [ylca0/idalib-cli](https://github.com/ylca0/idalib-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-17 -->
