## cirreum-persistence-nosql

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a .NET 10.0 C# class library that provides NoSQL persistence abstractions for the Cirreum ecosystem. It defines interfaces and base classes for entity management, repository patterns, and persistence operations without external dependencies.

## Development Commands

### Build and Package
```bash
# Build the project
dotnet build

# Build in Release mode with local versioning
dotnet build -c Release

# Create NuGet package (local builds include -rc suffix)
dotnet pack

# Clean build artifacts
dotnet clean
```

### Project Management
```bash
# Restore dependencies
dotnet restore

# Build documentation
dotnet build  # Generates XML documentation automatically
```

## Architecture Overview

### Core Entity Hierarchy
- **`IEntity`**: Base interface defining Id, EntityType, and PartitionKey
- **`BaseEntity`**: Abstract record implementing IEntity with basic functionality
- **`Entity`**: Full-featured entity extending BaseEntity with ETag, TTL, and audit capabilities
- **`SoftDeleteEntity`**: Adds soft delete and restore functionality to Entity

### Repository Pattern Interfaces
- **`IReadOnlyRepository<TEntity>`**: Query, retrieval, counting, and pagination operations
- **`IWriteOnlyRepository<TEntity>`**: Create, update, delete, and restore operations  
- **`IBatchRepository<TEntity>`**: Batch operations for multiple entities
- **`IRepository<TEntity>`**: Consolidated interface combining all repository types

### Key Design Patterns

1. **Partition Key Strategy**: Entities override `GetPartitionKeyValue()` to define partitioning logic
2. **Audit Fields**: Automatic tracking with timezone support using IANA identifiers
3. **Soft Deletes**: Built-in soft delete with restore counting and metadata
4. **ETag Concurrency**: Optimistic concurrency control through ETag support
5. **TTL Support**: Time-to-live functionality for automatic entity expiration
6. **Patch Operations**: Expression and path-based partial updates

### Entity Configuration Attributes
- **`[Container("name")]`**: Specifies container/collection name
- **`[PartitionKeyPath("/path")]`**: Defines partition key path for persistence layer
- **`[UniqueKey("keyname", "/path")]`**: Declares unique key constraints

### Indexing Policy Attributes
- **`[IndexingPolicy(IndexingMode.Consistent)]`**: Class-level, sets indexing mode and `Automatic` flag
- **`[IncludedPath]`** / **`[IncludedPath("/path/?")]`**: Property-level, auto-derives path from `[JsonPropertyName]` or camelCase name if not explicit
- **`[ExcludedPath("/path/*")]`**: Class-level, `AllowMultiple = true`
- **`[CompositeIndex("groupName", order, position)]`**: Property-level, `AllowMultiple = true`, groups by name, orders by position
- **`[SpatialIndex(SpatialType.Point)]`**: Property-level, `AllowMultiple = true`

### Indexing Enums (framework-agnostic mirrors of Cosmos SDK types)
- **`IndexingMode`**: `Consistent`, `Lazy`, `None`
- **`CompositePathSortOrder`**: `Ascending`, `Descending`
- **`SpatialType`**: `Point`, `LineString`, `Polygon`, `MultiPolygon`

### JSON Serialization Conventions
- Uses `System.Text.Json` with specific property names:
  - `_etag` for ETag values
  - `_ts` for raw timestamp (Unix seconds)
  - `ttl` for time-to-live values
- Most audit properties use JsonIgnore to control serialization

## Code Style Guidelines

- **Target Framework**: .NET 10.0 with latest C# language version
- **Nullable Reference Types**: Enabled throughout
- **Coding Style**: Follow .editorconfig rules (tabs, PascalCase, etc.)
- **Interfaces**: Prefix with 'I' (enforced by naming rules)
- **Documentation**: XML documentation required for public APIs
- **Records**: Prefer record types for entity definitions
- **Expression Bodies**: Use for simple properties, avoid for methods/constructors

## Important Implementation Notes

1. **Partition Strategy**: All batch operations require entities to share the same partition
2. **Inheritance Hierarchy**: Choose appropriate base class - BaseEntity for minimal, Entity for full features
3. **Async Patterns**: All repository operations are async with proper cancellation token support
4. **Pagination**: Support both continuation token (preferred) and offset-based pagination
5. **Audit Metadata**: Automatically populated by persistence layer, not set manually
6. **Soft Delete Queries**: Use `includeDeleted` parameter to control visibility

---
> Source: [cirreum/Cirreum.Persistence.NoSql](https://github.com/cirreum/Cirreum.Persistence.NoSql) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
