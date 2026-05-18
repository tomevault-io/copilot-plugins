## windows-commands

> Windows PowerShell command compatibility for local terminal usage

# Windows Command Compatibility Rules

This workspace runs on Windows (PowerShell). Use Windows-compatible command patterns when suggesting or running terminal commands.

## Use These

- `Select-String` instead of `grep`
- `Get-ChildItem` instead of `find`
- `Get-Content` instead of `cat`
- `Select-Object -First/-Last` instead of `head`/`tail`
- `Measure-Object` instead of `wc`
- `Get-Command` / `where.exe` instead of `which`

## Avoid These (Unix-only by default)

`grep`, `sed`, `awk`, `cut`, `head`, `tail`, `wc`, `find`, `which`, `sudo`, `chmod`, `touch`.

## Project-Specific Examples

```powershell
# Logs search
spacetime logs broth-bullets-local | Select-String -Pattern "cairn" -CaseSensitive:$false

# Rust source search
Get-ChildItem -Recurse -Filter "*.rs" | Select-String -Pattern "GrassAppearanceType"

# Count matching log lines
(spacetime logs broth-bullets-local | Select-String -Pattern "error" -CaseSensitive:$false).Count
```

## Notes

- Forward slashes in paths usually work, but prefer PowerShell-friendly quoting.
- Keep commands copy-paste friendly for PowerShell users.

---
> Source: [SeloSlav/2d-multiplayer-survival-mmorpg](https://github.com/SeloSlav/2d-multiplayer-survival-mmorpg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
