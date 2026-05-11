## adodevmetrics

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Setup and Running

```bash
# Install dependencies
pip install -r requirements.txt

# Configure (copy template and fill in values)
cp .env.example .env

# Run
python main.py
```

Output: an Excel file named `{repository_name}_{YYYYMMDD_HHMMSS}.xlsx` in the working directory.

## Required Environment Variables

| Variable | Description |
|---|---|
| `ADO_ORGANIZATION_URL` | `https://dev.azure.com/your-org` |
| `ADO_PROJECT` | Azure DevOps project name |
| `ADO_REPOSITORY` | Repository name |
| `ADO_BRANCH` | Branch to collect commits from |
| `ADO_PAT` | Personal Access Token (scopes: Code-Read, Pull Request Threads-Read) |
| `MONTHS_BACK` | Number of months to look back |

Optional: `REQUESTS_CA_BUNDLE` for corporate SSL certificates.

## Architecture

Flat single-package structure — all modules at root level. Execution flows linearly through collectors into an exporter:

```
main.py → config.py → ado_client.py → commit_collector.py → pr_collector.py → excel_exporter.py
                                            ↕
                                       commit_cache.py (SQLite)
```

### Module Responsibilities

- **[main.py](main.py)** — Orchestrates the full pipeline: load config → create API client → collect commits → collect PR stats → export Excel.
- **[config.py](config.py)** — Loads `.env` via `python-dotenv`, validates required vars, returns a `Config` dataclass.
- **[ado_client.py](ado_client.py)** — Creates authenticated `GitClient` from the `azure-devops` SDK. Provides a `@with_retry()` decorator (exponential backoff, up to 5 attempts) for HTTP 429 and 5xx errors.
- **[commit_collector.py](commit_collector.py)** — Paginates commits (100/page), filters merge commits and author≠committer cases, computes line-level diffs per commit, uses SQLite cache to skip already-processed commits.
- **[commit_cache.py](commit_cache.py)** — SQLite manager (`commit_cache.db`). Stores `(repository, commit_id, lines_added, lines_deleted)`. Uses WAL journal mode. Commits are treated as immutable — no cache invalidation.
- **[pr_collector.py](pr_collector.py)** — Paginates PRs (100/page), tracks per-author counts: PRs created, PRs approved (vote ≥ 10), PRs commented.
- **[excel_exporter.py](excel_exporter.py)** — Generates a two-sheet workbook: "Commits" (per-commit rows) and "Summary" (aggregated per-author stats). Auto-fits columns, freezes headers.

### Key Design Decisions

**Commit filtering rules** (commit_collector.py):
- Merge commits detected by message prefix: `Merged PR`, `Merge remote-tracking branch`, `Merge branch`, `Merge pull request`.
- Author/committer must match — compared by email first, then name. This excludes cherry-picks and rebases.

**Change parsing** (commit_collector.py ~lines 62–87):
- Azure DevOps Python SDK returns `changes` as `[object]` (typed as raw dicts with camelCase keys), not `GitChange[]` as the REST docs suggest. `_get_field()` handles both SDK model objects and raw dicts for compatibility.

**Binary file exclusion** (commit_collector.py ~lines 18–26):
- 40+ binary extensions are skipped when counting line changes (`.png`, `.pdf`, `.exe`, `.xlsx`, etc.).

**SQLite cache** — persists between runs in `commit_cache.db`. On repeated runs against the same repository, already-seen commits are loaded instantly without re-fetching diffs from the API.

---
> Source: [dbobylev/adodevmetrics](https://github.com/dbobylev/adodevmetrics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
