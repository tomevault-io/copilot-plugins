## terminal-hygiene

> Minimize terminal spawning to prevent stale terminal accumulation


# Terminal Hygiene

Every Shell tool call spawns a new terminal that Cursor never auto-closes. Stale terminals accumulate fast.

## Rules

- **Prefer non-shell tools** — use Read, Grep, Glob, or SemanticSearch instead of `cat`, `grep`, `find`, `head`, `tail`, `ls` wherever possible. Only use Shell when you genuinely need shell execution.
- **Batch shell commands** — combine related commands with `&&` or `;` into a single Shell call instead of multiple separate calls.
- **Check running terminals first** — before starting a dev server or long-running process, read the terminals folder metadata (`head -n 10 *.txt` in the terminals folder) to avoid duplicating a process that's already running.
- **No exploratory shell commands** — don't use Shell just to "check" something you could read from a file directly.

## Examples

```
# BAD — three terminals spawned
Shell: ls backend/
Shell: cat backend/pyproject.toml
Shell: grep -r "import" backend/shared/

# GOOD — zero terminals spawned
Glob: backend/**
Read: backend/pyproject.toml
Grep: pattern="import" path=backend/shared/
```

---
> Source: [albusOS/sku-ops](https://github.com/albusOS/sku-ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
