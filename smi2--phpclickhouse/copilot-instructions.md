## phpclickhouse

> PHP client library for ClickHouse (`smi2/phpclickhouse`). Pure PHP, no dependencies beyond ext-curl and ext-json. Supports PHP 8.0+.

# CLAUDE.md — phpClickHouse

## Project Overview

PHP client library for ClickHouse (`smi2/phpclickhouse`). Pure PHP, no dependencies beyond ext-curl and ext-json. Supports PHP 8.0+.

## Principles

- Follow existing patterns in surrounding code — the codebase uses Doctrine coding standard (phpcs.xml.dist)
- Write tests for new functionality — all tests require a running ClickHouse instance
- Keep changes focused — avoid over-engineering, this is a lightweight HTTP client
- Security: all user-supplied values must go through `ValueFormatter::formatValue()` or bindings (`:param`), never raw string interpolation into SQL
- Use `declare(strict_types=1)` in every PHP file
- PSR-4 autoloading: `ClickHouseDB\` namespace maps to `src/`
- **Backward compatibility is sacred:** NEVER rename, remove, or change signatures of public methods. Public API must remain fully compatible across versions. New parameters — only with default values. Deprecate, but don't break.

## Architecture

```
Client (API) → Http (transport) → CurlerRequest/CurlerRolling (curl) → Statement (results)
                ↑                        ↑
            Settings              Query + Degenerations (SQL generation)
```

- **Client.php** — user-facing API: `select()`, `write()`, `insert()`, `selectAsync()`
- **Statement.php** — result wrapper with lazy parsing, implements Iterator
- **Settings.php** — connection config (timeouts, compression, HTTPS, readonly)
- **Transport/Http.php** — HTTP layer, auth methods (header/basic/query string), URL construction
- **Transport/CurlerRolling.php** — async execution via `curl_multi_*`
- **Query/Query.php** — SQL + format management, degeneration pipeline
- **Query/Degeneration/Bindings.php** — parameter binding (`:param` → escaped value)
- **Query/Degeneration/Conditions.php** — conditional SQL template blocks
- **Quote/ValueFormatter.php** — type-aware value escaping for SQL safety
- **Cluster.php** — multi-node cluster support with health checks

## Key Notes

### Running Tests

Tests require a running ClickHouse server. Docker Compose starts **two versions**:

```bash
docker-compose -f tests/docker-compose.yaml up -d
```

| Container | Version | Port | Purpose |
|-----------|---------|------|---------|
| `clickhouse-21` | 21.9 | 8123 | Backward compatibility (old MergeTree syntax, strings in JSON) |
| `clickhouse-latest` | 26.3.3.20 | 8124 | Modern CH (native types in JSON, HTTP 500 for mid-stream errors) |

**Two separate test suites:**

```bash
# ClickHouse 21.9 — all original tests
./vendor/bin/phpunit -c phpunit-ch21.xml

# ClickHouse 26.3 — shared tests + CH26-adapted tests
./vendor/bin/phpunit -c phpunit-ch26.xml
```

CH26-specific tests live in `tests/ClickHouse26/` and account for behavioral differences:
- Modern MergeTree syntax (`ORDER BY` instead of deprecated constructor args)
- JSON returns native numeric types instead of strings (`UInt64Test`)
- Temporary tables work without sessions (`SessionsTest`)
- Mid-stream errors return HTTP 500 instead of 200 (`StatementTest`)

Environment variables (defaults in `phpunit-ch21.xml` / `phpunit-ch26.xml`):

| Variable | CH 21 | CH 26 |
|---|---|---|
| `CLICKHOUSE_HOST` | `127.0.0.1` | `127.0.0.1` |
| `CLICKHOUSE_PORT` | `8123` | `8124` |
| `CLICKHOUSE_USER` | `default` | `default` |
| `CLICKHOUSE_PASSWORD` | _(empty)_ | _(empty)_ |
| `CLICKHOUSE_DATABASE` | `php_clickhouse` | `php_clickhouse` |

**Important:** `CLICKHOUSE_DATABASE` must NOT be `default` — tests DROP and recreate the database.

### Test Setup

Tests use the `WithClient` trait (`tests/WithClient.php`). It initializes `$this->client` and `$this->tmpPath` via `@before`. The database is created once per test run (global flag hack).

### Code Quality

```bash
# Static analysis (PHPStan level 5, baseline for legacy code)
./vendor/bin/phpstan analyse --memory-limit=512M

