## dotfiles

> Personal dotfiles. Files in this repo are the source of truth — `~/.config/*` are symlinks pointing here. Theme: Tokyo Night across all tools.

# CLAUDE.md

## Repo

Personal dotfiles. Files in this repo are the source of truth — `~/.config/*` are symlinks pointing here. Theme: Tokyo Night across all tools.

## Symlink map

| Repo path | Symlinked to |
|-----------|-------------|
| `shell/nushell/` | `~/.config/nushell` |
| `dev/nvim/` | `~/.config/nvim` |
| `shell/starship/starship.toml` | `~/.config/starship.toml` |
| `apps/kitty/` | `~/.config/kitty` |
| `wm/wallpapers/` | `~/.config/wallpapers` |
| `shell/zoxide/config.nu` | `~/.config/zoxide/config.nu` |
| `apps/yazi/` | `~/.config/yazi` |
| `apps/wallrizz/` | `~/.config/WallRizz` |
| `apps/whosthere/` | `~/.config/whosthere` |
| `wm/wayland/hypr/` | `~/.config/hypr` |
| `wm/wayland/waybar/` | `~/.config/waybar` |
| `wm/wayland/swaync/` | `~/.config/swaync` |
| `wm/cursor/icons-default/index.theme` | `~/.icons/default/index.theme` |
| `wm/applications/rofi-filebrowser.desktop` | `~/.local/share/applications/rofi-filebrowser.desktop` |
| `apps/btop/` | `~/.config/btop` |
| `apps/fastfetch/` | `~/.config/fastfetch` |
| `apps/cava/` | `~/.config/cava` |
| `apps/rofi/` | `~/.config/rofi` |
| `dev/claude/settings.json` | `~/.claude/settings.json` |
| `shell/scripts/cava-vibe` | `~/.local/bin/cava-vibe` |
| `system/nixos/configuration.nix` | `/etc/nixos/configuration.nix` (sudo) |

Run `./doller` to (re)create all symlinks. TUI shows status per link, backs up conflicts. `--dry-run` to preview, `--force` to skip prompt. `install.sh` is a shim that calls it.

---

## Nushell

**Config:** `nushell/config.nu` | **Aliases:** `nushell/aliases/<tool>/<tool>-aliases.nu`

### Shell behavior
- Vi mode — block cursor in normal, line in insert; `󱄅 N` / `󱄅 I` indicators
- History: SQLite, 100k entries, per-session, dedup, timestamps, auto-sync on entry
- Zoxide smart directory jumping (`z`, `zi`)
- `$EDITOR` / `$VISUAL` / `buffer_editor` all set to `nvim`

### Hooks
- **Pre-prompt title:** Updates kitty tab bar every prompt: `~/path 󰊢 branch [+staged ~modified ?untracked] | Nf Nd`
- **PWD change:** Auto-clears screen on `cd` to any non-home directory
- **Greeting:** Fastfetch + centered fortune "Quote of the Day" — only fires when `NU_BANNER=1` is in env (set by waybar distro logo kitty launch)

### FZF keybinds
| Key | Action |
|-----|--------|
| `Ctrl+R` | Fuzzy history search |
| `Ctrl+T` | Fuzzy file picker → insert path (fd-powered) |
| `Alt+C` | Fuzzy cd to directory |

### Alias modules
| Module | Key aliases |
|--------|-------------|
| `bat` | `b`, `bn` (numbered), `bp` (plain), `bl` |
| `docker` | 35+ aliases — build, image, container, network, volume; `dsta` (stop all) |
| `git` | 150+ aliases + helpers (`git_current_branch`, `git_main_branch`, worktree, rebase flows) |
| `nixos` | `nixos-edit`, `nixos-re-sw` (rebuild switch), `nixos-garbage` (7d+ cleanup) |
| `nvim` | `n` |
| `rip` | `rm` → rip (trash), `rd` (restore deleted) |
| `utils` | `grep`→rg, `find`→fd, `du`→dust, `top`→btop, `cb` (copy), `cbp` (paste) |
| `chezmoi` | `ch`, `chad`, `chap`, `chd`, `chda`, `chs` |

