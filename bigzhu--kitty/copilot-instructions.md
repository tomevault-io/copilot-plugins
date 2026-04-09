## kitty

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a kitty terminal emulator configuration repository that has been specifically configured to provide a tmux-like experience using kitty's native functionality. The configuration completely replaces tmux with kitty's built-in window management while preserving familiar tmux hotkeys.

## Architecture

### Configuration Structure
- **`kitty.conf`**: Main configuration file containing all settings, hotkey mappings, and theme includes
- **Platform-specific configs**: `macos.conf` and `linux.conf` provide platform-specific overrides
- **Launch configs**: `macos_launch.conf` and `linux_launch.conf` define startup behavior (currently disabled to prevent tmux auto-start)
- **Theme integration**: Uses `kitty-themes/kanagawa.conf` for color scheme
- **Documentation**: `tmux-style-hotkeys.md` contains complete hotkey reference

### Key Design Decisions
The configuration implements a complete tmux replacement strategy:
- **Ctrl+a prefix**: All hotkeys use Ctrl+a as prefix key, matching tmux behavior
- **Native functionality**: Uses kitty's built-in tabs (tmux windows) and windows (tmux panes)
- **tmux migration**: Auto-startup of tmux has been disabled in favor of direct zsh launch
- **Performance optimization**: Configured for minimal latency with `input_delay 0`, `repaint_delay 0`

## Configuration Management

### Applying Changes
- **Live reload**: Use `Ctrl+a r` to reload configuration without restart
- **Full restart**: Required for some changes like startup sessions or platform configs

### Platform Detection
Configuration uses `${KITTY_OS}` variable to include platform-specific files:
- macOS: Includes `macos.conf` 
- Linux: Includes `linux.conf`

## tmux-Style Hotkey System

The configuration provides complete tmux compatibility through kitty's native features:

### Tab Management (tmux windows)
- `Ctrl+a c`: New tab with current directory
- `Ctrl+a x` or `Ctrl+a &`: Close tab
- `Ctrl+a n/p`: Navigate tabs
- `Ctrl+a 0-9`: Jump to specific tab
- `Ctrl+a l`: Last used tab

### Window Management (tmux panes)  
- `Ctrl+a %` or `Ctrl+a \`: Vertical split
- `Ctrl+a "` or `Ctrl+a -`: Horizontal split
- `Ctrl+a w`: Close window
- `Ctrl+a o`: Next window
- `Ctrl+a ←→↑↓`: Navigate windows

### Copy/Paste System
- `Ctrl+a [`: Enter scrollback mode using kitty-scrollback.nvim
- `Ctrl+a ]`: Paste from clipboard
- **Advanced scrollback**: Uses kitty-scrollback.nvim plugin for enhanced functionality

### Scrolling System
- `Ctrl+k`: Scroll up one line (vim-style)
- `Ctrl+j`: Scroll down one line (vim-style)
- `Ctrl+u`: Scroll up one page
- `Ctrl+d`: Scroll down one page

## kitty-scrollback.nvim Integration

### Configuration Requirements
The configuration has been fully integrated with kitty-scrollback.nvim plugin:

1. **Remote Control**: `allow_remote_control yes` - Required for plugin communication
2. **Unix Socket**: `listen_on unix:/tmp/kitty` - Enables remote control via socket
3. **Shell Integration**: `shell_integration enabled` - Required for advanced features
4. **Plugin Configuration**: Located in `~/.config/nvim/lua/plugins/kitty-scrollback.lua`

### Plugin Features
- **Conflict-free scrollback**: Each instance runs independently, solving multi-tab swap file issues
- **Enhanced navigation**: Full Neovim functionality for searching, copying, and text manipulation
- **Read-only mode**: Automatically configured to prevent save prompts on exit
- **System clipboard integration**: Direct copying with `y` key in visual mode

### Available Commands
- `Ctrl+a [` or `Ctrl+Shift+h`: Open scrollback buffer in Neovim
- `Ctrl+Shift+g`: View last command output in Neovim
- `Ctrl+Shift+Right-click`: View clicked command output in Neovim

## Special Features

### Keyboard Enhancements
- **Shift+Enter**: Sends `\x1b\x0d` (Alt+Enter sequence) for cross-platform newline behavior
  - Compatible with both kitty Claude Code dialog and nvim command line
  - Allows multi-line input without command submission
- **Ctrl+Enter**: Sends `\x1b[13;5u` for enhanced terminal support

### Performance Configuration
- **Massive scrollback**: 1,000,000 lines to match tmux configuration
- **Zero latency**: Input and repaint delays set to 0
- **Layout system**: Multiple built-in layouts (tall, stack, fat, grid, horizontal, vertical, splits)

## Session Management

Kitty does not have native session persistence like tmux. When kitty crashes or is closed, the window layout and tabs are lost.

**Recommendation**: Use tmux for proper session management and crash recovery. Tmux maintains sessions in the background and can restore them after a crash or unexpected close.

## Theme and Appearance

### Current Theme
- **Primary**: kanagawa.conf from kitty-themes submodule
- **Font**: JetbrainsMono Nerd Font with auto variants
- **Platform fonts**: Different sizes for macOS (13.0) vs Linux (16.0)

### Theme Management
Themes are managed via the `kitty-themes` git submodule. To change themes:
1. Edit the `include` line in `kitty.conf`
2. Reload with `Ctrl+a r`

## Development and Maintenance

### Testing Configuration Changes
1. Make changes to relevant `.conf` files
2. Use `Ctrl+a r` to reload (for most changes)
3. Restart kitty if changes don't take effect
4. Test all tmux-style hotkeys to ensure compatibility

### Adding New Hotkeys
Follow the established pattern in `kitty.conf`:
- Use `ctrl+a>` prefix for all tmux-style bindings
- Group related functionality with comments
- Use `--cwd=current` for directory preservation
- Document new hotkeys in `tmux-style-hotkeys.md`

### Git Workflow
The repository tracks configuration evolution with detailed commit messages. When making changes:
- Always update documentation if adding/changing hotkeys
- Test on both platforms if possible
- Include performance impact in commit messages

## Troubleshooting Common Issues

### tmux Auto-Start Prevention
If kitty starts tmux automatically, check:
1. `startup_session` lines are commented in `macos.conf`/`linux.conf`
2. Launch configs contain `launch zsh` not tmux commands

### Hotkey Conflicts
If hotkeys don't work:
1. Check for conflicting system shortcuts
2. Verify `ctrl+a>` prefix syntax in mappings
3. Use `kitty --debug-input` to diagnose key events
4. **Key symbol names**: Use correct kitty key names:
   - `[` = `left_bracket`, `]` = `right_bracket`
   - `$` = `4` (or `shift+4`)
   - `&` = `shift+7`
   - `?` = `shift+slash`
   - `"` = `shift+apostrophe`
   - `%` = `shift+5`

### Configuration Reload Issues
If `Ctrl+a r` doesn't reload properly:
1. Some multi-key sequences require full kitty restart
2. Check kitty version compatibility (tested on 0.40.1)
3. Use `test_config.conf` for isolated testing

### Theme Issues
If themes don't apply:
1. Ensure `kitty-themes` submodule is properly initialized
2. Check `include` path in `kitty.conf`
3. Verify theme file exists and has proper syntax

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bigzhu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bigzhu)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
