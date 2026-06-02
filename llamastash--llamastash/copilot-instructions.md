## llamastash

> This file provides project-level guidance to coding agents (Claude Code, OpenCode, Codex, Copilot CLI) working in this repository. Treat it as authoritative alongside `CONTRIBUTING.md`; on conflict, prefer this file's specifics.

# AGENTS.md

This file provides project-level guidance to coding agents (Claude Code, OpenCode, Codex, Copilot CLI) working in this repository. Treat it as authoritative alongside `CONTRIBUTING.md`; on conflict, prefer this file's specifics.

## Source of truth

The implementation plan is the canonical design document:

- `docs/plans/2026-05-13-001-feat-llamatui-v1-launcher-plan.md` — v1 architecture, security contract, the nine Implementation Units (1: scaffold, 2: daemon/IPC, 3: GGUF, 4: discovery, 5: launch/supervisor, 6: TUI shell, 7: right-pane tabs, 8: non-interactive CLI, 9: release scaffolding), and what is explicitly out of v1.
- `docs/plans/2026-05-18-001-feat-init-wizard-doctor-pull-plan.md` — v2 plan covering R48–R80: init wizard, doctor diagnostic, `pull` MVP, recommender, fetch contract, snapshot bundling.
- `docs/brainstorms/llamatui-requirements.md` — origin requirements (R1–R46) that the v1 plan traces to.
- `docs/brainstorms/2026-05-18-init-wizard-requirements.md` — origin requirements (R48–R80) for v2.
- `docs/spikes/2026-05-19-*.md` — pre-implementation spike findings that anchor v2's design (hf-hub injection, GH Releases asset contract, brew Linux bottle, VRAM overhead).
- `docs/architecture.md` — stable user-facing summary of what's actually in the binary.

Before any non-trivial change, identify which Implementation Unit it falls under. PR descriptions should cite the unit; commit subjects often use `feat(unit5):` / `fix(unit3):` style.

## TODO tracking

`TODO.md` at the repo root is the single index of outstanding work. Any time
you add a TODO somewhere — a `TODO(...)` / `FIXME` comment in code, an
unchecked `- [ ]` in a plan or doc, a `todo:` frontmatter field on a spike,
or a deferred follow-up surfaced during review — also add a one-line entry
in `TODO.md` that links back to the source location. When you complete a
TODO, strike it from both places in the same change. The goal is that
`TODO.md` alone tells you everything still open without grep-walking the
tree.

## Docs stay in sync with code

Docs and code ship together. After any change that alters user-visible
behavior, the CLI / IPC surface, configuration shape, install paths, exit
codes, dependencies, scope boundaries, or architecture, update the affected
docs in the **same change** (same commit, same PR). Treat a PR that ships
code without the matching doc update as incomplete.
Agents working on this app must always keep core docs in sync (README,
AGENTS.md, feature docs, CHANGELOG, usage docs, install docs, and adjacent
user/developer docs touched by the change).

Files to review for drift on every change — skip the ones a change doesn't
touch, but check before assuming:

- `README.md` — install, quickstart, screenshots, feature list, exit-code
  table when present.
- `AGENTS.md` (this file) — scope boundaries, exit-code table, CLI agent
  surface, `status` IPC fields, build/test/lint, common gotchas.
- `INSTALL.md` — installer paths, prerequisites, and installation flows
  across supported environments.
- feature docs (`docs/brainstorms/*requirements*.md`, `docs/plans/*.md`,
  and feature-focused sections in `README.md` / `docs/usage.md`) — keep
  feature scope, status, and user-facing behavior aligned with shipped code.
- `CHANGELOG.md` — noteworthy user-visible changes land an entry under
  `[Unreleased]` (or the active release section). Internal-only refactors
  can be omitted. Entries must be **short, human-scannable one-liners** —
  not every small change earns a bullet, and bullets must not carry
  implementation detail or noise. Bundle related small changes into a
  single entry where it reads better, and link to the PR / commit (e.g.
  `(#123)` or short SHA) for anyone who wants the full story.
- `CONTRIBUTING.md` — workflow / contribution rules when they shift.
- `SECURITY.md` — only when the threat model or hardening surface shifts.
- `docs/architecture.md` — when modules, the IPC shape, lifecycle states,
  or the data-flow diagram change.
- `docs/usage.md` — CLI subcommands, flags, JSON output shapes,
  configuration keys, keybindings.
- `docs/troubleshooting.md` — new failure modes / error messages that an
  end user might hit.
