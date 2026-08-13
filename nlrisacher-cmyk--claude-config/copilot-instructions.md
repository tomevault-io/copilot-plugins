## claude-config

> At the start of processing each new user message, check if `.claude-vault-inbox.md` exists in the current working directory. If it exists and has content:

# Global Claude Instructions

## ClaudeVault Inbox

At the start of processing each new user message, check if `.claude-vault-inbox.md` exists in the current working directory. If it exists and has content:

1. Read it
2. Treat each entry as a pending message sent by the user via the ClaudeVault dashboard
3. Address the message(s) as part of your current response or as a priority task
4. After reading, delete the file: use the Bash tool to run `rm -f .claude-vault-inbox.md` (or the appropriate command for the platform)

The file format is Markdown. Each message is preceded by an HTML comment `<!-- vault:ID -->` and a bold timestamp header. Treat the content after the header as the message text.

Do not mention the inbox file to the user unless they ask about it — just act on the messages naturally.

## Bigbrain Auto-Load

At the start of every conversation, silently read the following files to load full context. Do not mention loading them unless the user asks — just have the context ready.

1. `~/.claude/projects/-home-nathan/memory/bigbrain.md` — infrastructure index
2. `~/.claude/projects/-home-nathan/memory/projects.md` — all active projects, status, and stack summaries

For detailed project specs, read the linked file under `projects/` when working on that project.

---
> Source: [nlrisacher-cmyk/claude-config](https://github.com/nlrisacher-cmyk/claude-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
