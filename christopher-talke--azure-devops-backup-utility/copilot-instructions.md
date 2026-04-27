## azure-devops-backup-utility

> `ado-backup` is a CLI tool that backs up Azure DevOps organisations to disk using

# CLAUDE.md - Azure DevOps Backup Utility

## Project overview

`ado-backup` is a CLI tool that backs up Azure DevOps organisations to disk using
only the Azure CLI (`az`) as the API transport. It requires no additional Python
dependencies beyond the standard library (PyYAML is optional). The entry point is
`src/cli.py`; run with `python src/cli.py --help`.

## Repository layout

```
src/
  cli.py          # Argument parsing, logging setup, config validation, entry point
  config.py       # BackupConfig dataclass; merges YAML → env vars → CLI args
  orchestrator.py # run_backup(): drives scope calls, handles fail-fast, compression
  azcli.py        # Thin subprocess wrapper around az CLI; git_clone(); retry logic
  backoff.py      # retry() with exponential backoff and jitter
  paginator.py    # paginate() helper for continuation-token APIs (rarely used directly)
  paths.py        # BackupPaths class; safe_name(); parse_org_url()
  writers.py      # Atomic JSON/binary file writes; SHA-256 hashing; restricted perms
  redact.py       # redact_dict(); strips sensitive keys + isSecret pattern before persisting
  compress.py     # tar.gz compression: repos / project / all modes
  inventory.py    # Inventory class: tracks entities, errors, writes manifest + JSONL errors
  verify.py       # verify_backup(): samples backed-up items and checks against live ADO
  scopes/
    org.py            # Organisation metadata
    projects.py       # Project list + metadata
    git.py            # Repo clone (--mirror), branches, tags, policies, repo ACLs
    boards.py         # Work items (WIQL), queries, tags, board config, team settings
    pipelines.py      # Pipeline definitions + build history
    pull_requests.py  # PRs across all repos
    artifacts.py      # Artifact feeds + packages
    dashboards.py     # Dashboard JSON
    permissions.py    # Organisation-level ACLs
    wikis.py          # Wiki metadata + pages
    testplans.py      # Test plans + suites

tests/              # pytest unit tests (no integration tests; mocking az CLI)
examples/           # config.yaml, GitHub Actions, Azure Pipelines workflow examples
dashboard/          # Azure Function + vanilla JS frontend for backup observability
  api/              # Python Azure Functions v2 endpoints (reads _indexes/ from Blob Storage)
  frontend/         # Static HTML/CSS/JS served by the function app
```

## How to run

```bash
# Install (editable)
pip install -e .

# Or run directly (no install needed)
PYTHONPATH=src python src/cli.py --org-url https://dev.azure.com/your-org

# With a config file
python src/cli.py --config examples/config.yaml

# Run tests
pytest
```

## Configuration precedence

CLI args override env vars, which override YAML file values, which override defaults.

| Setting | CLI flag | Environment variable | Default |
|---------|----------|----------------------|---------|
| Org URL | `--org-url` | `AZURE_DEVOPS_ORG_URL` | - (required) |
| PAT | - | `AZURE_DEVOPS_EXT_PAT` or `SYSTEM_ACCESSTOKEN` | - (uses az login) |
| Output dir | `--output-dir` | `ADO_BACKUP_OUTPUT_DIR` | `ado-backup` |
| Timeout (s) | - | `ADO_BACKUP_TIMEOUT` | `120` |

## Output directory structure

```
ado-backup/
  dev.azure.com/
    {org}/
      {YYYYMMDDTHHMMSSZ}/
        _indexes/
          manifest.json          # start/end time, entity counts, limits applied
          inventory.json         # per-file SHA-256 checksums
          errors.jsonl           # one JSON record per error
          verification_report.json  # written when --verify is used
        org/                     # organisation-level data
        projects/
          {project}/
            metadata/
            git/
              repos.json
              {repo}/            # bare git mirror clone
              {repo}_branches.json
              {repo}_tags.json
              {repo}_policies.json
              {repo}_permissions.json
            boards/
              work_items/
                index.json       # list of IDs
                {id}.json        # full work item
                {id}_revisions.json
                {id}/attachments/
            pipelines/
            pull_requests/
            artifacts/
            dashboards/
            wikis/
            test_plans/
```

