## model-counter

> **Package:** `rejoose/model-counter` — A Laravel package for ultra-efficient model counting using Redis-backed caching with scheduled database synchronization. Tracks high-volume metrics (downloads, views, likes, visits) without database bottlenecks.

# CLAUDE.md

## Project Overview

**Package:** `rejoose/model-counter` — A Laravel package for ultra-efficient model counting using Redis-backed caching with scheduled database synchronization. Tracks high-volume metrics (downloads, views, likes, visits) without database bottlenecks.

**PHP:** ^8.3 | **Laravel:** 11.x / 12.x | **License:** MIT

## Architecture

Three-phase design:

1. **Write** — Redis atomic `INCR`/`DECR` (sub-millisecond, no DB hit)
2. **Sync** — Scheduled `counter:sync` command batch-upserts Redis deltas to the database, then clears Redis keys
3. **Read** — Combines DB baseline + Redis delta for accurate current count

Redis key format: `{prefix}{model_type}:{owner_id}:{counter_key}[:interval:period_key]`

## Directory Structure

```
src/
  Counter.php                  # Main static facade (increment/decrement/get/set/reset/delete)
  ModelCounterServiceProvider.php  # Service provider, config, migrations, macros
  Console/SyncCounters.php     # Artisan counter:sync command (Redis→DB)
  Enums/Interval.php           # Day/Week/Month/Quarter/Year enum with period calculations
  Models/ModelCounter.php      # Eloquent model for DB persistence
  Traits/HasCounters.php       # Trait for models (counter methods + query scopes)
  Filament/                    # Optional Filament 4 admin panel integration
config/counter.php             # Package configuration (store, direct, prefix, batch_size, table_name)
database/migrations/           # Two migrations: create table + add interval columns
tests/
  Feature/CounterTest.php      # Core functionality tests
  Feature/RecountIntervalTest.php  # Interval recount tests
  Feature/RelationCounterTest.php  # Relation macro + scope tests
  TestCase.php                 # Base test case (SQLite in-memory, array cache, direct mode)
```

## Commands

```bash
# Run tests
composer test              # vendor/bin/pest

# Run tests with coverage
composer test-coverage     # vendor/bin/pest --coverage

# Code formatting (Laravel Pint, PSR-12)
composer pint              # vendor/bin/pint

# Static analysis (PHPStan level 5)
composer phpstan           # vendor/bin/phpstan analyse
```

## Testing

- Framework: **Pest PHP 3** on top of **Orchestra Testbench 9**
- Database: SQLite in-memory (`RefreshDatabase` trait)
- Default test config: `counter.store = array`, `counter.direct = true` (no Redis required)
- For Redis-specific tests, use the `useRedisCache()` helper in `TestCase`
- Follow AAA pattern (Arrange, Act, Assert)
- Tests live in `tests/Feature/` and `tests/Unit/`

## Code Style & Conventions

- **PSR-12** via Laravel Pint with custom rules: `simplified_null_return`, no `braces`, no `new_with_braces`
- **Strict types** in all files
- Full **type hints** on parameters and return types
- **PHPStan level 5** — all code must pass static analysis
- Indentation: 4 spaces (2 for YAML/JSON)
- Line endings: LF
- Encoding: UTF-8

## CI Pipeline

Defined in `.github/workflows/`:

- **tests.yml** — Matrix: PHP 8.3/8.4 x Laravel 11/12 x prefer-lowest/prefer-stable. Runs Pest with Redis 7 service container. Uploads coverage to Codecov.
- **lint.yml** — Runs PHPStan and Pint in parallel.

## Key Patterns

- **Polymorphic ownership:** Counters attach to any Eloquent model via `owner_type`/`owner_id`
- **Interval enum:** `Day`, `Week`, `Month`, `Quarter`, `Year` with `periodKey()` for Redis/DB keys
- **Direct mode:** Set `COUNTER_DIRECT=true` to bypass Redis and write straight to DB (useful for dev/testing)
- **Query scopes:** `withCounter()` and `orderByCounter()` use JOINs to avoid N+1
- **Atomic operations:** Redis INCR/DECR for writes; GETDEL during sync; insert-or-update with retry for race conditions
- **Recount:** `recount()` and `recountPeriods()` recalculate counters from source data using relationship macros

## Configuration (config/counter.php)

| Key              | Env Var                | Default          | Description                          |
|------------------|------------------------|------------------|--------------------------------------|
| `store`          | `COUNTER_STORE`        | `redis`          | Cache store (redis, array, custom)   |
| `direct`         | `COUNTER_DIRECT`       | `false`          | Write directly to DB, skip Redis     |
| `prefix`         | `COUNTER_PREFIX`       | `counter:`       | Redis key prefix                     |
| `sync_batch_size`| `COUNTER_SYNC_BATCH_SIZE` | `1000`        | Batch size for sync command          |
| `table_name`     | —                      | `model_counters` | Database table name                  |

## Git Conventions

- Commit messages: type(scope): description (e.g., `feat(counter): add weekly interval support`)
- Branch from `main` or `develop`
- PRs require tests and passing CI

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Rejoose)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Rejoose)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
