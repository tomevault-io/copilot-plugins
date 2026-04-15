## cacheutility

> CacheUtility is a thread-safe, generic wrapper for System.Runtime.Caching that provides advanced caching features for .NET applications. It's distributed as a NuGet package and includes:

# CacheUtility Project Overview

## Project Purpose
CacheUtility is a thread-safe, generic wrapper for System.Runtime.Caching that provides advanced caching features for .NET applications. It's distributed as a NuGet package and includes:

- Thread-safe memory caching operations
- Cache groups for organized data management
- Dependency relationships between cache groups
- Automatic cache population with custom methods
- Multiple expiration strategies (sliding/absolute)
- Bulk retrieval with GetAllByGroup()

## Key Project Structure

### Core Files
- [CacheUtility.cs](mdc:CacheUtility.cs) - Main implementation of the Cache class with all caching functionality
- [CacheUtility.csproj](mdc:CacheUtility.csproj) - Project file with NuGet package configuration
- [README.md](mdc:README.md) - Comprehensive documentation with usage examples
- [License.txt](mdc:License.txt) - MIT license for the project

### Test Project
- [CacheUtility.Tests/](mdc:CacheUtility.Tests) - Unit tests directory
- [GetAllByGroupTests.cs](mdc:CacheUtility.Tests/GetAllByGroupTests.cs) - Tests for the GetAllByGroup functionality

### Build & Packaging
- Target Framework: .NET 9.0
- Uses System.Runtime.Caching package (version 8.0.1)
- Configured for NuGet package generation on build
- Current version: 1.0.9

## Main Class: Cache
The Cache class is abstract and provides static methods for all caching operations. Key functionality includes:
- Get<TData>() methods for retrieving/populating cache items
- Remove() methods for cache item removal
- RemoveGroup() for clearing entire cache groups
- GetAllByGroup() for retrieving all items from a group
- SetDependencies() for establishing cache group relationships

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/onnop)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/onnop)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
