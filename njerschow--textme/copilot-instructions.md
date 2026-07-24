## textme

> When adding new commands to the daemon, **always update the `HELP_MESSAGE` constant** in `src/index.ts`.

# Claude iMessage Daemon - Development Notes

## Important Reminders

### Help Message Must Be Updated

When adding new commands to the daemon, **always update the `HELP_MESSAGE` constant** in `src/index.ts`.

The help message is displayed when users send `help` or `?`. It must accurately reflect all available commands.

Location: `src/index.ts` - search for `HELP_MESSAGE`

Current commands:
- `help` / `?` - Show help
- `status` - Show current status
- `interrupt` / `stop` / `cancel` - Stop current task
- `yes` / `no` (and variants) - Approval responses

### Adding a New Command Checklist

1. Add the command detection function (e.g., `isMyCommand()`)
2. Add the handler in the poll loop
3. **Update `HELP_MESSAGE` to include the new command**
4. Update this list in CLAUDE.md

### Project Notes System

The daemon includes an automatic project memory system for continuity across Claude sessions:

- **`PROJECT_NOTES.md`** (per-project): Claude reads this at session start and updates it after completing non-trivial work. Contains current state, recent session summaries (last 10), and known issues/TODOs. Created automatically by Claude on first meaningful work in a project.
- **`PROJECT_INDEX.md`** (repo root at `/Users/n/Documents/PassiveIncome/SendblueBase/textme/PROJECT_INDEX.md`): Central index of all active projects. Injected into every Claude context prompt for cross-project awareness. Claude updates the "Last worked on" date and "Recent notes" after each session.
- The system prompt in `src/claude-session.ts` instructs Claude on when/how to maintain these files.
- The `askClaude()` function in `src/index.ts` reads and injects `PROJECT_INDEX.md` into the context prompt.
- Do NOT manually edit `PROJECT_NOTES.md` files — they are auto-maintained by Claude.
- `PROJECT_INDEX.md` can be seeded/corrected manually if needed.

---
> Source: [njerschow/textme](https://github.com/njerschow/textme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