### Custom completions
Bat, Docker, GitHub CLI, Git, Less, Make, Man, Nix, SSH, Tar, tldr, uv, Zoxide

---

## Starship

**Config:** `starship/starship.toml`

- Single-line, no newline
- **Left:** Language icons (C, Python, Node, Rust, Go, Java, Docker) — appear only in matching project dirs
- **Right:** Username (purple `#bb9af7`; red `#f7768e` for root)
- No cmd_duration, no directory/git in starship (handled by nushell pre-prompt hook + kitty tab bar)

### Git status symbols
`⇡N` ahead · `⇣N` behind · `⇕` diverged · `~` conflicted · `*` stashed · ` ` modified · `++N` staged · `?` untracked · ` ` deleted

---

## Kitty

**Config:** `kitty/kitty.conf` | **Tab bar:** `kitty/tab_bar.py`

- Font: JetBrainsMono Nerd Font 12pt
- Tokyo Night colors, 75% opacity, frosted blur (via Hyprland window rule)
- Scrollback: 10,000 lines; padding 8px; block cursor, no blink
- Repaint 10ms, input delay 3ms, sync to monitor
- Remote control via Unix socket `/tmp/kitty`

### Custom tab bar (`tab_bar.py`)
Parses nushell pre-prompt title (`~/path 󰊢 branch [+1 ~2] | 5f 2d | 14:23`):
- **Single tab:** Title centered full-width
- **Multi-tab:** Segments — directory (blue), branch (purple), file counts (cyan), time (green)
- Colors: bg `#1a1b26`, blue `#7aa2f7`, purple `#bb9af7`, cyan `#7dcfff`, green `#9ece6a`

---

## Hyprland

**Config:** `wayland/hypr/hyprland.lua` (Lua-based)

### Autostart
`waybar`, `swaync`, `hypridle`, `awww-daemon` (wallpaper: `Nix Tokyo Night.png`, 2s fade), `hyprctl setcursor Layan-cursors 24`

### Appearance
- Gaps: in 5px / out 20px; border 3px; rounding 10px
- Active border: liquid gradient blue→purple→cyan (animated, angle loops)
- Inactive border: animated gradient — same hues as active at 33% opacity (`rgba(...55)`)
- Shadow: `#7aa2f7` 22% / inactive `#1a1b26` 12%
- Blur: 16px, 6 passes, vibrancy 0.3

### Display configuration
**Tool:** `wdisplays` GUI for display layout (Super+M)  
Configure monitor arrangement, resolution, refresh rate graphically.

### Keybindings (Super key)
| Binding | Action |
|---------|--------|
| `Super+Return` | Open kitty |
| `Super+Q` | Close window |
| `Super+F` | File manager (kitty -e yazi) |
| `Super+E` | Rofi app launcher |
| `Super+M` | Display configuration (wdisplays) |
| `Super+W` | WallRizz wallpaper picker |
| `Super+N` | Toggle swaync notification center |
| `Super+C` | Keybindings cheatsheet (fzf browser) |
| `Super+T` | Toggle dwindle split |
| `Super+Shift+L` | Lock (hyprlock) |
| `Super+Shift+W` | LAN discovery (whosthere) |
| `Super+R` | Reload config |
| `Super+Shift+S` | Region screenshot → clipboard |
| `Print` | Full screenshot → `~/Pictures/` |
| `Super+H/L/K/J` | Focus left/right/up/down |
| `Super+[0-9]` | Switch workspace |
| `Super+Shift+[0-9]` | Move window to workspace |
| `Super+Mouse Scroll` | Cycle workspaces |
| `Super+LMB drag` | Move window |
| `Super+RMB drag` | Resize window |
| `XF86Audio*` | Volume / mic via wpctl |
| `XF86Brightness*` | Brightness via brightnessctl |
| `XF86Audio Next/Play/Pause/Prev` | Playerctl |

### Window rules
- WallRizz: float, 900×600, centered
- Suppress app maximize requests globally

