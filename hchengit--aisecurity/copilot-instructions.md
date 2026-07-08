## aisecurity

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

AISecurity is a "General AI Security Layer" — a macOS menu-bar app plus a cross-platform Rust detection engine that guards AI agents (Claude Code, OpenClaw, etc.) and the host machine. All detection logic (pattern matching, scoring, redaction, policy) lives in Rust and is shared by every consumer; the Swift app and the standalone binaries are thin front-ends over it.

Note: the primary working directory (`AIS Learn`) contains only unrelated notes. The actual codebase is the `AISecurity/` working directory — do all work there.

## Architecture

Detection logic is written **once** in Rust (`SecurityCore/crates/security-core`) and consumed three ways:

1. **Swift macOS app** (`Sources/AISecurity`) — links the Rust `staticlib` through a C ABI. `build-rust.sh` compiles `security-core-ffi` into `CSecurityCore/lib/libsecurity_core_ffi.a` + a cbindgen-generated header; the `CSecurityCore` system-library target exposes it to Swift, and `RustBridge/SecurityCoreBridge.swift` wraps the C functions. Swift owns the UI, the launch-time integrity/keychain gating, file/process/TCC monitoring, and the encrypted Vault; it calls into Rust for every actual detection decision.
2. **Linux daemon** (`crates/security-linux`) — same core, native binary + TUI, no Swift.
3. **AI-agent integration binaries** (below) — link `security-core` directly as a Rust dependency.

### The daemon and the :7459 loopback service

The running app/daemon (`Core/SecurityDaemon.swift`) hosts an in-process HTTP listener on `127.0.0.1:7459` exposing `intent_verifier` and `privacy_router`. The AI-agent binaries are **thin relays** to this port so policy stays centralized in one running process:

- **`aisec-mcp`** — MCP server. Exposes `verify_intent` and `evaluate_privacy` as MCP tools (registered via `claude mcp add --scope user aisec …`). This is the `aisec` MCP server whose instructions appear in this session.
- **`intent-hook`** — Claude Code `PreToolUse` hook. Consults `intent_verifier` and returns allow/deny/ask; `install.sh` merges it into `~/.claude/settings.json`.
- **`privacy-router`** — local forward-proxy for outbound LLM API calls (`HTTPS_PROXY=http://127.0.0.1:7459`). Scans request bodies and applies allow/redact/warn/block.
- **`ai-exec`** — wraps a command in macOS `sandbox-exec` using the `[agents.*]` policy from config.

If the daemon isn't running, relays fail closed rather than making unguarded decisions.

### security-core module map

Each concern is one module in `crates/security-core/src/` with a matching config section. Detection modules: `threat_intent_parser` (7-layer scoring engine), `sensitive_data`, `prompt_injection`, `file_sanitizer`, `email_patterns`, `message_patterns`, `sender_whitelist`. Agent-facing policy: `intent_verifier`, `privacy_router`, `agent_policy`, `command_policy`, `model_verifier`/`model_vetting`, `package_vulns`, `threat_feeds`, `bypass`, `policy_audit`. Infrastructure: `config` (TOML + `MACSEC_*` env overrides), `path_resolver` (macOS vs Linux defaults), `encryption` (AES-256-GCM), `key_filter` (redaction), `wasm_sandbox` (wasmtime plugin loader), `tls_transport` (rustls log shipping), `vault`, `process_manager`, `local_services` (the :7459 handlers), `severity`, `alert`.

When adding a detection capability, add/extend the Rust module + its config section; the Swift `Modules/*.swift` file is a monitor/orchestrator that calls into it, not a second copy of the logic.

## Build & test

**Rust core** (from `SecurityCore/`):
```bash
cargo build --release -p security-core-ffi   # what build-rust.sh compiles for Swift
cargo test -p security-core                   # unit tests (~122)
cargo test --test cross_validation            # shared JSON suite validating Rust vs Swift parity
cargo test --workspace
cargo test -p security-core sensitive_data    # single module's tests
```

**Full macOS build** (from repo root):
```bash
./build-rust.sh release      # MUST run before/after any Rust change — Swift links the .a, edits to .rs are invisible until you rebuild it
swift build -c release
swift run AISecurity          # menu-bar app, no Dock icon (.accessory)
```
`.build/` (Swift) and `SecurityCore/target/` (Cargo) are build output.

