## zed-conf

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Zed editor configuration directory containing settings, keymaps, tasks, and themes for the Zed text editor. It's part of a larger dotfiles repository at `/home/isomo/.config/`.

## Configuration Files

### Core Configuration
- **settings.json** - Main editor configuration with vim mode, AI integration, and theme settings
- **keymap.json** - Custom key bindings (currently using defaults with commented examples)
- **tasks.json** - Task definitions for running commands, including LaTeX forward search
- **themes/** - Custom theme directory

### Key Features Configured
- **Vim Mode**: Enabled with vim_mode: true
- **AI Integration**: Copilot configured for edit predictions and chat (GPT-4.1)
- **LaTeX Support**: TeXLab LSP with latexmk build configuration
- **PDF Forward Search**: Okular integration for LaTeX development
- **GitHub Dark Theme**: Set as default for both light and dark modes

## Git Commit Convention
- Use emoji first to indicate commit type:
  - 🎉 `:tada:` - Initial commit or major feature
  - ✨ `:sparkles:` - New feature
  - 🐛 `:bug:` - Bug fix
  - 🔧 `:wrench:` - Configuration changes
  - 📝 `:memo:` - Documentation
  - 🚀 `:rocket:` - Performance improvements
  - 🎨 `:art:` - Code style/formatting
  - ♻️ `:recycle:` - Refactoring
  - 🔥 `:fire:` - Remove code/files
  - 📦 `:package:` - Add dependencies/submodules

---
> Source: [isomoes/zed-conf](https://github.com/isomoes/zed-conf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
