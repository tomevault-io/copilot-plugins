## sonicdpi

> Guidance for Claude when working in this repository. Keep this file accurate when the project structure, build steps, or invariants change.

# CLAUDE.md

Guidance for Claude when working in this repository. Keep this file accurate when the project structure, build steps, or invariants change.

## What this project is

**SonicDPI** is a cross-platform DPI-bypass engine for YouTube and Discord (including Discord voice / UDP RTP), written in Rust and dual-licensed MIT OR Apache-2.0. It is a userspace + kernel-hook hybrid: an OS-specific interceptor catches packets at the network layer and a platform-agnostic engine decides what to do with them (split, fake, reorder, drop).

The repo is a single Cargo workspace (`Cargo.toml` at the root) with four crates under `crates/`.

## Crate layout and responsibilities

```
crates/
├── sonicdpi-engine/     ← platform-agnostic core. NO OS-specific code.
├── sonicdpi-platform/   ← per-OS packet interception (WinDivert / NFQUEUE / pf rdr-to).
├── sonicdpi-cli/        ← `sonicdpi` binary: clap subcommands, service install, profile loader.
└── sonicdpi-tray/       ← `sonicdpi-tray` binary: end-user system-tray on/off toggle.
                            Spawns sonicdpi.exe as a child — does NOT load WinDivert in-process.
```

Layering rule: **`engine` knows nothing about packets-on-the-wire mechanics, `platform` knows nothing about strategies, `cli`/`tray` glue them.** If you find yourself adding a `cfg(target_os = ...)` to the engine, you're in the wrong crate.

### sonicdpi-engine — the parts to know
- `Engine` (lib.rs) — top-level entry the platform backend talks to. Owns `FlowTable`, `StrategyPipeline`, `TargetSet`, `DnsCache`, `Profile`.
- `flow.rs` — `FlowKey` / `Flow` / `FlowTable`. Per-connection state. Hot path.
- `strategy.rs` — `Strategy` trait + `StrategyPipeline`. The five built-in strategies are: `tls-multisplit`, `fake-multidisorder`, `hostfakesplit`, `quic-fake-initial`, `discord-voice-prime`.
- `tls.rs` / `dns.rs` — minimal protocol parsers (ClientHello SNI extraction, DNS A/AAAA cache for SNI→IP correlation).
- `packet.rs` / `builder.rs` — IPv4/IPv6 + TCP/UDP packet representation and rebuild via `etherparse` with correct checksum recomputation.
- `fakes.rs` / `embedded_fakes.rs` — fake TLS ClientHello / QUIC Initial / STUN / Discord-shape payloads, generated with per-run randomness so the binary isn't fingerprintable by static signature.
- `fooling.rs` — TCP TTL/MD5SIG/badseq/badsum/timestamp tricks for fake-packet decoys.
- `proxy.rs` — userspace transparent-proxy mode used by macOS and the `Probe` CLI subcommand.
- `profile.rs` — TOML profile schema + `build_pipeline()`. Three built-ins: `youtube-discord`, `youtube-discord-seqovl`, `youtube-discord-hostfakesplit`.

### sonicdpi-platform — backends behind the `Interceptor` trait
- `windows.rs` — WinDivert. Uses the **EV-signed** WinDivert 2.2.2 binaries from `vendor/windivert/x64/` (see `.cargo/config.toml`). Do not switch to the `vendored` feature of `windivert-sys` — that crate's `cl.exe`-built dll has a stack-overflow bug in its filter parser. Build script auto-copies `WinDivert.dll` next to `sonicdpi.exe`.
- `linux.rs` — Pure-Rust `nfq` crate (no GPL `libnetfilter_queue` linkage). Installs nftables rules at startup. Requires `cap_net_admin` (or root). `iptables-only` hosts (kernel < 6.17) are not supported.
- `macos.rs` — `pf rdr-to` redirects target ports to a transparent userspace proxy; `DIOCNATLOOK` ioctl recovers the original destination. **TCP-only in v0.2** — `IPPROTO_DIVERT` was removed from XNU. Discord voice on macOS is blocked on the NetworkExtension System Extension backend (tracked in `docs/macos-networkextension.md`).

