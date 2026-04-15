## shell-config

> - macOS (Apple Silicon) primary, Linux secondary, **never Windows**

# Shell-Config Cursor Rules

## Platform
- macOS (Apple Silicon) primary, Linux secondary, **never Windows**
- Bash 5.x required (macOS: `brew install bash`)
- Zsh 5.9 default interactive shell

## Critical Rules

### Bash 5.x Required
macOS users must install Homebrew bash: `brew install bash`
Modern features now allowed: `declare -A`, `${var,,}`, `readarray`, `|&`

### Non-Interactive Commands (REQUIRED)
- All commands must run without user input
- Must fail loudly with non-zero exit on error
- Never hang waiting for input

### Error Messages (REQUIRED FORMAT)
```bash
echo "❌ ERROR: gitleaks not installed" >&2
echo "ℹ️  WHY: Required for secrets scanning in pre-commit hooks" >&2
echo "💡 FIX: brew install gitleaks" >&2
exit 1
```

### File Limits
- Target: 600 lines
- Max: 800 lines (must split)

### Testing
- Every new function needs tests
- Run `./tests/run_all.sh` before commit

### Mandatory Patterns
- Trap handlers for temp files: `trap 'rm -f "$temp"' EXIT INT TERM`
- Source shared colors: `source "$(dirname "${BASH_SOURCE[0]}")/../common/colors.sh"`
- Quote all variables: `"$var"`
- Check deps before using:
```bash
if ! command -v tool >/dev/null 2>&1; then
    echo "❌ ERROR: 'tool' is not installed" >&2
    echo "ℹ️  WHY: Required for X functionality" >&2
    echo "💡 FIX: brew install tool" >&2
    exit 1
fi
```

### Security
- Never pipe curl to sh: download, verify, then execute
- Always quote variables to prevent injection

## Before Commit
```bash
shellcheck --severity=warning file.sh
./tests/run_all.sh
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nichm) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
