## dotfiles

> Personal config files (zsh, git, tmux, nano, mc) symlinked into `$HOME` via `install.sh`.

# Dotfiles

Personal config files (zsh, git, tmux, nano, mc) symlinked into `$HOME` via `install.sh`.

## Related Repos
- **Homelab** (`/Users/driversti/Projects/homelab`): Provisions LXC containers that deploy these dotfiles. The provisioning script (`provisioning/lxc-template/modules/30-user-yurii.sh`) clones this repo and runs `install.sh`. When changing configs here, also test on a provisioned LXC.

## Config Locations
- `zsh/.zshrc` — Oh My Zsh config with plugins
- `git/.gitconfig` — Git user config
- `tmux/.tmux.conf` — Ctrl-a prefix, mouse, intuitive splits
- `nano/.nanorc` — Line numbers, syntax highlighting, tabs-to-spaces
- `mc/` — Midnight Commander config + dark skin

## Notes
- Configs must be compatible with **Debian Trixie** (current LXC base)
- `install.sh` is idempotent — creates symlinks, backs up existing files to `~/.dotfiles_backup/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/driversti) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
