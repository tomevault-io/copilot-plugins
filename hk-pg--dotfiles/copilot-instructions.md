## dotfiles

> This repository contains personal dotfiles managed by `chezmoi`. It primarily focuses on a modular Zsh configuration.

# Gemini Context: Dotfiles (chezmoi)

This repository contains personal dotfiles managed by `chezmoi`. It primarily focuses on a modular Zsh configuration.

## Project Overview

- **Tooling**: [chezmoi](https://www.chezmoi.io/) for dotfiles management.
- **Shell**: [Zsh](https://www.zsh.org/) with a modular script structure.
- **Plugin Manager**: [antidote](https://getantidote.github.io/) for high-performance Zsh plugin loading.
- **Theme**: [Powerlevel10k](https://github.com/romkatv/powerlevel10k) for a fast and informative prompt.
- **Package Manager**: [Homebrew](https://brew.sh/) is assumed for many dependencies.

## Key Files & Directories

- `dot_zshrc`: Maps to `~/.zshrc`. It is a lightweight loader that sources all `.zsh` files from the user scripts directory.
- `dot_zsh/`: Maps to `~/.zsh/`.
    - `zsh_plugins.txt`: List of plugins to be loaded by `antidote` (e.g., syntax highlighting, autosuggestions).
    - `user_scripts/`: Contains modular configuration files:
        - `alias.zsh`: Shell aliases.
        - `path.zsh`: Environment path updates.
        - `antidote.zsh`: Initialization logic for the plugin manager.
        - `p10k.zsh`: Configuration for the Powerlevel10k theme.
        - `completions/`: Directory for custom shell completion scripts.

## Building and Running (chezmoi)

Since this is a `chezmoi` source directory, you interact with it using `chezmoi` commands:

- **Apply Changes**: `chezmoi apply` (updates the actual files in `$HOME`).
- **Edit File**: `chezmoi edit <path-to-file>` (opens the source file in your editor and applies on save if configured).
- **Add New File**: `chezmoi add <path-to-file>` (tracks a new file from `$HOME`).
- **View Changes**: `chezmoi diff` (see the difference between the source and the target).
- **Status**: `chezmoi status` (see what's changed).

## Development Conventions

- **Modularity**: Avoid adding large blocks of logic directly to `dot_zshrc`. Instead, create a new `.zsh` file in `dot_zsh/user_scripts/`.
- **Naming**: Use the `dot_` prefix for files that should be hidden in the home directory (e.g., `dot_zshrc` becomes `~/.zshrc`).
- **Plugins**: Add new Zsh plugins to `dot_zsh/zsh_plugins.txt` to be managed by `antidote`.
- **External Tools**: Many scripts (like `antidote.zsh`) check for the presence of `brew` before executing, maintaining portability where possible.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hk-pG)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hk-pG)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
