## ai-assist

> All new feature work and bug fixes MUST use git worktrees for isolation. This allows multiple Claude Code sessions to work on different features in parallel without conflicts.

## Feature Development Workflow

All new feature work and bug fixes MUST use git worktrees for isolation. This allows multiple Claude Code sessions to work on different features in parallel without conflicts.

### Starting Feature Work

When asked to work on a new feature, fix, or task:
1. Use the EnterWorktree tool with a descriptive name matching the feature (e.g., `add-dark-mode`, `fix-login-bug`)
2. Copy `.env` into the worktree: `cp /Users/onlinegrocery/Projects/ai-assist/.env .env`
3. Create a feature branch within the worktree
4. Complete all development within the worktree
5. Commit and push from the worktree

### Why Worktrees

- Each Claude Code session gets its own isolated working directory
- No merge conflicts between parallel sessions
- All worktrees share the same git history
- Clean separation of concerns per feature

### When NOT to Use Worktrees

- Quick one-line fixes or typo corrections
- Reading/exploring code without making changes
- If the user explicitly says to work in the current directory

### Cleanup

On completion, remind the user they can clean up with:
`git worktree remove .claude/worktrees/<name>`
Or list active worktrees with: `git worktree list`

## Visual Testing

After any code change that touches iOS UI, you MUST visually verify the changes:

1. **Start the server**: `./dev.sh` (auto-assigns a unique port from 8080-8089, clears DB, seeds test data). Read `.dev-port` to get the assigned port.
2. **Build and run the app**: Use `build_run_sim` with scheme `AIAssistApp` on an iPhone simulator
3. **Configure the app**: Open Settings (gear icon) in the app and set the port to match your `.dev-port` value
4. **Verify the UI**: Take screenshots and tap/swipe through affected screens to confirm the changes look correct
5. **Check for regressions**: Navigate to related screens to ensure nothing else broke visually

### Parallel Sessions

Multiple Claude Code sessions can run simultaneously. Each session gets its own server port and database automatically via `./dev.sh`. Port locks are stored in `data/ports/` and cleaned up on exit.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ith-harvey)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ith-harvey)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
