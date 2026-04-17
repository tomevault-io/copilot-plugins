## dbplus

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DBPlus is a Python database abstraction library providing a unified API across five database backends: SQLite, PostgreSQL, MySQL, Oracle, and DB2. The same application code runs unmodified on any supported database by changing only the connection URL.

**Current version:** 0.4.7 (pre-production, targeting 1.0)

## Setup & Installation

```bash
pip install .                    # Install from local clone
pip install dbplus               # Install from PyPI
```

No runtime dependencies are required. Database drivers are optional, installed per backend:
- SQLite: built-in (no install needed)
- PostgreSQL: `psycopg2`
- MySQL: `mysql-connector-python`
- Oracle: `oracledb`
- DB2: `ibm_db`

## Running Tests

Tests are standalone scripts in `kbtest/` that require actual database connections (no mocking framework). They manipulate `sys.path` to import from the local source tree.

```bash
python kbtest/testsqlite.py      # SQLite tests (needs a Chinook.db file)
python kbtest/mastertest.py      # Comprehensive integration tests
python kbtest/postgrestest.py    # PostgreSQL tests
python kbtest/mysql_test.py      # MySQL tests
python kbtest/oracletest.py      # Oracle tests
python kbtest/testdb2.py         # DB2 tests
```

There is no pytest/unittest harness. Each test file runs directly with Python.

## CI/CD

GitHub Actions CI (`.github/workflows/ci.yml`) triggers on push to `master` and deploys documentation to GitHub Pages via `mkdocs gh-deploy`. It does **not** run tests.

## Architecture

```
User Code
  → Database        (facade: URL parsing, connection lifecycle, transactions)
    → Statement     (SQL execution, parameter binding, cursor management)
      → BaseDriver  (abstract interface in drivers/__init__.py)
        → DB2 | MYSQL | ORACLE | POSTGRES | SQLITE  (concrete drivers)
          → Native libraries (ibm_db, mysql.connector, oracledb, psycopg2, sqlite3)
```

### Key classes

- **`Database`** (`dbplus/Database.py`): Main entry point. Parses a URL like `sqlite:///path.db` or `Postgres://user:pass@host:port/dbname`, dynamically loads the matching driver via `import_module(f"dbplus.drivers.{driver}")`, and exposes `query()`, `execute()`, `callproc()`, `transaction()`, `copy_to()`, `copy_from()`. Falls back to `DATABASE_URL` env var if no URL is provided.

- **`Statement`** (`dbplus/Statement.py`): Handles parameter binding. Converts unified syntax (`:named` params and `?` positional params) into driver-specific placeholders (`?` for SQLite/DB2, `%s` for Postgres/MySQL, `:` for Oracle). Uses a regex to parse parameters outside of string literals.

- **`Record`** (`dbplus/Record.py`): Wraps a single row. Supports dict-style (`record['col']`), index-based (`record[0]`), and attribute-based (`record.col`) access. Has `as_dict()`, `as_json()`, `as_model(pydantic_class)` conversions.

- **`RecordCollection`** (`dbplus/RecordCollection.py`): Lazy result set over Records. Supports iteration, slicing, `len()`, and conversion via `all()`, `one()`, `scalar()`, `as_DataFrame()`, `as_dict()`, `as_json()`.

- **`QueryStore`** (`dbplus/QueryStore.py`): Loads named SQL queries from `.sql` files or directories. Queries are defined with `-- name: query_name` comments and accessed as attributes.

- **`BaseDriver`** (`dbplus/drivers/__init__.py`): Abstract base class. Each driver implements `connect()`, `close()`, `execute()`, `iterate()`, `clear()`, `begin_transaction()`, `commit()`, `rollback()`, `last_insert_id()`, `callproc()`, and `get_placeholder()`.

### Adding a new driver

1. Create `dbplus/drivers/DRIVERNAME.py` with a class named `DBDriver` that extends `BaseDriver`
2. Implement all abstract methods
3. Set the appropriate `get_placeholder()` return value
4. The driver is auto-discovered by name from the connection URL scheme

### Parameter binding flow

The `Statement` class normalizes parameters across drivers. For non-Oracle drivers, named params (`:param`) and positional params (`?`) in SQL are parsed via regex, matched against provided args/kwargs, and rewritten to the driver's native placeholder. Oracle uses `:` natively so kwargs pass through directly.

### Transaction model

`Database.transaction()` is a context manager. It calls `begin_transaction()` on entry, `commit()` on clean exit, and `rollback()` on exception. Nested transactions are not supported and raise `DBError`. Manual `commit()`/`rollback()` inside a `with transaction()` block also raises `DBError`.

### Proxy to driver methods

`Database.__getattr__` proxies unknown method calls to the underlying driver, allowing driver-specific methods (like DB2's `columns()` or `server_info()`) to be called directly on the `Database` object.

## Documentation

Built with MkDocs Material. Source in `docs/index.md`, config in `mkdocs.yml`.

```bash
pip install -r requirements.txt  # mkdocs + mkdocs-material
mkdocs serve                     # Local preview
mkdocs build                     # Build static site
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/klaasbrant) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
