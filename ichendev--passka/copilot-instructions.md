## passka

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Passka is an AI-agent friendly credential manager written in Rust with a Swift iOS/macOS app. It stores credentials securely in the macOS Keychain and provides both a CLI tool and native app interface.

## Commands

```bash
# Build the entire workspace
cargo build

# Build release version (required for iOS FFI)
cargo build --release

# Run a single test
cargo test -p passka-core -- types::tests

# Build the FFI library (generates Swift bindings)
cargo build -p passka-ffi --release

# Run the CLI
cargo run -p passka-cli -- help

# Run a specific CLI command
cargo run -p passka-cli -- add my-creds --type api_key
cargo run -p passka-cli -- get my-creds
cargo run -p passka-cli -- list
cargo run -p passka-cli -- exec my-cred -- echo $API_KEY
```

## Architecture

### Crates

- **`passka-core`**: Core library containing:
  - `types.rs`: Credential types (UserPass, Cookie, ApiKey, AppSecret, Token, Custom) and data structures
  - `store/keychain.rs`: Keychain storage via `keyring` crate (secure credential storage)
  - `store/index.rs`: JSON-based index at `~/.config/passka/index.json` for metadata
  - `oauth.rs`: OAuth token refresh logic

- **`passka-cli`**: CLI application with subcommands defined in `cli.rs`, implemented in `commands/` directory

- **`passka-ffi`**: FFI bridge using `swift_bridge` crate. Generates Swift bindings in `generated/` directory for use by the iOS/macOS app

### Data Model

- **CredentialMeta**: Stored in index JSON (name, type, description, env_vars, timestamps)
- **CredentialData**: Actual secrets stored in Keychain (key-value fields)

### iOS App

Located in `app/`. Uses Swift Package Manager to link against the FFI library:
- Links to `target/release/libpasska_ffi.a`
- Uses `PasskaBridge` to call Rust functions
- Native SwiftUI views for credential management

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IchenDEV) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
