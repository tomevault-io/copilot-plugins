## emacs

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## About This Repository

This is a Doom Emacs configuration directory. Doom Emacs is a configuration framework for GNU Emacs that uses a declarative module system.

## File Structure

- `init.el` - Module declarations (what Doom modules are enabled and their flags)
- `config.org` - Literate configuration source (the actual configuration is written here in org-mode)
- `config.el` - Generated Emacs Lisp code from config.org (auto-tangled by Doom's literate module)
- `packages.el` - Package declarations for additional packages not included in Doom modules

## Configuration Architecture

This configuration uses **literate programming** via the `literate` module (enabled in init.el:197). This means:

- The primary configuration source is `config.org`
- Changes should be made to `config.org`, NOT `config.el`
- `config.el` is auto-generated from `config.org` when Doom syncs
- Emacs Lisp code blocks in `config.org` are tangled into `config.el`

## Essential Commands

After making configuration changes:

```bash
~/.config/emacs/bin/doom sync
```

This command:
- Installs/updates packages declared in `packages.el`
- Regenerates autoloads
- Tangles `config.org` into `config.el` (if using literate config)
- Syncs module changes from `init.el`

Other useful commands:

```bash
# Upgrade Doom and packages
~/.config/emacs/bin/doom upgrade

# Rebuild Doom (fixes most issues)
~/.config/emacs/bin/doom build

# Run Doom doctor to check for issues
~/.config/emacs/bin/doom doctor

# Purge orphaned packages and repos
~/.config/emacs/bin/doom purge
```

## Enabled Modules

Key enabled modules (from init.el):

- **Completion**: vertico, corfu (+orderless)
- **Editor**: evil (+everywhere), format (+onsave +lsp), snippets, whitespace (+guess +trim)
- **Tools**: LSP (+eglot), magit, tree-sitter
- **Languages**: emacs-lisp, java, json, markdown, org, python (+lsp +pyenv +tree-sitter), rust (+lsp +tree-sitter), sh (+lsp +fish), web, yaml, zig (+lsp +tree-sitter)
- **Config**: literate, default (+bindings +smartparens)

## Custom Packages

- `odin-mode` - Support for the Odin programming language (installed from GitHub)

## Modifying Configuration

1. **To change settings**: Edit `config.org`, add Emacs Lisp code in `#+begin_src emacs-lisp` blocks
2. **To enable/disable modules**: Edit `init.el`, uncomment/comment module lines
3. **To add packages**: Add `package!` declarations in `packages.el`
4. **Always run**: `~/.config/emacs/bin/doom sync` after changes
5. **Restart Emacs**: Configuration changes typically require restarting Emacs

---
> Source: [linuxunil/emacs](https://github.com/linuxunil/emacs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
