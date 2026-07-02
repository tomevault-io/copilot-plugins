## unraid-claude-code

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Unraid plugin that installs Claude Code CLI on Unraid servers. The entire plugin is defined in a single file: `claude-code.plg`.

## Architecture

The plugin file (`claude-code.plg`) is an XML document containing:
- Plugin metadata (version, author, URLs, minimum Unraid version)
- CHANGES section for release notes
- Embedded shell scripts (install, startup, removal)
- Embedded PHP for the web UI settings page
- File creation directives using `<FILE>` elements

Key components within the plugin:
- **Install script** (`install-claude.sh`): Installs Claude Code via the official native installer, sets up symlinks for persistent storage
- **Web UI** (`claude-code.page`): PHP page shown at Settings → Utilities → Claude Code
- **Startup script** (`rc.claude-code`): rc.d script that waits for network/array, then runs install

## Development Workflow

Serve the plugin locally for testing:
```bash
python3 -m http.server 8080
```

On Unraid terminal:
```bash
# Install from dev machine
plugin install http://YOUR_DEV_IP:8080/claude-code.plg

# Reinstall after changes
plugin remove claude-code.plg && plugin install http://YOUR_DEV_IP:8080/claude-code.plg
```

## Releasing

Uses date-based versioning (`YYYY.MM.DD`) per Unraid plugin conventions:
```bash
bump-my-version replace --new-version 2025.12.01
git push && git push --tags
```

The `.bumpversion.toml` config updates three version locations in `claude-code.plg` automatically:
- Plugin version attribute
- CHANGES section header
- Post-install message

---
> Source: [BrianPugh/unraid-claude-code](https://github.com/BrianPugh/unraid-claude-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
