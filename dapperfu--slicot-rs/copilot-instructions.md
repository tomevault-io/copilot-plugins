## slicot-rs

> Git user configuration for automated commits with model name specification


# Git User Configuration

## Requirement

Before committing, the git user SHALL be configured to identify automated commits from AI assistance. The configuration SHALL use OS-specific syntax based on the development environment.

## Name Configuration

The git `user.name` SHALL be configured to `"$(whoami) | Cursor.sh | <model>"` (or OS-specific equivalents) to identify automated commits from AI assistance.

### OS-Specific Syntax

The command syntax SHALL be selected based on the operating system and shell:

**OS-specific syntax mapping:**
- **Unix-like (Linux/macOS/Git Bash on Windows):** Use `$(whoami)` for username and `$(hostname)` for hostname
- **Windows CMD:** Use `%USERNAME%` for username and `%COMPUTERNAME%` for hostname
- **Windows PowerShell:** Use `$env:USERNAME` for username and `$env:COMPUTERNAME` for hostname

**Note:** Git Bash on Windows uses Unix syntax (`$(whoami)` and `$(hostname)`), not Windows CMD syntax.

### OS Detection and Command Selection

To determine which command syntax to use:

1. **Identify your shell environment:**
   - If using Git Bash (even on Windows), use Unix syntax
   - If using Windows Command Prompt (CMD), use Windows CMD syntax
   - If using Windows PowerShell, use PowerShell syntax
   - If using Linux/macOS terminal, use Unix syntax

2. **Use the appropriate syntax:**
   - Check your shell prompt or run `echo $SHELL` (Unix) or `echo %COMSPEC%` (Windows CMD) to identify your shell
   - Git Bash on Windows will show a Unix-like prompt and use Unix syntax

## Model Name Specification

The `<model>` placeholder SHALL be replaced with the actual model name. The literal strings `"<model>"` or `"model"` SHALL NOT be used.

### Valid Model Names

The following are valid model names that SHALL be used:
- **Auto** - For Auto (agent router)
- **Composer 1** - For Composer 1
- **GPT-5.1** - For GPT-5.1
- **Claude Sonnet 4** - For Claude Sonnet 4
- Any other actual model name currently in use

### Examples

**Unix-like (Linux/macOS/Git Bash):**
```bash
git config user.name "$(whoami) | Cursor.sh | Auto"
git config user.name "$(whoami) | Cursor.sh | Composer 1"
git config user.name "$(whoami) | Cursor.sh | GPT-5.1"
```

**Windows CMD:**
```cmd
git config user.name "%USERNAME% | Cursor.sh | Auto"
git config user.name "%USERNAME% | Cursor.sh | Composer 1"
git config user.name "%USERNAME% | Cursor.sh | GPT-5.1"
```

**Windows PowerShell:**
```powershell
git config user.name "$env:USERNAME | Cursor.sh | Auto"
git config user.name "$env:USERNAME | Cursor.sh | Composer 1"
git config user.name "$env:USERNAME | Cursor.sh | GPT-5.1"
```

**Incorrect Usage (SHALL NOT be used):**
```bash
git config user.name "$(whoami) | Cursor.sh | <model>"  # Literal placeholder
git config user.name "$(whoami) | Cursor.sh | model"     # Literal string "model"
```

## Email Configuration

Before committing, the git `user.email` SHALL be configured based on the operating system:

- **Unix-like (Linux/macOS/Git Bash on Windows):** `"$(whoami)@$(hostname).local"`
- **Windows CMD:** `"%USERNAME%@%COMPUTERNAME%.local"`
- **Windows PowerShell:** `"$env:USERNAME@$env:COMPUTERNAME.local"`

**Note:** Git Bash on Windows uses Unix syntax (`$(whoami)@$(hostname).local`), not Windows CMD syntax.

### Examples

**Unix-like (Linux/macOS/Git Bash):**
```bash
git config user.email "$(whoami)@$(hostname).local"
```

**Windows CMD:**
```cmd
git config user.email "%USERNAME%@%COMPUTERNAME%.local"
```

**Windows PowerShell:**
```powershell
git config user.email "$env:USERNAME@$env:COMPUTERNAME.local"
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dapperfu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
