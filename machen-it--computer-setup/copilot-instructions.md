## computer-setup

> Interactive shell setup script. Detects OS (mac/linux) and shell (bash/zsh) automatically, then walks through each setup step with a y/n prompt.

# computer_setup

Interactive shell setup script. Detects OS (mac/linux) and shell (bash/zsh) automatically, then walks through each setup step with a y/n prompt.

## Usage

```bash
bash setup.sh
```

## Adding a new step

1. Add any config/snippet files to `configs/` if needed.
2. In `setup.sh`, add a new section following the existing pattern:

```bash
# ── step N: description ───────────────────────────────────────────────────────
echo -e "${BOLD}Step N: your step name${RESET}"
echo "  One-line description of what this does."
echo ""

if ask "Install X?"; then
  # do the work
  success "X installed"
else
  skip
fi

echo ""
```

Use the `$OS` variable (`mac` / `linux`) and `$SHELL_RC` (path to active shell config) for any platform-specific logic.

## Variables available in setup.sh

| Variable        | Example values                      |
|-----------------|-------------------------------------|
| `$OS`           | `mac`, `linux`                      |
| `$SHELL_NAME`   | `zsh`, `bash`                       |
| `$SHELL_RC`     | `~/.zshrc`, `~/.bashrc`             |
| `$CLIPBOARD_CMD`| `pbcopy`, `wl-copy`, `xclip ...`    |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/machen-it) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
