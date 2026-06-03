## powershell-commit-gotcha

> Small Windows PowerShell commit-message gotcha


# PowerShell Commit Gotcha

- This repo is usually worked in from Windows PowerShell. Shell commands that include bash-style heredoc syntax like `<<'EOF'` can be parsed by PowerShell before bash receives them.
- For `git commit` message construction in PowerShell, prefer a PowerShell here-string or another PowerShell-safe approach instead of an inline bash heredoc.

---
> Source: [nopara73/LongevityWorldCup](https://github.com/nopara73/LongevityWorldCup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
