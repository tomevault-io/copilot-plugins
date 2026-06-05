## powershell-fix

> - Multi-line python -c commands

# CRITICAL POWERSHELL SAFETY RULES

## MEMORY: I will NEVER cause q^D^C issues in PowerShell again.

## NEVER USE THESE COMMANDS IN POWERSHELL ON WINDOWS:
- Multi-line python -c commands
- Complex string operations with quotes
- Commands that could cause parsing issues
- Any command that might result in q^D^C errors

## ALWAYS USE:
- Simple single-line commands
- File-based operations when possible
- Explicit output capture
- Safe command patterns

---
> Source: [wrm3/CBTrade](https://github.com/wrm3/CBTrade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
