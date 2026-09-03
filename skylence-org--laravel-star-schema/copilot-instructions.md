## laravel-star-schema

> Always delegate to specialized subagents when available:

# Laravel Star Schema Package

## Agent Delegation

Always delegate to specialized subagents when available:

- **Codebase exploration** → Use `Explore` agent for finding files, understanding structure, searching code
- **Laravel architecture** → Use `laravel-development:laravel-architect` for design decisions
- **Eloquent/database** → Use `laravel-development:eloquent-expert` for models, relationships, queries
- **Testing** → Use `laravel-development:testing-expert` for Pest tests
- **Security review** → Use `laravel-development:security-engineer` for security audits
- **Performance** → Use `laravel-development:optimization-expert` for optimization

## Skills

Use `/` skills for scaffolding:
- `/model-new`, `/migration-new`, etc.

## Tech Stack

- PHP 8.4+
- Laravel 11/12 (illuminate packages)
- Orchestra Testbench for package testing
- Pest for testing
- Databases: SQLite (local default), MySQL 8, PostgreSQL 17 (CI)

## Package Structure

This is a Laravel package (not an application). Key paths:
- `src/` — Package source code
- `config/star-schema.php` — Published config
- `database/migrations/` — Package migrations (auto-run)
- `tests/` — Pest test suite (Unit + Feature)
- `tests/Fixtures/` — Test models and fact definitions

## Running Tests - MUST FOLLOW

```bash
# Full suite (uses SQLite in-memory by default)
vendor/bin/pest --compact

# With parallel execution
vendor/bin/pest --parallel --compact

# Single file
vendor/bin/pest tests/Feature/StarQueryTest.php

# With coverage
vendor/bin/pest --coverage --min=50
```

- Always use `--compact` for cleaner output
- Local default is SQLite in-memory — CI tests against SQLite, MySQL 8, and PostgreSQL 17
- paratest only accepts a single path argument — do NOT pass multiple directories

## Code Quality - MUST RUN AFTER EDITING PHP FILES

After creating or editing any PHP file, ALWAYS run these commands on the modified files:

```bash
# Fix code with Rector (auto-refactoring)
vendor/bin/rector process path/to/file.php

# Fix code style with Pint
vendor/bin/pint path/to/file.php
```

Or for multiple files:
```bash
vendor/bin/rector process src/Path/To/Directory
vendor/bin/pint src/Path/To/Directory
```

## Git Hooks (CaptainHook)

Pre-commit hooks will run automatically:
1. Rector dry-run (on staged PHP files)
2. Pint dry-run (on staged PHP files)

Pre-push hooks:
1. Rector check (on changed PHP files vs main)
2. Pint check (on changed PHP files vs main)
3. PHPStan analysis

Commit message format (enforced):
```
type(scope): description

Types: feat, fix, docs, style, refactor, test, chore, build, ci, perf, revert
```

## Pint Rules (Laravel Preset + Strict)

