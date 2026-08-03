## job-harness

> job-harness is a Job Search OS packaged as a Codex and Claude Code plugin. Cursor can use this repository-level `AGENTS.md` for maintenance work, but the installable runtime is the plugin under `plugins/job-harness`.

# AGENTS.md — Agent Instructions

## Project Overview

job-harness is a Job Search OS packaged as a Codex and Claude Code plugin. Cursor can use this repository-level `AGENTS.md` for maintenance work, but the installable runtime is the plugin under `plugins/job-harness`.

It is NOT a tool for humans to use directly — it is the runtime environment in which an AI agent searches for jobs on behalf of a human.

The agent receives a natural-language job search request, runs the v2 search CLI, analyzes results, and presents findings.

## Philosophy

Search broadly across all available sources: job aggregators and employer career pages. Aggregators are useful, but they create a search bubble: some companies and vacancies appear only on employer sites. The v2 engine searches the full implemented catalog unless the user narrows sources explicitly.

The plugin is in active early development. Do not add compatibility shims,
legacy fallbacks, backward-compatible adapters, or code paths whose purpose is
preserving old behavior. Change the contract directly and update all callers,
fixtures, and tests in the same patch. `scripts/check_no_compat_comments.py`
rejects source comments that justify compatibility work.

## Architecture

```
plugins/job-harness/
├── .codex-plugin/plugin.json   # Codex plugin manifest
├── .claude-plugin/plugin.json  # Claude Code plugin manifest
├── .mcp.json                   # MCP server config (v1 legacy)
├── commands/                   # Claude Code slash command entrypoints
├── agents/                     # Claude Code agent entrypoints
├── skills/                     # Runtime skills and canonical workflows
├── scripts/                    # MCP server (v1) and artifact helpers
└── src/job_harness/
    ├── v2/                     # Primary contract-first search engine
    │   ├── application.py      # V2SearchApplication
    │   ├── cli.py              # job-harness-v2 entrypoint
    │   ├── contracts/          # SearchRequest, SourceScraper, records
    │   ├── postprocessing/     # Filter, dedupe, criteria plan
    │   ├── runtime/            # Orchestrator, HTTP fetcher, sources
    │   └── source_catalog.sql  # Source metadata and criteria declarations
    └── v1/                     # Legacy engine (MCP, browser, employer tools)
        ├── cli.py              # job-harness entrypoint
        ├── search_engine.py
        ├── scrapers/
        └── ...
```

## Plugin Components

This repo has one real plugin root: `plugins/job-harness`. Do not duplicate plugin runtime files at the repository root.

Development-only skills may live under `.agents/skills`. They are repository
maintenance guidance and must not be treated as plugin runtime skills.
When maintaining scraper code, source contracts, parser fixtures, or scraper
tests, read `.agents/skills/job-harness-scraper-development/SKILL.md`.

The plugin includes:

- **Commands**: `/job-search` (delegates to v2 workflow skill)
- **Runtime skills**: `job-search-workflow` (v2 CLI), `user-briefing`
- **Development skills**: `.agents/skills/job-harness-scraper-development`
- **Agent**: `job-searcher` — Claude Code entrypoint for the full automated workflow
- **v2 CLI**: `job-harness-v2 list-sources`, `job-harness-v2 search`
- **v1 MCP tools (legacy)**: `search_start`, `search_status`, `search_results`, … via `scripts/mcp-server.py`

`plugins/job-harness/skills/job-search-workflow/SKILL.md` is the canonical
search workflow. It owns v2 CLI usage, current search parameters, source catalog
inspection, append mode, artifact handling, and presentation rules. It uses
**`job-harness-v2` CLI**, not v1 MCP tools.
`plugins/job-harness/agents/job-searcher.md` and
`plugins/job-harness/commands/job-search.md` must remain thin entrypoints and
must not duplicate the skill workflow text.

**v2 search features to preserve:**

- Full catalog when `--source` is omitted; exact ids when repeated `--source` is used.
- Per-source raw limits from `source_catalog.sql`.
- Criteria capabilities per source (`native_request`, `structured_output`, `unsupported`).
- Append mode with shared run corpus and advancing `append_sequence`.

Legacy v1 CLI/MCP remains under `job_harness.v1` for maintenance.

## Verification

Run from the repo root:

`python scripts/verify_v2.py` — v2 lint, types, unit tests, and full-catalog live e2e. Use `python scripts/verify_v2.py --live-profile light` for the bounded two-source live e2e profile.

`python scripts/verify_repo.py full` — repository-wide gate including v1 tests

---
> Source: [feodal01/job-harness](https://github.com/feodal01/job-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
