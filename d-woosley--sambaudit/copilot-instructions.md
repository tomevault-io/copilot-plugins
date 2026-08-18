## sambaudit

> This file provides instructions for working with this Python CLI project.

# CLAUDE.md - Project Instructions for Claude Code

This file provides instructions for working with this Python CLI project.

## Project Overview

- **Project name**: sambaudit
- **Type**: Security / SMB auditing CLI tool
- **Core functionality**: Discovers SMB hosts via LDAP, enumerates shares, BFS-crawls directory trees with threads, indexes everything to SQLite
- **Target users**: Security engineers auditing internal file shares; pentesters (loud)

## Project Structure

```
sambaudit/
├── pyproject.toml       # Package configuration
├── README.md            # Project documentation
├── LICENSE              # MIT License
├── sambaudit/           # Main Python package
│   ├── __init__.py      # Package init
│   ├── run.py           # CLI entry point
│   ├── cli_args.py      # Argument parsing (ArgsParser)
│   ├── core.py          # Main orchestration (SambAudit)
│   ├── db.py            # SQLite operations (DatabaseManager)
│   ├── logger.py        # SQLite-backed logging (AuditLogger)
│   ├── auth.py          # SMB auth — NTLM/Kerberos (Authenticator)
│   ├── ldap_query.py    # LDAP host discovery (LDAPQuerier)
│   ├── smb.py           # Share/directory listing (SMBOperations)
│   ├── crawl.py         # BFS crawler (ShareCrawler)
│   └── throttle.py      # Resource monitor (Throttler)
```

## Key Files

- **pyproject.toml**: Package configuration — dependencies, entry point `sambaudit`
- **sambaudit/run.py**: CLI entry point — calls `parse_args()` then `SambAudit.run()`
- **sambaudit/core.py**: Main orchestration — discover → enumerate → crawl
- **sambaudit/db.py**: All SQLite CRUD — thread-local connections, WAL mode
- **sambaudit/crawl.py**: BFS with two thread pools (coord + dir) to avoid deadlock
- **sambaudit/logger.py**: Custom `SQLiteHandler` — all logs go to DB, optionally to console

## Development Commands

### Installation - IMPORTANT
**NEVER install packages with pip outside of a virtual environment.** Use pipx instead:

```bash
# Install in development mode using pipx
pipx install -e .

# Reinstall to pick up new dependencies
pipx install -e . --force
```

### Building
```bash
# Build package
pip build

# Build source distribution
python -m build --sdist
```

### Testing

**IMPORTANT: Run the full test suite before marking any task complete.**

pytest must run inside the pipx venv so it can import impacket and all other dependencies.
**Do NOT use `pytest`, `python -m pytest`, or `$(pipx environment ...)` — these all fail.**

The concrete pytest path is always:
```
~/.local/pipx/venvs/sambaudit/bin/pytest
```

If it is missing (no output from `ls ~/.local/pipx/venvs/sambaudit/bin/pytest`), inject it first — this only needs to be done once:
```bash
pipx inject sambaudit pytest
```

Then run tests:
```bash
# Run all tests
~/.local/pipx/venvs/sambaudit/bin/pytest tests/

# Run with coverage report
~/.local/pipx/venvs/sambaudit/bin/pytest tests/ --cov=sambaudit

# Run a single test file
~/.local/pipx/venvs/sambaudit/bin/pytest tests/test_db.py -v
```

Tests live in `tests/` — one file per module (`test_db.py`, `test_auth.py`, etc.).

**Rules:**
- Every new module must have a corresponding `tests/test_<module>.py`
- Every bug fix must include a regression test that fails without the fix
- When a method changes behaviour, update its test to match
- Tests mock external services (Impacket SMB, LDAP, psutil) but use real SQLite for `db.py` tests
- Run the full suite after every change — do not mark a task done until it passes

### Running
```bash
# Show help
sambaudit --help

# Single target, anonymous
sambaudit --target 192.168.1.50

# Full LDAP discovery with NTLM auth
sambaudit --dc dc01.corp.local -D CORP -u alice -p 'S3cr3t!'

# LDAP when DNS cannot resolve the DC — --dc for Kerberos SPN, --dc-ip for TCP
sambaudit --dc dc01.corp.local --dc-ip 192.168.1.10 -D corp.local

# Kerberos via ccache (--kerberos is set automatically when --ccache is given)
sambaudit --dc dc01.corp.local --dc-ip 192.168.1.10 --ccache /tmp/user.ccache

# Share discovery only (no crawl)
sambaudit --dc dc01.corp.local -D CORP -u alice -p 'S3cr3t!' -sd

# Hosts file with optional IP override (hostname=ip skips DNS, useful when DNS is wrong)
sambaudit --hosts-file hosts.txt

# Resume an interrupted run
sambaudit --resume

# Debug output to console
sambaudit --target 192.168.1.50 -d
```

