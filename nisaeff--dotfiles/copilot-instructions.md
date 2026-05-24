## dotfiles

> - **Lint all scripts**: `find . -name "*.sh" -exec shellcheck {} \;`

# Agent Guidelines for Dotfiles Repository

## Build/Lint/Test Commands

### Shell Script Validation
- **Lint all scripts**: `find . -name "*.sh" -exec shellcheck {} \;`
- **Lint single script**: `shellcheck path/to/script.sh`
- **Syntax check**: `bash -n path/to/script.sh`

### Configuration Validation
- **TOML files**: `python -c "import tomllib; tomllib.load(open('file.toml', 'rb'))"`
- **JSON/JSONC files**: `python -c "import json; json.load(open('file.json'))"`
- **Python configs**: `python -m py_compile path/to/config.py`

## Code Style Guidelines

### Shell Scripts
- Use `#!/bin/bash` shebang; prefer `[[ ]]` over `[ ]` for conditionals
- Lowercase variables with underscores: `my_variable`; always quote: `"$variable"`
- Use section headers: `# -----------------------------------------------------`
- Exit codes: `exit 0` for success, `exit 1` for errors
- Check for existing sessions/processes before creating new ones

### Python Configuration (qutebrowser)
- Import from qutebrowser modules: `from qutebrowser.config.configfiles import ConfigAPI`
- Use type hints for config objects: `config: ConfigAPI = config  # type: ignore`
- Group settings logically with comments (General, Content, Wayland, etc.)
- Include hardware-specific comments (display resolution, Wayland compatibility)

### Configuration Files
- **Indentation**: 4 spaces for YAML/JSON/Python, tabs for KDL/others
- Group related settings with descriptive comments
- Include hardware context in comments (CPU, GPU, display specs)
- Follow tool conventions: KDL for niri, TOML for alacritty/starship, JSONC for waybar

### Documentation
- Place `.md` files in `~/dotfiles/[app-name]/` NOT in `~/.config/`
- Use hyphenated names: `mouse-config.md`, `niri-config.md`
- Include hardware details (USB IDs, display specs) and user preferences
- Document reasoning for non-obvious configuration choices

---
> Source: [NIsaeff/dotfiles](https://github.com/NIsaeff/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