- `docs/plans/*.md` — tick the corresponding unit checkbox `[ ]` → `[x]`
  when the work lands; never invent retro-plans, but do keep checkboxes
  accurate.
- `config.example.yaml` — when a config key is added, removed, renamed,
  or its default changes.
- `Cargo.toml` — keywords / categories / description on any feature that
  changes the binary's positioning.
- `TODO.md` — per the section above.

If a change makes an existing doc statement wrong, fix or remove the
statement; don't leave the contradiction for the next reader. If you
introduce a new user-facing concept that none of the above docs cover yet,
pick the doc closest in scope and add a section there rather than spawning
a new file.

## Scope boundaries

The v1 contract — these are deliberate omissions, not gaps:

- **Loopback-only, same-UID.** The daemon binds two loopback TCP listeners on `127.0.0.1`: a JSON-RPC control plane on `:11436` (bearer-token authed; the token + URL land in `$XDG_STATE_HOME/llamastash/runtime.json`, mode `0600`) and the OpenAI-compat proxy (see next bullet). Neither listener accepts LAN traffic in v0.0.2. `--host` / `--listen` / `--bind` / `--api-key` / `--ssl-*` are refused if passed via `advanced[]` to `start_model`, and `LLAMA_ARG_*` env vars are stripped before spawn.
- **OpenAI-compat proxy carved out of the v1 R34 deferral.** A loopback HTTP/1.1 listener is enabled by default. In normal mode it prefers `127.0.0.1:11435` so a local Ollama daemon on `11434` can coexist; in Ollama-compat mode it prefers `127.0.0.1:11434`. It speaks `/health`, `/v1/models`, `/v1/chat/completions`, `/v1/completions`, `/v1/embeddings`, `/v1/rerank` so OpenAI-compatible agents (OpenCode, Pi, OpenAI SDKs) attach via one stable URL. Same-machine threat model — no auth, no TLS, no LAN binding, no peercred (the listener is plain loopback HTTP, not the IPC socket). The rest of R34 — Anthropic `/v1/messages`, MCP, network exposure, auth/TLS, idle eviction, fallback tuning — stays deferred. See `docs/plans/2026-05-21-001-feat-proxy-router-plan.md`.
- **`llamastash pull`** graduated in v2 from the v1 `unimplemented!` shim. MVP shape: `llamastash pull <owner/repo[:filename.gguf]>` — downloads via the [`hf-hub`](https://crates.io/crates/hf-hub) crate (0.5 line, resolves the same `reqwest 0.12` we pin elsewhere) into the canonical HF cache layout that discovery already scans. The TUI's `d` HuggingFace pull dialog is the interactive face of this primitive; the CLI `llamastash pull <slug>` stays the only non-TUI browse surface (the dialog is TUI-only, no HTTP / MCP equivalent in v2). The `cli/output.rs::list_json` / `favorites_json` / `CatalogRow::name` JSON shapes stay byte-stable.
- **`llamastash init` / `llamastash doctor`** are v2 surfaces. Init is the first-run wizard + maintenance tool; doctor is the read-only diagnostic. Both honor `--json` per the v2 plan §"init/doctor mode/flag decision matrix". `init` is **interactive by default**; agents that need non-interactive runs pass `--recommended` (`--yes` remains a hidden alias with identical behavior, and both flags can be combined) and may pre-answer individual prompts with `--install`, `--model`, and `--config-step`.
- **CLI color policy.** Every human-readable output uses ANSI colors when stdout is a TTY, `NO_COLOR` is unset, and `--no-colors` was not passed. Any one of those three conditions silences color. `--json` output is byte-stable regardless of the color policy. Padded report tables (`list`, `status`, `presets list`, `favorites list`, `last-params`, `daemon status`) are TTY-gated by the same three off-conditions: when piped or color-disabled, every command emits the same `\t`-separated rows as before so `awk -F\t` / `column -t` pipelines keep working. Agents pin against `--json`, not the TTY rendering.
- **Single binary, three roles.** The TUI, CLI, and daemon are all `llamastash`. Daemon spawns on demand when TUI/CLI attach and find the socket missing.
- **Catppuccin Macchiato is the default theme.** Five themes ship total (Macchiato, Latte, Gruvbox Dark, Solarized Dark, Monochrome). Themes are hard-coded palettes; no dynamic loading.

## Build, test, lint

```bash
cargo build                                                # release: cargo build --release
cargo test --features test-fixtures                        # full suite — required for CI parity
cargo test --features test-fixtures --test <name>          # one integration binary
cargo test --features test-fixtures <substring>            # filter by test name
cargo fmt --all -- --check
cargo clippy --all-targets --features test-fixtures -- -D warnings
make audit                                                 # maintainer audit bundle: Outputs to `target/audit`. Tests + release build + deps/security/unsafe/coverage artifacts
make audit-summary                                         # headline summary from `target/audit`
```

Look at the `Makefile` for more commands, including `make uat-*` for the manual UAT runs.

`--features test-fixtures` is required for the integration suite. It enables:

- the `fake_llama_server` binary (`tests/fixtures/fake_llama_server.rs`) that integration tests spawn instead of a real `llama-server` — answers `/health`, `/v1/models`, streaming `/v1/chat/completions`, `/v1/embeddings`, `/v1/rerank`, with deliberate failure-injection markers in request bodies.
- the `_test_sleep` IPC method used by drain-timeout tests (never exposed in release builds because the feature is opt-in and not in the default set).
- `src/gguf/test_fixtures` (`FixtureBuilder`, `build_minimal_gguf`).

`--features uat` enables the maintainer-only `llamastash uat` subcommand (hidden from `--help`; the release binary on crates.io and Homebrew bottles never ships it). The orchestrator drives a 5-step real-hardware lifecycle and emits a structured JSON report — see [`docs/testing/hardware-uat.md`](docs/testing/hardware-uat.md) for setup and run instructions. The release workflow audits that `--features uat` is never enabled in shipped binaries, while its pre-build `release-gate` job runs cold CPU-only UAT on Linux and macos.

Two-space indentation is enforced by `rustfmt.toml`. Clippy denies `shadow_unrelated` crate-wide; rename rather than reuse `let` bindings inside the same scope.

## End-to-end CLI validation (required for user-visible changes)

A passing `cargo test` is necessary but **not sufficient**. After any change to a CLI subcommand, IPC response shape, daemon lifecycle, TUI panel, or anything else a user would notice, **actually run the CLI** against a real daemon and verify the behavior with your own eyes. Test suites can pass while the binary is broken — stale daemons, missed env vars, deferred restarts, schema drift between client and server all hide behind green CI.

Minimum E2E loop after any user-facing change:

```bash
cargo build --bin llamastash
# If a daemon is already running from an older binary, kill it first —
# the running process is using a deleted binary and won't pick up your
# changes until restarted:
target/debug/llamastash daemon stop                     # or: --force when stale
target/debug/llamastash daemon start                    # backgrounds + writes runtime.json
target/debug/llamastash daemon status --json | jq .     # shape sanity-check
target/debug/llamastash list                            # the change you just made
target/debug/llamastash status --json | jq .daemon      # confirm new IPC fields
target/debug/llamastash                                 # TUI: pan through every visible panel
```

For TUI changes specifically, **launch the TUI and look at the panel you touched** — golden snapshots catch byte-exact regressions but not "the field is empty in real life because the running daemon doesn't surface it yet." A fresh daemon restart is part of the validation.

When E2E surfaces a regression the test suite missed (stale daemon, missing IPC field, wrong port, etc.), add a regression test before fixing — that's the gap the suite needs covered.

## Running the daemon locally

```bash
cargo run -- daemon start                # foreground; logs to terminal, Ctrl-C to stop
cargo run -- list                        # in another terminal
cargo run                                # opens the TUI against the same daemon
cargo run -- daemon stop
```

Attach surface: clients read `$XDG_STATE_HOME/llamastash/runtime.json` (mode `0600`) to discover the daemon's control-plane URL + bearer token. The file is `{"schema_version":1,"ipc_url":"http://127.0.0.1:<port>","ipc_token":"<bearer>","started_at_unix":<ts>,"daemon_pid":<pid>}` — the URL/token live under `ipc_url` / `ipc_token`. Note that under a `LLAMASTASH_STATE_DIR` override the file sits directly in that dir (no `llamastash/` subdir). Override the state directory with `LLAMASTASH_STATE_DIR=/path` for side-by-side daemons, or use `LLAMASTASH_IPC_URL` + `LLAMASTASH_IPC_TOKEN` (both required together) for clients that don't want to read runtime.json. If wedged, deleting `runtime.json` and `daemon.pid` in the state dir is safe — next `daemon start` rebinds clean.

For full path isolation (e.g. integration tests, the maintainer UAT command, side-by-side daemon experiments), pair `LLAMASTASH_STATE_DIR` with `LLAMASTASH_CONFIG_DIR`, `LLAMASTASH_CACHE_DIR`, and `HF_HOME` so state, config, cache/logs, and the HF cache all redirect together. Each variable is a verbatim override; empty values are treated as unset. See `docs/usage.md §Environment variables`.

## Architecture in one breath

```
TUI / CLI ──HTTP+Bearer──► Control plane (127.0.0.1:11436, loopback, bearer token)
OpenCode / Pi / SDK ──HTTP──► Proxy listener (127.0.0.1:11434, loopback, no auth)
                          │
                          ├── Discovery (scan + watch + caches)
                          ├── GGUF parser (metadata + identity)
                          ├── Process supervisor (spawn / probe / stop)
                          ├── Resource monitor (RAM/VRAM/CPU)
                          └── Persisted state (favorites / presets / running)
```

- **Wire format.** JSON-RPC 2.0 envelopes carried in `POST /rpc` request/response bodies. `src/ipc/methods.rs` is the dispatch table; `src/daemon/control_plane.rs` is the hyper service in front of it.
- **Model lifecycle.** `Launching → Loading → Ready → Stopping → Stopped`, plus `Error{cause}`. Transitions are guarded — once Stopping or Error, the model never moves out. The supervisor health-probes `/health` every 500 ms during Loading. See `src/daemon/supervisor.rs`.
- **Process survival.** `llama-server` children get their own session via `setsid`, so they outlive the daemon. On daemon restart, an orphan sweep re-adopts each entry in `state.running` only after three-factor confirmation: PID alive, recorded port answering, and `/v1/models` advertising the recorded model. The identity match accepts either the full canonical path (older llama-server echoed the `-m` value) or a bare basename (b9245+ reports only the file name as `id`); a *differing* full-path id is still rejected as the PID-reuse guard.
- **Model identity.** `(canonical absolute path, BLAKE3 of header bytes)`. Renames survive; symlinks dedupe to target; split GGUFs collapse to shard 1.
- **Persistence.** `$XDG_STATE_HOME/llamastash/state.json` (durable user state: favorites, presets, last-params, running snapshot) plus `runtime.json` (per-instance URL + bearer token, deleted on shutdown). Both written via the shared `util::atomic_write::write_secure` path (`*.tmp.<rand>` + `fsync` + atomic rename + parent dir `fsync`), mode `0600` on Unix. Parse failure on `state.json` → `state.json.broken-<ts>` quarantine, boot with defaults.

## CLI agent surface (Units 8 + 10/13)

Every read-and-mutation command supports `--json` and emits a wrapped object: `{"models":[…]}`, `{"favorites":[…]}`, `{"presets":[…]}`, `{"last_params":[…]}`, `{"stopped":[…],"count":N}`, `{"steps_ran":[…],"install":{…},"model":{…},"config":{…},"smoke":{…},"hardware":{…}}` for `init`, `{"schema_version":1,"findings":[{"id":…,"severity":…,"message":…,"fix_hint":…,"safe_to_log":true}],"baseline":{…}}` for `doctor`. Stable shapes for agent consumption. Exit codes follow `<sysexits.h>` numerically but with project-specific meanings — pin against the table in `src/cli/exit_codes.rs`, not the libc constants. `stop --all` in a non-TTY context refuses without `--yes`. The IPC `capabilities` method enumerates supported methods so clients can feature-detect.

### Exit-code table

| Code | Constant               | Meaning                                        |
| ---- | ---------------------- | ---------------------------------------------- |
| 0    | `SUCCESS`              | Success                                        |
| 64   | `USAGE`                | Bad CLI usage (clap rejection)                 |
| 65   | `DAEMON_UNREACHABLE`   | Daemon socket missing / timeout                |
| 66   | `MODEL_NOT_FOUND`      | Model reference matched zero or multiple       |
| 67   | `LAUNCH_FAILED`        | `start_model` accepted but supervisor failed   |
| 68   | `STOP_FAILED`          | `stop_model` / `stop_all` returned an error    |
| 69   | `PULL_FAILED`          | Standalone `llamastash pull` failed            |
| 70   | `BINARY_NOT_FOUND`     | `llama-server` not on PATH / config            |
| 71   | `UNKNOWN`              | Catch-all (anyhow bubble-up)                   |
| 72   | `INIT_ABORTED`         | Init pre-smoke abort (integrity / daemon stop) |
| 73   | `INIT_DOWNLOAD_FAILED` | Init's model-download step failed              |
| 74   | `INIT_SMOKE_FAILED`    | Init reached smoke but probe didn't pass       |

`llamastash uat` (maintainer-only, `--features uat`) emits a parallel
set of synthetic codes inside its JSON report's
`failure_summary.exit_code` — `10` (preflight backend mismatch), `11`
/ `12` / `13` (smoke HTTP / parse / status), `124` (timeout), `130`
(SIGINT). Full table in [`docs/testing/hardware-uat.md`](docs/testing/hardware-uat.md) §UAT synthetic exit codes.

### `status` IPC fields (kdash-style dashboard wiring)

The `status` method response carries the following top-level objects beyond the legacy `models` / `external` / `gpu` shapes:

- `host` — always an object (no `null`). Populated by the daemon's host-metrics sampler at 1 Hz. Fields: `cpu_pct` (f32, 0..=100 mean across cores), `ram_used_bytes` / `ram_total_bytes` (u64), `gpu_util_pct` / `gpu_mem_used_bytes` / `gpu_mem_total_bytes` / `gpu_temp_c` (each `Option`, omitted on backends that don't surface them), `gpu_backend` (string), `gpu_device_count` (u32).
  - `gpu_backend` values: `"cpu_only"`, `"nvidia"`, `"amd"`, `"apple_metal"`, `"unknown"` (Vulkan-only fallback), or the sentinel `"unsampled"` returned in the brief window between daemon start and the sampler's first tick. Clients gating UI on backend kind should treat `"unsampled"` as "not yet known", not as a real reading.