## Common Tasks

### Adding a new command
1. Edit `sambaudit/cli_args.py` - add argument in the relevant `_add_*_args()` method
2. Edit `sambaudit/core.py` - add logic in the appropriate workflow method

### Adding dependencies
1. Edit `pyproject.toml` - add to `dependencies` list
2. Confirm with user before adding new packages
3. Reinstall with pipx to load new dependencies: `pipx install -e . --force`

### Adding static files
1. Add files to `sambaudit/static/`
2. Update `pyproject.toml` under `[tool.setuptools.package-data]`

## Important Notes

- Entry point is defined in `pyproject.toml` under `[project.scripts]`
- Logger supports colored output for console and structured format for files
- Debug mode (`-d`) enables verbose logging
- **Always use pipx for installation** - never pip directly
- **Logging output is ignored** - do not commit log files to git

## Authentication Architecture

### Kerberos / ccache
- `--ccache FILE` sets `KRB5CCNAME` and automatically enables `--kerberos`
- `--kerberos` is implied when `--ccache` is provided — no need to pass both
- The Kerberos TGT principal is read from the ccache via `impacket.krb5.ccache.CCache` and shown in the identity string (e.g. `alice@CORP.LOCAL`)
- Env var: `SAMBAUDIT_CCACHE`

### DC IP separation (`--dc-ip`)
- `--dc` is always the **FQDN** — used for Kerberos SPN (`ldap/dc01.corp.local`, `cifs/dc01.corp.local`)
- `--dc-ip` is the **IP for TCP connections** — used when local DNS cannot resolve the DC
- `LDAPConnection(url, base_dn, dstIp)` — FQDN in URL, IP as dstIp
- `SMBConnection(remoteName, remoteHost)` — FQDN as remoteName, IP as remoteHost
- `auth.connect(host, host_ip=ip)` — same separation in SMBOperations
- Env var: `SAMBAUDIT_DC_IP`

### Hosts file format
- One entry per line: plain `hostname` or `hostname=ip` to override DNS
- `hostname=ip` is essential when DNS returns unreachable internal IPs

### Env file loading
- `.env` in CWD is loaded first; falls back to `~/.sambaudit` if not found
- Supported vars: `SAMBAUDIT_DC`, `SAMBAUDIT_DC_IP`, `SAMBAUDIT_DOMAIN`, `SAMBAUDIT_USERNAME`, `SAMBAUDIT_CCACHE`

## LDAP / DNS Internals

- LDAP uses **impacket's built-in LDAP module** (`impacket.ldap.ldap`, `impacket.ldap.ldapasn1`) — not ldap3
- `LDAPQuerier._dns_query()` — raw UDP DNS A-record resolver; queries the DC as nameserver when `--dc-ip` is set, bypassing system DNS
- `LDAPQuerier._pick_ip()` — when a hostname has multiple A records, prefers non-RFC1918 addresses (10/8, 172.16/12, 192.168/16) as more likely to be directly reachable
- If the discovered hostname matches `dc_host`, `dc_ip` is used directly without a DNS lookup

## Crawl Behaviour

### System share filtering
- `IPC$`, `SYSVOL`, `NETLOGON`, `PRINT$`, `FAX$` are skipped during crawl by default
- Override with `--include-system-shares`
- Share discovery (`-sd`) still enumerates and reports all shares; the filter only applies to the crawl phase

### Thread model
- `dir_pool` (`--threads` workers, default 20) — shared across all shares; performs the actual directory listings
- `coord_pool` (one thread per share) — drives the per-share BFS loop; coordinator threads are lightweight (mostly waiting on futures)
- All shares advance simultaneously; the dir_pool distributes work dynamically

### Progress display
- Active shares show Rich progress bars with `searched / known dirs`
- When a share completes, the bar is hidden and replaced with a one-line summary: `✓  hostname\\share  N dirs  M files  X.Xs`

## Code Style - STRICT REQUIREMENTS

### PEP 8 and OOP Best Practices
- **Methods must be small** - Each method should accomplish only ONE task
- **Use PEP 8 section separators** - Separate code sections with:
  ```python
  # ----------------------------------------------------------------------
  # Section Name
  # ----------------------------------------------------------------------
  ```
