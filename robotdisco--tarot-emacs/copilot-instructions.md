## tarot-emacs

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

tarot emacs is a plugin for emacs that will draw a tarot card deck for you, and provide you a reading.

## Developer Context
- I am new to Emacs Lisp development and the Elisp package ecosystem. Do not assume I know any emacs command, variable, or function other than basic emacs file editing functionality.
- Please explain all Elisp idioms and conventions when using them
- Please explain the emacs entities and concepts I will have to operate with, teaching me how to think about emacs itself when developing for it.
- Prefer clear, readable code over clever/terse approaches
- Please explain the workflows and toolings that are used an emacs lisp software development lifecycle; prefer simple solutions over complex or advanced ones, to teach me the concepts gradually.

## Conventions
- Follow standard Elisp package conventions (file headers, namespace prefixing, provide/require)
- Prefix all public functions and variables with `tarot-`
- Include docstrings on all public functions
- Write ERT tests for core logic (deck construction, card drawing, displayin revealed cards, spreads)

## Design Decisions
- Cards are plists (e.g., `(:name "The Fool")`) — adopted out of convenience, not a deliberated choice. Could revisit if the schema outgrows plists.
- Major arcana have `:name` only. Minor arcana have `:rank` and `:suit` only — no `:name` stored, it is always derived on demand.
- `tarot-card-name` is the single accessor for any card's display name — always use this, not `plist-get :name` directly. It handles both arcana types.
- Major vs. minor arcana are implicitly distinguished by presence of `:rank`. No `:type` field — YAGNI until an iteration forces the distinction into the open.
- `tarot-shuffle` is the stable public API for shuffling. `tarot--shuffle` contains the Fisher-Yates implementation, kept separate as it may be extracted to a generic utility later.

## Commands
- Load for development: Open tarot.el in Emacs and run `M-x eval-buffer`
- Run tests interactively: Open tarot.el in Emacs and run `M-x ert-run-tests-interactively`
- Run all checks (lint + tests): `just`
- Run tests only: `just test`
- Auto-indent source file: `just fmt`
- See all available recipes: `just --list`

## Dev Environment
- `shell.nix` provides `just` via nix — run `direnv allow` once after cloning to activate automatically
- Or enter manually with `nix-shell`

## Pre-commit Linting Routine
The `just` command runs the full routine automatically. To run steps manually in Emacs:
1. `C-x h` then `C-M-\` — select all and re-indent to standard Elisp style (or `just fmt`)
2. `C-x C-s` — save
3. `M-x byte-compile-file RET RET` — check for errors, undefined refs, wrong arg counts (check `*Compile-Log*` buffer)
4. `M-x checkdoc` — verify docstrings follow Elisp conventions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RobotDisco) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
