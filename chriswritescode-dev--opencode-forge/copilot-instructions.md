## opencode-forge

> The forge plugin writes logs to:

# Agent Guidelines

## Log File Location

The forge plugin writes logs to:

```
~/.local/share/opencode/forge/logs/forge.log
```

This path can be overridden via the `logging.file` configuration option in `forge-config.jsonc`.

## Bundled Skills

The following skills are bundled with the plugin and auto-installed to `~/.config/opencode/skills/` on first run:

| Skill | Purpose |
|-------|---------|
| `tdd` | Test-driven development with red-green-refactor loop |

Source files are in `skills/` directory.

## Project Conventions

- All commits must be meaningful and follow conventional commit standards
- No emojis in commit messages
- Always check for existing patterns before adding new code
- Functions should be single responsibility and reusable
- Remove dead code to keep the codebase clean

---
> Source: [chriswritescode-dev/opencode-forge](https://github.com/chriswritescode-dev/opencode-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