### Input
- Keyboard layout: Spanish (es); Capslock → Escape (keyd)
- Touchpad: natural scroll off; 3-finger horizontal swipe → workspace switch
- Epic mouse: −0.5 sensitivity

---

## Hyprlock

**Config:** `wayland/hypr/hyprlock.conf`

- Background: blurred current wallpaper (8px, 3 passes) + fallback `#1a1b26`
- No loading bar, hide cursor, no grace period
- Password field: 300×50px, blue border `#7aa2f7`, centered (−80px offset)
- Clock: 72pt bold, HH:MM, 1s refresh
- Date: 18pt, `Day, Month DD` format, 60s refresh

---

## Hypridle

**Config:** `wayland/hypr/hypridle.conf`

| Timeout | Action |
|---------|--------|
| 5 min | Dim to 20% brightness |
| 10 min | Lock session (loginctl) |
| 11 min | Turn off display |
| 30 min | Suspend system |

Before sleep → lock. After wake → display on.

---

## Waybar

**Config:** `wayland/waybar/` (modular jsonc + styles)

### Modules
| Module | Description |
|--------|-------------|
| `custom/user` (󱄅) | NixOS GTK menu (NixOS logo icon): Edit config, Rebuild Switch/Test, Update Flake, Garbage Collect, Generations, Rice (opens fastfetch+fortune kitty). Each action opens kitty, tees output to `/tmp/nix-<action>.log`, waits Enter to close |
| `custom/eyecare` (󰛐) | 20-20-20 eye break timer. Lives in `group/time` pill with clock. Fires swaync notification every 20min, auto-resets. Click to reset early. State file: `~/.local/state/eyecare-start`. Script: `wayland/waybar/scripts/eyecare` |
| Temperature | CPU temp, critical at 90°C |
| Memory | % usage, tooltip shows used/total GiB |
| CPU | % usage, updates 10s, warn 75%, critical 90% |
| Clock time | HH:MM, tooltip 12h AM/PM |
| Clock date | DD-MM + calendar popup on click |
| Network | WiFi signal bars, SSID/IP/freq tooltip; click → network menu; right-click → toggle WiFi |
| `custom/vpn` (󰦝) | vpnc VPN manager. Green=connected (IP tooltip), red=disconnected. Click → VPNC TUI fzf menu: connect/disconnect/import/edit configs. Configs at `~/.config/vpns/` (untracked) |
| Bluetooth | Connected/battery % tooltip; click → bluetooth menu |
| Notifications | swaync integration, icon changes per state (normal/DND/inhibited) |
| Updates | Hourly package check; click → update TUI; right-click → refresh |
| Idle inhibitor | Toggle prevent screen sleep |
| MPRIS | Current track title - artist |
| PulseAudio output | Volume %, scroll ±5%, click mute; tooltip shows device |
| PulseAudio input | Mic level, muted state, scroll to change |
| Backlight | Brightness %, scroll to adjust |
| Battery | % + charge icon; notifications at 20% warn / 10% critical / 100% full |
| Power menu (󰍜) | Lock / Suspend / Hibernate / Reboot / Shutdown / Logout |

### Styling
Tokyo Night palette; buttons with 20px border-radius; layer top, dock mode.

---

## Swaync

**Config:** `wayland/swaync/`

- Position: top-right, 400px wide, 400×600 control center
- Animations: slide-right, 200ms open / 100ms close
- Timeouts: 5s default, 3s low, 0 (sticky) critical
- Widgets: title bar + clear-all, DND toggle, notifications list, MPRIS player (72px rounded art)
- Keyboard shortcuts enabled

---

## WallRizz

**Config:** `wallrizz/`

- Kitty window, class `wallrizz`, 90% opacity, lists `~/.config/wallpapers`
- `awww@daniel.js`: random transitions (fade/wipe/wave/grow/center/outer), 1s duration, 60fps, random position
- `color-backend.sh`: ImageMagick kmeans extraction of 16 hex colors per image for theme sync

---

## NixOS

**Config:** `nixos/configuration.nix`

