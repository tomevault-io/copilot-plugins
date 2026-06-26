## agentgem

> Use **git worktrees** to isolate concurrent sessions. When more than one agent or

# CLAUDE.md

## Concurrent sessions

Use **git worktrees** to isolate concurrent sessions. When more than one agent or
session may be working in this repo at the same time, create a dedicated worktree
per session instead of sharing the main checkout:

```bash
git worktree add ../agentgem-<task> -b <task>
```

This keeps each session's branch, working tree, and build artifacts (`dist/`,
`tsconfig.tsbuildinfo`) separate, avoiding cross-session interference. Remove the
worktree when the work is merged or abandoned:

```bash
git worktree remove ../agentgem-<task>
```

---
> Source: [ninemindai/agentgem](https://github.com/ninemindai/agentgem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
