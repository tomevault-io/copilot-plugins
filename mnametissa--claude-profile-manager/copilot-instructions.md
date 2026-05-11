## claude-profile-manager

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claude Profile Manager is a bash-based tool that enables multiple Claude Code profiles via a `-u` flag. Profiles are stored as separate config directories (`~/.claude-<profile-name>`).

## Architecture

```
claude-profile-manager.sh   # Main entry, dispatcher, exports claude-profiles()
lib/
  config.sh                 # Global vars (CLAUDE_DEFAULT_PROFILE, paths)
  utils.sh                  # Profile helpers (profile_dir, profile_exists, etc.)
  claude-wrapper.sh         # claude() wrapper, handles -u and --yolo flags
  cmd-*.sh                  # Command implementations (one file per command group)
  completions.sh            # Shell completions
examples/
  agents/                   # Example agent .md files
  skills/                   # Example skill .md files
```

Key design:
- `claude()` function wraps native claude, intercepts `-u <profile>` to set `CLAUDE_CONFIG_DIR`
- Profiles live at `~/.claude-<name>/` with standard Claude config structure
- Agents/skills can be local (profile) or global (`~/.claude-shared/`)
- `jq` required for permission commands

## Testing

Manual testing only. Install locally and test commands:

```bash
bash install.sh
source ~/.bashrc  # or ~/.zshrc
claude-profiles add testprofile
claude -u testprofile
```

## Conventions

- No AI mention in commits
- Short commit messages
- Use `list` not `tree` for directory display

---
> Source: [MNametissa/claude-profile-manager](https://github.com/MNametissa/claude-profile-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