### sonicdpi-cli — `sonicdpi` binary
Subcommands: `run`, `install`, `uninstall`, `profiles`, `show`, `probe`. `service.rs` handles winsvc / systemd unit / launchd plist generation.

### sonicdpi-tray — end-user toggle
`elevation.rs` checks for admin/root upfront, `engine_guard.rs` spawns and supervises the `sonicdpi` child, `icons.rs` embeds tray PNGs. Uses `tray-icon` + `winit`.

## Build, test, lint

```bash
# Workspace build (all crates)
cargo build --workspace
cargo build --workspace --release

# Tests — engine logic only (no privileged network ops in unit tests).
# Integration tests that need real interceptors are gated on `--ignored`.
cargo test --workspace --lib

# Required to pass CI:
cargo fmt --all -- --check
cargo clippy --workspace --all-targets    # treated as -D warnings via RUSTFLAGS
cargo deny check licenses                 # license allowlist in deny.toml
```

Toolchain: `rust-toolchain.toml` pins `stable` with `rustfmt` + `clippy` components. MSRV is **1.78** (set in workspace `Cargo.toml`).

### Platform-specific build deps
- **Windows**: nothing extra — WinDivert binaries are vendored at `vendor/windivert/x64/`. The `.cargo/config.toml` sets `WINDIVERT_PATH` and bumps the link-time stack to 8 MiB on `x86_64-pc-windows-msvc` and `i686-pc-windows-msvc`.
- **Linux** (CI + dev): `sudo apt-get install -y libnetfilter-queue-dev nftables`. nftables is a runtime dep too.
- **macOS**: nothing extra.

### CI matrix (`.github/workflows/ci.yml`)
- `fmt` — Linux only.
- `clippy` — Linux + Windows + macOS, `-D warnings`.
- `test` — same matrix, `cargo test --workspace --lib`.
- `deny` — Linux only, `cargo-deny check licenses`.

## License invariants — DO NOT VIOLATE

Code in this repo is **MIT OR Apache-2.0**. Consequences:
- **Never copy code from GPL/LGPL projects** (zapret, GoodbyeDPI source, byedpi source). Their *technique catalogs* are fair game for re-implementation; their source is not. `CONTRIBUTING.md` makes this a hard no.
- **WinDivert is LGPL-3** and explicitly carved out in `deny.toml` (`exceptions`). It is dynamically linked via `WinDivert.dll` so the rest of the binary stays permissive — keep it that way (no static linking, no embedding the WinDivert source). Documented in `THIRD_PARTY_LICENSES.md`.
- The `licenses` allowlist in `deny.toml` is the source of truth for acceptable transitive deps. Adding a crate with a non-allowlisted license **must** be discussed in the PR — don't silently expand the allowlist.

## Things that look wrong but aren't

- **`multiple-versions = "allow"` in deny.toml** — duplicate `windows`/`windows-sys`/`windows-targets` are a known version-skew between `windivert` 0.48, `socket2` 0.52, and the elevation check (`windows-sys` 0.59). We re-evaluate when upstream consolidates; do not "fix" by pinning.
- **Linux backend uses `nfq` (pure Rust), not `libnetfilter_queue`** — deliberate to avoid the GPL linkage.
- **macOS backend uses pf, not divert** — `IPPROTO_DIVERT` was removed from the XNU kernel. There is no path back; the long-term fix is the NetworkExtension System Extension backend.
- **`include_inbound` defaults to false** in `InterceptorConfig` — outbound-only is enough for the v0.2 strategies. Don't flip it without reading `flow.rs` first.
- **Per-run randomness in fake payloads** is intentional anti-fingerprinting (see `embedded_fakes.rs`). Don't refactor toward a static byte string.
- **The engine has zero allocation on the hot path once a flow is established.** Adding `Vec::new()` or `String::from(...)` inside `Strategy::on_packet` is a regression — push allocations into setup.

## Profiles (TOML)

Profile schema is documented in `crates/sonicdpi-engine/src/profile.rs` and shown end-to-end in `README.md`. To experiment: `sonicdpi show youtube-discord -o my.toml`, edit, then `sonicdpi run --profile my.toml -v`.

