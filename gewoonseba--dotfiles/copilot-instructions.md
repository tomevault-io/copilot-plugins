## dotfiles

> This repository contains GNU Stow-managed dotfiles shared across Sebastian's Linux

# AGENTS.md

This repository contains GNU Stow-managed dotfiles shared across Sebastian's Linux
desktop and MacBook. Changes to shared configuration may affect either or both
machines; preserve platform-specific behavior and directory variants where they
exist.

## Machine Access

Before accessing either machine, or changing SSH or Git configuration, read the
`machine-ssh` skill at `agents/.agents/skills/machine-ssh/SKILL.md`. It documents
how to identify the current machine, the SSH alias for reaching the other machine,
and the dedicated SSH and Git credential behavior that must be preserved.

## Repository Guidance

Follow the architecture, setup commands, Stow conventions, script requirements,
and behavioral guidelines in `CLAUDE.md`. In particular:

- Keep each Stow package laid out as if its contents were rooted at `$HOME`.
- Put install and maintenance scripts in `scripts/.local/bin/`.
- Keep shared configuration portable across Linux and macOS; use the existing
  platform-specific packages when behavior must differ.
- Make surgical changes and leave unrelated working-tree changes untouched.

---
> Source: [gewoonseba/dotfiles](https://github.com/gewoonseba/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
