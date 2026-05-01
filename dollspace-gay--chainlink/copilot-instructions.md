## chainlink

> Track tasks across AI sessions. Data in `.chainlink/issues.db`.

# Chainlink Issue Tracker

Track tasks across AI sessions. Data in `.chainlink/issues.db`.

## Commands

```bash
# Issues
chainlink create "title" [-p high] [-d "desc"]
chainlink list [-s all|closed] [-l label] [-p priority]
chainlink show|update|close|reopen|delete <id>
chainlink subissue <parent> "title"

# Organization
chainlink comment <id> "text"
chainlink label|unlabel <id> <label>
chainlink block|unblock <id> <blocker>
chainlink blocked|ready

# Sessions
chainlink session start|end|status|work <id>
chainlink session end --notes "handoff context"
```

## Workflow

1. `session start` → see previous handoff
2. `session work <id>` → mark focus
3. Work, add comments
4. `session end --notes "..."` → save context

## Best Practices

- Start sessions when beginning work
- Use `ready` to find unblocked issues
- Use subissues for tasks >500 lines
- End with handoff notes before context compresses

---

*Language rules, security requirements, and testing guidelines are in `.chainlink/rules/` and auto-injected based on detected project languages.*

---
> Source: [dollspace-gay/chainlink](https://github.com/dollspace-gay/chainlink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
