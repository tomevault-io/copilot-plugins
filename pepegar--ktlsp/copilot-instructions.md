## ktlsp

> This repository is a Cargo workspace:

# ktlsp Agent Notes

## Repository Layout and Boundaries

This repository is a Cargo workspace:

- `crates/ktcore` contains reusable parsing, indexing, inference, resolution, and diagnostics logic.
- `crates/ktlsp` contains the editor-facing LSP server, workspace adapter, dependency/JDK indexing,
  and the `bench` support binary.
- `crates/ktcheck` is the command-line static-analysis frontend built on `ktcore`.

Put reusable semantic behavior in `ktcore` when it does not depend on LSP types or editor state.
Keep protocol conversion, capability advertisement, request tracing, and editor lifecycle behavior
in `ktlsp`. For Kotlin/Java parity work, follow `docs/architecture/language-abstraction.md`: prefer
shared normalized facts and one feature algorithm over adding more language branches to
`workspace`.

## Harness Expectations

When changing ktlsp behavior, use the scriptable editor harness before calling the work done. The
focused Rust tests are necessary, but they do not prove that a real editor can start ktlsp,
observe capabilities, send requests, and receive useful results. Use `cargo test --workspace` for
the broad Rust suite after focused iteration.

Use `dev/ktlsp-harness.sh` for editor-facing validation:

- `dev/ktlsp-harness.sh basic` — first smoke for initialization and local/cross-file goto in a
  disposable two-file project.
- `dev/ktlsp-harness.sh features` — use for references, completion, auto-import, member goto, and
  did-change behavior.
- `dev/ktlsp-harness.sh library` — use when dependency-source indexing, default imports, stdlib, or
  external library symbols may be affected.
- `dev/ktlsp-harness.sh java` — use for the broad Java editor surface, including navigation,
  symbols, completion, diagnostics, refactors, hierarchy, and visual features.
- `dev/ktlsp-harness.sh semantic` — use when validating result chains, receiver-style scope
  functions, alias-backed receivers, or KMP source-set narrowing.
- `dev/ktlsp-harness.sh project --root <dir> --file <file>` — use for an ad hoc health check on an
  existing Kotlin or Java project.
- `dev/ktlsp-harness.sh java-project --root <dir> --file <file.java> --token <symbol>` — use for
  Java library-source goto and references in an existing Gradle project; add `--occurrence N` when
  the default second textual occurrence is not the desired probe.
- `dev/ktlsp-harness.sh emacs-project --root <dir> --file <file>` — use for Eglot compatibility and
  repeated semantic-token, highlight, and diagnostic requests against an existing project.
- `dev/ktlsp-harness.sh gradle-live` — use for the focused `dev/gradle-sample` probe, including
  project and library inference, goto, and diagnostics through a real LSP client.
- `dev/ktlsp-harness.sh comprehensive` — use for the broad `dev/gradle-sample` sweep across stdlib,
  serialization, coroutines, Okio, project inference, goto, and diagnostics.

The Neovim scenarios require `nvim` on `PATH`; `emacs-project` requires `emacs`.

Each harness run prints an artifact directory under `/tmp/ktlsp-harness/`. Inspect
`artifacts/summary.txt` first, then `artifacts/<scenario>.out` for client assertions and captured
command output. For Neovim scenarios, `xdg-state/nvim/lsp.log` contains the editor LSP log and
ktlsp stderr. Request traces are written to `artifacts/trace-events.jsonl` and converted to
`artifacts/trace.json` when possible. If a request returns no result, search the JSONL trace for
`outcome:"empty"` and check the recorded file, cursor, and symbol.

## Adding New Features

For a new LSP feature or behavior slice:

1. Choose the correct boundary: reusable semantic behavior in `ktcore`; protocol and editor state
   in `ktlsp`; CLI-only policy in `ktcheck`.
2. Add focused Rust tests for the pure behavior where possible. For shared behavior, cover both
   Kotlin and Java inputs when both languages are affected.
3. Add or extend an editor probe when the feature is editor-visible.
4. Route that probe through `dev/ktlsp-harness.sh` as a scenario or part of an existing scenario.
5. Document which harness scenario proves the feature in `README.md`.
6. Verify the smallest relevant harness scenario plus any broader scenario affected by the change.

Prefer disposable projects under the harness run directory for small reproductions. Use committed
fixtures under `dev/` only when the fixture is broadly useful or expensive to generate.

## Process-wide CPU Profiling

Set `KTLSP_FLAMEGRAPH=/tmp/ktlsp.svg` when starting the real language-server binary to sample CPU
work across the full server lifetime. The profiler starts before the LSP service is constructed, so
it includes Tokio worker threads and background project, dependency, and JDK indexing—not only
request handlers or the synthetic benchmark.

For a reproducible large-project initialization profile, build the release binary and drive it
through the benchmark client, which waits for project and library indexing before shutting the
server down:

```sh
cargo build --release --bin ktlsp

KTLSP_CACHE_DIR=/tmp/ktlsp-perf-cache \
KTLSP_FLAMEGRAPH=/tmp/ktlsp-profile.svg \
python3 dev/bench_initialize.py \
  --binary "$PWD/target/release/ktlsp" \
  --root /path/to/project \
  --timeout 120
```

The SVG is finalized only after a clean LSP `shutdown`/`exit` sequence; killing the process may
leave no profile. Collect flamegraphs separately from timing baselines because sampling adds
measurable overhead. Use distinct cache directories when comparing cold and warm behavior, and
write generated profiles under `/tmp`, not into the repository.

## Debug/Cache Rules

- The harness already redirects `KTLSP_CACHE_DIR`, `KTLSP_TRACE`, and `XDG_STATE_HOME` into its run
  directory. Set `KTLSP_CACHE_DIR` explicitly for other scripted runs; do not write to the real
  `~/.cache/ktlsp` from agent sessions.
- Keep stdout reserved for LSP JSON-RPC. Logs belong on stderr, Neovim logs, or JSONL artifacts.
- Do not leave generated logs, traces, or temporary projects in the repository root.
- If `HOME` is redirected for a test, preserve real `CARGO_HOME` and `RUSTUP_HOME` unless the goal is
  specifically to test a cold Rust toolchain environment.

---
> Source: [pepegar/ktlsp](https://github.com/pepegar/ktlsp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
