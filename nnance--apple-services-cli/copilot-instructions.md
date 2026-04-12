## apple-services-cli

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Native macOS CLI (Swift) that provides fast access to Apple Calendar and Contacts. Replaces slow `osascript`-based shell scripts with direct EventKit and Contacts.framework calls. Single binary, no external dependencies.

## Build & Run

```bash
swift build                    # Debug build
swift build -c release         # Release build

# Run (debug)
.build/debug/apple-services calendar list
.build/debug/apple-services contacts search "Jane"
```

## Install

The release binary must be codesigned with `entitlements.plist` (calendar + contacts entitlements) before installing. The binary also has a bundle identifier (`com.nicknance.apple-services`) embedded via Info.plist through a linker flag in Package.swift.

```bash
swift build -c release
codesign --force --sign - --entitlements entitlements.plist .build/release/apple-services
sudo cp .build/release/apple-services /usr/local/bin/apple-services
```

## TCC Permissions

The binary requires macOS TCC grants for Calendar (Full Access) and Contacts. TCC attributes permissions to the **host terminal app** (e.g., Terminal.app, Warp), not the binary itself. The terminal app must have Calendar and Contacts access granted in **System Settings > Privacy & Security**. On first run from a new terminal app, macOS should prompt for access.

## Testing

No test suite or linter is configured. Test manually by running subcommands against real Calendar/Contacts data.

## Architecture

- **Swift Package Manager** project (swift-tools-version 5.9, macOS 13+)
- **Single entry point**: `Sources/apple-services/main.swift`
- **Subcommand dispatch**: `apple-services <service> <action> [args...]` where service is `calendar` or `contacts`
- **Argument parsing**: Raw `CommandLine.arguments` (no ArgumentParser dependency)
- **Frameworks**: EventKit (calendar ops), Contacts.framework (contact ops), Foundation (JSON, dates)

### Key Services

- **CalendarService** — wraps `EKEventStore` for list/today/events/search/create/details/delete
- **ContactsService** — wraps `CNContactStore` for search/get/list/create

### Output Conventions

- All output is JSON to stdout via `JSONSerialization`
- Errors: `{"error": "..."}` with exit code 1
- Success messages (create/delete): `{"message": "..."}`
- Date input format: `"MM/DD/YYYY HH:MM:SS"`
- Date output format: ISO 8601
- Default calendar: `$APPLE_CALENDAR_NAME` env var, fallback `"Calendar"`

## Implementation Reference

See `PLAN.md` for the detailed implementation roadmap, including exact JSON output schemas that must match the shell scripts being replaced and all error handling cases.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nnance)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nnance)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
