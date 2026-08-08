## dn7-panel

> A single static Rust binary that manages a Linux host via an on-box web console

# DN7 Panel — agent guide

A single static Rust binary that manages a Linux host via an on-box web console
(monitoring, web terminal, containers, website, files). One crate at the root +
helper crates under `crates/`. No backend, no panel token, no runtime deps.

## Architecture constitution

- **Read [`ARCHITECTURE.md`](ARCHITECTURE.md) before any structural change.** It
  is the project constitution: layer responsibilities, the dependency direction
  (`web → app → core`; `infra → app → core`; `contracts` depends on nothing
  above `core`), the per-layer "禁止项" lists, and the file/function size limits.
- The rules are **enforced** by `tests/architecture.rs` (directory-level deny +
  module allowlist + a `core` serde whitelist). A PR that violates the
  dependency direction or the limits should fail review — don't add `arch-allow`
  exceptions without a tracked reason.

## Hard invariants

- **Zero external dependencies at runtime.** No shelling out to system programs
  (`ss`/`ip`/`useradd`/`systemctl`/…) and no `docker`/`nginx`/`openssl` CLI — all
  of it is pure Rust. The reverse proxy is the in-process edge (`crates/dn7-edge`,
  `infra::website` control plane), not external nginx.
- **Fully static musl binary.** Release builds must stay statically linked (no
  `DT_NEEDED`). Don't pull in C-linked crates.

## Versioning & releases

- **Single source of truth: [`release.toml`](release.toml).** It carries three
  independent knobs — `codename`, `version` (`<year>.<major>.<minor>`, Apple-style
  year = calendar + 1, so 2026 → `27`), and `build` — plus per-language `[notes]`.
  Display form everywhere is `<codename> <version> (build <build>)`, e.g.
  **`Phanes 27.0.0 (build 1)`**; the build number shows on every release.
- **`build` is an INDEPENDENT monotonic integer.** Bump it by hand for any
  published build; it never resets and is never derived from the version (so you
  never get an awkward `27G99 → …01`). Codenames advance one-per-year through the
  Greek primordials in cosmogonic order (Phanes → Chaos → Gaia → …).
- **CI (`.github/workflows/release.yml`) gates every push; publishes only on a
  build bump.** It reads `release.toml` and:
  - always runs fmt / clippy / tests (verify the push compiles);
  - publishes a GitHub Release **only when `build` moved** (no `b<build>` tag yet):
    **each build is its own release**, tagged `b<build>`, named `<codename>
    <version> (build <build>)`, **always Latest** (never a prerelease). Older
    builds' releases are retained (just no longer Latest), so every build stays
    downloadable.
- **Self-update pulls straight from GitHub releases**, racing several mirror
  "lines" (github direct + proxies, in `src/infra/support/fetch/`) and using the
  fastest reachable one. The panel reads the latest `(version, build)` from
  `releases.json` and pins the binary to that build's `b<build>` release; updates
  are compared by **(version, build)**, so a pure build bump reaches deployed
  panels. There is no separate distribution channel and no user-visible source
  picker.
- To cut a release: edit `release.toml` (bump `build`, and `version` for a real
  version; refresh `[notes]`) and push. Everything else is automatic. `codename` +
  `build` compile into the binary via `build.rs`; `version` rides in through
  `Cargo.toml` (`CARGO_PKG_VERSION`, stamped by CI). Keep the self-update tag
  format `v<version>` — `infra::support::fetch` parses the version out of it.

## Build & test workflow

- The Linux build/test runs in a **Lima VM named `dn7`** (`/work/panel` mount) —
  not on the macOS host. Another process may own the build; don't kick off
  `cargo build`/`test` blindly.
- Gate every change on: `cargo fmt && cargo clippy --workspace --all-targets --
  -D warnings && cargo test --workspace` (the `--workspace` is REQUIRED — without
  it the helper crates under `crates/` are skipped; CI runs clippy `-D warnings`).
- Touched the UI? Run `node scripts/check_i18n.js` (4 languages) from the repo
  root.
- For local foreground runs use `DN7_NO_GUARDIAN=1 dn7-panel panel`; never
  hand-run the top-level `dn7-panel` (it performs a real system install).

---
> Source: [Digital-Network-7/DN7-Panel](https://github.com/Digital-Network-7/DN7-Panel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
