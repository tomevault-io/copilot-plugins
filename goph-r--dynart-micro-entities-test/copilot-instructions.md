## dynart-micro-entities-test

> **dynart-micro-entities-test** is the test suite for [dynart-micro-entities](../dynart-micro-entities). It follows the same pattern as `dynart-micro-test`: a separate repository that symlinks the library under test via a Composer path repository.

# CLAUDE.md

## Project Overview

**dynart-micro-entities-test** is the test suite for [dynart-micro-entities](../dynart-micro-entities). It follows the same pattern as `dynart-micro-test`: a separate repository that symlinks the library under test via a Composer path repository.

Both `dynart/micro-entities` (`../dynart-micro-entities`) and `dynart/micro` (`../dynart-micro`) are symlinked, so library changes are reflected immediately.

## Running Tests

```bash
# from this directory
php vendor/bin/phpunit --testsuite unit --stderr
php vendor/bin/phpunit --testsuite integration --stderr   # requires MariaDB
php vendor/bin/phpunit --stderr
```

PHPUnit 9.5, config in `phpunit.xml.dist`. Two test suites: `unit` (no DB) and `integration` (requires MariaDB).

## Project Structure

```
src/
  Integration/
    IntegrationTestCase.php   abstract base: wires Config, MariaDatabase, EntityManager,
                              MariaQueryBuilder, QueryExecutor from test.ini for each test
  Entities/
    TestUser.php              id (PK AI), name, email, active, created_at
    TestPost.php              id (PK AI), user_id (FK→TestUser cascade), title, body, published_at
  StubConfig.php              ConfigInterface stub — returns defaults
  StubLogger.php              LoggerInterface stub — discards all output
  StubEvents.php              EventServiceInterface stub — records emitted event names in $emitted[]
  TestDatabase.php            Database subclass with no-op connect(), backtick escapeName()
  TestHelper.php              createEntityManager() and registerEntity() via reflection
tests/
  Unit/                       No DB required; uses TestDatabase + stubs
  Integration/                Requires MariaDB; each test creates/drops its own tables
configs/
  test.ini                    DB connection config for integration tests
```

## Configuring Integration Tests

Edit `configs/test.ini`:

```ini
database.default.dsn           = "mysql:host=localhost"
database.default.name          = micro_entities_test
database.default.username      = root
database.default.password      =
database.default.table_prefix  = te_
```

> **Note:** The DSN value must be quoted if it contains `=`. Bare `=` in values breaks
> `parse_ini_file` with `INI_SCANNER_TYPED`.

Create the database before first run:

```sql
CREATE DATABASE micro_entities_test CHARACTER SET utf8;
```

Integration tests manage their own tables (create in `setUp`, drop in `tearDown`).

## Key Patterns

- **`IntegrationTestCase`** lives in `src/Integration/` (not `tests/`) so the autoloader can find it before PHPUnit alphabetically loads `DatabaseTest` which extends it.
- **Table drop order** — always drop child tables (FK references) before parent tables to avoid constraint errors: `dropTable(TestPost::class)` then `dropTable(TestUser::class)`.
- **Condition param names** in `Database::update()` must differ from column names in the SET clause to avoid the merged params array overwriting the update value.

---
> Source: [goph-R/dynart-micro-entities-test](https://github.com/goph-R/dynart-micro-entities-test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-27 -->
