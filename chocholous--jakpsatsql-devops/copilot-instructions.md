## jakpsatsql-devops

> - Language: Python 3.13

# jakpsatsql-devops — Czechitas Provisioner

## Stack

- Language: Python 3.13
- Package manager: uv (pyproject.toml + uv.lock, NO pip/venv/requirements.txt)
- Dependencies: snowflake-connector-python, cryptography, rich, pyinstaller, pytest
- Testing: pytest
- Auth: Snowflake JWT (RSA key pair)
- Target platform: macOS (arm64), Windows

## Testing Conventions

- Run all tests: `make test`
- Unit tests only (no Snowflake): `uv run pytest tests/test_unit.py tests/test_phase1_setup.py -v`
- Integration tests (need Snowflake): `SF_KEY_PASSPHRASE=<pp> uv run pytest tests/test_integration.py -v -m integration`
- Test location: `tests/` in project root
- Naming: `test_*.py`
- Fixtures: `tests/conftest.py`
- TSV fixtures: `tests/fixtures/`
- Skip integration if `provisioner_key.p8` or `SF_KEY_PASSPHRASE` missing

## Code Quality Standards

- No `CREATE OR REPLACE` — always use `IF NOT EXISTS` for idempotency
- No hardcoded credentials — passphrase via env var `SF_KEY_PASSPHRASE` or interactive prompt
- Fail-fast TSV validation before any Snowflake connection
- No silent errors — all SQL failures logged and reported
- Configuration constants at top of file (SF_ACCOUNT, SF_USER, etc.) — NOT env vars
- `ruff` for linting: `uv run ruff check .`

## Project Structure

- `provision.py` — main provisioning script (~300 lines)
- `tests/` — test suite
- `tests/fixtures/` — sample TSV files for tests
- `tests/conftest.py` — shared fixtures
- `docs/` — plan and documentation
- `provisioner_key.p8` — encrypted RSA private key (committed, AES256)
- `provisioner_key.pub` — RSA public key (gitignored — goes into Snowflake)
- `setup_snowflake.sql` — SQL for ACCOUNTADMIN to run once
- `build.sh` — PyInstaller build script
- `run.command` — macOS double-click wrapper
- `run.bat` — Windows batch wrapper
- `SPUSTENI.txt` — operator instructions (plain text)

## Config Management

- Snowflake constants hardcoded in `provision.py` (not sensitive, part of code):
  - `SF_ACCOUNT = "gn56074.west-europe.azure"`
  - `SF_USER = "PROVISIONER"`
  - `SF_ROLE = "ROLE_PROVISIONER"`
  - `SF_WAREHOUSE = "COMPUTE_WH"`
  - `SF_DATABASE = "COURSES"`
  - `DEFAULT_KEY = "provisioner_key.p8"`
- Passphrase via `SF_KEY_PASSPHRASE` env var (or interactive getpass prompt)
- `.env` gitignored, never committed
- `provisioner_key.pub` gitignored
- `provisioner_key_plain.p8` gitignored

## Gitignore Rules

Critical files that MUST be gitignored:
- `provisioner_key.pub` — public key goes into Snowflake
- `provisioner_key_plain.p8` — never keep plain key on disk
- `.venv/` — uv virtualenv
- `dist/` — PyInstaller output
- `build/` — PyInstaller build artifacts
- `*.spec` — PyInstaller spec files
- `.env` — local env vars

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chocholous)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/chocholous)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
