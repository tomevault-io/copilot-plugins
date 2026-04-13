## databricks-scala-ground-truth-corpus

> This file provides guidance to Claude Code when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Project Overview

Scala ground truth corpus for Databricks course content. Implements production-ready Spark, MLlib, Delta Lake, and Structured Streaming patterns with comprehensive test coverage using Popperian falsification methodology.

## Build and Development Commands

```bash
# Build
sbt compile                    # Compile main sources
sbt "Test / compile"           # Compile test sources

# Testing
sbt test                       # Run all tests
sbt "testOnly *DataFrameOps*"  # Run specific test class

# Formatting
scalafmt                       # Format all Scala files
scalafmt --check               # Check formatting (CI)

# Coverage
sbt clean coverage test coverageReport  # Generate coverage report
# Report at: target/scala-2.12/scoverage-report/index.html

# Quality tiers
make tier1                     # On-save: fmt-check + compile
make tier2                     # Pre-commit: tier1 + test
make tier3                     # Pre-push: tier2 + coverage
make tier4                     # CI/CD: tier3 + doc

# Documentation
sbt doc                        # Scaladoc API docs
make book                      # mdBook documentation
make book-serve                # Serve at localhost:3000
```

## Architecture

### Domain Modules

| Domain | Package | Purpose |
|--------|---------|---------|
| **Spark Fundamentals** | `com.paiml.databricks.spark` | DataFrame ops, SQL, UDFs, window functions, joins |
| **ML/MLflow** | `com.paiml.databricks.ml` | Feature engineering, pipelines, evaluation, tuning |
| **Delta Lake** | `com.paiml.databricks.delta` | CRUD, MERGE, time travel, CDC, schema evolution |
| **Streaming** | `com.paiml.databricks.streaming` | Stream processing, windowed aggregation, joins |

### Key Dependencies

- **Spark 3.5.4** (Scala 2.12)
- **Delta Lake 3.3.0**
- **ScalaTest 3.2.19** + **ScalaCheck 1.18.1**
- **sbt-scoverage 2.2.2** (95% minimum coverage)

## Quality Standards

- **95% minimum test coverage** (enforced by sbt-scoverage)
- **Zero scalac warnings** (via `-Xlint`, `-Ywarn-*` flags)
- **ScalaCheck property tests** for all pure functions
- **SharedSparkSession** trait for integration tests

## Commit Format

```
feat|fix|test|docs|refactor: message (Refs DSGTC-XXX)

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
```

## Testing Patterns

- Unit tests: Pure function testing without SparkSession
- Integration tests: Use `SharedSparkSession` trait (local[2] mode)
- Property tests: ScalaCheck generators for input validation
- Delta tests: Temp directories via `Files.createTempDirectory`
- Streaming tests: Memory sink + `processAllAvailable()` pattern

## File Template

```scala
package com.paiml.databricks.{domain}

import org.apache.spark.sql.{DataFrame, SparkSession}

/** Brief description of module purpose.
  *
  * Detailed explanation of patterns and usage.
  */
object ModuleName {

  /** Method documentation. */
  def methodName(df: DataFrame): DataFrame = {
    // Implementation
  }
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/paiml)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/paiml)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
