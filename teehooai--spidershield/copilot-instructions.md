## spidershield

> > Security scanner and runtime guard for MCP servers.

# AGENTS.md — SpiderShield

> Security scanner and runtime guard for MCP servers.
> Static analysis, policy enforcement, DLP, prompt injection detection.

## What This Project Does

SpiderShield is an open-source security linter for MCP (Model Context Protocol) servers and AI agent skills. Think "npm audit for MCP tools." It scans tool descriptions, security patterns, architecture quality, and licensing, then outputs a 0-10 SpiderScore.

## Quick Start

```bash
pip install spidershield

# Scan an MCP server from GitHub
spidershield scan https://github.com/owner/repo

# Check an AI agent config (Claude Code / OpenClaw)
spidershield agent-check /path/to/config

# Run as MCP server (for integration with Claude Desktop, etc.)
spidershield serve
```

## Project Structure

```
src/spidershield/
  cli.py              -- Click CLI entry point
  models.py            -- Pydantic V2 models (ScanReport, SecurityIssue, etc.)
  server.py            -- MCP server mode (scan_mcp_server tool)
  scanner/
    runner.py          -- 4-stage scan pipeline (descriptions → security → architecture → license)
    description_quality.py -- Tool description scoring (7 criteria)
    security_scan.py   -- 46+ static security patterns
    architecture_check.py -- Code quality checks
    license_check.py   -- License detection
  agent/
    scanner.py         -- Agent config security audit
    skill_scanner.py   -- Skill malware/injection detection (20+ patterns)
    toxic_flow.py      -- Dangerous capability combination detection
    pinning.py         -- SHA-256 content pinning for rug-pull detection
  rewriter/
    runner.py          -- Template + LLM description rewriter
  hardener/runner.py   -- Security fix suggestions
```

## Build & Test

```bash
# Install in dev mode
pip install -e ".[dev]"

# Run tests (use empty env vars to skip Supabase, use local SQLite)
SUPABASE_URL= SUPABASE_SERVICE_KEY= python -m pytest tests/ -x -q

# Run a specific test
python -m pytest tests/test_scanner.py -x -q
```

## Key Commands

| Command | What It Does |
|---------|-------------|
| `spidershield scan <repo>` | Full security scan of an MCP server |
| `spidershield agent-check <path>` | Audit agent config + installed skills |
| `spidershield rewrite <repo>` | AI-powered tool description improvement |
| `spidershield harden <repo>` | Generate security fix suggestions |
| `spidershield guard` | Runtime security guard mode |
| `spidershield serve` | Run as MCP server |

## Conventions

- Python 3.11+, type hints on all functions
- Pydantic V2 for data models
- Rich console for CLI output
- Click for CLI framework
- Tests in `tests/` directory
- Security issue codes: TS-E (errors), TS-W (warnings), TS-C (config), TS-P (pinning)

## SpiderRating API

For programmatic access to ecosystem-wide security ratings (5,899 servers rated):

```
GET https://api.spiderrating.com/v1/public/score/{owner}/{repo}
GET https://api.spiderrating.com/v1/public/check?tool={name}
```

Website: https://spiderrating.com

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/teehooai)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/teehooai)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
