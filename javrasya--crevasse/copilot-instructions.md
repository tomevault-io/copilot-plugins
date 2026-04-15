## crevasse

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Crevasse is a data migration tool for Apache Iceberg tables, implemented as a Gradle plugin. It uses Groovy DSL scripts to perform schema migrations and can auto-generate migration scripts from Avro schema changes.

## Build Commands

```bash
./gradlew clean build              # Full build with tests
./gradlew build -x test            # Build without tests
./gradlew publishToMavenLocal      # Publish to local Maven repository
./gradlew test                     # Run all tests
./gradlew :crevasse-iceberg:test   # Run tests for specific module
```

## Module Architecture

```
crevasse/
├── crevasse-plugin/          # Gradle plugin entry point (Groovy)
├── crevasse-iceberg/         # Core migration logic (Java)
├── crevasse-bundled/         # Shaded dependencies (Guava relocation)
└── examples/                 # Example project demonstrating usage
```

### crevasse-plugin
Main Gradle plugin implementing `Plugin<Project>`. Key classes:
- `CrevasseMigrationPlugin.groovy` - Registers extension and tasks
- `RootExtension.groovy` - Plugin DSL configuration
- `IcebergHandler.groovy`, `GlueCatalogHandler.groovy`, `HadoopCatalogHandler.groovy` - Catalog configurations
- `MigrationExecutorTask.groovy` - Executes migration scripts
- `MigrationScriptGeneratorTask.groovy` - Generates migrations from Avro schemas

### crevasse-iceberg
Core Java module containing:
- `MigrationExecutor.java` - Orchestrates migrations, tracks state in Iceberg table properties
- `MigrationScriptGenerator.java` - Compares Avro schemas, generates Groovy migration scripts
- `MigrationContext.java` - Transaction context for schema changes
- `MigrationBaseScript.java` - Base class for Groovy DSL scripts
- `ops/` - Table operations (AddColumnsOps, RemoveColumnOps, ModifyColumnOps, etc.)
- `schema/` - Type system (Column, ColumnType, PrimitiveColumnType, ListColumnType, etc.)

## Plugin Usage

The plugin provides two Gradle tasks:
- `./gradlew generateMigrationScripts` - Auto-generate migrations from Avro schema changes
- `./gradlew applyMigrations` - Apply pending migrations to Iceberg tables

Migration scripts are stored in `scriptDir/namespace/tablename/migration_<number>.groovy`.

Migration state is tracked in Iceberg table properties:
- `crevasse.migration.state` - Latest applied migration number
- `crevasse.migration.last-applied-at` - Timestamp of last migration

## Key Dependencies

- Apache Iceberg 1.5.1
- Groovy 5.0.3 (crevasse-iceberg), Gradle's internal Groovy (crevasse-plugin)
- Apache Avro 1.11.1
- Hadoop 3.2.0
- Spock 2.4
- Java 11 (source compatibility)
- Gradle 9.2.1
- Lombok 1.18.42

## MCP and Tool Usage

When looking up library versions, compatibility info, or documentation:
1. **Use Context7 first** - Call `mcp__context7__resolve-library-id` then `mcp__context7__get-library-docs` before resorting to web search
2. Only use WebSearch if Context7 doesn't have the needed information

## Testing

Uses Spock framework for Groovy specs and JUnit 5 for Java tests. Test files located in:
- `/crevasse-iceberg/src/test/groovy/`
- `/crevasse-plugin/src/test/groovy/`

## Example Project

See `/examples/crevasse-iceberg-example/` for a working demonstration:
1. Run `./gradlew publishToMavenLocal` from root
2. In example project: `./gradlew generateMigrationScripts`
3. In example project: `./gradlew applyMigrations`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/javrasya)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/javrasya)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
