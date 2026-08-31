## data-analyst

> This document defines development guidelines for the data‑analyst framework.

# AGENTS.md

This document defines development guidelines for the data‑analyst framework.

## Principles
- Keep data operations deterministic and auditable.
- Treat metadata as first‑class output, not a side effect.
- Prefer composable, testable modules over monolithic flows.
- Isolate projects by default (data, scripts, secrets, environments).
- Keep the agent layer thin and optional.

## Python tooling (uv)
We use `uv` for all Python development workflows. This is mandatory for installs, running scripts, and tests.
Do not use `pip`, `python -m pip`, or `python` directly for project tasks; always use `uv`.

### Dependency management
- Add dependencies: `uv add <package>`
- Add dev dependencies: `uv add --dev <package>`
- Sync environment: `uv sync`

### Running code
- Run modules or scripts: `uv run python path/to/script.py`
- Run app locally: `uv run uvicorn app.main:app --reload` (from apps/api)

## Beads (progress tracking)
We use Beads (bd) for issues, progress tracking, and run logs. All work must be represented by Beads issues and updated as it progresses.

### Setup (protected branch workflow)
- Sync branch: `beads-sync` (configured via `bd migrate sync beads-sync`).
- Only these files are committed on main:
  - `.beads/.gitignore`
  - `.gitattributes`
- Beads issue data is stored on the `beads-sync` branch (`.beads/issues.jsonl`, `.beads/metadata.json`, optional `.beads/config.yaml`).
- Local runtime files (SQLite db, daemon files) are not committed.

### Daily workflow
- Create issues for any planned work:
  - `bd create "Implement CSV ingestion"`
  - `bd q "Short task"` (quick capture)
- Start work by updating status:
  - `bd update <issue-id> --status in_progress`
- Log decisions, results, and key outputs as comments:
  - `bd comments add <issue-id> "Summary of result + links"`
- Mark complete:
  - `bd update <issue-id> --status closed` or `bd close <issue-id>`
- Sync to the beads branch regularly:
  - `bd sync`

### Merging beads metadata to main
- Check status:
  - `bd sync --status`
- Create a PR from `beads-sync` → `main` (recommended for protected branches).
- After merge, update local state:
  - `git checkout main && git pull`
  - `bd import`

### Optional daemon
- `bd daemon start --auto-commit` (commits updates to `beads-sync` automatically).
- Avoid `--auto-push` unless you want fully automated sync.

## Beads (progress tracking)
We use Beads (bd) for issues, progress tracking, and run logs. All work must be represented by Beads issues and updated as it progresses.

### Setup (already initialized)
- Repo is initialized with `bd init`.
- Sync branch configured: `beads-sync`.
- Beads metadata lives in `.beads/`.

### Daily workflow
- Create issues for any planned work:
  - `bd create "Implement CSV ingestion"`
  - `bd q "Short task"` (quick capture)
- Start work by updating status:
  - `bd update <issue-id> --status in_progress`
- Log decisions, results, and key outputs as comments:
  - `bd comments add <issue-id> "Summary of result + links"`
- Mark complete:
  - `bd update <issue-id> --status done` or `bd close <issue-id>`
- Sync to the beads branch regularly:
  - `bd sync`

### Dependencies and structure
- Use dependencies to model task order:
  - `bd dep add <child-id> <parent-id>`
- Use epics for larger work:
  - `bd epic create "MVP"` then attach issues to the epic.

### Health checks
- `bd doctor --fix` if anything looks misconfigured.
- `bd status` or `bd list` for a quick overview.

### Notes
- The JSONL files under `.beads/` are tracked; DB/daemon files are ignored.
- If a clone is missing the sync branch, run `bd migrate sync beads-sync`.
- Optional editor integration: `bd setup claude` (adds hooks for context injection).

## Architecture summary
- **apps/api**: FastAPI service (projects, datasets, runs, artifacts).
- **apps/web**: Svelte control plane UI.
- **packages/core**: profiling, analysis, metadata inference.
- **packages/connectors**: ingestion sources (file, DB).
- **packages/runtime**: job execution, logging, environment resolution.
- **packages/reporting**: markdown/HTML/PDF reports.
- **projects/**: per‑project workspaces (gitignored).

## Agent layer guidance
- Use an agent framework (e.g., PydanticAI) only for orchestration and user assistance.
- Agent tools must map to safe, deterministic actions: ingest, profile, analyze, report.
- Avoid “auto‑modify data” tools without explicit user confirmation.
- Store all agent actions as run logs with inputs and outputs.

## Project isolation
- Each project has:
  - data/raw, data/staging, data/processed
  - scripts/
  - artifacts/
  - metadata/
  - secrets/ (encrypted or env‑backed)
  - env/ (definition file, not a shared runtime)

## API conventions
- All writes are idempotent when possible.
- All runs emit: status, logs, artifact list, timestamps.
- Artifacts are immutable once created.
- Use explicit run types: ingest, profile, analyze, report.

## Metadata conventions
- Persist metadata as JSON + markdown in project workspace.
- Always include schema snapshots and column stats.
- Capture candidate keys, duplicates, and join hints.

## Reporting conventions
- Reports are generated from deterministic data + templates.
- Store raw data used for report generation in artifacts.
- Export both Markdown and HTML; PDF is optional in MVP.

## Error handling
- Fail fast with clear error messages and recovery hints.
- Log structured errors with dataset/run IDs.
- Never swallow exceptions in worker/runtime layers.

## Testing
- Unit tests for core analysis + metadata inference.
- Integration tests for ingestion and report generation.
- Snapshot tests for metadata outputs.

## Security
- Secrets never stored in logs.
- Project workspaces must not escape root workspace.
- Validate file uploads and paths.

## Definition of done (MVP)
- One‑click project creation.
- CSV ingest → profiling → analysis → report with artifacts.
- UI shows datasets, run history, and reports.
- Metadata docs generated for each dataset.

## Landing the Plane (Session Completion)

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **PUSH TO REMOTE** - This is MANDATORY:
   ```bash
   git pull --rebase
   bd sync
   git push
   git status  # MUST show "up to date with origin"
   ```
5. **Clean up** - Clear stashes, prune remote branches
6. **Verify** - All changes committed AND pushed
7. **Hand off** - Provide context for next session

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - that leaves work stranded locally
- NEVER say "ready to push when you are" - YOU must push
- If push fails, resolve and retry until it succeeds

---
> Source: [leokeba/data-analyst](https://github.com/leokeba/data-analyst) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