The three built-ins, in priority order:
1. `youtube-discord` — default. multisplit + seqovl=568, fake-multidisorder + md5sig, quic-fake-initial × 6, discord-voice-prime × 6.
2. `youtube-discord-seqovl` — fallback when the ISP filters MD5SIG. Multisplit + seqovl=568 only (Flowseal-style).
3. `youtube-discord-hostfakesplit` — fallback when the DPI matches SNI on first sight before reassembly. Rewrites SNI to `www.google.com`.

## Adding a new strategy — checklist

1. New module under `crates/sonicdpi-engine/src/` implementing `Strategy`. Keep it allocation-free on the hot path.
2. Plumb a TOML section in `profile.rs::Profile::build_pipeline`.
3. Unit test in the same file — feed it a synthetic `Packet`, assert the `Action`. No real sockets.
4. One-paragraph entry in `docs/research-techniques-2026.md` with a citation (zapret thread, ntc.party post, paper). This is required by `CONTRIBUTING.md`.
5. If it needs new fake bytes: drop them under `crates/sonicdpi-engine/builtin/` and document how they were captured (own machine only, no third-party pcaps).

## Privileges and where they're needed

- **Windows**: `sonicdpi.exe` requires admin (WinDivert driver load). The CLI checks via `windows-sys` upfront and bails with a clear message if not elevated. The tray binary itself does **not** need admin — it spawns the elevated child.
- **Linux**: needs `cap_net_admin` (`sudo setcap cap_net_admin=eip ./sonicdpi`) or run via the systemd unit, which sets it.
- **macOS**: needs root for pf rule install (`sudo ./sonicdpi install`).

## Running and debugging locally

```bash
# Foreground, default profile, debug logs:
sudo ./sonicdpi run -vv                  # Linux/macOS
.\sonicdpi.exe run -vv                   # Windows (admin shell)

# Probe several profiles against a host without installing anything:
./sonicdpi probe -H rr1.googlevideo.com --profiles youtube-discord,youtube-discord-seqovl

# Logs:
journalctl -u sonicdpi -f                # Linux (systemd)
tail -f /var/log/sonicdpi.log            # macOS
Event Viewer → SonicDPI                   # Windows
```

`tracing-subscriber` reads `RUST_LOG` and the `-v` count maps to: `info` / `debug` / `trace`.

## Roadmap (from CONTRIBUTING.md, tracked as GH issues)

- v0.2 — finish byte-rewriter (`Replace` / `InjectThenPass` actually emit packets).
- v0.3 — macOS NetworkExtension backend with UDP support (Discord voice on Mac).
- v0.4 — adaptive probing harness (auto-rotate ALT profiles).
- v0.5 — minimal Tauri GUI for end-users.

## What contributions are NOT accepted

From `CONTRIBUTING.md`, repeated here because it's load-bearing:
- Anything that helps a censor identify SonicDPI traffic.
- GPL-licensed code copied from zapret / GoodbyeDPI / byedpi sources.
- Fake payloads containing real user data or third-party PII.

## Style

- Module-level `//!` docs explain *what this module is for* — keep them up to date when responsibilities shift.
- `anyhow::Result` at binary boundaries (CLI / platform-open paths), `thiserror` for engine error enums (`engine` is a library, callers will want to match).
- `tracing` for logs, never `println!` outside `cli/main.rs`'s human-output paths (`profiles`, `show`).
- No `unwrap()` on packet-parsing paths. Malformed packets must be `Action::Pass`-ed through, never panicked on.

## Useful files for orientation

- `README.md` — user-facing feature list, install steps, profile format.
- `README.ru.md` — Russian translation (keep in sync when changing user-facing claims).
- `CONTRIBUTING.md` — workflow, what's accepted, what isn't.
- `THIRD_PARTY_LICENSES.md` — WinDivert LGPL carve-out.
- `docs/research-techniques-2026.md` — current technique catalog and citations.
- `docs/platform-backends.md` — interceptor internals and per-OS gotchas.
- `docs/macos-networkextension.md` — design notes for the v0.3 macOS backend.

---
> Source: [by-sonic/sonicdpi](https://github.com/by-sonic/sonicdpi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