### System
- Boot: systemd-boot, quiet, latest kernel, max 3 generations
- Audio: PipeWire + PulseAudio compat layer + ALSA 32-bit + RTKit
- Display: Hyprland enabled, X server + dconf + polkit
- Docker: rootless mode
- Printing: CUPS
- Network: NetworkManager, hostname `nixos`
- Locale: `en_US` / `es_ES` regional / Europe/Madrid timezone
- Shell: bash auto-launches nushell for interactive sessions
- Fonts: JetBrains Mono Nerd Font + Material Icons

### Notable user packages
Wayland: `hyprlock`, `hypridle`, `awww`, `waybar`, `swaync`, `hyprpicker`, `blueman`, `fastfetch`, `fortune`, `fzf`
Terminal: `kitty`, `starship`, `zoxide`, `yazi`, `bat`, `ripgrep`, `fd`, `delta`, `btop`, `dust`, `rm-improved`, `mapscii`
Dev: `neovim`, `git`, `git-lfs`, `gcc`, `uv`, `nodejs`, `claude-code`
Apps: `brave`, `teams-for-linux`, `keepass`, `veracrypt`

### Custom packages
- `wallrizz` v1.4.0 (auto-patch-elf)

### Private certificates
Certs live at `~/.config/certs/*.crt` — outside the repo, untracked. `configuration.nix` loads them automatically via `SUDO_USER` at rebuild time. If the directory is absent, no certs are added (graceful). To add a cert: drop a `.crt` file in `~/.config/certs/` and run `nixos-re-sw`.

---

## Useful aliases (nushell)

| Alias | Command |
|-------|---------|
| `nixos-re-sw` | `sudo nixos-rebuild switch` |
| `nixos-edit` | `sudo nvim /etc/nixos/configuration.nix` |
| `nixos-garbage` | `sudo nix-collect-garbage --delete-older-than 7d` |
| `n` | `nvim` |
| `z` | zoxide jump |
| `rm` | rip (trash, recoverable) |
| `rd` | restore from rip trash |
| `grep` | ripgrep |
| `find` | fd |
| `du` | dust |
| `top` | btop |
| `cb` | wl-copy |
| `cbp` | wl-paste |

---

## Non-Nix tools (uv-managed)

Some tools aren't in nixpkgs. These are installed via `uv tool install` (uv is in system packages) and live in `~/.local/share/uv/tools/`. Binaries exposed at `~/.local/bin/`.

| Tool | Install | Purpose | Status |
|------|---------|---------|--------|
| `claude-swap` (`cswap`) | `uv tool install claude-swap` | Multi-account Claude Code manager — switch accounts, track quota, auto-switch before rate limits | ⏳ pending nixpkgs / custom derivation |

To reinstall after a fresh system: `uv tool install claude-swap && echo "Y" \| cswap list` (registers current Claude account).
To upgrade: `cswap upgrade` or `uv tool upgrade claude-swap`.

> **TODO:** Package these as proper Nix derivations once in nixpkgs, or write `buildPythonApplication` derivations. Track at: https://github.com/NixOS/nixpkgs/issues (search "claude-swap").

---

## What NOT to do

- Don't edit files in `~/.config/` directly — they're symlinks, edits land in the repo automatically.
- Don't add helix back.
- Don't symlink `hardware-configuration.nix` on different hardware — regenerate with `nixos-generate-config` first.
- Don't put certs in the repo — they live at `~/.config/certs/`, outside version control.
- Don't remove `nofail` from the SHARED-PART mount — it must not block boot if missing.
- Don't set `NU_BANNER=1` in a normal kitty launch — it triggers fastfetch/fortune on every new shell in that window.
- Don't add styles to `wayland/waybar/styles/modules-*.css` — those files are **not imported** anywhere and have zero effect. All waybar CSS goes in `wayland/waybar/style.css`.
- Don't symlink cursor dirs to `~/.config/cursors` expecting X fallback — standard path is `~/.icons/`. `~/.icons/default/index.theme` controls the X11 default cursor (was `Inherits=NixCursor`, fixed to `Inherits=Layan-cursors`).

---
> Source: [daniel-g0/dotfiles](https://github.com/daniel-g0/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
