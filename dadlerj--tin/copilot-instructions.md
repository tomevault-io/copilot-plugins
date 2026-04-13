## tin

> Thread-based version control for conversational coding. Wraps git, treating conversation threads as the primary unit of change.

# Tin

Thread-based version control for conversational coding. Wraps git, treating conversation threads as the primary unit of change.

## Important: Use Tin Commands

**Always use built-in tin commands instead of directly modifying `.tin/` storage on disk.** For example:
- Use `tin thread delete` instead of `rm .tin/threads/<id>.json`
- Use `tin add --unstage` instead of editing `.tin/index.json`

Direct file manipulation can leave the repository in an inconsistent state (e.g., orphaned staging entries, missing index updates).

## Quick Reference

```bash
tin init                  # Initialize (also runs git init if needed)
tin status                # Show current branch and staged threads
tin branch [name]         # List or create branches
tin checkout <ref>        # Switch branch/commit (restores git state)
tin add <thread-id>       # Stage a thread (or --all)
tin commit -m "msg"       # Commit staged threads
tin log                   # Show commit history
tin thread list           # List all threads
tin thread show <id>      # Display thread conversation
tin thread delete <id>    # Delete a thread (use -f for committed/active)
tin hooks install         # Install Claude Code hooks (--global for ~/.claude)
tin hooks uninstall       # Remove hooks
tin amp pull [id|count]   # Pull threads from Amp CLI
```

## Project Structure

```
cmd/tin/main.go           # Entry point
internal/
  commands/               # CLI command implementations
  model/                  # Message, Thread, TinCommit, Branch structs
  storage/                # .tin directory operations
  hooks/                  # Claude Code hook handlers + installation
```

## Data Model

- **Message**: hash (merkle chain), role, content, timestamp, tool_calls, git_hash_after
- **Thread**: id (first message hash), agent, messages[], parent refs for branching
- **TinCommit**: id, message, thread refs, git_commit_hash

Storage in `.tin/`: threads/, commits/, refs/heads/, index.json (staging), HEAD

## Agent Integrations

### Claude Code

Hooks auto-track conversations:
- `SessionStart` → new thread
- `UserPromptSubmit` → append human message, auto-stage
- `Stop` → append assistant response + tool calls + git hash
- `SessionEnd` → mark thread complete

Slash commands: `/branches`, `/commit [msg]`, `/checkout [branch]`

### Amp

Pull threads from Amp CLI:
```bash
tin amp pull                    # Pull latest thread
tin amp pull 5                  # Pull 5 most recent threads
tin amp pull T-019b7d09-...     # Pull specific thread by ID
tin amp pull https://ampcode.com/threads/T-...  # Pull by URL
```

Threads are deduplicated by Amp thread ID (stored in `AgentSessionID`).

## Building

```bash
go build -o tin ./cmd/tin
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dadlerj)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dadlerj)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
