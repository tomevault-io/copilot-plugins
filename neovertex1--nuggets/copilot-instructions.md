## nuggets

> This project uses `nuggets` for persistent holographic memory. The `nuggets` CLI is installed and available.

# Nuggets — Holographic Memory

This project uses `nuggets` for persistent holographic memory. The `nuggets` CLI is installed and available.

## When to use Nuggets

**Before searching for files or code patterns**, check memory first:
```bash
nuggets recall "what you're looking for"
```
If found, use the answer instead of running Grep/Glob/Read.

**After discovering something useful**, cache it:
```bash
nuggets remember <nugget> "<key>" "<value>"
```

## What to remember

- File locations: `nuggets remember locations "auth handler" "src/auth/middleware.ts:47"`
- Commands: `nuggets remember project "test cmd" "pytest tests/ -v"`
- Patterns: `nuggets remember project "error handling" "uses Result type, never throws"`
- User preferences: `nuggets remember prefs "style" "2-space indent, no semicolons"`
- Bug fixes: `nuggets remember debug "CORS error" "add origin to allowlist in config.ts"`

## What NOT to remember

- Anything longer than a sentence (values should be short)
- Temporary context that won't matter next session
- Information already in CLAUDE.md (don't duplicate)

## Commands

```bash
nuggets remember <nugget> <key> <value>   # Store (auto-creates nugget)
nuggets recall <query>                     # Search all nuggets
nuggets recall <query> --nugget <name>     # Search specific nugget
nuggets forget <nugget> <key>              # Remove a fact
nuggets list                               # Show all nuggets
nuggets facts <nugget>                     # Show facts in a nugget
```

---
> Source: [NeoVertex1/nuggets](https://github.com/NeoVertex1/nuggets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