# Code style (Doctrine standard)
./vendor/bin/phpcs
```

### SQL Injection Prevention

- Never concatenate user input directly into SQL strings
- Use bindings: `$client->select('SELECT * FROM t WHERE id = :id', ['id' => $value])`
- `ValueFormatter::formatValue()` escapes strings via `addslashes()` + single-quote wrapping
- Arrays in bindings are expanded to `IN (val1, val2, ...)` lists automatically
- `Expression` and `Type` objects bypass escaping — use only for trusted values

### Auth Methods

Four methods available in `Transport/Http.php`:
- `AUTH_METHOD_NONE` (0) — no auth (trusted/proxy setups)
- `AUTH_METHOD_HEADER` (1, default) — `X-ClickHouse-User` / `X-ClickHouse-Key` headers
- `AUTH_METHOD_QUERY_STRING` (2) — URL parameters
- `AUTH_METHOD_BASIC_AUTH` (3) — HTTP Basic Auth

### Query Formats

Supported output formats set via `Query::setFormat()`: JSON, JSONCompact, JSONEachRow, CSV, CSVWithNames, TSV, TSVWithNames, TabSeparated, and others.

### Async Queries

Async pattern uses a queue:
1. `$client->selectAsync()` — queues request (returns Statement immediately)
2. `$client->executeAsync()` — blocks until all queued requests complete
3. Access results from returned Statement objects

`CurlerRolling` limits concurrent connections to 10 by default.

### Streaming & Bulk Inserts

- `insertBatchFiles()` — async insert from CSV/TSV files
- `insertBatchStream()` — stream-based inserts
- `StreamInsert`, `StreamRead`, `StreamWrite` — gzip-capable stream wrappers

### Native Query Parameters

Server-side typed parameter binding — SQL injection impossible at protocol level:

```php
$db->selectWithParams('SELECT * FROM t WHERE id = {id:UInt32}', ['id' => 42]);
$db->writeWithParams('INSERT INTO t VALUES ({id:UInt32}, {name:String})', ['id' => 1, 'name' => 'Alice']);
```

### Per-Query Settings

Override settings for individual queries without changing global config:

```php
$db->select('SELECT ...', [], null, null, ['max_execution_time' => 300]);
```

### Generators

Memory-efficient iteration for large resultsets:

```php
foreach ($db->selectGenerator('SELECT * FROM huge_table') as $row) { ... }
```

### Releases

Version format: `MAJOR.YY.MDD` — example `1.26.412`

| Segment | Meaning | Example |
|---------|---------|---------|
| `MAJOR` | Critical API changes. Currently always `1` | `1` |
| `YY` | Year (last two digits) | `26` = 2026 |
| `MDD` | Month + day or version within month | `412` = April 12 |

How to release:

1. Add entry to `CHANGELOG.md` with date and version: `### YYYY-MM-DD [Release X.YY.MDD]`
2. Commit and push to master
3. Create GitHub Release: `gh release create X.YY.MDD --title "X.YY.MDD" --notes "..."` — body is the CHANGELOG section for this version
4. Tag is created automatically by `gh release create`

### CI

GitHub Actions (`.github/workflows/tests.yml`): PHP 8.0–8.4 × ClickHouse 21.9 + 26.3, PHPStan, PHPCS.

Legacy Travis CI config (`.travis.yml`) still present.

### File Structure

```
src/
├── Client.php, Statement.php, Settings.php, Cluster.php
├── Exception/          # ClickHouseException hierarchy (DatabaseException with CH error name, query ID)
├── Query/              # SQL generation, bindings, conditions
├── Quote/              # Value formatting and escaping
├── Transport/          # HTTP/curl layer (IPv6, curl options, AUTH_METHOD_NONE)
└── Type/               # Int64, UInt64, Decimal, UUID, IPv4, IPv6, DateTime64, Date32, MapType, TupleType
tests/                  # PHPUnit tests for CH 21.9
tests/ClickHouse26/     # Adapted tests for CH 26.3
doc/                    # Documentation (types, native-params, generators, exceptions, etc.)
example/                # 29 usage examples
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/smi2) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-12 -->