Key rules to follow when writing code:
- `declare(strict_types=1)` required in all files
- Use `DateTimeImmutable` over `DateTime`
- Use strict comparison (`===`, `!==`)
- Use `mb_*` string functions
- No superfluous elseif/else blocks (early return)
- Import all classes, constants, functions (no `\` prefix)

Class element order:
1. Traits (`use`)
2. Cases (enums)
3. Constants (public → protected → private)
4. Properties (public → protected → private)
5. Constructor/Destructor
6. Magic methods
7. Abstract methods
8. Public static → Public → Protected static → Protected → Private static → Private

## Rector Rules

Active rule sets:
- `deadCode` - Remove unused code
- `codeQuality` - Improve code quality
- `codingStyle` - Consistent coding style
- `typeDeclarations` - Add type hints
- `privatization` - Make private when possible
- `earlyReturn` - Convert to early returns
- PHP version upgrades

## PHPStan (Level 10)

Strict static analysis with larastan. Baseline file at `phpstan-baseline.neon` — do not add new entries to the baseline. Fix errors properly instead.

## Writing Code - Preventive Guidelines

To avoid Pint/Rector fixes, write code correctly from the start:

```php
<?php

declare(strict_types=1);

namespace Skylence\StarSchema;

use Illuminate\Database\Eloquent\Model;

final class Example extends Model
{
    // Properties first
    protected $guarded = [];

    // Then constructor if needed

    // Then public methods
    public function calculate(): float
    {
        return $this->orders->sum('total');
    }

    // Private methods last
    private function format(): string
    {
        return number_format($this->calculate(), 2);
    }
}
```

Key patterns:
- Always `declare(strict_types=1)`
- Always import classes (no `\DateTime`, use `use DateTime`)
- Use `final` on classes when not extended
- Use early returns instead of nested if/else
- Add return type hints to all methods
- Use strict comparisons (`===` not `==`)
- Use `sprintf()` instead of string interpolation (`"{$var}"`)

## Multi-Database Compatibility - MUST FOLLOW

This package runs on SQLite, MySQL, and PostgreSQL. All migrations and queries must work across all three.

### Migration rules:
- **Index/foreign key names must be ≤ 64 characters** (MySQL limit)
- **`->after('column')` only works on MySQL** — PostgreSQL/SQLite ignore it
- **Use `DB::table()->upsert()` instead of raw `ON CONFLICT`/`ON DUPLICATE KEY`** — the Eloquent method handles dialect differences

### Query rules:
- Date truncation is handled by database adapters (`MySqlAdapter`, `PgsqlAdapter`, `SqliteAdapter`) — never write raw date SQL directly
- Always use the `DateAdapter` interface for any new date-related SQL
- Test with SQLite locally; CI will catch MySQL/PostgreSQL issues

## Writing Pest Tests - MUST FOLLOW

### Test Structure

Tests live in `tests/Unit/` and `tests/Feature/`. All tests use `TestCase` which extends Orchestra Testbench with SQLite in-memory.

### Test Conventions

```php
<?php

declare(strict_types=1);

use Carbon\CarbonImmutable;
use Skylence\StarSchema\StarQuery;
use Skylence\StarSchema\Tests\Fixtures\Order;
use Skylence\StarSchema\Tests\Fixtures\OrderFact;

// Use beforeEach to set up test data and schema
beforeEach(function (): void {
    DB::statement('DROP TABLE IF EXISTS orders');
    DB::statement('CREATE TABLE orders (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        order_number TEXT,
        total REAL DEFAULT 0,
        quantity INTEGER DEFAULT 0,
        customer_id INTEGER DEFAULT 1,
        ordered_at DATE
    )');

    app(StarSchemaRegistry::class)->registerFact(new OrderFact);
});

// Test naming: describe what it does, not what method it calls
it('queries sum per month with gap filling', function (): void {
    Order::insert([
        ['order_number' => 'O-001', 'total' => 100, 'ordered_at' => '2025-01-15'],
        ['order_number' => 'O-002', 'total' => 200, 'ordered_at' => '2025-01-20'],
    ]);

    $results = StarQuery::fact('test_orders')
        ->between(
            CarbonImmutable::create(2025, 1, 1),
            CarbonImmutable::create(2025, 1, 31),
        )
        ->perMonth()
        ->sum('total');

    expect($results)->toHaveCount(1)
        ->and($results[0]->value)->toBe(300.0);
});
```

### Test Rules

1. **Always use `declare(strict_types=1)`**
2. **Use `it()` with descriptive strings** — not `test()`, not `/** @test */`
3. **Use `expect()` fluent assertions** — not `$this->assert*()` PHPUnit style
4. **Chain assertions with `->and()`** to keep related checks together
5. **Use `beforeEach()`** for shared setup, not helper methods per test
6. **Create test tables directly with SQL** since this is a package (no app migrations)
7. **Register facts/dimensions in `beforeEach()`** via `StarSchemaRegistry`
8. **Use fixture classes** in `tests/Fixtures/` for test models and definitions
9. **Use Pest datasets** for parameterized tests (adapters, ranges, etc.)
10. **Avoid mocking** — test against real SQLite database

### Test Coverage Requirements

Every test file MUST include tests beyond the happy path:

1. **Boundary tests** — empty datasets, single row, large ranges, zero values
2. **Edge cases** — gap filling across year boundaries, quarter transitions, empty results
3. **Filter combinations** — single filter, multiple filters, whereIn with arrays
4. **All aggregation types** — sum, avg, count, min, max for each feature
5. **All time grains** — daily, weekly, monthly, quarterly, yearly

### Fixture Models

Test models go in `tests/Fixtures/`. They should be minimal:

```php
// tests/Fixtures/Order.php
final class Order extends Model
{
    protected $guarded = [];
    public $timestamps = false;
}

// tests/Fixtures/OrderFact.php
final class OrderFact implements FactDefinition
{
    public function name(): string { return 'test_orders'; }
    public function sourceModel(): string { return Order::class; }
    public function query(): Builder { return Order::query(); }
    // ...
}
```

## Workflow - MUST FOLLOW

After completing all tasks:

1. **Run Rector + Pint** on all modified PHP files
2. **Run tests** to verify nothing is broken
3. **Run PHPStan** if types or interfaces changed
4. Only then mark work as complete

---
> Source: [skylence-org/laravel-star-schema](https://github.com/skylence-org/laravel-star-schema) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
