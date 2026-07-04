## garlemald-client

> manages its own Wine runtime. Single Rust crate, edition 2024, Rust 1.95.

# AGENTS.md — agent guide for Garlemald Client

Instructions for AI coding agents (OpenAI Codex and other `AGENTS.md`-aware tools)
working in this repository. **Claude Code / Claude Agent SDK: see
[`CLAUDE.md`](CLAUDE.md)** — it is the twin of this file. Keep the two in sync when
you change one.

Read this before you write code. The deeper reference docs are in `docs/` (linked
throughout).

---

## What this is

Garlemald Client is a **cross-platform launcher for FINAL FANTASY XIV v1.23b** (the
original 2010 1.0 release, not A Realm Reborn). It detects an installed 1.x client,
patches it forward to client version **`2012.09.19.0001`** (CRC32-verified ZiPatch
apply), runs a WebView login against a private server, and launches `ffxivgame.exe` —
on macOS (incl. Apple Silicon), Linux, and Windows. On macOS/Linux it downloads and
manages its own Wine runtime. Single Rust crate, edition 2024, Rust 1.95.

---

## Build, run

```sh
cargo build --release
cargo run --release
```

- **Windows:** build the **32-bit** target — `cargo run --release --target i686-pc-windows-msvc`
  (it reads the suspended 32-bit `ffxivgame.exe` thread context to patch it).
- **Linux:** needs the GTK3 / WebKit2GTK / X11 / Wayland / GL dev libraries and
  system Wine; see [`docs/dev-environment.md`](docs/dev-environment.md).
- **macOS Apple Silicon:** needs Rosetta 2 at run time (the managed Wine engine is x86).

The login WebView runs as a subprocess: `cargo run -- --login-webview <url>` is the
launcher re-execing itself (`src/main.rs` → `login::run_webview`).

### Logging

- Launcher: `log` + `env_logger`, default `info` (`src/main.rs`). Override with
  `RUST_LOG`, e.g. `RUST_LOG=garlemald_client::patcher=trace`.
- Game under Wine (mac/linux): `<data_dir>/logs/wine.log`; raise `WINEDEBUG` via
  Developer Settings. Optional winsock packet tracing via the `ws2_32-proxy/` DLL.

### State (not in the repo)

Per-user dirs via `directories` `ProjectDirs::from("me","stegall","garlemald-client")`
(`src/config/paths.rs`): `config_dir()` holds `preferences.toml` (the selectable
server list is baked in from `src/servers/default_servers.toml`); `data_dir()` holds
the Wine `prefix/`, `runtime/` (macOS), and `logs/`. Reset = delete
those files; see [`docs/dev-environment.md`](docs/dev-environment.md).

---

## Where the modules live

Single crate, `src/`:

| Module          | Owns                                                                              |
|-----------------|----------------------------------------------------------------------------------|
| `app/`          | `eframe`/`egui` GUI — `launcher_window`, `patcher_window`, `settings_window`, `developer_window` |
| `servers/`      | server registry (`ServerDefinition { name, address, login_url }`; `default_servers.toml`) |
| `patcher/`      | patch download (`ureq`) + apply worker + `manifest.rs` (`PATCH_URL_BASE`)         |
| `patch_format/` | the ZiPatch format (decompress + apply file deltas)                               |
| `login/`        | the `--login-webview` subprocess + the `ffxiv://login_success?sessionId=` handshake |
| `crypto/`       | Blowfish encryption of the game launch arguments (`build_launch_arguments`)       |
| `launcher/`     | `GameLaunchRequest` + `launch_game` (`game_launch.rs`); PE patches (`pe_patch.rs`) |
| `platform/`     | per-OS: `windows.rs` (native Win32), `macos.rs`/`linux.rs` + `wine.rs` (managed Wine) |
| `config/`       | `paths.rs`, `preferences.rs`                                                      |
| `version.rs`    | launcher version + `FFXIV_BOOT_VERSION` / `FFXIV_GAME_VERSION`                    |

Architecture + the client↔server (WebView login / patch / launch handoff) flow:
[`docs/architecture.md`](docs/architecture.md).

---

## The license header (REQUIRED on every new Rust file)

Every `.rs` file (including `build.rs`, examples, tests) **must** open with this
header, verbatim — copy it from a sibling file. Markdown / TOML files carry none.

```rust
// garlemald-client — cross-platform launcher for FINAL FANTASY XIV 1.x private servers
// Copyright (C) 2026  Samuel Stegall
//
// This program is free software: you can redistribute it and/or modify
// it under the terms of the GNU Affero General Public License as published
// by the Free Software Foundation, either version 3 of the License, or
// (at your option) any later version.
//
// This program is distributed in the hope that it will be useful,
// but WITHOUT ANY WARRANTY; without even the implied warranty of
// MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
// GNU Affero General Public License for more details.
//
// You should have received a copy of the GNU Affero General Public License
// along with this program.  If not, see <https://www.gnu.org/licenses/>.
//
// SPDX-License-Identifier: AGPL-3.0-or-later
```

Porting code from a new upstream? Also credit it in [`NOTICE.md`](NOTICE.md).

---

## Branching & PR flow

- Branch off **`develop`** (the default/integration branch); never commit to
  `develop`/`main` directly. PR **into `develop`**.
- `develop` → `main` is the release path; releases are tag-driven (see
  [`docs/RELEASING.md`](docs/RELEASING.md)). Don't hand-bump the version — the release
  workflow owns `Cargo.toml`'s version.
- Link the issue in the PR.

---

## The gates your work MUST pass (CI runs them across Windows/macOS/Linux)

```sh
cargo fmt --all --check
cargo clippy --all-targets -- -D warnings
cargo build --all-targets --locked
cargo test --locked
```

`--locked` means: if you change dependencies, commit the updated `Cargo.lock`. **Do
not open a PR until these are green.**

---

## Hard rules

1. **AGPL header on every new `.rs` file** (above).
2. **Never cross-commit between repos.** This repo is one of several independent git
   repositories in a shared workspace (e.g. `Garlemald-Server`); commit here only.
3. **Keep CI green** before raising a PR.
4. **Branch off `develop`, PR into `develop`.**
5. **Respect the `platform/` abstraction** — OS-specific code (Wine, Win32, WebView,
   paths) lives behind it; you can't fully test another OS locally, so lean on CI.
6. Keep this file and [`CLAUDE.md`](CLAUDE.md) **in sync**.

---

## Reference docs

- [`docs/architecture.md`](docs/architecture.md) — pipeline, modules, client↔server flow.
- [`docs/dev-environment.md`](docs/dev-environment.md) — build/run, logging, state, local-server loop.
- [`docs/agents.md`](docs/agents.md) — running an agent on an issue (human-facing).
- [`CONTRIBUTING.md`](CONTRIBUTING.md) — the contribution workflow.
- [`docs/RELEASING.md`](docs/RELEASING.md) — branching & release automation.
- [`NOTICE.md`](NOTICE.md) — upstream attribution.

---
> Source: [swstegall/Garlemald-Client](https://github.com/swstegall/Garlemald-Client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