- **One class per file** - Each unique type of class should be in its own file
- **Type hints required** - Use type hints for all function signatures
- **Docstrings required** - Add docstrings to all public functions and classes

### Security First
- **Security is the #1 priority**
- **All major security decisions must prompt the user** - Never make security decisions automatically
- Validate all user inputs
- Never log sensitive information (passwords, tokens, keys)
- Use the logger's redaction feature for sensitive data

### Import Organization
- **ALL imports must be at the top of the file — ALWAYS**
- **NEVER import inside a method, function, class body, or conditional block**
- **No mid-file imports** - Organize imports at the top only
- Group imports in this order:
  1. Standard library
  2. Third-party packages
  3. Local application imports

### Logging Standards
- **Use non-debug output sparingly** - Only for critical user-facing messages
- **Debug log everything** that would be helpful during debugging
- Use the provided logger.py for consistent logging
- The logger provides colored output for console and structured format for files

## Preferences
- Ask before committing to git
- Prefer editing existing files over creating new ones
- Run tests after making changes
- Keep code simple — no over-engineering
- No unnecessary comments or docstrings

### Data Format Preferences
- **Configuration/settings**: Use JSON for storing config and settings information
- **Output format**: Use CSV for data output
- Avoid YAML, CSV (for config), TOML, or MD for configuration storage

## Workflow
- When something goes sideways, stop and re-plan — don't keep pushing
- After finishing a task: run typecheck, tests, and lint before calling it done

## Style
- Prefer small, focused functions
- Use early returns over nested conditionals

---

## Code Style

- Functions should do one thing. If you need the word "and" to describe it, split it.
- Name variables after what they contain, functions after what they do.
- Don't abbreviate names. `getUserProfile` not `getUsrProf`. Clarity beats brevity.
- No commented-out code. Delete it. Git remembers.
- Handle errors explicitly. Don't swallow exceptions or ignore error returns.
- Keep files under 300 lines. If a file is growing, extract a module.
- Imports go at the top, grouped: stdlib, external packages, internal modules.

### Python-Specific Rules
- Use type hints for all function signatures.

---

## Testing

- Write tests that verify behavior, not implementation details.
- Each test should have one clear assertion. Name it after what it proves.
- Use `describe` blocks to group related tests. Use `it` or `test` for individual cases.
- Test the public API of modules, not internal functions.
- Prefer real dependencies over mocks. Only mock external services (APIs, databases).
- Every bug fix must include a regression test that fails without the fix.
- Run the full test suite before marking any task complete: `pytest`

---

## New Python Modules

**IMPORTANT**: Before adding any new Python module to this project, you MUST:
1. Verify the module is a legitimate, trusted package (check PyPI, GitHub, etc.)
2. Confirm the module is NOT a typosquatting or fake package
3. Present your findings to the user and get confirmation before adding
4. Only add modules that are well-maintained and have a good security history
5. After adding, reinstall with pipx: `pipx install -e . --force`

## Troubleshooting

- **Import errors**: Ensure package is installed (`pipx install -e .`)
- **Entry point not found**: Check `[project.scripts]` in pyproject.toml
- **Static files missing**: Verify paths in `[tool.setuptools.package-data]`
- **New dependencies not loading**: Run `pipx install -e . --force`

---

## Phase 2: Web UI (`webui/`)

### Architecture

- `webui/` is a separate Python package registered as the `sambaudit-web` entry point in `pyproject.toml`
- The Flask server binds to `127.0.0.1` by default; `--host 0.0.0.0` exposes it on all interfaces (warns at startup)
- The server runs over HTTPS using a self-signed certificate stored in `~/.local/share/sambaudit/ssl/`
- Authentication is required — password is auto-generated on first run and printed to stdout; user must change it on first login
- The webui opens the SQLite DB **read-only** (`uri=True`, `?mode=ro`) — it never writes to the database
- Routes are Flask Blueprints in `webui/routes/` — one blueprint per feature area (pages, api_dashboard, api_explorer)
- SQL queries live in `webui/queries/` — route handlers never contain raw SQL

### Frontend Rules (strict)

- **All CSS goes in `static/css/style.css`** — no inline `style=` attributes, no `<style>` blocks anywhere in templates or JS
- CSS custom properties (variables) define all colors, spacing, and typography tokens at `:root`
- File-type category colors are defined as CSS variables and used consistently in both the pie chart and the file explorer
- **Alpine.js components live in `static/js/components/`** — one file per component, each exports a single Alpine `x-data` factory function registered with `Alpine.data()`
- All JS dependencies (Alpine.js, Chart.js, marked.js, DOMPurify, highlight.js) are **vendored locally** in `static/js/vendor/` — no CDN dependencies
- All data comes from `fetch()` calls to `/api/v1/` endpoints — no server-side data injection into `<script>` tags