- `daemon.build` — semver string from `CARGO_PKG_VERSION`; matches `--version`.
- `daemon.server_path` — absolute path to the `llama-server` binary the daemon resolved at startup. `null` when unset.
- Per-model rows in `models[]` carry `latest_rss_bytes: Option<u64>` and `latest_cpu_pct: Option<f32>` from the per-launch resource sampler. Both are `None` until one tick (~1 s) after launch.
- `proxy` — `{enabled: bool, listen: Option<String>, status: "disabled" | "listening" | "port_in_use" | "unbound", bind_error: Option<String>}`. `listen` is the attempted address (`"127.0.0.1:<port>"`) on every state except `disabled`, where it is `null`. `bind_error` is non-null only on `unbound` (unexpected bind failure beyond port-in-use).

`status.gpu` is **live**: when the host-metrics sampler is attached, it reflects the freshest GPU probe. Late driver loads / hotplug changes propagate within one sampler tick rather than staying pinned to the boot snapshot.

All of these fields land in the CLI's `status --json` output too (`src/cli/output.rs::status_json`), so agents that consume the CLI surface get the same view as raw IPC clients.

## Conventions

- Less jargons and more straightforward facts and numbers. Keep jargon only if it helps marketing or understanding.
- Until first release is made, do not add code/docs etc for backward compatibility/legacy etc.
- Prefer using commands from the Makefile for common tasks (`make build`, `make test`, `make lint`) to internalize the standard flags and avoid mistakes like forgetting `--features test-fixtures` on tests.
- Conventional-commit prefixes: `feat:`, `fix:`, `refactor:`, `test:`, `docs:`, `chore:`. Unit-scoped variants are common (`feat(unit8): …`).
- Inline `#[cfg(test)] mod tests` per file is the default; integration tests under `tests/` for daemon-spawning scenarios.
- Comments explain **why**, not **what**. No multi-paragraph doc blocks unless the constraint is genuinely non-obvious. Don't reference task IDs or PR numbers in comments — those rot.
- No `#[allow(...)]` without a one-line reason.
- **Keybinding labels are never hardcoded in UI.** Help bars, footers, hints, popup affordances ("Press `q` to quit", `[Ctrl+S] save`, etc.) must derive their key text from the active `KeyMap` (`src/tui/keybindings.rs` — `Binding::label` / `Binding::description`), never from inline string literals. The keymap is the single source of truth so that user overrides from the config file are reflected everywhere the binding is surfaced. When adding a new action: add it to the `Action` enum and the appropriate `*_BINDINGS` slice with a `label`/`description`, then look that binding up at render time (e.g. via `KeyMap::bindings_for(Focus)` or a focused helper) — do not duplicate the literal key string in the widget.

