## bc-dev-cli

> A .NET CLI tool for Business Central (BC) development. Downloads BC compiler artifacts from Microsoft CDN and compiles AL extensions.

# bcdev

A .NET CLI tool for Business Central (BC) development. Downloads BC compiler artifacts from Microsoft CDN and compiles AL extensions.

## Build and Test

```bash
dotnet build
dotnet test
```

## Architecture

### Core Services

- **ArtifactService** (`src/Services/ArtifactService.cs`): Downloads BC compiler artifacts from Microsoft CDN using HTTP Range requests for efficient partial ZIP downloads. Caches artifacts locally.
- **CompilerService** (`src/Services/CompilerService.cs`): Wraps the AL compiler (alc) for compiling AL extensions.
- **SymbolService** (`src/Services/SymbolService.cs`): Handles symbol file downloads from NuGet feeds.

### Key Patterns

#### Concurrency Control with SemaphoreSlim
Multiple parallel CI jobs downloading the same BC version can cause race conditions. Use `SemaphoreSlim` to serialize access to cache directories per version:
```csharp
private static readonly Dictionary<string, SemaphoreSlim> _downloadLocks = new();
```

#### Completion Marker for Caching
Use a `.extraction-complete` marker file to signal that multi-step extraction is fully complete. This prevents fast-path cache checks from succeeding when extraction is partial:
```csharp
var completionMarker = Path.Combine(targetPath, ".extraction-complete");
await File.WriteAllTextAsync(completionMarker, DateTime.UtcNow.ToString("O"));
```

#### Partial ZIP Downloads
For large archives (>1GB), download only necessary parts using HTTP Range requests:
1. Download EOCD (last 64KB) to find Central Directory location
2. Download Central Directory to get file index
3. Download only needed files (compiler, client DLLs)

This reduces download from ~3GB to ~270MB.

#### Platform-Specific Builds
The AL compiler is distributed inside a VSIX file with platform-specific directories:
- `extension/bin/win32/` - Windows
- `extension/bin/darwin/` - macOS
- `extension/bin/linux/` - Linux

Use MSBuild conditions in `.csproj` for platform-specific files:
```xml
<ItemGroup Condition="$([MSBuild]::IsOSPlatform('OSX'))">
  ...
</ItemGroup>
```

### Platform Considerations

#### ARM64 macOS (Apple Silicon)
The BC compiler (alc) is x64-only. On ARM64 Macs:
- Rosetta 2 is required to run the compiler
- First execution triggers Rosetta's AOT compilation/caching
- Concurrent access during this initial compilation can cause "code signature supplement failed" errors (race condition)
- The `SemaphoreSlim` pattern above prevents this race condition

## Testing Guidelines

### Test Fixture Management
Copy test app files to a temporary directory before testing to isolate tests and prevent modification of source files:
```csharp
var tempDir = Path.Combine(Path.GetTempPath(), $"bcdev-test-{Guid.NewGuid()}");
CopyDirectory(testAppPath, tempDir);
try { /* run test */ }
finally { Directory.Delete(tempDir, recursive: true); }
```

### Exception Handling for Cleanup
Use try-catch in cleanup code to prevent test failures due to cleanup errors:
```csharp
try { Directory.Delete(tempDir, true); }
catch { /* Ignore cleanup errors */ }
```

### Diagnostic Logging
Use `ITestOutputHelper` extensively for debugging CI failures:
```csharp
_output.WriteLine($"Compiler: {compilerPath}");
_output.WriteLine($"Compilation success: {result.Success}");
```

### Do NOT Skip Tests
Tests should fail if expected functionality is not present. Do not automatically skip tests when environmental prerequisites aren't met. If the compiler isn't runnable, the test should fail to surface the issue.

### Test Categories
Use `[Trait("Category", "Slow")]` for tests that download large artifacts or take significant time.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/navipartner) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
