## dotfiles

> This repo owns a small dotfiles payload: shell config, app config, Brewfile

# AGENTS.md - Dotfiles

## Scope

This repo owns a small dotfiles payload: shell config, app config, Brewfile
tiers, global `mise` config, varlock schema, and shared agent guidance.

## Rules

- Keep this repo boring and repeatable.
- Prefer `mise` for runtimes and project tasks.
- Prefer Homebrew for system tools and GUI apps.
- Keep Brewfiles curated; do not mirror every installed package.
- Do not add bespoke terminal themes, generated shell widgets, or local browser profiles here.
- Do not store secrets in tracked files. Use 1Password and varlock references.
- Keep machine-specific overrides in `~/.zshrc.local`.
- Keep `.agents/skills/` out of this repo; install skills through `thesammykins/skills`.

## Boundaries

- `new-mac` orchestrates fresh-machine setup.
- `dotfiles` owns shell/app config and Brewfile tiers.
- `skills` owns local/custom skills and upstream skill manifests.
- `codex-stuff` owns Codex plugin packaging.

## Validation

```bash
mise run check
mise run audit
DOTFILES_DRY_RUN=1 DOTFILES_LINK_MODE=safe DOTFILES_INSTALL_DEV=1 DOTFILES_INSTALL_WORKSTATION=1 bash scripts/install.sh
```

---
> Source: [thesammykins/dotfiles](https://github.com/thesammykins/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
