## dotfiles

> Personal configuration files (dotfiles) for Linux/macOS development environment including nvim, i3, tmux, zsh, and other tools. Managed with GNU Stow for symlink management.

# Agent Guidelines for Dotfiles Repository

## Repository Overview
Personal configuration files (dotfiles) for Linux/macOS development environment including nvim, i3, tmux, zsh, and other tools. Managed with GNU Stow for symlink management.

## Build/Deploy Commands
- `make dump-keybindings` - Export Cinnamon keybindings to file
- `make load-keybindings` - Import keybindings from file
- `ansible-playbook ansible-home/.ansible/deploy-open-webui.yml -i ansible-home/.ansible/hosts.ini` - Deploy with Ansible
- `stow tool-name` - Create symlinks for a specific tool configuration
- `stow -D tool-name` - Remove symlinks for a specific tool configuration
- `tmux source-file ~/.tmux.conf` - Reload tmux configuration
- `nvim --headless "+Lazy! sync" +qa` - Update Neovim plugins without UI

## Testing
**No test suite** - This is a configuration repository. Testing is done by manual verification, starting new sessions, opening nvim to check LSP/plugin functionality, and sourcing shell configs.

## Code Style Guidelines

### Lua (Neovim, Awesome WM, Yazi configs)
**Indentation:** Use tabs for indentation
**File Structure:**
- nvim plugins: Each plugin in `nvim/.config/nvim/lua/plugins/plugin-name.lua`
- Use `return {}` pattern for plugin specifications
```lua
return {
    {"author/plugin", config = function() ... end},
}
```

**Keybinding Pattern:**
```lua
vim.keymap.set("n", "<leader>ff", builtin.find_files, { desc = "Telescope find files" })
vim.g.mapleader = " "  -- Set before loading plugins
```

**Error Handling:** Use `pcall()` for safe module loading (see awesome/rc.lua). Wrap git/system calls in error checks:
```lua
if vim.v.shell_error ~= 0 then
    vim.api.nvim_echo({{ "Failed to clone lazy.nvim:\n", "ErrorMsg" }}, true, {})
    os.exit(1)
end
```

**LSP Configuration:**
```lua
local capabilities = require("cmp_nvim_lsp").default_capabilities()
lspconfig.gopls.setup({
    capabilities = capabilities,
    filetypes = {"go", "gomod", "gowork", "gotmpl"},
    settings = { completeUnimported = true, usePlaceholders = true },
})
```

### Shell Scripts (Bash/Zsh)
**Shebang:** Use `#!/usr/bin/env bash` or `#!/usr/bin/env sh`
**Quoting:** Always quote variables to prevent word splitting
**Exit on errors:** Use `set -euo pipefail` where appropriate

**Tmux Config:**
- Use `set -g` for global options, `set-window-option` or `setw` for window options
- Use `bind-key` with descriptive comments
- Conditional locale setting for WSL:
```bash
if-shell "locale -a | grep -q 'C.utf8'" "set-environment -g LANG C.utf8"
```

**Error Checking:**
```bash
if ! command -v docker &> /dev/null; then
    echo "Docker not found"
    exit 1
fi
```

### Python (Ranger commands)
**Imports:** Use standard import style with `from __future__ import` for Python 2/3 compatibility
**Class Structure:** Use docstrings for command documentation, implement `execute()` and `tab()` methods
```python
class my_edit(Command):
    """Documentation string visible in :c help"""
    def execute(self):
        if self.arg(1):
            target_filename = self.rest(1)
```

### YAML (Ansible playbooks)
**Indentation:** 2 spaces (strict)
**Best Practices:** Use descriptive task names, `when:` for conditionals, `register:` to capture command output, `changed_when:` to control when tasks report changes

## File Organization
- Each tool has its own directory matching stow structure
- Config files go in `tool-name/.config/tool-name/`
- Root dotfiles (`.zshrc`, `.tmux.conf`) go in `tool-name/` directory
- Use GNU Stow for symlink management
- Example: `nvim/.config/nvim/`, `tmux/.tmux.conf`, `zsh/.zshrc`

