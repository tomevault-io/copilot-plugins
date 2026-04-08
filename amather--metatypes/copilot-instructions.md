## metatypes

> Generates one unified registration method:

# MetaTypes Project

A C# source generator that produces compile-time metadata for classes, structs, records, and enums to reduce reflection overhead at runtime. Implements a vendor-based architecture that allows extensibility through:

- Custom type discovery methods
- Vendor-specific metadata extensions via partial class generation

> **Development Status**: This project is in active development and has not yet reached a versioned release. Documentation focuses on current features without version tracking, deprecated/obsolete markers, or "NEW" feature annotations until the first stable release.

## Quick Start

Primary showcase (Entity Framework Core vendor):
```bash
cd samples/Vendor/EfCore/Sample.EfCore.SingleProject
dotnet run
```

Basic functionality:
```bash
cd samples/Sample.Console
dotnet run
```

## Vendor Samples

### Entity Framework Core Vendor
📂 **[EfCore Vendor Documentation](./samples/Vendor/EfCore/CLAUDE.md)**

Features:
- Table metadata (names, keys, foreign keys)
- DbContext collections for organized entity access
- Cross-assembly entity discovery
- DI integration with service provider extensions

Samples:
- `samples/Vendor/EfCore/Sample.EfCore.SingleProject/` - Single project showcase
- `samples/Vendor/EfCore/Sample.EfCore.MultiProject/` - Multi-project scenario

### Statics Vendor
📂 **[Statics Vendor Documentation](./samples/Vendor/Statics/CLAUDE.md)**

Features:
- Static method discovery across assemblies
- Service pattern metadata generation
- Repository generation with consistent async APIs
- DI integration for static method collections

Status: ✅ Complete

## Project Structure

### Core Components
- **src/MetaTypes.Abstractions/** - Core interfaces and attributes intended to be used by consumers ([CLAUDE.md](src/MetaTypes.Abstractions/CLAUDE.md))
- **src/MetaTypes.Generator.Common/** - Shared generator infrastructure ([CLAUDE.md](src/MetaTypes.Generator.Common/CLAUDE.md))
- **src/MetaTypes.Generator/** - Main source generator implementation

### Vendor System
- **EfCore Vendor** - Entity Framework Core extensions with table mapping and relationships
- **Statics Vendor** - Service method discovery and repository generation for static classes

## Recent Architecture Changes

1. **Target-Namespace-Specific DI Extensions** - Generate DI extension methods based on target namespace
2. **Cross-Assembly Mode** - Unified provider generation that consolidates types from multiple assemblies
3. **Vendor-Specific DI Methods** - Separate DI registration methods for each vendor
4. **Vendor Folder Structure** - Consistent vendor pattern across all projects
5. **Pluggable Vendor System** - Vendor-agnostic core with specialized extensions
6. **Repository Generation** - Statics vendor generates repositories with consistent async APIs
7. **Diagnostic Analyzers** - Roslyn analyzers validate attribute usage at design time

## Diagnostic Analyzers

MetaTypes includes Roslyn diagnostic analyzers that validate attribute usage and configuration in real-time within the IDE. Diagnostics are vendor-specific and automatically enabled based on the discovery methods configured in `metatypes.config.json`.

### Key Features

- **Real-time Validation** - Errors and warnings appear as you type in the IDE
- **Vendor-Specific** - Each vendor defines its own diagnostic rules
- **Configuration-Based** - Diagnostics are enabled when their corresponding discovery method is active
- **Standard Suppression** - Use `#pragma warning disable MTXXX` or `.editorconfig` to suppress warnings

### Diagnostic ID Convention

All diagnostic IDs follow the pattern: `MT{VendorPrefix}{Number}`

Examples:
- `MTSTAT0001` - Statics vendor diagnostics
- `MTEFCORE0001` - EfCore vendor diagnostics

### Release Tracking

**IMPORTANT**: When adding, modifying, or removing diagnostic rules:

1. **Update `AnalyzerReleases.Unshipped.md`** - Add new rules to this file during development
2. **Update diagnostic descriptor files** - Add the new `DiagnosticDescriptor` in the vendor's diagnostics file (e.g., `StaticsDiagnostics.cs`, `EfCoreDiagnostics.cs`)
3. **Update `DIAGNOSTICS.md`** - Document the new diagnostic in the comprehensive diagnostics documentation

When releasing a new version:
- Move entries from `AnalyzerReleases.Unshipped.md` to `AnalyzerReleases.Shipped.md`
- Create a new release section with the version number

These files are required by the Roslyn analyzer infrastructure and must be maintained correctly to avoid build warnings (RS2008).

For complete diagnostic documentation, see **[Diagnostics Guide](./docs/DIAGNOSTICS.md)**.

## Generated DI Extension Methods

The generator produces DI extension methods based on the target namespace where the generator runs:

### Cross-Assembly Mode
```json
{
  "MetaTypes.Generator": {
    "Discovery": { "CrossAssembly": true }
  }
}
```

Generates one unified registration method:
```csharp
// Register all discovered MetaTypes from multiple assemblies
services.AddMetaTypesSampleConsole();
```

### Single-Project Mode  
```json
{
  "MetaTypes.Generator": {
    "Discovery": { "CrossAssembly": false }
  }
}
```

Generates separate methods per assembly:
```csharp
services.AddMetaTypesSampleBusiness();
services.AddMetaTypesSampleAuth();
```

### Vendor-Specific Methods
```csharp
// EfCore vendor extensions
services.AddMetaTypesSampleConsoleEfCore();

// Statics vendor extensions  
services.AddMetaTypesSampleConsoleStatics();

// Statics repository extensions
services.AddMetaTypesSampleConsoleStaticsRepositories();

// Service retrieval - Individual entity types
var efCoreTypes = serviceProvider.GetEfCoreMetaTypes();
var staticsTypes = serviceProvider.GetStaticsMetaTypes();

// Service retrieval - DbContext collections
var dbContexts = serviceProvider.GetEfCoreDbContexts();
var specificContext = serviceProvider.GetEfCoreDbContext<MyDbContext>();

// Service retrieval - Repository collections
var staticsRepositories = serviceProvider.GetServices<IStaticsRepository>();
var entityRepositories = serviceProvider.GetServices<IEntityRepository>();
var userRepository = serviceProvider.GetService<UserRepository>();
var globalRepository = serviceProvider.GetService<GlobalRepository>();
```

## Configuration

Basic project setup with `metatypes.config.json`:
```json
{
  "MetaTypes.Generator": {
    "Generation": { "BaseMetaTypes": true },
    "Discovery": {
      "CrossAssembly": true,
      "Methods": ["MetaTypes.Attribute", "MetaTypes.Reference"]
    }
  }
}
```

## Documentation

- **[Complete Documentation](./docs/README.md)** - Architecture, configuration, and API reference
- **[Configuration Guide](./CONFIG.md)** - Detailed configuration options
- **[Diagnostics Guide](./docs/DIAGNOSTICS.md)** - Diagnostic analyzer infrastructure and rules
- **[Migration Guide](./MIGRATION.md)** - Upgrading from previous versions

## Build Commands

```bash
dotnet build                    # Build entire solution
dotnet test                     # Run all tests
dotnet build samples/           # Build all samples
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/amather)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/amather)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