**Full install** (builds everything, installs the `.app` + LaunchAgent, installs the agent binaries, wires the MCP server and PreToolUse hook):
```bash
./install.sh     # see uninstall.sh to reverse
```

## Config

Runtime config: `~/.mac-security/config.toml` (template: `config.toml.example`). One `[section]` per module. `MACSEC_*` environment variables override individual keys at highest priority (e.g. `MACSEC_MODE`, `MACSEC_SCAN_DIRS`).

`[general].mode` is `PRODUCTION | TESTING | DEVELOPMENT` and gates safety behavior — e.g. in `PRODUCTION` the default encryption passphrase is rejected, so `SECURITYCORE_PASSPHRASE` must be set for at-rest encryption of the whitelist/config secrets.

## Conventions that matter here

- **Fail closed.** Startup and the relays deliberately refuse to proceed when a security precondition can't be met (code-signature mismatch in a production install, missing Keychain master key, daemon unreachable) rather than degrading silently. Preserve this — don't add fallbacks that quietly downgrade to plaintext or unguarded paths.
- **Rust is the single source of truth** for detection. The `cross_validation` test enforces Rust/Swift parity; if you change scoring or patterns, keep that test green.
- **Custom rules are WASM.** User plugins go in `~/.mac-security/rules/` and run sandboxed in wasmtime (no fs/network), exporting `name`, `analyze`, and `alloc`. Extend `wasm_sandbox.rs` rather than adding ad-hoc rule-loading paths.

## Security practices (this is a security-enforcement product)

The hard gates are in CI (`.github/workflows/ci.yml`) — they fail the build, so they
can't quietly rot. This section is the judgment CI can't automate.

**Before a change is done, all of these pass (CI enforces them):**
- `cargo clippy --workspace -- -D warnings` — zero warnings. No new `#[allow(...)]`
  without a one-line justification comment.
- `cargo deny check advisories bans sources` — no un-accepted dependency advisories.
  Accepted ones live in `SecurityCore/deny.toml` with a reason; **prune that list as
  fixes land, and never add a new ignore without a reason and a tracking note.**
- `cargo test --workspace` passes **and is deterministic.** Never let a test depend on
  a process-global (env var, `static` DB/connection, a shared file like the real
  `~/.mac-security/bypass`) without a serial guard or explicit isolation — flaky tests
  hide real bugs (see the `bypass` and `threat_feeds` test guards for the pattern).
- Rust↔Swift parity: if you touch scoring/patterns, keep `cross_validation` green, and
  remember `./build-rust.sh` must run for Swift to see `.rs` changes.

**Crown-jewel code — changes here get a threat-model review, not just a unit test:**
the bypass/critical-secret floors (`bypass.rs`, `privacy_router.rs`), intent & command
policy (`intent_verifier.rs`, `command_policy.rs`), the MCP/PreToolUse trust boundary
(`aisec-mcp`, `intent-hook`, `local_services.rs`), and the Rust↔Swift FFI
(`security-core-ffi`). For these, ask **"how would a compromised or prompt-injected
agent evade or disable this?"** and prefer fail-closed. Enforcement inputs an agent can
control — env vars, config it can write, transcript contents — are IN scope. Many of the
worst historical findings were cross-module interaction bugs (floor asymmetry between the
daemon and the hook; the MCP relay trusting a redirectable daemon URL), so review the
trust boundary, not just the diffed lines.

**Deeper passes (not per-commit):** run the `security-audit-pipeline` skill per milestone
or before releases, and fuzz the hand-rolled parsers (`local_services.rs` HTTP,
`email_scanner`, the PyPI/manifest parser). A model finds the interesting logic bugs;
fuzzers and `cargo deny` find the complete boring set — use both.

**WASM sandbox:** runs on `wasmtime` 43 with fuel metering (bounds guest instructions per
call), a linear-memory cap, and a host-side result-size cap — see `wasm_sandbox.rs`.
wasmtime ships frequent security releases, so keep it current; the supply-chain gate flags
new advisories. Any new plugin entry point must preserve those three limits.

---
> Source: [hchengit/AISecurity](https://github.com/hchengit/AISecurity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
