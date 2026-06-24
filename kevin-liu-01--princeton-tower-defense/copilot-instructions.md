## use-ripgrep

> Use ripgrep (rg) instead of grep for all search operations


# Use ripgrep Instead of grep

NEVER use `grep` in shell commands. Always use `rg` (ripgrep) instead.

- `rg` is faster, respects `.gitignore` by default, and provides better output formatting.
- All `grep` flags have `rg` equivalents — prefer the native `rg` flags.

Common translations:

| grep | rg |
|------|-----|
| `grep -r "pattern" .` | `rg "pattern"` |
| `grep -rn "pattern"` | `rg -n "pattern"` |
| `grep -rl "pattern"` | `rg -l "pattern"` |
| `grep -i "pattern"` | `rg -i "pattern"` |
| `grep -E "regex"` | `rg "regex"` (ERE by default) |
| `grep --include="*.ts"` | `rg -t ts` or `rg -g "*.ts"` |

---
> Source: [Kevin-Liu-01/Princeton-Tower-Defense](https://github.com/Kevin-Liu-01/Princeton-Tower-Defense) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
