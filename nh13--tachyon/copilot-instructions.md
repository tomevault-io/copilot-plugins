## tachyon

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`tachyon` is a single-binary, zero-dependency Rust probe that measures a host's **memory access rate under whatever contention exists right now**. It exists so benchmark wall times from shared cloud hosts can be told apart from real code regressions: the same unchanged binary measured 18.89 s and 25.01 s on different `m7i.4xlarge` instances, and the score is the control variable that flags such a host.

Read `README.md` (the argument for the design) and `src/lib.rs`'s module doc (the same argument, with the measurements) before changing anything under `src/`.

`CONTRIBUTING.md` is the authority on the change rules, testing rationale, and dependency policy. This file covers what an agent needs on top of that: where things live, and which claims about this codebase are easy to get wrong.

## Commands

```bash
cargo ci-fmt     # fmt --all -- --check
cargo ci-lint    # clippy --all-features --all-targets -- -D warnings
cargo ci-test    # test --locked --all-features
```

These three aliases live in `.cargo/config.toml` and are exactly what CI runs. `--locked` is deliberate — it fails on a stale lockfile rather than silently re-resolving.

Single test: `cargo test chain_is_a_single_full_length_cycle`. Integration tests only: `cargo test --test cli`. Whole crate builds in ~1 s; there is no nextest and no watch setup, on purpose.

Optional pre-commit hook (runs `ci-fmt` + `ci-lint`): `./scripts/install-hooks.sh`.

Manual smoke of the release binary — the thing `cargo test` cannot stand in for, see below:

```bash
cargo build --release --locked
./target/release/tachyon --seconds 2 --working-set-mb 32 --json
```

## The rule that governs changes

**A change to what determines a reading is a change to what past readings mean.** Scores get recorded beside benchmark timings and compared months later; if the access pattern, the chain construction, the default working set, or the units change, every previously recorded score silently becomes incomparable. There is no error message for that.

That covers `src/`, but not only `src/` — `[profile.release]` in `Cargo.toml` is part of the measurement (the optimised build is the one people run), and so is `rust-toolchain.toml`, since a different compiler can generate a different chase.

Classify any such change and say so in the PR description, per `CONTRIBUTING.md`:

- **No change to what is measured** (refactor, docs, CI, tests), or
- **Changes what is measured** — then include before/after scores from the *same machine* and a note on interpreting old readings.

`CHANGELOG.md` carries a `### Measurement` heading for the second kind.

## Architecture

Three files, and the split is meaningful:

- **`src/lib.rs`** — the measurement, and nothing else. `ProbeConfig` in, `ProbeResult` out via `run()`; `slots_for` is also public, for callers sizing a working set. `ProbeResult` stores raw counts (`accesses`, `elapsed`, `threads`, `working_set_bytes`) and derives `million_accesses_per_sec()` (the score) and `ns_per_access()` (the sanity check) on demand, so the stored record cannot disagree with the reported one. Note `working_set_bytes` is the memory actually walked — the request rounded down to whole cache lines — not the value passed in.
- **`src/main.rs`** — hand-rolled arg parsing, JSON and human formatting, exit codes. No measurement logic. `parse_args` returns `Result<Args, String>` so every rejection is testable in-process.
- **`tests/cli.rs`** — runs the built binary via `CARGO_BIN_EXE_tachyon`.

Design invariants, and whether anything actually pins each one:

