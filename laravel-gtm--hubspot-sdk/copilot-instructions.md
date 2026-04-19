## hubspot-sdk

> All PHP packages in this project MUST pass PHPStan analysis at **level 8**. This is non-negotiable.


# PHPStan Level 8 Rules

All PHP packages in this project MUST pass PHPStan analysis at **level 8**. This is non-negotiable.

## What Level 8 Enforces

Level 8 is cumulative — it includes everything from levels 0–7 plus its own checks:

| Level | What It Adds |
|-------|-------------|
| 0 | Basic checks: unknown classes, functions, methods on `$this`, wrong argument counts, undefined variables |
| 1 | Possibly undefined variables, unknown magic methods/properties (`__call`, `__get`) |
| 2 | Unknown methods on all expressions (not just `$this`), PHPDoc validation |
| 3 | Return type checking, types assigned to properties |
| 4 | Basic dead code detection: always-false `instanceof`, dead `else` branches, unreachable code |
| 5 | Argument types passed to methods and functions |
| 6 | Missing typehints |
| 7 | Partially wrong union types (methods that only exist on some types in a union) |
| **8** | **Calling methods and accessing properties on nullable types** |

## Configuration

The `phpstan.neon` or `phpstan.neon.dist` file MUST specify level 8:

```neon
parameters:
    level: 8
    paths:
        - src
        - tests
```

## Key Rules for Writing Level 8 Compliant Code

### Always Null-Check Before Accessing Nullable Types

Level 8's primary addition is strict nullable type checking. Never call methods or access properties on a value that could be `null`.

```php
// BAD — PHPStan level 8 error: Cannot call method format() on DateTimeInterface|null
function getDate(?DateTimeInterface $date): string
{
    return $date->format('Y-m-d');
}

// GOOD — null check before access
function getDate(?DateTimeInterface $date): string
{
    if ($date === null) {
        return 'N/A';
    }

    return $date->format('Y-m-d');
}
```

### Use Strict Return Types

```php
// BAD — missing return type, and nullable return not handled by callers
function findUser(int $id)
{
    return User::find($id);
}

// GOOD — explicit nullable return type signals callers must handle null
function findUser(int $id): ?User
{
    return User::find($id);
}
```

### Type-Narrow Union Types Properly

```php
// BAD — method only exists on one type in the union
/** @param string|int $value */
function process(string|int $value): string
{
    return $value->toString();
}

// GOOD — narrow the type first
function process(string|int $value): string
{
    if (is_int($value)) {
        return (string) $value;
    }

    return $value;
}
```

### Provide Complete PHPDoc for Complex Types

```php
// BAD — PHPStan cannot infer array shape
function getConfig(): array
{
    return ['host' => 'localhost', 'port' => 3306];
}

// GOOD — PHPDoc describes the array shape
/** @return array{host: string, port: int} */
function getConfig(): array
{
    return ['host' => 'localhost', 'port' => 3306];
}
```

### Declare Strict Types in Every File

```php
<?php

declare(strict_types=1);
```

### Use Baseline for Legacy Code Only

If adopting level 8 in an existing codebase, generate a baseline to avoid blocking work:

```bash
vendor/bin/phpstan analyse --generate-baseline
```

Then work to eliminate baseline entries over time. **New code must never add to the baseline.**

### Common Mistakes

- Forgetting to null-check return values from methods like `find()`, `first()`, `get()` that may return `null`
- Using `@var` annotations to suppress errors instead of fixing the actual type issue
- Adding `mixed` types to avoid writing proper type declarations — level 8 still requires handling nullable access
- Ignoring PHPStan errors with `@phpstan-ignore-next-line` instead of fixing the code
- Not running `composer analyse` before committing — always verify locally

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/laravel-gtm) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
