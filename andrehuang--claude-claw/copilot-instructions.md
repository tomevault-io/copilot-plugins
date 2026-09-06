## claude-claw

> These rules activate when relevant and are invisible otherwise. They improve Claude Code's behavior for multi-agent work, long sessions, and cross-session continuity.

# Claude-Claw: Global Behavior Rules

These rules activate when relevant and are invisible otherwise. They improve Claude Code's behavior for multi-agent work, long sessions, and cross-session continuity.

## Multi-Agent Git Safety

When multiple agents (subagents, background agents, worktree agents) are working on the same repository:

- Do not create, apply, or drop git stash entries unless explicitly requested by the user. Stashes are global state and affect all agents.
- When committing, scope to your changes only. Stage specific file paths -- never use `git add -A` or `git add .` in multi-agent contexts.
- Do not switch branches or create/modify git worktrees unless explicitly asked. Branch state is shared.
- When you encounter unrecognized files or changes from other agents, leave them alone and focus on your assigned work.
- Prefer grouped commit-pull-push cycles over many tiny syncs. Frequent pushes create merge conflicts for concurrent agents.
- Before editing a file, check if it was recently modified by another agent (via git log or git diff). If so, coordinate or wait.

## Context Preservation

- When conversations get long, proactively save critical state to memory before information is lost: current task goals, key decisions made, file paths being edited, unresolved questions, and next steps.
- Before context compression: preserve exact identifiers (UUIDs, hashes, commit SHAs, file paths), active task state, and pending TODOs. These are the details most likely to be lost in summarization.
- After receiving compressed context: re-read critical files from disk rather than relying on compressed summaries. The files are the source of truth, not the summary.

## Pre-Compaction Memory Flush

- When you notice the conversation is approaching context limits (many long tool results, extensive back-and-forth, system reminders about compression), proactively write a structured summary to memory.
- Include: task objective, current progress, key decisions and their rationale, blockers, exact file paths and line numbers being worked on, and what to do next.
- Write to today's daily log for ephemeral state, and to project memory for decisions with long-term value.

## Daily Logs

- At the start of each session, check if a daily log exists for today at the project's `memory/daily/YYYY-MM-DD.md`. If it exists, read it for context on what happened earlier today.
- Also read yesterday's daily log if it exists, for continuity across days.
- During complex work, append notable events to today's daily log: decisions made, files modified, problems encountered, things to remember for next session.
- Daily logs are ephemeral -- they provide day-to-day continuity but are NOT long-term memory. Important insights should be promoted to proper memory files.
- Keep daily log entries concise: one line per event, with a timestamp or rough time-of-day marker.
- Format: `- HH:MM -- Brief description of what happened`

## Cross-Project Memory

- When the user asks about something that may span multiple projects (e.g., "what did I work on this week", "have I seen this pattern before", "what's my current workload"), search memory across projects by reading MEMORY.md index files from `~/.claude/projects/*/memory/MEMORY.md`.
- When writing global memory (user routines, preferences, cross-project knowledge), always write to the global memory directory.
- When writing task-specific or project-specific memory, write to the current project's memory directory.
- Never duplicate the same memory across tiers -- reference by file path instead.

## Memory Tier Awareness

Three memory tiers exist, each with a distinct purpose:

| Tier | What goes here | What does NOT go here |
|------|---------------|----------------------|
| **Global** | User identity, learned routines, preferences, cross-project knowledge | Project-specific task state |
| **Project-local** | Task state, handoffs, evaluations, architecture decisions | User preferences (those are global) |
| **Daily log** | Today's events, decisions, files touched | Long-term knowledge (promote to proper memory) |

When saving memory, always ask: "Will this matter in a different project?" If yes, it's global. "Will this matter next month?" If yes, it's project-local memory (not just a daily log entry).

---
> Source: [andrehuang/claude-claw](https://github.com/andrehuang/claude-claw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
