## claude-enhanced-insights

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Enhanced reimplementation of Claude Code's `/insights` command with raised limits. Three scripts form a multi-machine insights pipeline: sync sessions from remote machines, diagnose available data, and generate comprehensive HTML reports by analyzing session transcripts via the Claude API.

## Architecture

**Data flow:** Remote machines → `insights_sync.sh` (rsync) → `~/.claude/projects/*.jsonl` → `enhanced_insights.py` (analyze) → HTML/JSON reports in `~/`

### enhanced_insights.py (core engine)
- OAuth token management: reads `~/.claude/.credentials.json`, auto-refreshes expired tokens
- Session discovery: scans `~/.claude/projects/` for UUID-named and agent-* JSONL files
- Metrics extraction: parses tool usage, languages, tokens, errors, git activity from session messages
- Facet extraction: sends transcripts to Claude API for structured analysis, caches results in `~/.claude/usage-data/facets/`
- Report generation: 8 parallel Claude API calls for different report sections, then renders HTML with embedded CSS/charts
- Configuration knobs are at the top of the file (lines 36-62) — token limits, batch sizes, parallelism

### insights_sync.sh
- Rsyncs `.jsonl` session files from configured remote hosts to local `~/.claude/projects/`
- Remote hosts configured in `REMOTES` array at top of file — edit these for your environment
- Uses `--ignore-existing` to avoid re-transferring already-synced sessions
- Requires SSH key-based auth (uses `BatchMode=yes`) and `rsync` on both ends
- Skips unreachable hosts gracefully (5-second connect timeout)
- Supports `--dry-run` to preview what would be synced without transferring
- Warns if >50 new sessions are synced (since the original `/insights` has a per-run cap)

### insights_diagnostic.sh
- Read-only audit of `~/.claude/projects/` and the facet cache — no side effects
- Cross-platform: handles GNU coreutils (Linux) and BSD coreutils (macOS) via portable `stat`/`find` helpers
- Reports per-project: total session files, agent sub-sessions, regular sessions, tiny files (<1KB likely excluded), date ranges
- Shows facet cache status: cached count, size on disk, uncached sessions needing API processing
- Checks for existing HTML report and parses its metadata (date, size, subtitle)
- Explains the path-hashing scheme: same absolute project path on different machines produces the same hash, so synced sessions merge correctly

## Running

```bash
# Full analysis run
python3 enhanced_insights.py

# Preview what would be processed (no API calls)
python3 enhanced_insights.py --dry-run

# Filter by project or date
python3 enhanced_insights.py --project "*claude*" --after 2026-02-01

# Override model
python3 enhanced_insights.py --model claude-sonnet-4-20250514

# Sync sessions from remote machines first
bash insights_sync.sh
bash insights_sync.sh --dry-run

# Diagnose available session data
bash insights_diagnostic.sh
```

## Dependencies

Python: `anthropic`, `httpx` (plus stdlib). No requirements.txt exists — install manually with `pip install anthropic httpx`.

## Security Notes

- Output files (HTML report, JSON dump, facet cache) are written with `0o600` permissions
- The OAuth CLIENT_ID is the standard public Claude Code client ID — not a secret
- Remote hosts in `insights_sync.sh` should be edited for your environment
- User prompts (first 200 chars) are persisted in facet cache and reports — inherent to the tool's purpose

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dmaynor)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dmaynor)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
