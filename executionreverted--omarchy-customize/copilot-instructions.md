## omarchy-customize

> This repository installs an end-user Omarchy rice. It is not an Omarchy source

# Repository guide for coding agents

## Purpose

This repository installs an end-user Omarchy rice. It is not an Omarchy source
fork. Keep every change inside this repository or the user's normal config
directories. Never modify `~/.local/share/omarchy/`; Omarchy owns that tree and
updates it independently.

## Source of truth

- `config/quickshell/command-center/`: Quickshell UI and helper processes
- `config/omarchy/themes/nvimgelion/`: the custom Omarchy theme
- `config/hypr/`: complete lock/idle configs copied by the installer
- `hypr/`: small marked blocks merged into existing Hyprland config
- `config/gtk-3.0/` and `config/gtk-4.0/`: Nautilus/GTK styling
- `fonts/`: bundled display-font assets installed into the user font directory
- `media/`: README preview and demo capture
- `install.sh` and `uninstall.sh`: the public installation contract

Edit the repository copy first, then deploy it to `~/.config` for live testing.
Do not treat a live config edit as the final source.

## Design constraints

- Preserve the Nvimgelion/NERV palette: warm black, coral orange, muted purple,
  restrained green, and pale lavender.
- Evangelion is a display face. Use it for real titles and large numbers, not
  dense project rows or long body text.
- ASCII panels must fit by width first and crop by height. User art comes from
  `~/.ascii` and must not overlap neighboring content.
- Keep all three MAGI layouts usable at 2560x1600 and avoid hard-coding one
  machine's absolute home path.
- Content must remain visible without an entrance animation completing.
- Controls that look interactive must remain functional and pointer-accessible.
- The pet must stay below the dock when the dock is focused, and dragging it
  must not activate UI underneath.

## Safe workflow

1. Read the current file and inspect related bindings before editing.
2. Preserve unrelated user changes in a dirty worktree.
3. Use the repository installer pattern; never write into Omarchy's managed
   source tree.
4. Copy changed command-center files into
   `~/.config/quickshell/command-center/` for a live check.
5. Restart Quickshell cleanly after QML changes.
6. Restart Waybar, Mako, or Hypridle after changing their configuration.
7. After Hyprland changes, run `hyprctl reload` and `hyprctl configerrors`.
8. Capture all three MAGI tabs when changing typography or layout.

## Validation

Run the checks relevant to the change:

```bash
python -m py_compile config/quickshell/command-center/*.py
bash -n install.sh uninstall.sh config/quickshell/command-center/*.sh
git diff --check
hyprctl reload
hyprctl configerrors
```

For QML, restart the live shell and inspect its log:

```bash
qs kill --pid "$(pgrep -f '^qs -p .*/command-center' | head -1)"
qs -p ~/.config/quickshell/command-center -d
qs log --pid "$(pgrep -f '^qs -p .*/command-center' | head -1)" -t 200 --no-color
```

Do not force-kill `hyprlock`. Lock-screen captures require the user to unlock
normally so the active session is never stranded.

## Git hygiene

- Do not commit `__pycache__`, editor backups, generated previews outside
  `media/`, or personal files from `~/.ascii`.
- Before committing screenshots or recordings, inspect the actual frames for
  open terminals, chat text, usernames, hostnames, private repository names,
  tokens, notifications, and other accidental personal data. Metadata-only
  checks are not enough.
- Keep commits focused and preserve the configured author identity.
- Update README feature and install documentation when public behavior changes.

---
> Source: [executionreverted/omarchy-customize](https://github.com/executionreverted/omarchy-customize) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
