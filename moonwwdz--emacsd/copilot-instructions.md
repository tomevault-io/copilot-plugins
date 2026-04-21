## emacsd

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a personal Emacs configuration repository (`~/.emacs.d`) with a modular structure. The configuration is designed for daily use and includes support for multiple programming languages, org-mode, and various productivity tools.

## Installation & Setup

```bash
pip3 install epc orjson sexpdata six setuptools paramiko rapidfuzz --break-system-packages
git clone https://github.com/moonwwdz/emacsd.git ~/.emacs.d
cd ~/.emacs.d
git submodule init
git submodule update
```

## Architecture

### Core Configuration Files

- **`init.el`** - Main configuration entry point that loads all modules
- **`lisp/init-*.el`** - Modular configuration files:
  - `init-packages.el` - Package management and installation
  - `init-ui.el` - User interface and theme settings
  - `init-better-default.el` - Enhanced default settings
  - `init-org.el` - Org-mode configuration
  - `init-keyboard.el` - Keybinding definitions

### Custom Modules

- **`lisp/moonwwdz-*.el`** - Personal custom utilities:
  - `moonwwdz-golang.el` - Go development setup
  - `moonwwdz-shell.el` - Shell configuration
  - `moonwwdz-dict.el` - Dictionary integration
  - `moonwwdz-helper.el` - General helper functions

### Package Structure

- **`git-package/`** - Git submodules with custom packages:
  - `lsp-bridge` - LSP client with async completion
  - `evil` - Vim emulation layer
  - `emacs-rime` - Rime input method integration
  - `company-english-helper` - English word completion
  - `nov` - EPUB reader
  - `org-modern` - Modern org-mode styling
  - `mastodon` - Mastodon client

- **`theme/`** - Custom color themes (molokai variants)

- **`elpa/`** - Installed ELPA packages

## Key Features

### Development Support
- Go development with LSP integration
- JavaScript/JS2 mode
- Rust mode
- Python development with pipenv integration
- Git integration via magit

### Org-Mode
- HTML export functionality
- Modern styling with org-modern
- Capture templates with `C-c c`

### Input & Dictionary
- Rime input method support
- English-Chinese dictionary lookup
- Youdao and Mac dictionary integration

### UI/UX
- Evil mode for Vim keybindings
- Company mode for auto-completion
- Dired sidebar for file navigation
- Tab bar mode for workspace management

## Essential Keybindings

| Keyboard | Action |
|----------|--------|
| `C-c C-l` | Edit org-mode links |
| `C-c c` | Org-capture |
| `C-c g` | Magit status |
| `C-x C-n` | Toggle dired sidebar |
| `C-c d` | Mac dictionary lookup |
| `C-c y` | Youdao dictionary |
| `M-.` | Jump to definition (xref) |
| `M-,` | Return from definition |

## Development Notes

- Configuration uses a modular approach with separate files for different aspects
- Custom packages are managed as git submodules in `git-package/`
- Personal custom utilities are prefixed with `moonwwdz-`
- The setup supports both Chinese and English input methods
- Uses modern Emacs features while maintaining compatibility

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/moonwwdz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
