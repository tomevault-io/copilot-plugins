## claude-sorted-linked-list

> A production-quality PHP library implementing a `SortedLinkedList` data structure.

# SortedLinkedList — PHP Library

## Project overview

A production-quality PHP library implementing a `SortedLinkedList` data structure.
Full specification: [`requirements.md`](./requirements.md).

Key constraints:
- Maintains ascending order at all times (no `sort()`/`usort()` internally).
- Stores either `int` or `string` — type fixed at instantiation, no mixing.
- Strict type safety — no silent coercion, all public APIs fully typed.
- Implements `Countable`, `IteratorAggregate`, `toArray()`.
- Compatible with all currently supported PHP versions (defined in `composer.json`).

---

## Tooling

| Tool | Command | Purpose |
|---|---|---|
| PHPUnit / Pest | `./vendor/bin/pest` | Unit tests |
| PHPStan | `./vendor/bin/phpstan analyse` | Static analysis (level max) |
| Psalm | `./vendor/bin/psalm` | Static analysis (strict mode) |
| PHP-CS-Fixer | `./vendor/bin/php-cs-fixer fix --dry-run` | Code style check |
| Infection | `./vendor/bin/infection` | Mutation testing |

CI runs all of the above on every push and pull request. All steps must pass.

---

## Code standards

- `declare(strict_types=1)` in every PHP file.
- All parameters, return types, and property types explicitly declared — no `mixed`.
- PHPStan level max + Psalm strict: zero errors allowed.
- PSR-12 / PER code style enforced by PHP-CS-Fixer.
- Value objects use `readonly` properties or `readonly class`.
- Custom exceptions extend appropriate SPL base class.
- No `var_dump`, `print_r`, `die`, `exit` in production code.

---

## Automated workflow

Use `/php-implement <task description>` to run the full TDD cycle automatically:

1. **php-pro** implements the feature and writes tests.
2. **php-test-analyst** analyses test coverage and mutation resistance. If critical gaps are found, php-pro adds missing tests and the analysis reruns.
3. **php-code-reviewer** reviews the source code. If issues ≥ 80 confidence are found, php-pro fixes them and review reruns.
4. Done — all agents report clean.

```
/php-implement Add the SortedLinkedList::removeAt method with out-of-bounds guard
```

---

## Agents

This project ships three specialist sub-agents. Invoke them from the `/agents` dialog or via the `Agent` tool.

### `php-pro`

**When to use:** główny agent do pisania kodu PHP — implementacja klas, interfejsów, serwisów, wzorców projektowych.

Co umie:
- PHP 8.3+: `readonly`, enums, Fibers, match, named args, first-class callables
- PSR-12, SOLID, DDD, hexagonalna architektura
- PHPUnit / Pest, PHPStan level 9, Psalm
- Laravel i Symfony patterns

**Typowe wywołanie:**
```
Implement the SortedLinkedList class using php-pro.
```

---

### `php-code-reviewer`

**When to use:** after writing or modifying any PHP class, before committing, or before opening a PR.

What it checks:
- Strict typing and type safety (PHPStan max / Psalm strict compatibility)
- SOLID design, immutability, PHP 8.x idioms (`match`, `readonly`, enums, named args)
- SortedLinkedList-specific contracts: sorting invariant, O(n) complexity, `addAll` transactional behaviour
- PSR-12 / PER compliance

Reports only high-confidence issues (≥ 80/100) — no noise.

**Typical invocation:**
```
Review the current git diff with php-code-reviewer.
```

---

### `php-test-analyst`

**When to use:** after implementing a feature or fixing a bug, before running Infection, or as a pre-PR test coverage check.

What it checks:
- All 9 correctness contracts from `requirements.md` (sorting invariant, type enforcement, duplicates, transactional `addAll`, boundary queries, iteration, custom comparator)
- Mutation resistance against common Infection mutators (comparison, arithmetic, return-value, method-removal)
- Test quality: behaviour-focused assertions, data providers, edge case isolation

Outputs critical gaps with minimal PHPUnit/Pest examples that would kill surviving mutants.

**Typical invocation:**
```
Analyse test coverage for the recently added SortedLinkedList methods with php-test-analyst.
```

---

## Definition of Done

A change is ready to merge when:

- [ ] All CI steps pass (tests, PHPStan, Psalm, PHP-CS-Fixer, Infection)
- [ ] `php-code-reviewer` reports no issues ≥ 80 confidence
- [ ] `php-test-analyst` reports no critical gaps (9–10)
- [ ] Mutation score meets the threshold defined in `infection.json`
- [ ] No type inspection warnings in IDE

---
> Source: [zawiszaty/claude-sorted-linked-list](https://github.com/zawiszaty/claude-sorted-linked-list) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-08 -->
