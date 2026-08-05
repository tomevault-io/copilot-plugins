## metal2vulkan

> Operating guide for any AI agent working in this repository. Follow it unless the user

# AGENTS.md

Operating guide for any AI agent working in this repository. Follow it unless the user
explicitly overrides a rule for the current task.

## What this project is

**metal2vulkan** is a standalone Rust crate and CLI that translates **Metal AIR** (LLVM bitcode
or sanitized `.ll`) to **Vulkan SPIR-V** with a native emitter (no LLVM `llc` in the product path).

- **License:** LGPL-3.0-or-later (`LICENSE`)
- **Publishable crate:** `metal2vulkan` (library + `metal2vulkan` binary)
- **Optional workspace member:** `validation/` (`metal2vulkan-validation`, `publish = false`) —
  Metal oracle / Vulkan executor helpers and offline tools such as `spirv_delta` (uses `rspirv`
  there only; the product crate does **not** depend on `rspirv`)

This tree is **not** the paravirtual GPU monorepo. Do not reintroduce monorepo paths (`host/`,
`vm/`, `kb/`, `journal/`), device/protocol code, or private capture corpora.

**Orientation for humans and agents:**

| Doc | Role |
|---|---|
| `README.md` | Install, CLI, quick library use |
| `docs/ARCHITECTURE.md` | Pipeline, structurizer/relooper, retry cascade |
| `docs/REFLECTION.md` | Consumer binding metadata (`ShaderReflection`) |
| `docs/VALIDATION.md` | Developer validation ladder (harvest, drift, A/B) |
| `CONTRIBUTING.md` | Day-to-day build/test loop |
| `src/env_vars.rs` | Authoritative `METAL2VULKAN_*` registry (`--help`) |

## Components (start here)

| Path | Role |
|---|---|
| `src/` | Product library + CLI (`translate*`, native emitter, passes, reflect) |
| `tests/` | Integration tests for the product crate |
| `examples/` | Small runnable examples |
| `validation/` | Optional oracle/executor tooling (not published to crates.io) |
| `docs/` | Architecture and consumer guides |
| `scripts/` | Dev utilities (`mtlb-extract`, harvest, A/B, drift, SPIR-V grammar regen) |
| `.github/workflows/ci.yml` | Format, clippy `-D warnings`, serial tests on `ubuntu-26.04` + `macos-26` |

Crown-jewel code lives under `src/native/`, `src/passes/`, and `src/reflect/`. Prefer structural
fixes over one-off workarounds.

## Ground rules

### Translation honesty

- **Structure and semantics over names.** Decide emit/lowering from IR structure (types, storage
  classes, access chains, AIR metadata ABI) — **never** from a hardcoded function, type, variable,
  or corpus-style identifier. A name-keyed branch that green-lights one shader while failing
  identically shaped others is a defect.
  - **Allowed:** dispatch on **stable ABI symbols** that are part of the AIR/LLVM contract
    (`air.*`, `llvm.*` families). Prefer a structural test when one exists.
  - **Honest FALLBACK:** unsupported inputs return `Err` / CLI `FALLBACK`. Do not emit
    wrong-but-valid SPIR-V to silence a gate.
- **No env-gated product paths.** Product translation behavior must not branch on env vars. Keep
  `METAL2VULKAN_*` for operational settings, tool path overrides, and default-off diagnostics /
  measurement only (`src/env_vars.rs`).
- **Unknown stays unknown.** Do not invent semantics for unsupported opcodes, types, or layouts.
  Fail visibly and leave a test or doc note if the gap is real.
- **Fix causes, not observations.** Temporary probes may explain a failure; committed behavior
  needs a structural/ABI justification, not matching one workload’s bytes by special-case.

### Claims and evidence

- **A claim is only as broad as its evidence.** Verifying one path does not license “all / never /
  zero” statements. Scope replies, commits, and docs to what you actually ran.
- Say **verified** only for commands you executed and results you saw; use **expected** /
  **unaudited** for the rest.

### Measurement before polish

- Prefer the smallest change that can be **tested** (unit/integration, local A/B, or a synthetic
  `.ll` fixture). Do not ship third-party captured shaders or mined corpora.
- When fixing a bug class, add or update a **test** (or a clear measurable check) so it cannot
  regress silently.

### Tooling and temp files

- External tools on some paths: **`llvm-dis`**, **`spirv-val`**, and for validation **`spirv-diff`**
  / **`spirv-as`**. Resolve via PATH or `METAL2VULKAN_<TOOL>` overrides.
