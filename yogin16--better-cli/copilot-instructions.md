## better-cli

> When building or improving CLI tools, follow these practices:


# Better CLI Rules

When building or improving CLI tools, follow these practices:

- stdout = data only. stderr = errors, warnings, progress. Never mix.
- Support `--json` with consistent envelope: `{"status": "ok|error", "data": {...}}`
- Output must guide the next action — suggest follow-up commands
- Semantic exit codes: 0=success, 2=usage, 3=not-found, 4=permission, 75=transient
- Every error: what + context + fix command + docs link
- Every prompt needs a flag bypass (`--yes`, `--force`, `--name=value`)
- No ANSI when piped; respect `NO_COLOR`
- Flags over positional args
- Config: flags > env vars > config files > defaults
- Never accept secrets via flags
- Support `--fields`, `--quiet`, `--dry-run`
- Create commands output the resource identifier

Full guide: see SKILL.md

---
> Source: [yogin16/better-cli](https://github.com/yogin16/better-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
