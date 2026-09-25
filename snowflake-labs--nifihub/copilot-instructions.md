## nifihub

> NiFi Hub coding conventions and build instructions


## Project Overview

NiFi Hub is a repository of independently-versioned Apache NiFi extension bundles and versioned flow definitions.
All Java code uses package `com.snowflake.nifihub`.

## Building

NiFi Hub uses Maven. Never use `javac` directly. Each bundle builds independently:

```bash
./mvnw clean verify -Pcontrib-check -f <bundle-path>/pom.xml
```

The root POM has no `<modules>` block. Do NOT run `mvn install` from root expecting to build all bundles.

## Code Style

1. Mark all possible variables `final` (method arguments, local variables, member variables, catch variables).
2. No star imports. Import each class individually.
3. No underscores in class names, variables, or filenames.
4. Line width: up to 200 characters.
5. 4-space indentation, no tabs.
6. Use SLF4J loggers, never `System.out.println`.
7. Use `.formatted()` for string formatting instead of `+` concatenation.
8. Include Apache 2.0 license header in all source files.
9. Avoid creating trivial 1-2 line private methods called only once.
10. Private/helper methods should not appear before the first public/protected method that calls them.

## Testing

- Unit tests use `nifi-mock` framework via `TestRunner` from `TestRunners.newTestRunner()`.
- Test classes are named `Test<ClassName>.java`.
- Code coverage must be >= 80%.
- Never use `assertDoesNotThrow`. Just call the method directly.
- Never add `// Given`, `// When`, `// Then` comments.
- Use JUnit assertions (never the `assert` keyword).

## Package Naming

Java packages follow: `com.snowflake.nifihub.<category>.<bundlename>`

Example: `com.snowflake.nifihub.example` for the example bundle.

## Bundle Structure

Every bundle follows this structure:
```
nifi-<name>-bundle/
├── pom.xml            (packaging: pom, parent: nifihub-parent)
├── SKILL.md           (agent-readable bundle documentation)
├── nifi-<name>-processors/
│   ├── pom.xml        (packaging: jar)
│   └── src/
└── nifi-<name>-nar/
    └── pom.xml        (packaging: nar)
```

## Ending Conditions

Before considering a task complete:
1. `./mvnw clean verify -Pcontrib-check -f <bundle>/pom.xml` passes
2. Code coverage >= 80%
3. SKILL.md is present and up-to-date
4. All code follows the style rules above

---
> Source: [Snowflake-Labs/nifihub](https://github.com/Snowflake-Labs/nifihub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