- Scratch files under the OS temp dir (or a caller-supplied `tmp`) must be **removed as soon as
  the tool no longer needs them**. The CLI removes its work directory on success and before
  `process::exit` on FALLBACK. Do not reintroduce long-lived dumps under fixed `/tmp/...` paths.
- FALLBACK **repro bundles** under `$TMPDIR/metal2vulkan-repros` (or `METAL2VULKAN_REPRO_DIR`) are
  intentional and may be kept for debugging.

### What not to commit

- Apple-owned binaries, guest disk images, mined metallib/AIR corpora, or private golden sets
- Contents of `validation/corpus/local/`, generated `validation/tests/corpus_*.rs`, raw `*.metallib`
  / `*.air` / `*.spv` dumps (all gitignored — see `validation/corpus/README.md`)
- `target/`, `Cargo.lock` (gitignored for this library-first tree), `.cache/`
- Name-keyed special cases “just to pass one case”

**Allowed (committed):** synthetic fixtures under `validation/fixtures/public/`;
`validation/corpus/drift-ledger.jsonl` (AIR + SPIR-V hashes); `tolerances.jsonl` and
`broken.jsonl` (AIR fingerprints + tolerance / skip metadata — **no** shader bodies).

## Layout & ownership

- **Product logic** stays in the `metal2vulkan` crate (`src/`).
- **Optional offline/oracle work** stays in `validation/` (may depend on `rspirv`, MoltenVK, a
  Vulkan ICD). Do not pull that into the published crate’s default dependency graph.
- **Grammar tables** under `src/spirv_binary/*_generated.rs` are regenerated via
  `scripts/regen-spirv-grammar/` from Khronos SPIRV-Headers + rspirv-autogen; do not hand-edit.
- **Reflection** for consumers: `src/reflect/` + `docs/REFLECTION.md`. Binding numbers must stay
  aligned with the interface pass ABI (set 0; bases 0 / 32 / 64 / 96).

## Build, test, CI

Always run Rust tests **serially**:

```sh
cargo fmt --all
cargo clippy --workspace --all-targets -- -D warnings
cargo test -p metal2vulkan -- --test-threads=1
cargo test -p metal2vulkan-validation -- --test-threads=1
```

- **MSRV:** see `rust-version` in `Cargo.toml` (currently 1.87).
- **Warnings are hard failures.** CI sets `RUSTFLAGS=-Dwarnings` and runs clippy with
  `-D warnings`. Treat any new rustc or clippy warning as a bug you must fix before committing —
  do not leave `#[allow(...)]` noise to paper over real issues, and do not ask reviewers (or a
  later CI run) to clean up after you.
- **Features:** `serde` enables `ShaderReflection` JSON and CLI `--emit-meta`.
- CI runners: **`ubuntu-26.04`** and **`macos-26`** (not `*-latest`).

When diagnosing a hang, prefer reading full `cargo test` output over piping through `head`/`grep`
(easy to hide a later failure).

## Code style

- **Format before every commit.** Run `cargo fmt --all` (repo `rustfmt.toml`) on the whole
  workspace so the diff you commit already matches what CI’s `fmt --check` expects. If a change
  is pure formatting, keep it in its own commit when mixed with logic would obscure the review.
- **Clippy must be clean.** Before you commit Rust edits, run
  `cargo clippy --workspace --all-targets -- -D warnings` and clear every finding. Match
  surrounding module style; prefer small, reviewable files and structural fixes over growing
  special-case tables.
- Public API and CLI changes should update `README.md` / `docs/` when consumer-visible.

## Git workflow

- Prefer **focused commits** with a clear subject and body (what / why / how verified).
- **Pre-commit gate (Rust):** `cargo fmt --all`, then clippy with `-D warnings` as above. Do not
  commit unformatted code or known clippy/rustc warnings; CI will fail the same checks.
- Do not force-push shared history unless the user asks.
- Do not commit secrets, large binaries, or lockfiles that this repo intentionally ignores.

## The loop (unit of work)

1. **Scope** — smallest testable increment; name how you will verify it.
2. **Validate** — read the relevant code and `docs/`; do not assume monorepo facts.
3. **Change** — implement; keep product paths free of env gates and name keys.
4. **Test** — serial `cargo test` for the packages you touched; full workspace clippy with
   `-D warnings` when Rust sources changed.
5. **Document** — update `docs/` or comments only when the durable contract changed.
6. **Commit** — `cargo fmt --all` first, then one concern per commit when practical. Only commit
   when fmt and clippy are clean.

**Done** = it works under the checks above, claims match evidence, fmt/clippy are green, and the
tree stays publishable as a general AIR→SPIR-V translator.

---
> Source: [steelbrain/metal2vulkan](https://github.com/steelbrain/metal2vulkan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
