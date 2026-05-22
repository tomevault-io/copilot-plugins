## explainanythingbackup

> - Always prioritize being concise even at the expense of grammar

# My defaults
- Always prioritize being concise even at the expense of grammar
- Always prefer to re-use existing code rather than writing new code
- After every code block you write, always run lint, tsc, and build and write corresponding unit tests and run them before writing the next code block
- Prefer TypeScript, strict mode
- For debugging, also use playwright mcp server in headless mode unless asked
- Always start by reading /docs/docs_overall/getting_started.md
- Always run frontend and backend in tmux so that can easily tail logs for debugging when needed
- Include comments at the top of every file explaining in 1-2 sentences what the file does.

## Debugging with tmux Dev Servers

Dev servers are automatically started by SessionStart hooks. Access logs via:

```bash
# 1. Find your instance
cat /tmp/claude-instance-*.json | jq -r '.instance_id'
# Output: abc123

# 2. View server logs (last 100 lines)
tmux capture-pane -t claude-abc123-backend -p -S -100

# 3. View frontend logs
tmux capture-pane -t claude-abc123-frontend -p -S -100

# 4. Or use log files directly
tail -100 server-abc123.log
tail -100 client-abc123.log
```

For full debugging workflow, see: `docs/planning/tmux_usage/using_tmux_recommendations.md`

## Server Management Rules

**NEVER manually start dev servers.** The project uses on-demand tmux server management.

### What NOT to do:
- `npm run dev` or `npm run dev:server` directly
- `next dev` or any direct Next.js server commands
- Starting servers in background with `&`
- Any manual server startup

### What TO do instead:
- Run `npm run test:e2e` - servers start automatically via `ensure-server.sh`
- Use `./docs/planning/tmux_usage/ensure-server.sh` if you need a server outside tests
- Check `/tmp/claude-instance-*.json` for running server URLs
- View logs via `tmux capture-pane -t claude-<id>-backend -p -S -100`

Servers are managed by the tmux infrastructure and will auto-shutdown after 5 minutes idle.
See: `docs/planning/tmux_usage/using_tmux_recommendations.md`

## Project Planning Rules

### Workflow Enforcement

Before editing code files, you must:
1. Read `/docs/docs_overall/getting_started.md`
2. Read `/docs/docs_overall/project_workflow.md`
3. Create todos using TodoWrite

These prerequisites are tracked automatically. Code edits will be blocked until completed.

### Active Project Detection

Your active project is determined by the current git branch. If the branch is `fix_bug_20260108`, your project folder is `/docs/planning/fix_bug_20260108/`.

### Planning Files

1. **Never use internal plan files** - Do not write to `.claude/plans/`. All planning content goes in the project's `_planning.md` file.

2. **Project folder structure:**
   - `_status.json` - Workflow state (auto-managed)
   - `_research.md` - Research findings
   - `_planning.md` - Brainstorm + implementation plan
   - `_progress.md` - Execution tracking

### Bypassing Enforcement

For quick fixes or emergencies:

1. **Branch prefix bypass** - Use `hotfix/`, `fix/`, `docs/`, or `chore/` prefix
2. **Environment variable** - `WORKFLOW_BYPASS=true claude`

Main/master branches and legacy projects (without `_status.json`) are automatically exempt.

---
> Source: [alcmdbackup/explainanythingbackup](https://github.com/alcmdbackup/explainanythingbackup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
