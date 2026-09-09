## dotfiles

> User-specific, non-config data. Maps `local/.local/` → `~/.local/`.

# local — user-specific data (stow package)

## Purpose

User-specific, non-config data. Maps `local/.local/` → `~/.local/`.

## Ownership

`~/.local/bin` scripts, dictionaries, keyboard backups, and other durable user data. Owns `local/.stow-local-ignore`.

## Local Contracts

- Stowed via `just stow local`; `local/.local/*` symlinks into `~/.local/`.
- `AGENTS.md` and `__repoImages` are excluded from stow via `.stow-local-ignore`.

## Work Guidance

(none)

## Verification

(none)

## Child DOX Index

No children.

---
> Source: [edheltzel/dotfiles](https://github.com/edheltzel/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
