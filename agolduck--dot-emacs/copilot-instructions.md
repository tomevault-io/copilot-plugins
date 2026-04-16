## dot-emacs

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a personal Emacs configuration using a modular, layered architecture. All custom modules use the `w-` prefix and reside in `lisp/`. Package management is handled by `straight.el`.

## Architecture

### Loading Hierarchy

The configuration loads in layers, each building on the previous:

```
init.el
├── w-minimal         → Built-in packages only, core settings
│   ├── w-programming-minimal
│   ├── w-file-minimal
│   ├── w-ui-minimal
│   └── w-font
├── w-straight        → Package manager bootstrap
├── w-essential       → Core third-party packages
│   ├── w-pyim, w-ace-window, w-term-extra, w-dired-extra, w-git
│   ├── w-search-extra, w-minibuffer, w-company
│   ├── w-lisp, w-programming-essential
│   └── w-projectile
└── w-full            → Full feature set
    ├── w-dired-plus, w-term-full, w-org
    ├── w-programming-full
    └── w-aider       → AI coding assistant (aidermacs)
```

### Key Directories

- `lisp/` - Core configuration modules (all prefixed `w-`)
- `site-lisp/` - Local utility packages
- `var/` - Runtime data (bookmarks, history, caches)
- `straight/` - Package management
  - `repos/` - Shared package sources (symlinked)
  - `straight-<version>/` - Version-specific builds

### Package Management

Uses `straight.el` with version-specific isolation via `straight-base-dir`. Packages declared with `(straight-use-package ...)`. Built-in packages are explicitly declared to prevent external reinstallation:

```elisp
(straight-use-package '(org :type built-in))
```

## Key Bindings

All user commands use `M-SPC` as prefix:

- `M-SPC SPC` - `execute-extended-command`
- `M-SPC a` - Aider menu (aidermacs-transient-menu)
- `M-SPC g s` - magit-status
- `M-SPC s p` - consult-ripgrep (or consult-grep)
- `M-SPC f r` - crux-recentf-find-file
- `M-SPC v` - expand-region

## Completion System

- Minibuffer: vertico + orderless + consult + marginalia + embark
- In-buffer: company-mode
- Chinese input: pyim

## Project Detection

Custom `w/project-try-local` detects projects by these markers (in priority order):
1. `.project`
2. `go.mod`, `Cargo.toml`, `project.clj`, `pom.xml`, `package.json`
3. `Makefile`, `README.org`, `README.md`

## AI Integration

Aidermacs (aider integration) is configured in `w-aider.el`:
- Default model: deepseek
- Commit messages in Chinese with specific format
- API keys from auth-source

## Language Support

Tree-sitter modes enabled for: Go, TypeScript, YAML, CMake, Dockerfile. Custom quickrun commands for Maven tests, Deno, Clojure, Scala.

## Terminal True Color

For true color in terminal Emacs (v27.1+):
```bash
TERM=xterm-direct emacs -nw
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aGolduck) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
