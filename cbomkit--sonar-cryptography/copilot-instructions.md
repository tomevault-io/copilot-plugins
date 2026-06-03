## sonar-cryptography

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **Sonar Cryptography Plugin (CBOMkit-hyperion)** - a SonarQube plugin that detects cryptographic assets in source code and generates Cryptographic Bill of Materials (CBOM) in CycloneDX format. Part of the [CBOMKit](https://github.com/cbomkit) toolset.

**Supported languages/libraries:**
- Java: JCA (100%), BouncyCastle light-weight API (100%)
- Python: pyca/cryptography (100%)

## Build Commands

```bash
# Build with tests
mvn clean package

# Build without tests
mvn clean package -DskipTests

# Build specific module
mvn clean package -pl java

# Format code (Google Java Format, AOSP style)
mvn spotless:apply

# Check formatting
mvn spotless:check

# Check code style
mvn checkstyle:check
```

## Testing

```bash
# Run all tests
mvn test

# Run tests for specific module
mvn test -pl java
mvn test -pl mapper
mvn test -pl engine

# Run specific test class
mvn test -Dtest=SimpleGuidelineTest

# Run specific test method
mvn test -Dtest=SimpleGuidelineTest#testBasicAssertion
```

**Testing Framework:** JUnit 5 + AssertJ + SonarQube Test Fixtures

**Detection rule tests:**
- Extend `TestBase` (in `java/src/test/java/com/ibm/plugin/TestBase.java`)
- Use `CheckVerifier` from SonarQube
- Test files (actual code to analyze) go in `src/test/files/`, not `src/test/java/`
- Implement `asserts()` method to verify detection store values and translated nodes

## Architecture

Multi-module Maven project (Java 17):

```
sonar-cryptography-plugin/    # Main SonarQube plugin entry point
├── CryptographyPlugin.java   # Plugin registration
├── OutputFileJob.java        # CBOM output handler
engine/                       # Core detection engine
├── detection/                # DetectionStore, Finding classes
├── rule/                     # IDetectionRule interface
java/                         # Java language support (JCA, BouncyCastle)
├── rules/detection/          # Java detection rules
python/                       # Python language support (pyca/cryptography)
mapper/                       # Translation layer to CBOM model
├── model/                    # Core data model (Algorithm, Key, Protocol, etc.)
├── ITranslator.java          # Main translation interface
enricher/                     # Adds algorithm details to findings
output/                       # CBOM/CycloneDX generation
common/                       # Shared utilities
rules/                        # Shared rule definitions
```

**Data flow:** Source code → Language module (AST) → Engine (detection) → Mapper (translation) → Enricher → Output (CBOM)

## Code Style

- **Formatting:** Google Java Format (AOSP style) via Spotless - runs on `mvn package`
- **License:** Apache 2.0 header required in all Java files (applied by Spotless)
- **Checkstyle rules:** No unused imports, camelCase lambda params, max 5 boolean operators, private utility constructors, @Override required

## Running with SonarQube

```bash
docker-compose up  # Starts PostgreSQL + SonarQube
```

Plugin JAR is built to `sonar-cryptography-plugin/target/` and copied to `.SonarQube/plugins/`.

## Key Documentation

- `docs/LANGUAGE_SUPPORT.md` - Extending for new languages/libraries
- `docs/DETECTION_RULE_STRUCTURE.md` - Writing detection rules
- `docs/TROUBLESHOOTING.md` - Testing configuration guide

---
> Source: [cbomkit/sonar-cryptography](https://github.com/cbomkit/sonar-cryptography) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