- **The chain is a single full-length cycle** (Sattolo's algorithm — `j` drawn from `[0, i)`, not `[0, i]`). An arbitrary permutation decomposes into short cycles, and a walker caught in one revisits a handful of lines that then sit in cache — turning a memory probe into a cache probe. *Pinned by `chain_is_a_single_full_length_cycle`.*
- **The chain is a dense `u32` array covering the whole working set** — sixteen slots per 64-byte line, so the allocation is exactly the size requested. Hops are deliberately *not* spread one-per-line; the full-length cycle means a line's other slots come back millions of hops later, long after eviction. *Pinned by `slots_cover_the_whole_working_set`.*
- **Chains are built before the clock starts**, so allocation and permutation are outside the measurement. *Not pinned by any test — structural.*
- **`black_box` guards the chase** against the optimiser deleting pure computation with an unused result. *Not pinned by any test; see below for why that is hard.*
- **The probe must not resemble the code under test.** Sharing code would let a real regression move the probe, cancelling itself out when you normalise. This is why the probe is a pointer chase and not anything drawn from the workloads it is used against. *Not pinnable by a test — a review question.*

## The optimiser-elision trap

Two intuitive things about this failure mode are false, and both were stated wrongly across this repo's docs before being corrected. Do not reintroduce them:

1. **`cargo test` cannot catch it.** `CARGO_BIN_EXE_tachyon` points at the binary built in the test's own profile, so `tests/cli.rs` runs the *debug* build, where nothing is elided. Only the CI `probe` job builds `--release`.
2. **The symptom is not `accesses: 0`.** `accesses` counts completed *batches*, not completed loads, so eliding the inner chase makes the outer loop nearly free and the count *inflates* — measured at ~2.4e12 for a 0.4 s run. An `accesses > 0` assertion passes straight through it.

What actually catches it is the **plausibility bound on `ns_per_access`**, asserted in `tests/cli.rs` and again in CI (`20 < ns_per_access < 2000`).

## Testing conventions

**Absolute scores are deliberately never asserted.** They depend on the host, so a hard number would fail on a busy CI runner — which is the tool working, not a bug. Tests assert *shape* and *plausible range* only, and the CI range is wide on purpose.

If you add a flag it needs cases in three places: `a_usage_error_exits_nonzero_and_explains_itself` (invalid values), `usage_documents_every_flag_the_parser_accepts`, and `help_exits_zero_and_documents_every_flag`. The latter two are what stop a flag shipping undocumented. If you touch chain construction, the cycle-property tests must still hold.

## Dependencies

There are none, and adding one needs an argument that outweighs this: the probe gets baked into benchmark images and its readings are compared across months, so each dependency is another thing that can change what it measures without anyone noticing. Hence the hand-rolled CLI instead of `clap`, the `format!`ed JSON instead of `serde_json`, and the hand-rolled xorshift64\* RNG. `forbid(unsafe_code)` is crate-level: an instrument that could have UB is not trustworthy.

## Output contract

`--json` emits **one flat line**, shaped to merge straight into a benchmark run's `meta.json` beside the `instance_id`. Keep it single-line and unnested — `json_is_a_single_flat_object` checks both.

Field names are pinned by `json_field_names_are_pinned` and by CI, because downstream joins on stored scores break silently when one is renamed. Treat a rename as a measurement-affecting change. The keys are unnamespaced (`threads`, `accesses`, `elapsed_s`), so a flat merge assumes the consuming `meta.json` does not use those names for its own values — worth checking before wiring up a new consumer.

`version` is in the record on purpose: it is the only thing in the output that says which measurement semantics produced a number.

## CI

`.github/workflows/check.yml` runs four jobs. `test` and `probe` run on **both x86_64 and arm64** because characterising hardware is the whole point and the two have different cache hierarchies; `lint` and `msrv` are host-independent and run on one runner. `msrv` exists because `rust-toolchain.toml` pins `stable` while `Cargo.toml` claims 1.85 — without it, nothing would notice the claim going stale. Third-party actions are pinned by commit hash with a version comment; the workflow deliberately carries only `actions/checkout` and relies on `rust-toolchain.toml` as the single toolchain source of truth.

## Style

`rustfmt` with the committed `rustfmt.toml` (`use_small_heuristics = "Max"`), clippy pedantic with warnings denied. Comments explain *why*; in this crate the "why" is almost always "because otherwise the probe would measure the wrong thing" — name which thing.

---
> Source: [nh13/tachyon](https://github.com/nh13/tachyon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-08 -->
