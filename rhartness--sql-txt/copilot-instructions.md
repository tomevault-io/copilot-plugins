## sql-txt

> SQL.txt project purpose, layered architecture, and key interfaces


# SQL.txt Project Overview

**SQL.txt** is a lightweight, embeddable .NET database engine that persists schemas, metadata, and row data in human-readable text files.

## Goals

1. **Learning tool** — Readable, inspectable database engine for understanding storage engines, query parsing, CRUD semantics, metadata, schema evolution, and indexing.
2. **Practical embedded datastore** — Minimal dependency, NuGet-installable local database requiring only a package reference, writable directory, and optional CLI/consumer tooling.

## Layered Architecture

- **Contracts** — Shared models, interfaces, exceptions (no dependencies)
- **Core** — Domain primitives, validation, utilities (depends on Contracts)
- **Storage** — On-disk persistence (depends on Contracts, Core)
- **Parser** — SQL-like command parsing (depends on Contracts, Core)
- **Engine** — Execution layer (depends on Contracts, Core, Storage, Parser)
- **Cli / SampleApp** — Consumer applications (depend on Engine)

## Key Interfaces

- `IDatabaseEngine` — Execute commands, open databases, RebalanceTableAsync
- `ICommandParser` — Parse text to command objects
- `ISchemaStore` — Schema read/write
- `ITableDataStore` — Row data read/write
- `IMetadataStore` — Metadata persistence
- `IFileSystemAccessor` — File I/O abstraction
- `IRowSerializer` / `IRowDeserializer` — Row format handling

## Project Responsibilities

| Project | Responsibility |
|---------|----------------|
| SqlTxt.Contracts | Table/column definitions, command contracts, result contracts, exceptions |
| SqlTxt.Core | Data types, validation, identifier normalization, serialization helpers |
| SqlTxt.Storage | Directory structure, schema/metadata/row file I/O, format versioning |
| SqlTxt.Parser | Tokenizer, parser, command AST/DTO, error reporting |
| SqlTxt.Engine | Create db/table, insert, select, update, delete, table scans |
| SqlTxt.ManualTests | Concurrency, sharding (fixed and VARCHAR), performance; **add manual tests for each major feature** when applicable (register in `ManualTestProgram.cs`) |

## Build Types

- **CLI exe** — SqlTxt.Cli
- **NuGet API DLL** — SqlTxt package; async API for embedding
- **Service** — SqlTxt.Service (Phase 2)

## API and Concurrency

- All DB functions via async API (ExecuteAsync, ExecuteQueryAsync, OpenAsync)
- Lock coordinator: Phase 1 single-writer; Phase 2 multi-reader + WITH (NOLOCK)

## Phase Status

| Phase | Status |
|-------|--------|
| Stage 0 | Done |
| Phase 1 | Done |
| Phase 2 | Done |
| Phase 3 | Done |
| Phase 3.5 | Done |
| Phase 4 | Next |
| CTE Phase | Planned |
| Phase 5 | Planned |
| Phase 6 | Planned |
| Phase 7 | Planned (Statistics) |

**Current focus:** Phase 4 (query enrichment). Implementation and efficiency bars: [Phase4_Implementation_Plan.md](../../docs/plans/Phase4_Implementation_Plan.md) and **Phase4_0x_*.md** sub-plans. See [02_Post_Phase3_Features.md](../../docs/specifications/02_Post_Phase3_Features.md) and [00-sql2023-compliance-roadmap.md](../../docs/roadmap/00-sql2023-compliance-roadmap.md). Phase 3.5 is complete: [Phase3_5_Storage_Efficiency_Plan.md](../../docs/plans/Phase3_5_Storage_Efficiency_Plan.md).

**Strategic:** SQL:2023 alignment per phase; dual storage backends (text | binary); configurable sharding (20MB default); rebalance API; Knuth-style efficiency; statistics-ready design.

## Storage Layout

- Root folder = database name
- `db/` — Database properties (manifest; defaultMaxShardSize 20MB; storageBackend: "text" | "binary")
- `Tables/` — User tables; each folder has `<TableName>.txt` or `.bin` (per storageBackend), `_STOC.txt`/`.bin`, `_PK.txt`/`.idx`, `_FK_*.txt`/`.idx`, `_INX_*.txt`/`.idx`
- `~System/` — System metadata (prefix `~` = engine-managed)
- `Views/`, `Procedures/`, `Functions/` — Phase 6 (views, stored procedures, functions)

## Reference

See [docs/specifications/01_Initial_Creation.md](docs/specifications/01_Initial_Creation.md) for full specification. See [docs/specifications/02_Post_Phase3_Features.md](docs/specifications/02_Post_Phase3_Features.md) for post-Phase 3 roadmap. For performance and efficiency requirements, see [docs/architecture/10-performance-and-efficiency.md](docs/architecture/10-performance-and-efficiency.md). SQL:2023 mapping: [docs/architecture/11-sql2023-mapping.md](docs/architecture/11-sql2023-mapping.md). Full SQL:2023 feature list: [docs/roadmap/01-sql2023-feature-registry.md](docs/roadmap/01-sql2023-feature-registry.md).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rhartness) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
