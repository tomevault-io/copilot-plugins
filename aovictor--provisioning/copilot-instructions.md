## provisioning

> Provisioning on macOS – How this repo works

Provisioning on macOS – How this repo works
===========================================

Overview
--------
This repository automates bootstrapping a macOS development environment. The main entrypoint is `install.sh`, which installs Xcode CLTs, applies your Vim/Zsh configuration, installs Homebrew packages from `brewfile`, sets up Oh My Zsh with Powerlevel10k, installs a Nerd Font, configures Ghostty with Catppuccin themes, and adds common `asdf` language plugins.

Repo layout (key files)
-----------------------
- `install.sh`: One-shot bootstrap script. Run from the repo root.
- `brewfile`: Homebrew bundle manifest used by `brew bundle` to install packages.
- `zshrc` and `p10k.zsh`: Zsh and Powerlevel10k configurations copied to your home directory.
- `vimrc` and `vim/`: Vim configuration and colors (Dracula).
- `ghostty/` and `ghostty_config`: Local Catppuccin Ghostty themes and a sample Ghostty `config`.
- `dracula-theme-master/`: Dracula themes for various apps (reference assets).

What the installer does (high level)
------------------------------------
1. Safety check: refuses to run as `root`.
2. Xcode CLTs: `xcode-select --install` (required for compilers and some Brew formulae).
3. Vim: copies `vimrc` to `~/.vimrc` and `vim/` to `~/.vim`.
4. Dracula iTerm assets: clones Dracula iTerm to `~/git/dracula` (theme assets only; no auto-apply).
5. Permissions fix (if present): takes ownership of `/usr/local/lib/pkgconfig`.
6. Homebrew setup:
   - Copies `brewfile` to `~/.brewfile`.
   - Ensures `brew` shell environment lines are in `~/.zshrc`.
   - Runs `brew bundle` to install formulae/casks listed in `brewfile`.
7. Oh My Zsh + Powerlevel10k:
   - Installs Oh My Zsh.
   - Installs Powerlevel10k theme.
   - Downloads MesloLGS Nerd Font to `~/Library/Fonts`.
   - Copies `zshrc` → `~/.zshrc` and `p10k.zsh` → `~/.p10k.zsh`.
8. Ghostty theming/config:
   - Clones `catppuccin/ghostty` and copies its `themes/` to `~/.config/ghostty/themes`.
   - Copies repo `ghostty_config` to `~/.config/ghostty/config`.
9. asdf language plugins: appends asdf init line to `~/.zshrc` and adds plugins for Go, Ruby, Python, Terraform.

Prerequisites
-------------
- macOS (Apple Silicon supported). The script assumes Homebrew is installed in `/opt/homebrew` (Apple Silicon default).
- Homebrew installed. The installer line for Homebrew in `install.sh` is effectively disabled; install Brew first:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

How to use
----------
Run from the repo root. Do not use `sudo`.

```bash
cd /Users/victorao/git/provisioning
chmod +x install.sh
./install.sh
```

After installation
------------------
- Terminal font: select a MesloLGS NF font in your terminal (Powerlevel10k recommends MesloLGS NF Regular).
- Ghostty theme: Ghostty will see themes in `~/.config/ghostty/themes`. The provided `ghostty_config` sets sensible defaults; adjust per preference.
- Zsh theme: Powerlevel10k will prompt for configuration on first shell start. You can reconfigure later with `p10k configure`.

Notes and behavior
------------------
- Idempotency: most steps are safe to re-run. `brew bundle` is idempotent; `cp -f` overwrites configs.
- Non-root: the script exits immediately if run as `root`.
- Homebrew path: the script adds `eval "$([/opt/homebrew]/bin/brew shellenv)"` to `~/.zshrc`. If you use Intel (`/usr/local`), adjust accordingly.
- Ghostty themes source: the script clones `catppuccin/ghostty` during install even though a `ghostty/` directory exists in the repo. This ensures you get the latest theme set; you can delete the cloned folder afterwards if you prefer.

Troubleshooting
---------------
- Command Line Tools prompt does not appear: open `App Store` → install Xcode, or run `sudo xcode-select --switch /Library/Developer/CommandLineTools` after installing CLTs.
- Homebrew not found: install Brew (see prerequisites) and re-run `./install.sh`.
- Powerlevel10k glyphs look wrong: ensure MesloLGS NF is installed and selected in your terminal profile.
- asdf not initialized: open a new shell or `source ~/.zshrc` so `asdf` is on PATH, then run `asdf plugin list` to verify.

Manual alternatives (from README)
---------------------------------
`README.md` also documents manual setup steps for Pyenv, apps, and themes if you prefer a guided, non-automated setup.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aovictor) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
