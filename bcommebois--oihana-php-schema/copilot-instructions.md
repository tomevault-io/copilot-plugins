## oihana-php-schema

> This document outlines the conventions and technical details for the Gemini assistant to follow in this project.

# Gemini Project Guidelines

This document outlines the conventions and technical details for the Gemini assistant to follow in this project.

## 1. Language and Communication

- **User Interaction:** The primary language for interacting with the assistant (prompts) is **French**.
- **Generated Content:** All generated code, documentation, tests, and commit messages must be written in **English**.

## 2. Project Context: oihana/php-schema

- **PHP Version:** Requires PHP >= 8.4.
- **Purpose:** This is a PHP library that provides an object-oriented, strongly-typed implementation of the Schema.org vocabulary with JSON-LD serialization and recursive hydration.
- **Core Dependencies:** It relies on `oihana/php-core` and `oihana/php-reflect`.
- **Namespaces (PSR-4):**
    - `org\schema\` → `src/org/schema`
    - `fr\ooop\` → `src/fr/ooop` (for extensions like Pagination)

### Core Architecture

- **Base Entity:** Most classes extend `org\schema\Thing`.
- **Core Logic:** Functionality is often provided by traits, especially `ThingTrait`, which handles reflection-based hydration from arrays and `JsonSerializable` logic.
- **Property Constants:** Property names **must** be referenced via constants from `org\schema\constants\Prop` (e.g., `Prop::NAME`, `Prop::ADDRESS`). This is critical for avoiding typos and aligning with the internal reflection mechanisms.
- **Serialization:** The `jsonSerialize()` method automatically includes `@type` and `@context` and omits null values.
- **Hydration:** Deep/recursive hydration relies on utilities from the `oihana/php-reflect` package.

## 3. PHPDoc Standards

All PHPDoc blocks must adhere to the following format for examples.

- **Example Tag:** Code examples must be placed within an `@example` tag.
- **Markdown Format:** The code itself must be inside a Markdown code block with the language specified (e.g., `php`).

### Correct Format

```php
/**
 * Describes a Person, living or dead.
 *
 * @example
 * ```php
 * use org\schema\constants\Prop;
 * use org\schema\Person;
 *
 * $person = new Person([
 *     Prop::NAME => 'John Doe',
 *     Prop::EMAIL => 'john.doe@example.com'
 * ]);
 *
 * echo $person->get(Prop::NAME); // "John Doe"
 * ```
 */
class Person extends Thing
{
    // ...
}
```

## 4. Common Commands

- **Install dependencies:** `composer install`
- **Run all tests:** `composer test`
- **Generate API docs:** `composer doc`

## 5. Testing

- **Framework:** PHPUnit 12. Configuration is in `phpunit.xml` (TestDox is enabled).
- **Run all tests:** `composer test` or `./vendor/bin/phpunit`
- **Run a single test file:** `composer test ./tests/org/schema/ThingTest.php`
- **Run a single test class:** `./vendor/bin/phpunit --filter ThingTest`
- **Run a single test method (by regex):` `./vendor/bin/phpunit --filter '/::test_serialization$/'`
- **Guideline:** New features or bug fixes must be accompanied by corresponding unit tests.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BcommeBois) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
