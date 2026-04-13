## homebrew-env

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a macOS development environment setup repository that manages Homebrew packages through a Brewfile. It provides a declarative way to manage development tools, applications, and VSCode extensions across macOS systems.

## Common Commands

### Environment Management
```bash
# Install/update packages from Brewfile
brew bundle

# Check environment sync with Brewfile
brew bundle check

# Generate Brewfile from current environment
brew bundle dump --describe --force

# Clean up unused packages
brew bundle cleanup --dry-run
brew bundle cleanup

# Update Homebrew and all packages
brew update && brew upgrade
```

### Setup Commands
```bash
# Create symlink to home directory (for new setups)
ln -s $(pwd)/Brewfile ~/Brewfile

# Install from specific Brewfile location
brew bundle --file=/path/to/Brewfile
```

## Architecture

- **Brewfile**: Central configuration file containing all package definitions (formulae, casks, VSCode extensions)
- **README.md**: User documentation in Japanese with setup instructions and troubleshooting
- The repository is designed to be symlinked to `~/Brewfile` for global use

## Package Categories in Brewfile

- **Core development tools**: Git, Docker, Node.js, Python, Ruby, Go, Java
- **Database systems**: MySQL, PostgreSQL, Redis  
- **Development applications**: VSCode, IDEs (IntelliJ, PyCharm), Docker Desktop
- **Productivity tools**: Alfred, 1Password, Slack, Raycast
- **System utilities**: Terminal emulators (iTerm2, Warp), file management tools
- **VSCode extensions**: Comprehensive development extension pack covering multiple languages and frameworks

## Notes

- All documentation is in Japanese
- Repository follows symlink-based installation pattern typical for dotfiles
- Includes extensive VSCode extension management through `vscode` entries
- Uses multiple Homebrew taps for specialized tools

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/iorionda)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/iorionda)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
