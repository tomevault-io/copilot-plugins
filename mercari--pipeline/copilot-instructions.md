## pipeline

> transforms:

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Mercari Pipeline is a configuration-driven data pipeline framework built on Apache Beam. It enables running various data pipelines on Cloud Dataflow, Apache Spark, and Apache Flink without writing code - just YAML/JSON configuration files.
It also includes a Server feature as an auxiliary tool to create, debug, and deploy pipelines.

## Documents (`src/main/resources/server/docs/`)

1. `module/` - Pipeline module config
2. `options/` - Pipeline options config
3. `system.md` - Pipeline system config
4. `server/` - Pipeline server

### Documents for Developer (`docs/developer/`)

1. `docs/developer/pipeline/pipeline/` - Document for the Pipeline itself
2. `docs/developer/pipeline/server/` - Document for Pipeline Server

## Architecture

### Entry Point
- `MPipeline.java` - Main entry point that loads config, creates Pipeline, and applies sources/transforms/sinks

### Configuration System (`config/`)
- `Config.java` - Parses YAML/JSON config from GCS, local files, base64, or Parameter Manager
- Supports FreeMarker templating with `${args.varName}` syntax for runtime variable substitution
- Config can import other config files via `system.imports`

### Module System (`module/`)
The pipeline is built from three module types, discovered via classpath scanning with `@Module` annotations:

**Sources** (`module/source/`) - Data input:
- BigQuery, Spanner, Bigtable, Datastore, Firestore
- PubSub, Kafka (streaming)
- JDBC, Storage (files), HTTP, Drive

**Transforms** (`module/transform/`) - Data processing:
- SelectTransform - Field selection/transformation with expressions
- AggregationTransform - Grouping and aggregation
- BeamSQLTransform - SQL-based processing
- OnnxTransform - ML inference
- PartitionTransform, LookupTransform, etc.

**Sinks** (`module/sink/`) - Data output:
- BigQuery, Spanner, Bigtable, Datastore, Firestore
- PubSub, Storage (files), JDBC, Iceberg

### Core Module Classes
- `Module.java` - Base class for all modules
- `MCollection.java` - Wrapper around PCollection with schema metadata
- `MCollectionTuple.java` - Container for multiple named MCollections
- `MElement.java` - Universal data element handling Avro, Row, Entity, Struct types
- `Schema.java` - Unified schema representation

### Utilities (`util/`)
- `schema/` - Schema utilities for Avro, Row, Entity, Struct, Proto conversions
- `pipeline/` - Pipeline utilities (Filter, Select, Aggregation, Query)
- `cloud/` - Cloud service utilities (GCS, BigQuery, Spanner, PubSub)
- `domain/` - Domain-specific utilities (ML, text analysis, JDBC)

### Server (`server/`)
- `PipelineApiServer.java` - REST API server for pipeline validation
- `PipelineMcpStreamableServer.java` - MCP server for AI integration

## Adding New Modules

1. Create class in appropriate package (`source/`, `transform/`, or `sink/`)
2. Extend `Source`, `Transform`, or `Sink` base class
3. Add `@Module(name = “modulename”)` annotation
4. Implement `expand()` method returning `MCollectionTuple`
5. Module is auto-discovered via classpath scanning

## Configuration File Structure

```yaml
system:
  args:
    myVar: “value”
  imports:
    - base: “gs://bucket/”
      files: [“common.yaml”]

sources:
  - name: input1
    module: bigquery
    parameters:
      query: “SELECT * FROM table”

transforms:
  - name: process1
    module: select
    inputs: [input1]
    parameters:
      fields: [...]

sinks:
  - name: output1
    module: spanner
    inputs: [process1]
    parameters:
      projectId: myproject
      instanceId: myinstance
      databaseId: mydatabase
      table: mytable
```

## Build Commands

```bash
# Build and create FlexTemplate container (default: Dataflow runner)
mvn clean package -DskipTests -Dimage={region}-docker.pkg.dev/{project}/{repo}/dataflow:latest

# Build for local execution (DirectRunner)
mvn clean package -DskipTests -Pdirect -Dimage=“{region}-docker.pkg.dev/{project}/{repo}/direct”

# Build API server
mvn clean package -DskipTests -Pserver -Dimage=“{region}-docker.pkg.dev/{project}/{repo}/server”

# Run all tests
mvn test

# Run a single test class
mvn test -Dtest=ConfigTest

# Run a single test method
mvn test -Dtest=ConfigTest#testMethodName
```

## Maven Profiles

- `dataflow` (default) - Cloud Dataflow runner
- `direct` - Local DirectRunner for testing
- `server` - Pipeline API server (WAR)
- `prism` - PrismRunner for portable execution
- `flink` - Apache Flink runner
- `spark` - Apache Spark runner

## Key Dependencies

- Java 21
- Apache Beam 2.70.0
- Google Cloud Platform SDKs (BigQuery, Spanner, Datastore, etc.)
- Jetty EE11 12 (For Server)

---
> Source: [mercari/pipeline](https://github.com/mercari/pipeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
