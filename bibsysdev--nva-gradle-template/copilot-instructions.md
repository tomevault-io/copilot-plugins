## nva-gradle-template

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this
repository.

## Project Overview

This is a multi-module Java project using Gradle as the build tool. It serves as a template for
NVA (Norwegian National Research Repository) projects and demonstrates modern Java development
practices with comprehensive static analysis, code formatting, and testing.

## Project Structure

- **Root module**: Contains build logic configuration and aggregated tasks
- **build-logic/**: Custom Gradle convention plugins defining shared build configuration
- **common/**: Shared library module with model classes (Book, Chapter, LibraryService)
- **calculator/**: Example application module with math operations, depends on common module

## Key Technologies & Tools

- **Java 23** with Amazon Corretto toolchain
- **Gradle** with custom convention plugins and composite builds
- **JUnit 5** for testing
- **Jacoco** for code coverage (100% coverage required)
- **Spotless** for code formatting (Google Java Format)
- **PMD** for static analysis
- **ErrorProne** for additional compile-time checks

## Essential Commands

### Build & Test

```bash
./gradlew build          # Build all modules, run tests, apply formatting
./gradlew test           # Run all tests
./gradlew clean build    # Clean build from scratch
```

### Individual Module Operations

```bash
./gradlew :common:test          # Test specific module
./gradlew :calculator:build     # Build specific module
```

### Code Quality & Formatting

```bash
./gradlew spotlessApply     # Apply code formatting
./gradlew spotlessCheck     # Check code formatting
./gradlew check            # Run all checks (tests, PMD, coverage)
```

### Coverage Reports

```bash
./gradlew testCodeCoverageReport    # Generate aggregated coverage report
./gradlew showCoverageReport       # Show coverage report location
./gradlew verifyCoverage          # Verify 100% coverage requirement
```

### Static Analysis

```bash
./gradlew pmdMain          # Run PMD on main source
./gradlew pmdTest          # Run PMD on test source
```

### Dependency Management

```bash
./gradlew dependencyUpdates    # Check for dependency updates
./gradlew dependencies        # Show dependency tree
```

## Build Logic Architecture

The project uses a sophisticated build system with custom convention plugins:

### Convention Plugins (build-logic/src/main/groovy/)

- **nva.root-module-conventions**: Root-level tasks, modern JaCoCo report aggregation
- **nva.java-conventions**: Java compilation, testing, static analysis setup
- **nva.formatting-conventions**: Spotless formatting configuration

### Key Build Features

- **Composite builds**: build-logic/ is included as a separate build
- **Version catalog**: gradle/libs.versions.toml manages all dependency versions
- **Modern coverage aggregation**: Uses Gradle's `jacoco-report-aggregation` plugin with automatic
  subproject detection
- **Automatic formatting**: Spotless runs before build and test tasks
- **Multiple static analysis tools**: PMD, ErrorProne, plus standard Java compiler warnings

## Testing Strategy

- **JUnit 5** with parameterized tests
- **Mockito** for mocking
- **Hamcrest** for assertions
- **100% coverage requirement** enforced by Jacoco
- All tests must pass for build to succeed

## Code Quality Standards

- **Google Java Format** with automatic import reordering
- **PMD** rules via config/pmd/ruleset.xml
- **ErrorProne** static analysis during compilation
- **Gradle build scripts** formatted with Greclipse
- **Markdown/config files** formatted with consistent spacing

## Development Workflow

1. Make code changes
2. Run `./gradlew build` (automatically formats and tests)
3. Use `./gradlew showCoverageReport` to verify coverage
4. The build will fail if:
   - Tests fail
   - Coverage drops below 100%
   - Code formatting is incorrect
   - PMD or ErrorProne violations exist

## Module Dependencies

- **calculator** depends on **common**
- Both modules apply **nva.java-conventions**
- Root module applies **nva.root-module-conventions**

## Running Single Tests

```bash
./gradlew :calculator:test --tests "math.CalculatorTest"
./gradlew :common:test --tests "model.LibraryServiceTest"
```

## Important Notes

- Code formatting is automatically applied during build/test
- Coverage verification runs as part of `check` task
- PMD and ErrorProne configured to be strict but not fail builds (warnings only)
- Java toolchain ensures consistent Java 23 usage across environments

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BIBSYSDEV) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
