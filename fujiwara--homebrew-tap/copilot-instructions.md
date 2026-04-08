## homebrew-tap

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Homebrew tap (`fujiwara/tap`) containing formulae for CLI tools authored by [@fujiwara](https://github.com/fujiwara). Formulae are Ruby `.rb` files in the repository root (and occasionally in `Formula/`).

## Prerequisites

Install [maltmill](https://github.com/Songmu/maltmill) via [aqua](https://aquaproj.github.io/):

```bash
aqua i
```

## Commands

```bash
# Update a specific formula to the latest release
make update/<tool-name>.rb

# Update all formulae that have linux support (uses `grep -l linux *.rb`)
make update-all

# Create a new formula for a fujiwara/<tool-name> GitHub repo
make create/<tool-name>

# Test a formula locally (requires Homebrew installed)
make test/<tool-name>.rb
# Equivalent to: HOMEBREW_DEVELOPER=true brew install --build-from-source <tool-name>.rb
```

## Formula Patterns

There are two formula styles:

1. **Multi-platform (modern)** — Used by most tools. Provides pre-built binaries for macOS (arm64/amd64) and Linux (arm64/amd64) with `on_macos`/`on_linux` blocks and per-architecture `url`/`sha256`. Includes a `head` block for building from source via Go.

2. **Single-binary (legacy)** — A few older tools (e.g., `aswrap.rb`, `nssh.rb`) use a single URL pointing to one binary.

When adding or updating a formula, use `maltmill` which handles version bumping, URL generation, and SHA256 computation automatically.

## Per-formula maltmill options

The Makefile supports per-formula maltmill options via `OPTS_<tool-name>` variables. When a GitHub release contains multiple archives for different tools, use the `-asset` option to select the correct one.

```makefile
# Example: simplemq-cli release includes both simplemq-cli and simplemq-localserver archives
OPTS_simplemq-cli = -asset simplemq-cli
```

The `update/%` target expands `$(OPTS_$*)` automatically, so formulae without a matching variable run maltmill with no extra options.

## CI/CD

- **daily.yml** — Runs twice daily. Uses `make update-all` to check for new releases, builds changed formulae from source, and opens a PR via `gh` CLI. PR description is auto-generated using GitHub Copilot SDK (`.github/scripts/generate-pr-body.mjs`) with a regex-based fallback.
- **diff-test.yml** — On non-master branch pushes, tests only changed `.rb` files by building from source on both Ubuntu and macOS.
- **test.yml** — On master pushes, builds all formulae from source on both Ubuntu and macOS.
- **dependabot.yml** — Monthly updates for GitHub Actions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fujiwara)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/fujiwara)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
