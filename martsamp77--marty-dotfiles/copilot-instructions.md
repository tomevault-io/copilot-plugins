## marty-dotfiles

> Chezmoi templates and shell bootstrap files in this dotfiles repository


# Chezmoi and shell bootstrap

## Chezmoi templates (`*.tmpl`)

- Preserve **Go template** syntax: `{{`, `}}`, `{{-`, `-}}`, `if`, `range`, and chezmoi data (e.g. `.chezmoi.os`).
- Do not reformat template tags in a way that breaks parsing.
- After non-trivial template edits, prefer `chezmoi diff` or `chezmoi apply --dry-run` when chezmoi is available.

## `dot_*` source files

- `dot_*` deploys to `$HOME` with a leading dot (see [CLAUDE.md](../../CLAUDE.md)).
- Keep line endings and encoding consistent with the rest of the file (often LF for shell config).

## Shell scripts (`*.sh`, `install.sh`)

- Use POSIX-safe patterns for Linux/macOS bootstrap.
- Match existing style for `set -euo pipefail` and error handling.

## Windows

- PowerShell under `windows/` follows normal PowerShell practices; this rule does not load full org-wide PowerShell standards unless those rules exist in this `.cursor` folder.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/martsamp77) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
