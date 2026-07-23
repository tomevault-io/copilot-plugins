## simpleorm

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SimpleORM is a Delphi ORM library that simplifies CRUD operations. It supports multiple database connection drivers (FireDAC, RestDataware, UniDAC, Zeos) and works with both VCL and FMX frameworks. The project is written in Object Pascal (Delphi).

## Build & Installation

- **Package manager**: [Boss](https://github.com/HashLoad/boss) (`boss install academiadocodigo/SimpleORM`)
- **Manual install**: Add the `src/` directory to the Delphi Library Path
- **Package file**: `SimpleORM.dpk` (design-time package, no component installation required)
- **Project group**: `SimpleORM_Group.groupproj` contains the library and sample projects
- **Test suite**: `tests/SimpleORMTests.dpr` — DUnit console runner. Open in Delphi IDE, compile and run. Use `-pause` flag to keep console open after tests

## Repository Structure

- `src/` - All library source files (`.pas`)
- `samples/` - Example projects (FireDAC, FMX, Horse, UniDAC, RestDW, Validation, ActiveRecord)
- `docs/plans/` - Design and implementation plan documents
- `assets/`, `public/` - Logo images
- Root: project files (`.dpk`, `.dpr`, `.dproj`, `.groupproj`), `boss.json`, `README.md`

## Architecture

### Core Interfaces (`SimpleInterface.pas`)

All core contracts are defined here. Key interfaces:
- **`iSimpleDAO<T>`** - Main DAO interface for CRUD operations (Insert/Update/Delete/Find), batch operations (InsertBatch/UpdateBatch/DeleteBatch/BulkInsert), Exists, Ask (NL Query), FindAs (DTO mapping), ForceDelete, and Logger
- **`iSimpleDAOSQLAttribute<T>`** - Fluent SQL builder (Fields/Where/Join/OrderBy/GroupBy/Skip/Take), accessed via `iSimpleDAO.SQL` and returned via `.&End`
- **`iSimpleQuery`** - Database connection abstraction (SQL/Params/ExecSQL/Open/DataSet) with transaction control (StartTransaction/Commit/Rollback/InTransaction) and SQLType
- **`iSimpleRTTI<T>`** - RTTI introspection for entity-to-SQL mapping, including SoftDeleteField detection
- **`iSimpleSQL<T>`** - SQL generation from entity metadata, with pagination (Skip/Take/DatabaseType) and soft delete support
- **`iSimpleQueryLogger`** - Query logging interface (`SimpleLogger.pas`), with default `TSimpleQueryLoggerConsole` implementation

### Entity Mapping (`SimpleAttributes.pas`)

Custom attributes for mapping Delphi classes to database tables:
- `Tabela('NAME')` - Maps class to table name
- `Campo('NAME')` - Maps property to column name (optional if property name matches column)
- `PK`, `FK`, `AutoInc`, `NotNull`, `NotZero`, `Ignore`, `NumberOnly`, `Automapping` - Field metadata
- `Bind('FIELD')` - Maps form components to entity properties for automatic UI binding
- `Display('NAME')` - Grid column header text
- `Format(size, precision)` - Field formatting and validation constraints (MaxSize/MinSize validated by TSimpleValidator)
- `HasOne('entity', 'fk')`, `BelongsTo('entity', 'fk')` - Eager-loaded relationships (auto-loaded in Find)
- `HasMany('entity', 'fk')`, `BelongsToMany('entity', 'fk')` - Relationship attributes (use `TSimpleLazyLoader<T>` from `SimpleProxy.pas` for lazy loading)
- `SoftDelete('FIELD_NAME')` - Class-level attribute enabling logical deletion (DELETE becomes UPDATE, SELECT auto-filters)
- `Email`, `MinValue(n)`, `MaxValue(n)`, `Regex('pattern', 'message')` - Validation attributes
- `Enumerator('TYPE')` - PostgreSQL-style enum type casting

### Data Flow

1. **Entity class** is annotated with attributes (`SimpleAttributes.pas`)
2. **`TSimpleRTTI<T>`** (`SimpleRTTI.pas`) reads attributes via RTTI to extract table name, fields, PK, and values
3. **`TSimpleSQL<T>`** (`SimpleSQL.pas`) generates SQL statements (INSERT/UPDATE/DELETE/SELECT) using RTTI data
4. **`TSimpleDAO<T>`** (`SimpleDAO.pas`) orchestrates: generates SQL via `TSimpleSQL`, fills parameters via `TSimpleRTTI`, executes via `iSimpleQuery`
5. **Query drivers** (`SimpleQueryFiredac.pas`, `SimpleQueryRestDW.pas`, `SimpleQueryUnidac.pas`, `SimpleQueryZeos.pas`, `SimpleQueryHorse.pas`) implement `iSimpleQuery` for their respective connection libraries

### Horse Integration

Three units enable full-stack integration with Horse (ExpxHorse) web framework:

- **`SimpleSerializer.pas`** - Entity <-> JSON conversion via RTTI using `[Campo]` attribute names as JSON keys. Methods: `EntityToJSON<T>`, `JSONToEntity<T>`, `EntityListToJSONArray<T>`, `JSONArrayToEntityList<T>`
- **`SimpleHorseRouter.pas`** - Auto-generates CRUD routes on a Horse app: `TSimpleHorseRouter.RegisterEntity<T>(THorse, Query)` creates GET/POST/PUT/DELETE endpoints. Supports custom path, callbacks (OnBeforeInsert/OnAfterInsert/OnBeforeUpdate/OnBeforeDelete)
- **`SimpleQueryHorse.pas`** - REST client driver implementing `iSimpleQuery` via HTTP. Usage: `TSimpleQueryHorse.New('http://server:9000', 'token')`. Translates SQL operations to REST calls. Supports Bearer token auth and custom headers via `OnBeforeRequest`

**Server setup (3 lines per entity):**
```pascal
TSimpleHorseRouter.RegisterEntity<TProduto>(THorse, LQuery);
THorse.Listen(9000);
```

**Client setup (swap one line):**
```pascal
// Instead of: TSimpleQueryFiredac.New(Connection)
LDAO := TSimpleDAO<T>.New(TSimpleQueryHorse.New('http://localhost:9000'));
```

### RTTI Helpers (`SimpleRTTIHelper.pas`)

Class helpers on `TRttiProperty`, `TRttiType`, and `TRttiField` that provide attribute-checking methods (e.g., `EhChavePrimaria`, `IsAutoInc`, `FieldName`, `DisplayName`, `IsHasOne`, `IsBelongsTo`, `IsEmail`, `HasFormat`, `IsSoftDelete`). These are used extensively throughout `SimpleRTTI.pas` and `SimpleValidator.pas`.

### Entity Base Class (`SimpleEntity.pas`)

`TSimpleEntity` provides `Parse` (from Form, DataSet, or JSON), `ToJSON`, and `SaveToFileJSON`. `TSimpleEntityList<T>` extends `TObjectList<T>` with similar capabilities.

### Lazy Loading Proxy (`SimpleProxy.pas`)

`TSimpleLazyLoader<T>` extends `TObjectList<T>` and defers query execution until first access (Count/ToArray). Used for HasMany relationships - instantiate in entity constructors.

### Query Logging (`SimpleLogger.pas`)

`iSimpleQueryLogger` interface with `Log(SQL, Params, DurationMs)`. Default implementation `TSimpleQueryLoggerConsole` outputs via OutputDebugString (Windows) or WriteLn (console). Attach via `TSimpleDAO.Logger(...)`.

### Validation (`SimpleValidator.pas`)

`TSimpleValidator.Validate(object)` checks constraints using RTTI: `NotNull`, `NotZero`, `Format` (string length), `Email`, `MinValue`, `MaxValue`, `Regex`. Raises `ESimpleValidator` with accumulated error messages.

### Conditional Compilation

- `{$IFNDEF CONSOLE}` guards all UI-binding code (form bind, VCL/FMX components)
- `{$IFDEF FMX}` / `{$IFDEF VCL}` switches between FMX and VCL component types
- `{$IF RTLVERSION > 31.0}` guards newer RTTI type kinds (`tkMRecord`)

## Conventions

- Portuguese naming in helper methods (e.g., `EhChavePrimaria` = "is primary key", `EhChaveEstrangeira` = "is foreign key", `Tem<T>` = "has attribute")
- Fluent interface pattern throughout: all methods return `Self` for chaining
- All main classes follow `TSimpleXxx.New(...)` constructor pattern returning an interface
- Properties must be declared in the `published` section for RTTI attribute mapping to work
- SQL uses parameterized queries with `:fieldname` syntax
- Transaction control is explicit: call `StartTransaction`/`Commit`/`Rollback` on `iSimpleQuery`
- `TSQLType` (Firebird, MySQL, SQLite, Oracle) is set on the query driver and propagated for database-specific SQL (pagination)
- Pagination uses fluent `.SQL.Skip(n).Take(n).&End.Find` pattern
- Batch operations (`InsertBatch`/`UpdateBatch`/`DeleteBatch`) auto-wrap in transactions
- `SimpleJSON.pas` is deprecated - use `SimpleJSONUtil.pas` for all JSON operations
- **Every new feature MUST include a sample project** in `samples/` demonstrating the feature — without a sample, the feature is NOT complete
- **Every new feature MUST include DUnit tests** in `tests/` — without tests, the feature is NOT complete
- **Never create `.dproj`, `.res`, or `.dfm` files** — these are generated exclusively by the Delphi IDE. Only create `.dpr` (program files) and `.pas` (unit files)
- **Minimum Delphi version**: 10.2 Tokyo — NO inline variables, NO managed records
- `BulkInsert` generates single INSERT with multiple VALUES (batches of 100) for performance
- `Exists(field, value)` is a convenience method for checking record existence without loading
- `FindAs<R>` maps query results to DTO classes without `[Tabela]` attribute
- `Ask(question)` on DAO uses AI to convert natural language to SQL and execute
- `TSimpleEventBus` provides global observer pattern for entity Insert/Update/Delete events
- `Automapping` attribute enables convention-over-configuration (class name = table name)
- `TSimpleSwagger` generates OpenAPI 3.0 specs from registered entities via RTTI
- `TSimpleSeeder` generates realistic test data via AI based on entity RTTI schema
- `TSimpleAutoIndex` analyzes slow queries and suggests CREATE INDEX via AI
- `TSimpleQueryOptimizer` analyzes SQL and suggests optimizations via AI
- `TSkillTelegram`/`TSkillDiscord` send notifications when entities change
- `TSimpleExportSheets` exports entity data to Google Sheets via API

---
> Source: [academiadocodigo/SimpleORM](https://github.com/academiadocodigo/SimpleORM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
