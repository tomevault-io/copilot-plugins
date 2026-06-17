## context-audit-openclaw-skill

> **Monthly context file audit for OpenClaw agent workspaces**

# context-audit

**Monthly context file audit for OpenClaw agent workspaces**

## Description

Scans agent workspace context files (MEMORY.md, AGENTS.md, TOOLS.md, etc.) for token bloat, month-over-month growth, and cross-workspace duplication. Posts a report for human review — never edits files automatically.

## When to Use

- "audit context files"
- "check workspace bloat"
- "context health report"
- "scan for duplicate context"

## What It Does

1. Scans OpenClaw bootstrap context files at workspace root by default (`AGENTS.md`, `MEMORY.md`, etc.)
2. Estimates token count per file (`bytes / 4`)
3. Compares against configurable per-file thresholds
4. Detects month-over-month growth via stored baselines
5. Finds duplicate paragraphs across workspaces (MD5 of normalized text)
6. Counts files in `memory/` directories (flags accumulation)
7. Posts report to configured notification channel

## Scripts

### `scripts/audit-context.sh`

Main audit script.

**Usage:**
```bash
# Full audit with notification
./scripts/audit-context.sh

# Dry run — print report only
./scripts/audit-context.sh --dry-run

# Custom config
./scripts/audit-context.sh --config /path/to/config.sh

# Override workspaces via CLI
./scripts/audit-context.sh --workspace ~/agent1 --workspace ~/agent2
```

**Options:**
- `--dry-run` — Print report only, skip notification
- `--config PATH` — Config file path
- `--workspace DIR` — Workspace to scan (repeatable, overrides config)
- `--all-root-md` — Scan all root `*.md` files instead of bootstrap-only set
- `--notify-channel CH` — Notification channel (e.g., `telegram`)
- `--notify-target ID` — Notification target (e.g., chat ID)
- `--baseline-dir DIR` — Baseline storage directory

### `scripts/schedule-audit.sh`

Creates a monthly cron job via OpenClaw gateway API.

**Usage:**
```bash
# Default: 1st of each month, 10am PT
./scripts/schedule-audit.sh

# Custom schedule
./scripts/schedule-audit.sh --schedule "0 10 1 * *"

# Different timezone
./scripts/schedule-audit.sh --tz "Europe/London"
```

## Configuration

Copy `config.example.sh` to `~/.config/context-audit/config.sh` and customize:

- **WORKSPACES** — Array of workspace directories to scan
- **CONTEXT_FILES** — Exact root files to audit (bootstrap set by default)
- **ALL_ROOT_MD** — Set `true` to scan all root `*.md` files
- **THRESHOLDS** — Per-file token limits (warn:critical)
- **MEMORY_FILE_WARN** — Max files in memory/ before warning
- **DUP_MIN_WORDS** — Minimum paragraph size for duplicate detection
- **GROWTH_WARN_PERCENT** — Month-over-month growth threshold
- **NOTIFY_CHANNEL / NOTIFY_TARGET** — Where to send reports

## Dependencies

- bash, jq, fd
- openclaw CLI (optional, for notifications)

## Limitations

- Token estimation is approximate (byte count / 4, ~85-90% accurate for English)
- Duplicate detection uses paragraph-level granularity (blank-line separated blocks)
- First run has no baseline, so no growth deltas are reported

---
> Source: [kesslerio/context-audit-openclaw-skill](https://github.com/kesslerio/context-audit-openclaw-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
