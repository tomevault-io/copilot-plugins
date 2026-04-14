## system

> Logging and auto-commit workflow for responses


## Logging and Auto-Commit Workflow

### Where to look for logs:
- **project_structure_final.txt** — logs of last program run (first 100 lines)
- **~/.system_cli/logs/cli.log** — full logs of all runs
- **~/.system_cli/logs/errors.log** — errors only
- **~/.system_cli/logs/debug.log** — detailed debug logs

### Log format:
```
2025-12-17 07:49:22 | INFO | system_cli.cli | cli_main:4800 | CLI started with arguments
```

### Auto-commit at end of response:
After completing work, run:
```bash
./auto_commit.sh "Summary of what was done"
```

This will:
1. Save response to `.last_response.txt`
2. Create a git commit
3. Post-commit hook regenerates `project_structure_final.txt` with latest logs
4. Amend commit with updated structure

### Quick commands:
```bash
# View last 100 lines of logs in real-time
tail -f ~/.system_cli/logs/cli.log

# View only errors
cat ~/.system_cli/logs/errors.log

# View detailed debug logs
cat ~/.system_cli/logs/debug.log

# Save response and create commit
./auto_commit.sh "Your response summary"
```

### When to use auto-commit:
- At the end of each important response
- After completing a feature or fix
- When you want to save progress with logs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/olegkizyma008-rgb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
