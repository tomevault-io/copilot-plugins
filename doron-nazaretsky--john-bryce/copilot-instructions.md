## john-bryce

> Add or modify executable code cells in pg-dba MyST lessons. Use when creating new lessons with runnable SQL or bash, migrating cells to the python kernel, or when the user asks about %%sql, %%bash, jupysql, or live code execution in lessons.


# MyST Executable Code Cells (pg-dba)

## Kernel and Magics

All executable lessons use the **Python kernel** with two cell magics:

| Magic | Use for |
|-------|---------|
| `%%sql` | Raw SQL (SELECT, INSERT, DDL, etc.) — connects via `DATABASE_URL` |
| `%%bash` | Shell commands (psql, ls, export, apt, etc.) |

## Adding a New Executable Lesson

1. **Frontmatter** — set kernelspec to python3:
   ```yaml
   ---
   kernelspec:
     name: python3
     language: python
     display_name: Python 3
   ---
   ```

2. **Setup cell** (hidden, first code cell) — load jupysql; no connection string needed:
   ````markdown
   ```{code-cell} python
   :tags: [remove-input]
   %load_ext sql
   ```
   ````

3. **Content cells** — prefix with `%%sql` or `%%bash`:
   ````markdown
   ```{code-cell} python
   %%sql
   SELECT * FROM pg_stat_activity LIMIT 5;
   ```
   ````

   ````markdown
   ```{code-cell} python
   %%bash
   psql -c "\dt"
   ```
   ````

## When to Use Which Magic

- **`%%sql`** — Pure SQL: `SELECT`, `WITH`, `INSERT`, `UPDATE`, `DROP`, `CREATE`, `ALTER`, etc. Jupysql auto-connects from `DATABASE_URL` (set in `compose.yml`).
- **`%%bash`** — Anything that needs a shell: `psql -c`, `psql << EOM`, `ls`, `export`, `alias`, heredocs, system commands. Env vars (`PGHOST`, `PGUSER`, `PGDATABASE`, `PGPASSWORD`) are already in the container.
  - **Transaction control** — Always use `%%bash` with a `psql` heredoc when the example relies on explicit `BEGIN`/`COMMIT`/`ROLLBACK` or `SET LOCAL`. The `%%sql` magic wraps each cell in its own transaction, which interferes with transaction-level settings like `synchronous_commit`.

## Running OS-Level Commands on the db Container

The `workspace` container is separate from the `db` container that runs PostgreSQL. Commands that inspect the PostgreSQL process tree (`ps`), the data directory (`$PGDATA`, `pg_ctl`), or local filesystem paths on the server must be routed to the `db` container.

Two patterns are available inside `%%bash` cells:

**`pgdb` wrapper** — use for scripts that contain no inner single quotes. `pgdb` calls `docker exec "${DB_CONTAINER:-db}" bash -c "$*"`, so the entire single-quoted string is passed as one argument to `bash -c`. Multi-line scripts work by embedding newlines inside the single quotes:

````markdown
```{code-cell} python
%%bash
pgdb 'pg_ctl status -D $PGDATA'
```
````

````markdown
```{code-cell} python
%%bash
pgdb 'DB_OID=$(psql -A -t -c "SELECT oid FROM pg_database WHERE datname = current_database();")
echo "OID: $DB_OID"
ls "$PGDATA/base/$DB_OID" | head -10'
```
````

**`docker exec -i` with heredoc** — use when the script contains inner single quotes (e.g. SQL table name literals in `psql -c "SELECT ... 'tablename' ..."`). The `-i` flag is required: without it, `docker exec` does not attach stdin, so bash has nothing to read the heredoc from and produces no output silently.

````markdown
```{code-cell} python
%%bash
docker exec -i -u postgres "$DB_CONTAINER" bash <<'PGDB'
FILEPATH=$(psql -A -t -c "SELECT pg_relation_filepath('my_table');")
ls -lh "$PGDATA/$FILEPATH"
PGDB
```
````

> **Never nest heredocs in a `%%bash` cell.** The `%%bash` magic executes the cell as a script, but `docker exec container bash <<'HEREDOC'` without `-i` silently produces no output because docker exec does not attach stdin by default. Always use `docker exec -i` when piping a heredoc.

The `DB_CONTAINER` env var is set to `db` in `compose.yml`. The Docker socket is mounted into `workspace` via `compose.yml` so the Docker CLI works inside the container.

## Configuration

- **Connection** — `DATABASE_URL` in `compose.yml`; jupysql reads it automatically.
- **Jupysql options** — `pyproject.toml` under `[tool.jupysql.SqlMagic]` (e.g. `displaycon`, `feedback`, `displaylimit`).

## Files Not Migrated

Bash-heavy lessons (apt, patroni, pgbackrest, su postgres, etc.) stay on `{code-cell} bash` and are not migrated. With global `kernelName: python3`, their live cells would need `%%bash` to work; they are left as reference/copy-paste material.

---
> Source: [doron-nazaretsky/john-bryce](https://github.com/doron-nazaretsky/john-bryce) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
