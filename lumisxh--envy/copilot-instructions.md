## envy

> `envy` is a fast, Rust-based utility that detects network context (e.g., Office vs. VPN vs. Home) and sets environment variables persistently in the Windows Registry or dynamically in the current shell.

# CLAUDE.md — envy environment manager

## Project overview

`envy` is a fast, Rust-based utility that detects network context (e.g., Office vs. VPN vs. Home) and sets environment variables persistently in the Windows Registry or dynamically in the current shell.

---

## Architecture

The tool follows a pipeline architecture:

1. **Config** → Parse `envy.toml` to get profile definitions
2. **Detect** → Probe network (DNS reachability, adapter names)
3. **Engine** → Match profiles against detection results
4. **Actuate** → Apply variables to Registry (Windows) or print shell commands

---

## Module map

| Module    | Responsibility                                                              |
| --------- | --------------------------------------------------------------------------- |
| `config`  | Parse `envy.toml`, define `Config` and `Profile` structs                    |
| `detect`  | Network sensors: DNS TCP check, PowerShell adapter query                    |
| `engine`  | Profile evaluation logic, returns matched profile                           |
| `env_win` | Windows Registry integration (gated behind `#[cfg(target_os = "windows")]`) |
| `output`  | Styled terminal output with consistent symbols                              |
| `error`   | `AppError` enum with `thiserror`                                            |

---

## CLI commands

| Command                | Description                                                             |
| ---------------------- | ----------------------------------------------------------------------- |
| `envy apply`           | Detect context → write to Windows Registry → broadcast WM_SETTINGCHANGE |
| `envy status`          | Show detected profile and its environment variables                     |
| `envy hook powershell` | Print `$Env:VAR = 'VAL'` lines for shell sourcing                       |
| `envy info`            | Styled dashboard with version, network, and variables                   |

---

## Rust coding rules

These rules apply to all code in this project. Follow them exactly.

### Error handling

- **Never** use `unwrap()` or `expect()` in non-test code — propagate errors with `?`
- Within modules: return `crate::error::Result<T>` (alias for `Result<T, AppError>`)
- Add new `AppError` variants before reaching for `anyhow` inside modules
- In `main.rs` command handlers: use `anyhow::anyhow!(...)` at the binary boundary for user-facing errors

### Visibility

- Default to `pub(crate)` — only use `pub` when necessary for external API
- Test-only functions get `#[cfg(test)]`

### Style preferences

- Prefer `match` over `if let` chains when there are multiple cases
- Use enums for domain states — not booleans or strings
- No `let-else` — use `match` or `if let` with early return
- No `async` — all I/O is blocking, no tokio

### Platform handling

- Use `#[cfg(target_os = "windows")]` for Windows-only code (Registry, PowerShell)
- Use `#[cfg(not(target_os = "windows"))]` for fallback implementations
- `winreg` dependency is gated: `[target.'cfg(windows)'.dependencies]`

---

## Patterns to follow

### Config loading

All config loading goes through `Config::load()` which uses `dirs::config_dir()` to find the config path:

- Unix: `~/.config/envy/envy.toml`
- Windows: `%APPDATA%\envy\envy.toml`

### Profile matching

`engine::evaluate()` iterates profiles in definition order:

1. Skip `default = true` profiles in first pass
2. Match if ALL conditions pass (DNS AND adapter)
3. Fall back to `default = true` profile if no match
4. Error if no default profile exists

### DNS checking

`detect::check_dns()` uses `TcpStream::connect_timeout` with 1-second timeout. Returns true if ANY server is reachable (short-circuits on success).

### Adapter checking

`detect::check_adapter()` runs PowerShell on Windows to get active adapter names. Returns true if ANY keyword is found in ANY adapter name.

### Output styling

Use `output::*` functions — never `println!` / `eprintln!` directly in command handlers:

- `ok()` — green `✓` for success
- `warn()` — yellow `!` for warnings
- `error()` — red `✗` for errors
- `header()` — bold underlined text
- `table_row()` — colored symbol + padded name + status
- `env_var()` — shows set/unset state of environment variables

---

## Configuration format

```toml
[profiles.vpn]
match_dns = ["172.21.2.201:53", "172.21.2.202:53"]
match_adapter = ["barracuda", "cudavpn"]
env = { SSL_CERT_FILE = "", UV_NATIVE_TLS = "" }  # empty = unset

[profiles.office]
match_dns = ["172.21.2.201:53"]
env = { SSL_CERT_FILE = "C:\\cert\\Root.pem" }

[profiles.home]
default = true
env = { SSL_CERT_FILE = "" }
```

---

## Testing conventions

- Unit tests live in `#[cfg(test)]` modules at the bottom of each file
- Use `tempfile::TempDir` for any test that touches the filesystem
- DNS tests use TEST-NET-1 (`192.0.2.0/24`) which is reserved and unreachable
- Adapter tests are gated with `#[cfg(target_os = "windows")]`
- This is a `bin` crate, so integration tests go as unit tests inside modules

---

## Things to avoid

- Do not use `unwrap()` or `expect()` in non-test code
- Do not modify `envy.toml` via CLI — it is read-only
- Do not add `async` — the design explicitly excludes tokio
- Do not add a workspace — this is intentionally a single crate
- Do not use `println!` / `eprintln!` in command handlers — use `output::*`
- Do not add network checks that can hang — always use timeouts
- Do not add platform-specific dependencies without `#[cfg]` gating

---

## Dependencies

| Crate       | Purpose                                    |
| ----------- | ------------------------------------------ |
| `clap`      | CLI argument parsing with derive macros    |
| `serde`     | Serialization/deserialization for config   |
| `toml`      | TOML config file parsing                   |
| `anyhow`    | Error handling at binary boundary          |
| `thiserror` | Typed error enum derivation                |
| `dirs`      | Platform-aware config directory resolution |
| `console`   | Styled terminal output                     |
| `winreg`    | Windows Registry access (Windows only)     |

---

## Versioning and releases

- Version is the single source of truth in `Cargo.toml` (`version = "x.y.z"`)
- Follow [Semantic Versioning](https://semver.org/): bump major for breaking changes, minor for features, patch for fixes
- Maintain `CHANGELOG.md` using [Keep a Changelog](https://keepachangelog.com/) format

### Release targets

| Target triple               | OS      | Arch                  |
| --------------------------- | ------- | --------------------- |
| `x86_64-unknown-linux-gnu`  | Linux   | x86_64                |
| `aarch64-unknown-linux-gnu` | Linux   | ARM64                 |
| `x86_64-apple-darwin`       | macOS   | x86_64                |
| `aarch64-apple-darwin`      | macOS   | ARM64 (Apple Silicon) |
| `x86_64-pc-windows-msvc`    | Windows | x86_64                |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LuMiSxh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