## Key design decisions

- **No pip dependencies at runtime.** `az devops invoke` is the only API transport.
  PyYAML is imported opportunistically; the code ships its own minimal YAML parser
  as a fallback.
- **Atomic writes.** `writers.write_json` / `writers.write_binary` write to a temp
  file in the same directory and `replace()` atomically. Partially-written files are
  cleaned up on error.
- **Directory permissions.** On Unix, `writers._secure_mkdir` chmod 0o700 newly
  created directories to prevent credential leakage from group-readable mounts.
- **PAT never in argv.** `azcli.git_clone` passes the PAT via `GIT_CONFIG_*`
  environment variables (not on the command line). `_mask_pat` redacts `--pat`
  arguments in debug log output.
- **Sensitive data redaction.** Every scope writes data through `redact.redact()`.
  `redact_dict` catches keys by name (`SENSITIVE_KEYS`), by dot-path
  (`SENSITIVE_PATHS`), and by the ADO `{"isSecret": true, "value": ...}` pattern.
- **Retry with backoff.** All `az` calls go through `backoff.retry()`. HTTP 429 and
  5xx responses raise `AzCliThrottled` which is the retryable exception.
- **Incremental backup.** Pass `--since YYYY-MM-DD` to filter work items, pipelines,
  and pull requests by changed date. Git repos use `git remote update --prune` when
  the bare clone already exists.
- **Exit codes.** `0` = success, `1` = backup errors, `2` = verification failures,
  errors abort with code `1` when `--fail-fast` is set.

## Components (--include / --exclude)

Valid values: `org`, `projects`, `git`, `boards`, `pipelines`, `permissions`,
`pull_requests`, `artifacts`, `dashboards`, `wikis`, `testplans`

All components are active by default. Pass `--include a,b` to back up only those
components, or `--exclude c,d` to skip specific ones.

## Verification (--verify)

After backup completes, `verify.verify_backup()` samples `--verify-samples` items
per category (default 3) from each project and compares against the live ADO
instance:

| Category | Check |
|----------|-------|
| git | HEAD SHA of default branch matches live |
| boards | `System.Rev` matches live work item |
| pipelines | `revision` field matches live build definition |
| pull_requests | `status` field matches live PR |
| wikis | Pages file is non-empty |
| artifacts | Package count matches live feed |

Items modified after the backup started are skipped (`_SKIP`). Results are written
to `_indexes/verification_report.json`. Exit code is `2` on any failure.

## Adding a new scope

1. Add a file `src/scopes/{name}.py` with a `backup_{name}(paths, inventory, org_url, project_name, *, pat, dry_run, ...)` function.
2. Import and call it in `orchestrator.py` inside the per-project loop.
3. Add the component name string to `ALL_COMPONENTS` in `config.py`.
4. Add it to the `--include`/`--exclude` help text in `cli.py`.
5. Write unit tests in `tests/test_scopes_{name}.py` - mock `azcli.az` / `azcli.invoke` and `writers.write_json`.

## Testing conventions

- Tests live in `tests/`; `pytest` discovers them automatically.
- `src/` is on `sys.path` via `pyproject.toml` `[tool.pytest.ini_options] pythonpath`.
- All tests are unit tests; they mock `subprocess.run` or higher-level `azcli.*` functions. No real `az` binary is required.
- Common pattern: `unittest.mock.patch("azcli.az", return_value=[...])` or `patch("writers.write_json")`.

## Security notes

- Never store a PAT in a config file (the code warns if it detects one).
- The canonical credential env var is `AZURE_DEVOPS_EXT_PAT`; `SYSTEM_ACCESSTOKEN` is the ADO pipeline built-in.
- `paths.safe_name()` strips `..` and unsafe characters to prevent path traversal from project or repo names.
- `config.validate()` explicitly rejects `output_dir` values containing `..`.
- Compression in `compress.py` checks the source is under the archive destination before deleting it.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/christopher-talke) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
