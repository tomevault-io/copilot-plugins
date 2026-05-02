## spec-driven-workshop

> This file provides guidance to AI coding agents when working with code in this repository.

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Project Overview

A spec-driven development workshop teaching engineers about agent-based development and lightweight formal methods through Java exercises. The core demonstrates JML (Java Modeling Language) specifications, property-based testing, and static analysis.

## Build Commands

All commands go through Make. The default target is `check`.

```bash
make tooling       # One-time: download JDK 21, OpenJML, Z3 (requires internet)
make check         # Format → compile (with Errorprone) → SpotBugs → Spotless check → tests
make check-jml     # check + OpenJML formal verification (extended static checking)
make test          # Tests only (JUnit 5 + jqwik + Fray via Surefire)
make format        # Auto-format with Spotless (Palantir style)
make clean         # Delete build artifacts
make jshell        # Interactive JShell with full classpath
make repl          # Clojure REPL with full classpath
make uberjar       # Fat JAR via maven-shade-plugin
make run           # Execute the uberjar
```

To run a single test class: `JAVA_HOME=$(make -s -p | grep '^JHOME' | cut -d= -f2) mvn test -Dtest=AppTest`

## Verification Pipeline

`make check` chains: Spotless format → Errorprone (extended type checking) → SpotBugs + FindSecurityBugs → JUnit 5/jqwik/Fray tests. `make check-jml` adds OpenJML's ESC pass using the Z3 SMT solver against JML annotations in source.

## Code Style

Palantir Java Format (modified Google Style) enforced by Spotless. Run `make format` before committing.

## Architecture

- **Java 21** compiled via project-local JDK in `tooling/jdk-21.0.7+6/`
- **Maven** (`pom.xml`) manages dependencies and plugins; profiles: `errorprone`, `openjml`, `dev`
- Source: `src/main/java/com/kevel/` — application code with inline JML specs (`/*@ ... @*/`)
- Tests: `src/test/java/com/kevel/` — JUnit 5 unit tests + jqwik property-based tests
- OpenJML binary: `tooling/openjml/openjml` — invoked directly for `check-jml-only`

## JML Specifications

JML specs live inline in Java source as `/*@ ... @*/` comments. Key constructs: `requires` (preconditions), `ensures` (postconditions), `pure` (no side effects). Use `// @ skipesc skiprac` to exclude methods from verification. OpenJML catches issues like integer overflow that tests may miss.
For more details see the [OpenJML Tuturial](https://www.openjml.org/tutorial/)

## Sandboxed Development

- **macOS**: `./orchard.sh` — Docker container (Ubuntu 22.04) with all tooling pre-installed
- **Linux**: `./bw-opencode` — Bubblewrap sandbox
- Git push/pull must be done on the host, not inside sandboxes

---
> Source: [adzerk/spec-driven-workshop](https://github.com/adzerk/spec-driven-workshop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
