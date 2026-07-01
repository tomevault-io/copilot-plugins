## fast-paginate

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Fast Paginate is a Laravel package that provides an optimized `limit`/`offset` pagination method using a "deferred join" technique. Instead of fetching full rows during pagination, it first retrieves only primary keys in an optimized subquery, then fetches full records for those specific IDs. This significantly improves performance on large datasets.

## Commands

**Run all tests:**
```bash
./vendor/bin/phpunit
```

**Run a specific test file:**
```bash
./vendor/bin/phpunit tests/Integration/BuilderTest.php
```

**Run a specific test method:**
```bash
./vendor/bin/phpunit --filter basic_test
```

**Note:** Tests require a MySQL database (8.4+). Configure via environment variables or `phpunit.xml`:
- DB_DATABASE=fast_paginate
- DB_USERNAME=test
- DB_PASSWORD=root

## Architecture

The package registers macros on Laravel's Eloquent Builder and Relation classes via `FastPaginateProvider`:

- **`FastPaginate`** (`src/FastPaginate.php`): Core pagination logic. Implements `fastPaginate()` and `simpleFastPaginate()` methods that:
  1. Clone the query and select only primary keys
  2. Run pagination on the key-only query
  3. Use the retrieved IDs in a `WHERE IN` clause on the original query
  4. Return full records with proper pagination metadata

- **`BuilderMixin`** (`src/BuilderMixin.php`): Adds `fastPaginate()` and `simpleFastPaginate()` methods to Eloquent Builder

- **`RelationMixin`** (`src/RelationMixin.php`): Adds the same methods to Eloquent Relations, with special handling for `BelongsToMany` and `HasManyThrough`

- **`ScoutMixin`** (`src/ScoutMixin.php`): Provides `fastPaginate()` on Laravel Scout builders (defers to standard pagination as Scout already optimizes)

### Fallback Behavior

The package automatically falls back to standard pagination when:
- Query contains `havings`, `groups`, or `unions`
- `perPage` is set to `-1` (get all records)
- Query contains incompatible expressions (detected via `QueryIncompatibleWithFastPagination`)

### Key Implementation Detail

Order-by columns that reference computed/aliased columns are preserved in the inner query to maintain correct ordering during the key-selection phase.

---
> Source: [aarondfrancis/fast-paginate](https://github.com/aarondfrancis/fast-paginate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
