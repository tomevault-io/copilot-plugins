## architecture

> Architecture — package layout, command structure, XDG layout, Nostr query flow


# zapstore-cli — Architecture

## Core Principle

`zapstore-cli` is a package manager for desktop platforms. It queries Nostr for app metadata, downloads and verifies binaries, and manages them on the filesystem. The filesystem is the sole source of truth — no separate state database.

## Package Layout

```
main.go                  Entry point, command dispatch
cmd/                     One file per subcommand (install, remove, update, list, search, cleanup)
nostr/                   Nostr relay queries — fetch apps (32267), releases (30063), assets (3063)
platform/                Platform detection (OS, arch) and asset filtering
store/                   Filesystem operations — install paths, symlinks, XDG layout
ui/                      Output formatting — Cargo-style action verbs, color, JSON mode
version/                 Version comparison (versionCode integers only)
install/                 Download, SHA-256 verification, binary placement
```

## XDG Filesystem Layout

```
$XDG_DATA_HOME/zapstore/packages/<hex-pubkey>-<app-id>/<version>/   Installed binaries
~/.local/bin/<app-id>                                                 Symlinks to active binary
$XDG_CACHE_HOME/zapstore/                                             Profile cache
$XDG_CONFIG_HOME/zapstore/config.env                                  User config
```

The package directory is prefixed with the publisher's hex pubkey for namespacing.

## Command Flow

### install / update
1. Query relay for app (32267) → release (30063) → asset (3063) matching platform+arch
2. Download binary to temp path
3. Verify SHA-256 against asset event `x` tag
4. Move to `packages/<pubkey>-<appid>/<version>/`
5. Symlink `~/.local/bin/<appid>` → versioned binary

### list
Read `packages/` directory — no relay query needed.

### search
NIP-50 full-text search on relay.

## Output Design

Status messages → stderr. Data (JSON) → stdout.
Cargo-style action-verb prefix: `   Resolving`, `  Downloading`, `   Verifying`, `  Installing`, `    Finished`.

---
> Source: [zapstore/zapstore-cli](https://github.com/zapstore/zapstore-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
