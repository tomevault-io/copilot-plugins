## jitdemo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Setup
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

# Run the app
streamlit run app.py

# Run all tests
pytest

# Run a single test
pytest tests/test_vault.py::test_vault_validate_denies_manual_revocation_before_jwt_expiry

# Syntax check after editing Python files
python -m py_compile <file.py>
```

## Architecture

This is an educational Streamlit demo showing Just-in-Time access vs standing credentials across three actor types (human, app, agent).

**Everything is in-memory. Restart = fresh state. This is intentional.**

### Request Flow

```
Persona runner (src/personas/*.py)
  → MiniVault.request_lease()           # policy lookup, TTL enforcement
  → DynamicDBCredentialsEngine.issue()  # JWT signing + Lease creation
  → returns (token_str, Lease)

Persona runner passes token to DBProxy
  → DBProxy._run_with_audit()
  → MiniVault.validate(token)           # TWO independent checks:
      1. Signer.decode() — cryptographic JWT verification
      2. LeaseManager lookup — rejects revoked leases (even if JWT still valid)
  → _enforce_scope() — allowlists op and table against TokenScope
  → executes SQL query
```

### Key Design Decisions

**Dual validation in `MiniVault.validate()`** (`src/vault.py:70`): JWT expiry alone is insufficient. A manually revoked lease must be rejected even if the JWT hasn't expired yet. The `LeaseManager` is the second independent gate.

**Table name interpolation in `DBProxy`** (`src/db_proxy.py:28`): Table names are interpolated directly into SQL — this is safe because the table name was already allowlisted from the vault-issued `TokenScope.tables` before reaching the query. Never add new tables to DBProxy without registering them in a policy first.

**`LeaseManager` immutability** (`src/lease_manager.py:41`): Revocation uses `model_copy(update={...})` to produce a new `Lease` instance — never mutates in place. All `Lease` and `AuditEvent` objects are Pydantic models; use `model_copy` for any state transitions.

**Background sweeper** (`src/lease_manager.py:59`): A daemon thread marks expired leases as revoked and emits `lease_expired` audit events. Started once per Streamlit session in `app.py:31`. Tests that need it call `start_sweeper()` / `stop_sweeper()` explicitly.

**Streamlit session state** (`app.py:24`): `MiniVault`, `sqlite3.Connection`, and `DBProxy` are initialized once per session under the `"vault"` key. All pages access shared state via `st.session_state`. Never re-initialize these objects on page load — the guard `if "vault" not in st.session_state` must stay.

### Module Map

| Module | Role |
|--------|------|
| `src/models.py` | Pydantic models: `Policy`, `Lease`, `TokenScope`, `AuditEvent` |
| `src/vault.py` | `MiniVault` — facade for all lease/policy/audit operations |
| `src/secrets_engine.py` | `DynamicDBCredentialsEngine` — JWT issuance + lease creation |
| `src/signer.py` | `Signer` — HS256 JWT encode/decode |
| `src/lease_manager.py` | `LeaseManager` — in-memory lease store + background sweeper |
| `src/audit.py` | `AuditLog` — append-only event log with `tail()` / `since()` |
| `src/db_proxy.py` | `DBProxy` — scope-enforcing SQLite wrapper |
| `src/personas/` | `human.py`, `agent.py`, `app.py` — generator-based scenario runners |
| `src/seed.py` | Populates SQLite tables and registers default policies |
| `src/ui_theme.py` | Streamlit CSS injection for the dark glassmorphism theme |
| `pages/` | Streamlit multipage UI (Standing Access, JIT Flow, Vault Admin, Audit Log) |

### Exceptions

Three domain exceptions live in `src/__init__.py`:
- `AccessDenied` — raised by vault validation and scope enforcement
- `InvalidToken` — raised by `Signer.decode()`
- `PolicyNotFound` — raised by `MiniVault.request_lease()`

## Language Conventions

Python 3.13 project. Use Python string syntax (single/double quotes, f-strings, triple-quoted strings). Never use JavaScript-style backtick literals.

## Verification Before Completion

Before declaring any task complete:
1. Run `python -m py_compile <file>` on every edited `.py` file
2. Run `pytest` and confirm all tests pass — report the count (e.g., `46/46 passing`)
3. If the Streamlit app was modified, verify it starts: `streamlit run app.py --server.headless true`

---
> Source: [sjit123/JITDemo](https://github.com/sjit123/JITDemo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-24 -->
