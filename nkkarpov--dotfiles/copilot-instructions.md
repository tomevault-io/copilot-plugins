## dotfiles

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a dotfiles repository that manages configuration files through symbolic links. The primary purpose is to maintain VSCode settings and keybindings in version control and deploy them to the appropriate system locations.

## Architecture

The repository uses a simple configuration-driven approach:

1. **dotfiles.conf**: Maps source files to target locations using the format `source_path:target_path`
2. **update_links.sh**: Bash script that reads dotfiles.conf and creates symbolic links from the repository to system locations
3. **vscode/**: Contains VSCode settings.json and keybindings.json

The linking script handles:
- Expanding tilde (~) in target paths
- Creating target directories if they don't exist
- Backing up existing files (non-symlink) to *.backup
- Replacing existing symlinks if they point to different sources
- Skipping already-correct symlinks

## Commands

### Install/Update Dotfiles
```bash
./update_links.sh
```
Reads dotfiles.conf and creates/updates symbolic links. Safe to run multiple times - it will skip already-correct links.

## Adding New Dotfiles

To add new configuration files:
1. Add the file to the repository (in an appropriate subdirectory)
2. Add a line to dotfiles.conf: `relative/path/in/repo:~/target/path`
3. Run `./update_links.sh`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nkkarpov) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