## Naming Conventions
- **Lua:** snake_case for variables/functions, PascalCase for classes
- **Shell:** lowercase_with_underscores for functions and variables
- **Python:** snake_case for variables/functions, PascalCase for classes
- **tmux:** lowercase_with_underscores for options (@powerkit_option_name)
- **File names:** lowercase with hyphens or underscores

## Security Guidelines

### Pre-Commit Sensitive Data Check
**CRITICAL:** Always scan for sensitive data before committing. Run these checks:
```bash
rg -i "password|passwd|api_key|apikey|secret|token|private_key|credentials|bearer" --glob '!.git/*'
rg -i "aws_access_key|aws_secret|github_token|gh_token|ssh_key" --glob '!.git/*'
find . -name "*.pem" -o -name "*.key" -o -name "*.env" -o -name "*_rsa" 2>/dev/null | grep -v ".git"
git ls-files | rg -i "ssh|gpg|key|secret|password|credential"
```

### What Should NEVER Be Committed
- SSH keys (`id_rsa`, `id_ed25519`, `*.pem`)
- GPG private keys
- API keys, tokens, or authentication credentials
- `.env` files with secrets (already in `.gitignore`)
- Email addresses or usernames (use placeholders in examples)
- IP addresses or hostnames (use `localhost` or `example.com`)

### Safe to Commit
- Keyboard shortcuts and keybindings
- Theme configurations and color schemes
- Public configuration without credentials
- Example configs with placeholder values

## Platform-Specific Considerations
- **WSL:** May need `C.utf8` locale instead of `en_US.UTF-8`
- **macOS:** Separate zsh config in `zsh-macos/` directory
- **Linux:** Use Linux-specific paths and tools

## .gitignore
Excluded files: `.aider*`, `.env`, `lazy-lock.json`, `.venv`

## Repository Structure
This repository is organized as a GNU Stow collection with each tool having its own directory:

- `zsh/` - Zsh shell configuration with Oh My Zsh
- `zsh-macos/` - macOS-specific Zsh configuration
- `tmux/` - Tmux terminal multiplexer configuration
- `alacritty/` - Alacritty terminal configuration  
- `kitty/` - Kitty terminal configuration
- `kitty-mac/` - macOS-specific Kitty configuration
- `i3/` - i3 window manager configuration
- `awesome/` - Awesome window manager configuration
- `polybar/` - Status bar configuration
- `rofi/` - Application launcher configuration
- `dunst/` - Notification daemon configuration
- `ranger/` - Terminal file manager configuration
- `yazi/` - Modern terminal file manager configuration
- `newsraft/` - RSS reader configuration
- `ansible-home/` - Ansible deployment automation
- `autorandr-home-desktop/` - Monitor profile management
- `ulauncher/` - Application launcher with extensions

## Working with This Repository
1. All configuration files are designed to be portable across Linux/macOS systems
2. Use GNU Stow to deploy configurations: `stow tmux`, `stow zsh`
3. For macOS systems, use `zsh-macos` and `kitty-mac` instead of `zsh` and `kitty`
4. For WSL, ensure proper Nerd Font setup in Windows Terminal
5. Keybinding management uses `make dump-keybindings` and `make load-keybindings` for Cinnamon desktop

## Key Features and Tools
- **Zsh**: Oh My Zsh with starship prompt
- **Tmux**: Includes tmux-powerkit plugin with powerline status bar
- **Alacritty**: GPU-accelerated terminal with Catppuccin themes
- **Kitty**: Alternative terminal with Catppuccin themes
- **i3**: Tiling window manager with configured keybindings
- **Awesome**: Lua-based window manager
- **Themes**: Consistent Catppuccin theming across most terminal tools
- **Tools**: Ranger, Yazi, Rofi, Dunst, Ulauncher, Newsraft

## Configuration Management  
- History settings in zsh are configured to prevent truncation and duplicate entries
- Tmux plugins require manual installation with `Ctrl+a I` after reloading configuration 
- All configurations are designed for reuse across different machines

## Target Audience
This repository is intended for experienced users who understand the tools and configuration concepts. README files are kept concise and focused on platform-specific differences rather than detailed explanations of basic functionality.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ragan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ragan)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