## Protected artifacts

Do not flag these for deletion or `.gitignore` during reviews — they are part of the engineering record:

- `docs/brainstorms/*` — origin requirements.
- `docs/plans/*.md` — implementation plans (living docs with progress checkboxes).
- `docs/solutions/*.md` — solution memos when present.
- `docs/benchmarks/*` — methodology doc, results pages, and the raw per-host run JSONs under `runs/` and `overhead/`. These are the published evidence behind the README's positioning claims; deleting or rewriting prior dated pages destroys the reproducibility contract documented in `docs/benchmarks/methodology.md`.
- `.context/compound-engineering/ce-review/*` — multi-agent review run artifacts.

## Built-in defaults table maintenance

The static `(architecture, gpu_backend) → TypedKnobs` defaults table
lives in `src/launch/defaults_table.rs`. When `data/benchmark-snapshot.json`
adds a new recommender pick, audit the table coverage:

- Architectures listed in the snapshot but missing from `COVERED_ARCHS`
  fall through to the conservative `*` row (which only seeds
  `n_gpu_layers: 99` on GPU backends).
- `FLASH_ATTN_ELIGIBLE` is opt-in only — extend it once measurement
  confirms a new architecture supports flash-attn cleanly on NVIDIA
  / Apple Metal. AMD/HIP flash-attn coverage stays uneven; leave to
  user override via `config.yaml arch_defaults`.
