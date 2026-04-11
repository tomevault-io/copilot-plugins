## dotfiles-tui

> Implement a master-stack window layout for Aerospace tiling window manager:

# Aerospace Master-Stack Layout Implementation

## Goal
Implement a master-stack window layout for Aerospace tiling window manager:
- Oldest window on the **left** (master)
- All newer windows stacked **vertically on the right**

## Challenges Encountered

### 1. Window Age Detection
- Aerospace doesn't provide window creation timestamps
- macOS window server order reflects focus/stacking order, not creation time
- **Solution**: State file tracking (`/tmp/aerospace-layout-state/workspace-{n}.state`)

### 2. Floating Windows Interference
- Aerospace floating windows are part of the tiling tree structure
- They insert themselves between tiling windows and break navigation
- **Solution**: Move floating windows to separate workspace (`FLOAT-TEMP`)

### 3. Temp Workspace Names
- Aerospace reserves names starting with underscore
- **Solution**: Use `LAYOUT-TEMP` and `FLOAT-TEMP`

### 4. `flock` Not Available on macOS
- Script initially used `flock` for locking which doesn't exist on macOS
- **Solution**: PID-based lock file mechanism

### 5. Normalization Setting
- `enable-normalization-flatten-containers = true` prevented nested layouts
- **Solution**: Set to `false` in aerospace config

## Final Implementation

### Script: `~/.config/aerospace/auto-layout.sh`

**Algorithm:**
1. Get current workspace
2. Load window order from state file (preserves age)
3. Append new windows (sorted by ID)
4. Move floating windows to `FLOAT-TEMP`
5. Move tiling windows to `LAYOUT-TEMP`
6. Move tiling back in order (oldest first)
7. Flatten workspace tree
8. For 3+ windows: join newest windows with `join-with left` to create vertical stack
9. Balance sizes

### Config Changes: `~/.config/aerospace/aerospace.toml`

```toml
# Disabled to allow nested containers for master-stack
enable-normalization-flatten-containers = false

# Keybindings
alt-r = 'exec-and-forget ~/.config/aerospace/auto-layout.sh'  # Manual re-layout
alt-f = 'workspace FLOAT-TEMP'  # Access floating windows

# Auto-trigger on new window
[[on-window-detected]]
run = ['exec-and-forget ~/.config/aerospace/auto-layout.sh']
```

## Usage

| Action | Result |
|--------|--------|
| `alt+enter` | Open new Ghostty window (triggers auto-layout) |
| `alt+r` | Manually re-apply layout (use after closing windows) |
| `alt+f` | Switch to floating windows workspace |

## Limitations

1. **Floating windows**: Must be kept on separate workspace (`FLOAT-TEMP`) to avoid interfering with tiling navigation
2. **Event-driven only**: No polling; `alt+r` required after closing windows
3. **State persistence**: State files in `/tmp` are cleared on reboot; window order resets

## Layout Result

```
┌─────────────┬─────────────┐
│             │   Window 2  │
│   Master    ├─────────────┤
│  (oldest)   │   Window 3  │
│             ├─────────────┤
│             │   Window 4  │
└─────────────┴─────────────┘
```

Navigation:
- From master: `right` → stack
- From stack: `left` → master, `up/down` → other stack windows

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mr-narender)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mr-narender)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
