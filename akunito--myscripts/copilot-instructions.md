## myscripts

> Globs: `*.sh`, `**/*.sh`

# Shell Script Standards

Globs: `*.sh`, `**/*.sh`

## Bash Script Requirements

- Always start with `#!/bin/bash` shebang
- Use `set -e` for error handling or explicit error checks
- Quote all variables: `"$variable"` not `$variable`
- Use arrays for multiple values: `paths=(...)`
- Check directory existence: `[ ! -d "$DIR" ] && mkdir -p "$DIR"`
- Use absolute paths or derive from script location: `SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"`
- Include comments for non-obvious logic
- Make scripts executable: `chmod +x script.sh`

## Path Handling

- Prefer `$HOME` or `$USER` over hardcoded usernames
- Handle both Linux (`/home/`) and macOS (`/Users/`) paths
- Use `readlink -f` or `realpath` for resolving symlinks (Linux)
- Use `dirname` and `basename` for path manipulation

## Error Handling

- Check command success: `if ! command; then ... fi`
- Use `||` for fallback: `command || echo "Error message"`
- Validate inputs before processing
- Provide meaningful error messages

## Backup Scripts

- Include timestamps in filenames: `backup-$(date +%Y-%m-%d_%H:%M:%S).tar.gz`
- Verify destination exists before writing
- Set permissions after operations: `chown -R user:user "$dest"`
- Use compression for large backups: `tar -czf`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/akunito) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