- Folklore-only flags (`mlock`, `no_mmap`, KV-cache quant types) stay
  unset at the table level until measurement supports them.

A TODO entry tracks the AMD/HIP `no_mmap` measurement follow-up.

## Common gotchas

- The CLI/TUI/daemon are one binary. `cargo run -- daemon start` and `cargo run` (TUI) attach to the same daemon via `runtime.json` (URL + bearer token) under the state dir — running two `cargo run` invocations in parallel without distinct `LLAMASTASH_STATE_DIR` will both attach to the same daemon.
- Integration tests bind to a temp dir per test (`unique_temp_dir(label)`); never share `state_dir` between tests, or they'll race the lockfile.
- `cargo build` (without `--features test-fixtures`) intentionally omits `fake_llama_server` and `_test_sleep`. CI runs both with and without the feature to catch accidental dependencies on test-only surface.
- `cargo install` artifacts deliberately exclude `src/gguf/test_fixtures` and the `_test_sleep` IPC method via feature gating — don't move them out from behind `#[cfg(any(test, feature = "test-fixtures"))]`.
- Release pipeline runs `publish-homebrew`, `publish-site`, and `publish-cargo` in parallel after the upstream build matrix; a single-job failure leaves channels diverged. Recovery is to re-run the failed job from the Actions UI (or `gh run rerun --failed <run-id>`). Pre-release tags (`vX.Y.Z-<suffix>`) skip all three downstream jobs by design so dry runs never write to external repos.
- `LLAMASTASH_BENCH_DISABLE_DEFAULTS=1` is a bench-internal escape hatch read by `src/launch/params.rs::resolve_layered`. When set, the resolver collapses to "User-labeled layers only" — preset/last-used/yaml-arch/built-in-arch defaults all skip. `scripts/bench/` sets it so `llamastash start` produces byte-identical argv to raw `llama-server` for Suite-A overhead comparison. Never set this in production runs; it disables the auto-tuning the launcher exists to do.

