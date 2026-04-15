## labkit

> Feature work happens in isolated git worktrees via the `wt` CLI:

# Agentic Workflow

## Worktrees

Feature work happens in isolated git worktrees via the `wt` CLI:

```bash
wt switch --create <branch> --yes   # create a new worktree
wt list                              # list all worktrees
wt remove <branch> --yes             # remove a worktree
```

Each worktree gets its own iTerm2 pane, named after the branch.

## Multi-agent work

When breaking a large task into parallel subtasks:
1. Spawn one pane per agent using `it2 session split --vertical`
2. Name each pane after its responsibility
3. Agents share the same codebase via git — coordinate through branches, not files
4. Use `it2 app broadcast on` to run the same shell command (e.g., `git pull`) in all panes at once
5. Tear down panes with `it2 session close -s <id>` when done

## Research before implementing

For non-trivial features, research the codebase and the web before writing code:
1. Grep for existing implementations of the concept
2. Read the project's key config files (package.json, tsconfig, etc.)
3. Check GitHub issues/PRs for prior art: `gh issue list`, `gh pr list`
4. Identify what already exists before proposing new dependencies

## Playwright for visual verification

Use `playwright-cli` to open the running dev server and verify UI changes visually:

```bash
playwright-cli open http://localhost:3000
playwright-cli screenshot
playwright-cli close
```

## Code quality

- Prefer editing existing files over creating new ones
- Avoid over-engineering — implement the minimum needed for the current task
- Don't add error handling for scenarios that can't happen in normal usage
- Avoid backwards-compat hacks for removed code

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dallen4) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
