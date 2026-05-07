## obsidian-hover-annotations

> - Symptom: Complex `node -e "..."` commands fail in PowerShell with parser errors before Node runs.

# Local Notes

## PowerShell Inline Node
- Symptom: Complex `node -e "..."` commands fail in PowerShell with parser errors before Node runs.
- Cause: PowerShell re-parses nested quotes, brackets, and backticks inside inline JavaScript.
- Fix: Prefer a here-string piped to Node, for example `@'...js...'@ | node`, instead of `node -e`.

---
> Source: [MaxMiksa/Obsidian-Hover-Annotations](https://github.com/MaxMiksa/Obsidian-Hover-Annotations) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