## Release & distribution

- Steady-state contract: `git tag vX.Y.Z && git push --tags` triggers `.github/workflows/release.yml`, which first runs `release-gate` (tests + cold CPU-only UAT on Linux and macos), then builds 4 target tarballs, uploads release assets, pushes the new Homebrew formula to `llamastash/homebrew-llamastash`, pushes the verified `install.sh` mirror to `llamastash/llamastash.github.io`, and publishes to crates.io. The full pipeline takes ~10-15 minutes on cold caches plus the pre-build validation time.
- First-time setup (creating org repos, minting tokens, configuring Pages) lives in [`docs/runbooks/release-0.0.1-bootstrap.md`](docs/runbooks/release-0.0.1-bootstrap.md) — every step has a `gh` CLI primitive.
- Pre-tag CI guards in `release-readiness` (ci.yml) catch most release-breaking PRs before tag time: `cargo publish --dry-run --locked`, crates.io name-availability against a publisher allowlist, CHANGELOG `[Unreleased]` header presence, Cargo.toml ↔ CHANGELOG version alignment, packager.py unit tests.
- Action SHA-pinning policy: trust-critical actions in release.yml (those alongside secrets) are pinned to commit SHAs; first-party `actions/*` are tag-pinned. Updates flow through Dependabot PRs (`.github/dependabot.yml`).

---
> Source: [llamastash/llamastash](https://github.com/llamastash/llamastash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
