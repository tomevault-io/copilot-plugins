## memory

> Memory management for important conversation points


# Conversation Memory

Track and save important points during conversations to `.cursor/memories/`.

## What counts as important
- User's vision and goals for the project
- Corrections the user makes to my mistakes (especially when they say "remember this")
- Solutions we brainstorm together
- Reasoning behind decisions we made

## Behavior
When I detect one of the above, I should:
1. Proactively offer to save it, OR
2. If it's clearly important (explicit correction, key decision), automatically append to today's memory file

## File format
Save to `.cursor/memories/YYYY-MM-DD.md` (one file per day, append entries).

Each entry format:
```
## HH:MM - [Category]

**Summary:** Brief description

**Details:** Context and reasoning if relevant

---
```

Categories: `Vision` | `Correction` | `Brainstorm` | `Decision`

## Guidelines
- Don't save trivial things or routine coding tasks
- When in doubt, ask before saving
- Keep summaries concise but capture the "why"

---
> Source: [tisu19021997/langclaw](https://github.com/tisu19021997/langclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
