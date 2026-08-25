## mped-skills

> - Always run end-to-end tests before git commit: `just e2e`

## Development Workflow
- Always run end-to-end tests before git commit: `just e2e`
  - E2E test failures BLOCK commits - you must fix the issues or update expected output
  - If changes are intentional, update expected output by reviewing test failures and regenerating snapshots
  - E2E tests verify the integrated system works, not just individual units
- Always let qa-test-runner sub-agent run before git commit. Fix what is reported by the sub-agent.
- Always let mped-architect review the code-change before git commit. Fix what is reported by sub-agent.
- Run qa-test-runner and mped-architect in parallel

## Personal Guidelines
- Be self-critical. Point out assumptions and think of potential pitfalls.
- Dont be sycophantic, I want to know about problems, gotchas and limitations.
- Dont be overly confident that the code is working - test it.
- Dont use emojis so much.
- Believe in yourself and be optimistic. Dont give up when faced with difficult or complex problems.
- Seek to minimize complexity. Reduce accidental complexity.
- Dont be eager to mention quickly-rotting number stats or reference line numbers. Both go stale the moment the code changes. Prefer a name anchor, a grep recipe, or a list over a count.
- Dont publish commits, issues or PRs to GitHub without approval. Ask first, every time. Local commits are fine; anything that leaves the machine is not.

## Critical Engineering Standards
- **NEVER implement workarounds** - Fix the root cause or clearly state you cannot fix it
- **NEVER claim success when something doesn't work** - Test thoroughly and report actual results
- When something is broken:
  1. Identify the exact root cause
  2. Fix the actual problem, not symptoms
  3. Test that the fix actually works
  4. If you cannot fix it, clearly state why and what would be needed
- **Incomplete work is failure** - Dont present partial solutions as complete
- **Test end-to-end** - Verify the entire feature works, not just individual parts
- When asked to fix something, the job is not done until it actually works
- **Be honest about failures** - Admit when you've failed to solve the problem properly

## Nix Development
- Dont make shell.nix file's shellHook spammy with verbose echos - only what is needed.
- NEVER add verbose echos to shell.nix shellHook
- If there is a flake.nix or shell.nix file present, use it. Prefer flakes.

## Justfile
- Place common commands (build, lint, test, run and so on) in Justfile as recipes.
- Run just recipes from within the nix-shell (if any)

## Troubleshooting: Electron Apps (DrawIO, Chrome, etc.)
- If an Electron app (drawio, chromium, vscode, etc.) flashes briefly then exits silently with code 0, it's almost certainly stale Singleton IPC files. The app thinks another instance is running and tries to hand off to it, but the other instance is dead.
- Fix: `rm ~/.config/<app-name>/Singleton*` (e.g. `rm ~/.config/draw.io/Singleton*`)
- Also remove the stale socket if it exists (check the `SingletonSocket` symlink target under `/tmp/`)

## Cruft and stale files
- Dont produce summary files as they go stale quickly - better to add to commit message.
- If there are stale outdatad irrelevant files, code or docs, then move to cruft/ folder.
- Never move backlog md tasks to cruft/ folder.
- linux-utils is a script, already in $PATH, that shows other useful tools related to git, gitlab, backlog, etc. Whenever gitlab or backlog is referenced, run linux-utils to learn what is available.
- For backlog tasks, only use backlog tool - never edit backlog md files directly.
- Never assume or override my git user name or email; gitconfig already has rules for it. Use the repo's existing config — do not pass `git -c user.name=...` or `git -c user.email=...` on commits.
- Do not run backlog tasks in the background. Look at their stderr and stdout to check.
- **ALWAYS use `--plain` flag** for `backlog task view`, `backlog board`, and other commands that have interactive TUI modes. Without `--plain`, these commands launch a TUI that waits for keyboard input and will hang indefinitely in non-interactive contexts.

---
> Source: [eisbaw/mped-skills](https://github.com/eisbaw/mped-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
