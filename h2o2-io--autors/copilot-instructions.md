## autors

> `auto_rs` is a cross-platform Rust workspace for automotive calibration,

# auto_rs contributor guide

`auto_rs` is a cross-platform Rust workspace for automotive calibration,
measurement, diagnostics, bus access, and symbol processing. It exposes native
Rust APIs and a stable C ABI.

## Workspace architecture

Dependencies should flow from foundational crates toward integration crates.
Avoid cycles and do not make file-format crates depend on transport or protocol
implementations.

| Layer | Crates | Responsibilities |
| --- | --- | --- |
| Foundation | `autors-util`, `autors-native`, `autors-runtime` | Safe utilities, dynamic libraries and Seed & Key, async runtime abstraction |
| File formats | `autors-a2l`, `autors-cdf`, `autors-dcm`, `autors-datafile`, `autors-dbc`, `autors-ldf`, `autors-asc`, `autors-blf`, `autors-ltrc`, `autors-mdf`, `autors-odx` | Calibration, network, measurement, and diagnostic data formats |
| Values and formulas | `autors-formula`, `autors-values` | Conversion formulas, checksums, and runtime calibration values |
| Symbols | `autors-symbols`, `autors-elf`, `autors-map` | Symbol parsing and A2L address updates |
| Bus access | `autors-can`, `autors-lin` | Cross-platform bus abstractions and optional hardware adapters |
| Scheduling | `autors-scheduler` | DBC/LDF-driven CAN and LIN remaining-bus simulation and transmission hooks |
| Transport | `autors-isotp` | ISO-TP over CAN and ISO 17987-2 diagnostic transport over LIN |
| Protocols | `autors-comm`, `autors-ccp`, `autors-xcp`, `autors-diag` | Shared DAQ state and CCP, XCP, UDS, KWP2000, and DoIP clients |
| Flash procedures | `autors-prm` | PRM/CNF parsing and execution |
| Integration | `autors-ffi` | C ABI, generated header, and foreign-language examples |

Intentional dependency edges include `autors-asc`/`autors-blf -> autors-can` for
`CanFrame`, `autors-blf`/`autors-ltrc -> autors-lin` for `LinFrame`,
`autors-ccp -> autors-xcp` for shared typed IF_DATA values, and
`autors-elf`/`autors-map -> autors-symbols` for address-update types.

## Runtime and feature model

The communication crates use an async core. The `runtime-tokio` feature selects
Tokio-backed runtime services, while `blocking` exposes synchronous facades that
drive the same async operations through `autors-runtime`. Both are enabled by
default. Keep async-only builds working with `--no-default-features` and blocking
builds working without relying on a caller-owned Tokio runtime.

`autors-can` enables `socketcan` by default on Linux. Hardware adapters are
opt-in `vendor-*` features; `all-vendors` enables all 17 adapters. `autors-lin`
uses the same convention for its three adapters. Optional native dependencies
must stay behind the feature that needs them, and platform-specific code must be
guarded with `cfg` at the narrowest practical boundary.

## Required behavior contracts

These two behaviors are deliberate and high priority:

1. Receive processing is cooperative by default. Devices and protocol masters
   must not start an implicit background receive thread. Callers advance work
   through `poll`, `poll_once`, or `CommKernel::poll_clients`. The explicit CAN
   dispatch helper remains opt-in. Blocking facades must drive the same async
   core instead of implementing a second state machine.
2. XCP exchange code queues callbacks while a mutable exchange is active, then
   delivers them after the exchange returns through `XcpMaster::drain_pending`.
   This prevents re-entrant mutable access while preserving callback order.
   New command paths must drain pending callbacks at the same lifecycle boundary.

Preserve externally visible protocol quirks when tests document them. If a
behavior appears odd, name the test after the concrete input/output contract and
explain why callers can observe it; do not describe implementation ancestry.

## Coding rules

- Follow standard Rust naming and module conventions. Prefer traits for shared
  behavior, enums for closed polymorphism, `Vec` for ordered collections, and
  `IndexMap` where stable insertion order is part of round-trip fidelity.
- Use each crate's `Error` and `Result` types. Avoid `unwrap` and `expect` outside
  tests, and never allow a panic to cross the FFI boundary.
- Keep `unsafe` confined to native, FFI, and hardware-driver boundaries. Every
  unsafe block needs a precise `SAFETY` explanation. `autors-util` forbids unsafe
  code.
- Parse XML with `serde` and `quick-xml`. Preserve unknown A2L content as tokens
  when a typed representation is unavailable.
- Keep public API documentation concise and behavioral: units, byte order,
  ownership, failure conditions, and state transitions matter more than type
  inventories.
- Comments and maintained documentation are English-only. Do not add historical
  source-attribution or implementation-history notes. Record compatibility behavior
  in terms of observable inputs and outputs.
- Keep target-specific code inside `autors-can`, `autors-lin`, `autors-native`,
  or the FFI boundary unless the design requires otherwise.
- Do not edit generated or external artifacts to make validation pass. Update
  their generator or the maintained source instead.
- Do not run `git commit` or `git push` unless the user explicitly requests it.

## Tests and validation

For a focused change, start with the affected package:

```text
cargo test -p <crate>
cargo clippy -p <crate> --all-targets -- -D warnings
```

Before handing off a workspace-wide change, run:

```text
cargo fmt --all -- --check
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo doc --workspace --no-deps
```

For runtime-sensitive crates, also check representative feature forms:

```text
cargo check -p <crate> --no-default-features
cargo check -p <crate> --no-default-features --features runtime-tokio
cargo check -p <crate> --no-default-features --features blocking
```

For bus adapters, validate default and full-adapter builds:

```text
cargo check -p autors-can
cargo check -p autors-can --all-features
cargo check -p autors-lin --all-features
```

When the targets are installed, cross-check the workspace for
`x86_64-pc-windows-msvc`, `x86_64-unknown-linux-gnu`, and
`x86_64-apple-darwin`. Do not claim a target passed when it was skipped because
the target or linker was unavailable.

The FFI acceptance surface includes the generated header, its ABI tests, and the
examples under `crates/autors-ffi/examples`. API changes must update all three.

## Change discipline

Keep changes scoped and preserve unrelated work in a dirty worktree. Add tests
for behavior changes, especially parser round trips, binary layout, timeout and
cancellation paths, callback ordering, feature gates, and FFI error handling.
Report validation failures exactly, including whether they predate the current
change or are caused by an unavailable toolchain or external dependency.

---
> Source: [H2O2-IO/autors](https://github.com/H2O2-IO/autors) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