### API Design

- All JSON API endpoints return `{"data": ..., "error": null}` envelope
- API routes are prefixed with `/api/v1/`
- Page routes return rendered Jinja2 templates that extend `base.html`
- The Flask app is created via an app factory `create_app(db_path, port)` in `webui/app.py`

### File Classification Categories

Files are classified into these 14 categories based on extension. Classification is done in a single `classify_extension(filename)` function — not scattered across routes or templates.

| Category | Extensions / Rule |
|----------|-------------------|
| Credentials | `.pem`, `.crt`, `.key`, `.pfx`, `.p12`, `.csr`, `.der` |
| Scripts | `.sh`, `.ps1`, `.bat`, `.cmd`, `.py`, `.rb`, `.pl`, `.vbs`, `.php`, `.asp`, `.psm1`, `.ksh`, `.zsh`, `.bash` |
| Config | `.conf`, `.cfg`, `.ini`, `.yaml`, `.yml`, `.properties`, `.json`, `.xml`, `.env` |
| Documents | `.doc`, `.docx`, `.xls`, `.xlsx`, `.ppt`, `.pptx`, `.pdf`, `.rtf`, `.csv`, `.odt`, `.ods`, `.odp` |
| Database | `.sql`, `.db`, `.mdb`, `.sqlite`, `.accdb`, `.dbf`, `.dump`, `.bak`, `.backup`, `.ldif` |
| Archives | `.zip`, `.tar`, `.gz`, `.tgz`, `.bz2`, `.7z`, `.rar`, `.xz`, `.cab` |
| Executables | `.exe`, `.dll`, `.so`, `.msi`, `.deb`, `.rpm`, `.apk`, `.jar`, `.war`, `.ear` |
| Media | `.iso`, `.img`, `.vmdk`, `.vdi`, `.dmg`, `.ova`, `.ovf`, `.bin` |
| Web | `.html`, `.htm`, `.js`, `.jsp`, `.aspx`, `.css`, `.cgi` |
| Logs | `.log`, `.out`, `.audit`, `.trace`, `.dmp` |
| Text | `.txt` |
| Backup/Temp | `.tmp`, `.swp`, `.old`, `.save`, `.orig` |
| Sensitive-name | Filename (lowercased) contains: `password`, `secret`, `credential`, `vault`, `token`, `apikey` |
| Other | Everything else |

Each category maps to a CSS variable color token (e.g. `--color-credentials: #e74c3c`). The color mapping is the canonical source of truth in `style.css` — JS reads these via `getComputedStyle` when building charts.

### Scan Status Indicator

Files in the explorer show a scan icon indicating whether the Snaffler-style regex pass has been run against that file's content. The `files` table will gain a `scan_status` column in a future phase. For now the column does not exist — the icon is always rendered as "unscanned" (grey). Do not add `scan_status` to the DB schema until the scanning feature is built.

### sambaudit-web CLI

```
sambaudit-web [--db PATH] [--host HOST] [--port PORT]
  --db PATH     SQLite database to read (env: SAMBAUDIT_DB, default: sambaudit.db in CWD)
  --host HOST   Interface to bind on (default: 127.0.0.1; use 0.0.0.0 to expose on all interfaces)
  --port PORT   Port to bind on (default: 8080)
```

The command is location-independent: it loads `.env` in CWD first, then falls back to
`~/.sambaudit`. Set `SAMBAUDIT_DB=/absolute/path/to/sambaudit.db` in `~/.sambaudit` to
run `sambaudit-web` from any directory without specifying `--db`.

### Web Request Logging

Every HTTP request (except `/static/` assets) is written to the `logs` table in the
SQLite database with `component=webui`. Level is `INFO` (2xx/3xx), `WARNING` (4xx), or
`ERROR` (5xx). Reuses `DatabaseManager.write_log()` — no separate DB writer needed.

### File Tree — Lazy Loading

The file explorer fetches children on demand: `GET /api/v1/explorer/children?parent_id=X`. This avoids loading the full file tree for large crawls. The root level (`parent_id` omitted) returns all hosts; expanding a host returns its shares; expanding a share returns its root-level directories and files.

### Installing webui Dependencies

Flask is the only new dependency. Add to `pyproject.toml` and reinstall:
```bash
pipx install -e . --force
```

---
> Source: [d-woosley/SambAudit](https://github.com/d-woosley/SambAudit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
