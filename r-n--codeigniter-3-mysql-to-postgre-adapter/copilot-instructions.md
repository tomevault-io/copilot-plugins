## codeigniter-3-mysql-to-postgre-adapter

> Guidance for any AI coding agent working in this repository. (Claude Code users: see `CLAUDE.md`, which carries the same content.)

# AGENTS.md

Guidance for any AI coding agent working in this repository. (Claude Code users: see `CLAUDE.md`, which carries the same content.)

## What this is

Drop-in compatibility layer that lets a CodeIgniter 3 app written for MySQL run on PostgreSQL **without rewriting application query code**. It is not a standalone app — it is a set of CI `MY_*` overrides + a helper that users copy into their existing CI3 project's `application/` folder.

There is **no build, no package manager, no lint config**. Do not invent commands for them. There is a small framework-free test harness for the SQL rewriter — see Testing below — but the rest of the adapter is validated by running it inside a real CI3 app against a Postgres DB.

## Testing

```
php tests/translator_test.php
```

Stubs `APPPATH` and `get_instance()->db` so `mysqlToPostgres()` runs standalone. Add a case here for any change to the rewriter — it's the only regression net that code has.

## Architecture

Query flow: `$this->db->query($sql)` → `MY_DB_postgre_driver::query()` → `mysqlToPostgres($sql)` rewrites MySQL SQL to Postgres SQL → `parent::query()`.

- **`application/database/drivers/postgre/MY_DB_postgre_driver.php`** — extends `CI_DB_postgre_driver`. (1) Routes every `query()` through `mysqlToPostgres()`. (2) Overrides every Query Builder method that names a table (`_insert`, `_insert_batch`, `_update`, `_delete`, `count_all`, `table_exists`, `list_fields`, `field_data`, `_list_columns`, `primary`, …) to wrap the table with `addSchema()`. Also `SET search_path` (schema quoted as an identifier so it survives hyphens/special chars) in `db_connect()`, and reimplements `insert_id()` via sequences, using `version_compare()` against the numeric part of `version()`.
- **`application/helpers/postgre_helper.php`** — translation core. `addSchema($table)` prefixes `"schema".table` (no-op if name is empty, already has `.` or `?`, or starts with `(` — a parenthesised expression/subquery). `mysqlToPostgres($query)` is a regex + recursive rewriter, cached per top-level query by `md5()`: masks every `'...'`/`"..."` string literal to an inert token first (so backtick stripping, schema injection, and date-fn rewrites never touch quoted data), strips backticks, rewrites `SHOW COLUMNS FROM` into an `information_schema` query, injects the schema after `FROM`/`JOIN`/`INSERT INTO`/`UPDATE`, maps MySQL date functions (`CURDATE`, `INTERVAL n UNIT`, `YEAR/MONTH/DAYNAME/WEEKDAY`, balanced-paren aware for nested calls) — then unmasks literals last.
- **`application/libraries/ParensParser.php`** — tokenizes SQL into a nested array by paren depth so `mysqlToPostgres` can recurse into subqueries (placeholders `?0`, `?1`, … mark nested groups, restored via `strtr()` so `?1` can't match inside `?10`) and run clause regexes per-level. Only strips outer parens when they wrap the whole string (`isWrapped()`), not just because the string starts with `(`.
- **`application/core/MY_Loader.php`** — overrides `database()` to find/instantiate the custom driver, swallows a `SET search_path` connect error (now dead code in practice, kept defensively).
- **`application/core/MY_URI.php`** — empty `CI_URI` subclass (placeholder).

## Editing the SQL translator (`mysqlToPostgres`)

The fragile heart — regex-based and order-dependent. Rules of engagement:
- Recursion has three modes via the `$parsed` arg: 0 = top-level / parse parens, 1 = subquery assembly, deeper = leaf. Confirm which mode a branch runs in before changing it. Literal masking and the cache apply only at `$parsed === 0`.
- Commented-out `consoleLog(...)` calls are debug tracing — uncomment to inspect parse tree / rewrite stages.
- Schema injection can double-prefix; that's why the `"schema"."schema"` → `"schema"` cleanups exist. Adding clause keywords to the injection regex can reintroduce duplicates — keep the cleanups.
- Run `php tests/translator_test.php` after every change. For anything it can't exercise, verify by running representative queries through `mysqlToPostgres()` inside a CI3 context (depends on `get_instance()->db`).

## Conventions

- Match CodeIgniter 3 style (PHP, `MY_` subclass prefix, `#[\AllowDynamicProperties]` on classes for PHP 8.2+).
- Consumer code accesses DB identifiers in **lowercase** — Postgres lowercases unquoted names.

---
> Source: [R-N/codeigniter-3-mysql-to-postgre-adapter](https://github.com/R-N/codeigniter-3-mysql-to-postgre-adapter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
