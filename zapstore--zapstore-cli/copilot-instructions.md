## vision

> Product vision — what zapstore-cli is, who uses it, what success means


# zapstore-cli — Vision

## What zapstore-cli Is

`zapstore-cli` is a command-line package manager for desktop platforms (Linux, macOS). It installs, updates, and manages apps distributed via Zapstore — using Nostr for metadata and Blossom for binaries.

It follows the XDG Base Directory Specification and integrates naturally into Unix workflows.

## Who Uses It

- Developers and power users who prefer the terminal
- CI/CD pipelines that need to install Zapstore-distributed tools
- Users on platforms without a native Zapstore GUI app

## What Success Means

- Users can install any Zapstore app in one command: `zapstore install <app-id>`
- Installed binaries are immediately available on PATH
- Updates are reliable and verifiable
- Piping and scripting work correctly (`--json`, `--quiet`)

## Non-Goals

- `zapstore-cli` does not publish apps (that's `zsp`)
- `zapstore-cli` does not have a GUI
- `zapstore-cli` does not support Android APK installation (that's the Flutter app)
- `zapstore-cli` does not manage user accounts or signing keys

---
> Source: [zapstore/zapstore-cli](https://github.com/zapstore/zapstore-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
