## openapi-diff

> This file is written for agentic coding tools working in this repo. It captures the

# Agent Notes for this Repository (openapi-diff)
This file is written for agentic coding tools working in this repo. It captures the
project's build/test/format commands and the house style that the CI and formatter
expect.

## At-a-glance
- Tech: multi-module Maven (Java), source/target = 1.8 (see `pom.xml`).
- Modules: `core/` (library), `cli/` (shaded CLI jar), `maven/` (Maven plugin),
  `maven-example/` (example usage; install/deploy skipped).
- CI runs `./mvnw ... verify` on JDK 8/11/17/21 (see `.github/workflows/*.yml`).
- Formatting: `com.coveo:fmt-maven-plugin` (see `pom.xml`).

## Cursor / Copilot rules
- Cursor rules: none found (no `.cursor/rules/` and no `.cursorrules`).
- Copilot rules: none found (no `.github/copilot-instructions.md`).

## Commands
Use the Maven wrapper (`./mvnw`) when possible to match CI.

### Build
```bash
# compile + run unit tests (all modules)
./mvnw test

# full CI-style verification (unit tests + packaging)
./mvnw verify

# clean build
./mvnw clean verify

# build just one module (and its deps)
./mvnw -pl core -am verify
./mvnw -pl cli -am package

# CI-style (adds some output + jacoco report)
./mvnw -V -B -ntp -ff verify jacoco:report
```

Notes:
- For faster local iteration, add `-DskipTests=true` when appropriate.

### Format / "lint"
```bash
# auto-format Java sources
./mvnw com.coveo:fmt-maven-plugin:format

# if the plugin supports it, fail if formatting differs (use in CI-like loops)
./mvnw com.coveo:fmt-maven-plugin:check
```

Git hook:
- A local pre-commit hook may run formatting (`.git/hooks/pre-commit` runs
  `./mvnw com.coveo:fmt-maven-plugin:format`). Don't rely on it; run format
  explicitly when you change Java files.

### Tests
```bash
# run all tests
./mvnw test

# run tests for a single module
./mvnw -pl core test
./mvnw -pl maven test

# run a single test class (Surefire)
./mvnw -pl core -Dtest=OpenApiDiffTest test

# run a single test method (JUnit 5 via Surefire)
./mvnw -pl core -Dtest=OpenApiDiffTest#testDiff test
```

Notes:
- Test classes must match Surefire includes (convention here is `*Test.java`).
- If you only changed `cli/`, tests likely live in `core/`; run `./mvnw -pl core test`.

### Docker-oriented build
The `docker` Maven profile disables tests/javadoc/sources and sets `fmt.skip`.

```bash
./mvnw -Pdocker package
docker build -t local-openapi-diff .
```

### Running the CLI from source

```bash
# build shaded jar
./mvnw -pl cli -am package

# run (jar name includes version; use the *-all classifier)
java -jar cli/target/openapi-diff-cli-*-all.jar --help
```

## Code style guidelines
### Formatting
- Always run `./mvnw com.coveo:fmt-maven-plugin:format` after editing Java.
- Don't fight the formatter; structure code so it reads well after formatting.
- Keep lines/blocks simple; prefer extracting helper methods over deeply nested
  ternaries or chained calls.

### Imports
- Let the formatter manage import ordering.
- Avoid wildcard imports.
- In tests, static imports are common and preferred for AssertJ:
  `import static org.assertj.core.api.Assertions.assertThat;`

### Java language level
- Code must compile with Java 8 (no `var`, records, text blocks, etc.).
- Avoid APIs added after Java 8 even if they compile on newer JDKs.

### Naming and structure
- Packages stay under `org.openapitools.openapidiff`.
- Classes: `PascalCase`; methods/fields: `lowerCamelCase`.
- Constants: `UPPER_SNAKE_CASE` and `static final`.
- Prefer "Diff" suffix for comparison components (`*Diff`, `*DiffResult`).
- Tests: `*Test` for unit tests; use descriptive test method names.

### Types, nulls, and collections
- Be explicit about ownership/mutability. Prefer returning empty collections over
  `null` when you control the API.
- When interacting with Swagger/OpenAPI models, expect nullable fields and guard
  accordingly.
- Prefer `final` for local variables/fields when it improves readability.

### Error handling and logging

- Library code: throw useful runtime exceptions for invalid input (see `OpenApiCompare.notNull(...)`).
- Avoid swallowing exceptions silently; log or rethrow.
- CLI / Maven plugin: prefer user-actionable messages; use SLF4J; use `MojoExecutionException`/`MojoFailureException`.

### Testing conventions
- Frameworks: JUnit Jupiter (JUnit 5) + AssertJ.
- Keep tests deterministic; avoid depending on network.
- Put fixtures in `*/src/test/resources/` and reference them in a way that works
  when Maven runs from the module directory.
- When adding behavior, add a focused test in `core/` unless the change is truly
  CLI-only or Maven-plugin-only.

## Repository hygiene
- Line endings: contributors recommend `git config core.autocrlf input`
  (see `CONTRIBUTING.md`).
- Prefer small, reviewable changes; add tests with fixes.
- Do not commit generated artifacts (`target/`, shaded jars, etc.).

---
> Source: [OpenAPITools/openapi-diff](https://github.com/OpenAPITools/openapi-diff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
