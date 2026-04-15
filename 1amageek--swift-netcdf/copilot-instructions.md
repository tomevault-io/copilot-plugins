## swift-netcdf

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

swift-netcdf is a Swift wrapper for the NetCDF (Network Common Data Form) C library. The project builds netCDF-c from source as a git submodule and exposes it to Swift through a C module wrapper.

## Architecture

### Three-Layer Structure

1. **netcdf-c (C library)**: Git submodule at `Sources/CNetCDF/netcdf-c`
   - Official Unidata netCDF C library
   - Built from source using CMake
   - Configured for minimal build (netCDF-3 only, no HDF5/DAP/NCZarr)

2. **CNetCDF (C module)**: Swift Package Manager C target at `Sources/CNetCDF`
   - Module map at `Sources/CNetCDF/module.modulemap` defines the C module
   - Built library artifacts in `Sources/CNetCDF/lib/` (generated, not in git)
   - Exposes C headers to Swift

3. **NetCDF (Swift wrapper)**: Swift library at `Sources/NetCDF/`
   - Public Swift API wrapping C functions
   - Type-safe Swift interfaces (NetCDFFile, NetCDFMode, NetCDFError)
   - Manages resource lifecycle and error handling

### Key Design Decisions

- **Source build required**: The netCDF C library must be built before Swift package can compile
- **Static linking**: Uses libnetcdf.a (static library) to avoid runtime dependencies
- **Minimal configuration**: netCDF built without HDF5, DAP, NCZarr to reduce complexity
- **Module map approach**: Uses modulemap to expose C library to Swift (not systemLibrary target)
- **Type-safe API**: Separate AccessMode and CreationMode types prevent API misuse
- **Swift-like interface**: Uses computed properties, URL-based APIs, and proper error handling

## Build Process

### Initial Setup (First Time Only)

```bash
# 1. Initialize git submodules
git submodule update --init --recursive

# 2. Build netcdf-c library
./scripts/build-netcdf.sh
```

This creates:
- `Sources/CNetCDF/lib/include/` - C headers
- `Sources/CNetCDF/lib/lib/libnetcdf.a` - Static library
- `.build/netcdf-build/` - CMake build artifacts (can be deleted)

### Regular Development

```bash
# Build Swift package
swift build

# Run tests
swift test

# Build in verbose mode (for debugging)
swift build -v
```

### Rebuild netcdf-c

If you need to rebuild the C library (e.g., after updating submodule or platform version):

```bash
# Clean previous build
rm -rf .build/netcdf-build Sources/CNetCDF/lib

# Rebuild
./scripts/build-netcdf.sh
```

**Important**: The build script automatically sets `CMAKE_OSX_DEPLOYMENT_TARGET` to match the platform requirements in Package.swift. If you update the platform versions (e.g., from macOS 15 to macOS 26), you **must** rebuild the NetCDF C library to avoid runtime crashes.

## Package.swift Configuration

The CNetCDF target in Package.swift:
- Uses `.target()` (NOT `.systemLibrary()`)
- `publicHeadersPath: "include"` points to umbrella header directory
- `sources: ["dummy.c"]` - includes a dummy C file for Xcode compatibility
  - The actual NetCDF library is pre-built and linked statically
  - `dummy.c` satisfies Xcode's requirement that targets have at least one source file
  - Command-line builds work with or without this file
- `cSettings: [.headerSearchPath("lib/include")]` - finds built headers
- `linkerSettings` links against pre-built `libnetcdf.a`
- Must exclude submodule directory and build artifacts with `exclude`

## Common Issues

### Runtime Crash in NC_create() or other NetCDF functions

**Symptom**: Tests crash inside NetCDF C library functions (like `NC_create()`)

**Cause**: Deployment target mismatch. The NetCDF library was built for a different macOS version than what the Swift package expects.

**Solution**:
1. Clean and rebuild the NetCDF C library:
   ```bash
   rm -rf .build/netcdf-build Sources/CNetCDF/lib
   ./scripts/build-netcdf.sh
   ```
2. Rebuild the Swift project:
   ```bash
   swift build
   ```

The build script automatically sets the correct deployment target (26.0) via `-DCMAKE_OSX_DEPLOYMENT_TARGET=26.0`.

### Xcode Build: "Build input file cannot be found: CNetCDF.o"

If building in Xcode fails with this error:
- Ensure `Sources/CNetCDF/dummy.c` exists
- Verify `Package.swift` includes `sources: ["dummy.c"]` in CNetCDF target
- This dummy file is required for Xcode but not for command-line builds

The issue occurs because Xcode requires at least one source file per target, even for targets that only link pre-built libraries.

### Missing netcdf_filter_hdf5_build.h

The built netCDF library may not install all headers referenced in the module map. If you encounter missing header errors:
- Check which headers are actually installed in `Sources/CNetCDF/lib/include/`
- Update `Sources/CNetCDF/module.modulemap` to only include available headers
- May need to adjust CMake build options in `scripts/build-netcdf.sh`

### Module CNetCDF not found

Ensure:
1. `./scripts/build-netcdf.sh` has been run successfully
2. `Sources/CNetCDF/lib/include/netcdf.h` exists
3. `Sources/CNetCDF/module.modulemap` exists and references correct header paths

### Linker errors

- Verify `Sources/CNetCDF/lib/lib/libnetcdf.a` exists
- Check that linker flags in Package.swift use correct relative paths
- netCDF may require system libraries (m, z, xml2) - add to linkerSettings if needed

## Module Map Structure

The `Sources/CNetCDF/module.modulemap` defines the C module interface:
- Header paths are relative to the CNetCDF directory
- Uses `lib/include/` prefix for built headers
- Links libnetcdf with `link "netcdf"` directive

## API Design

### Type-Safe Mode System

The API uses separate types for file access modes vs creation modes to prevent API misuse:

**AccessMode** (for opening existing files):
- `.read` - Maps to `NC_NOWRITE`, used with `nc_open()`
- `.write` - Maps to `NC_WRITE`, used with `nc_open()`

**CreationMode** (for creating new files):
- `.overwrite` - Maps to `NC_CLOBBER`, used with `nc_create()`
- `.exclusive` - Maps to `NC_NOCLOBBER`, used with `nc_create()`

This design prevents errors like passing a creation mode to an open operation, which would always fail at runtime with the C API.

### Convenience Initializers

Type-safe initialization methods prevent common mistakes:

```swift
// For opening existing files
init(reading: URL) throws              // Uses AccessMode.read
init(writing: URL) throws              // Uses AccessMode.write
init(opening: URL, mode: AccessMode)   // Explicit access mode

// For creating new files
init(creating: URL) throws                  // Uses CreationMode.overwrite
init(creating: URL, mode: CreationMode)     // Explicit creation mode
```

These APIs make intent clear and prevent accidentally trying to create a file with a read-only mode.

### Swift-Like API Patterns

The wrapper follows Swift conventions:

1. **Computed properties with throwing getters** instead of `getX()` methods:
   ```swift
   let count = try file.dimensionCount  // Instead of getDimensionCount()
   ```

2. **URL-based APIs** as primary interface with String convenience methods:
   ```swift
   init(reading: URL) throws           // Primary
   init(reading: String) throws        // Convenience wrapper
   ```

3. **Equatable error types** for testability:
   ```swift
   public enum NetCDFError: Error, CustomStringConvertible, Equatable {
       case fileNotFound(path: String)
       case fileAlreadyExists(path: String)
       // ... enables pattern matching in tests
   }
   ```

4. **Sendable conformance** for Swift 6 concurrency:
   ```swift
   public struct AccessMode: Sendable { ... }
   public struct CreationMode: Sendable { ... }
   ```

### Error Handling

NetCDFError provides:
- Specific cases for common errors (fileNotFound, permissionDenied, etc.)
- Generic `netcdfError(code:message:)` for C library errors
- Context-aware error messages including file paths
- Equatable conformance for testing specific error conditions

### Resource Management

The API provides both manual and automatic resource management:

**Manual cleanup**:
```swift
let file = try NetCDFFile(reading: url)
defer { try? file.close() }
// ... use file
```

**Automatic cleanup with scoped helpers**:
```swift
let dimCount = try NetCDFFile.withFile(reading: url) { file in
    try file.dimensionCount
}
```

The `withFile` methods ensure:
- File is always closed, even if body throws
- Close errors are propagated when body succeeds
- Body errors take precedence over close errors

## Testing

### Swift Testing Framework

This project uses **Swift Testing**, which is built into Swift 6.2 toolchain:
- No external package dependency required for testing
- Uses modern `@Test` and `@Suite` attributes
- Uses `#expect` macro instead of XCTest assertions
- Supports descriptive test names and better organization

Example:
```swift
import Testing
@testable import NetCDF

@Suite("NetCDF File Operations")
struct NetCDFTests {
    @Test("Create NetCDF file with URL")
    func createFileWithURL() throws {
        // Test implementation
    }
}
```

### Platform Requirements

The package requires:
- Swift 6.2 or later
- macOS 26.0+ / iOS 26.0+ / visionOS 26.0+ / tvOS 26.0+ / watchOS 26.0+

These platform versions ensure access to the latest Swift concurrency features and built-in Swift Testing framework.

## Concurrency Support

### Thread Safety

`NetCDFFile` is designed for single-threaded use. The underlying NetCDF C library is not thread-safe for concurrent operations on the same file handle. When using NetCDF files in concurrent contexts:

1. **One file per task**: Each concurrent task should open its own file handle
2. **Synchronize access**: If sharing a file handle, use Swift actors or locks to serialize access
3. **Sendable conformance**: Mode structs (`AccessMode`, `CreationMode`) conform to `Sendable` for safe cross-task usage

### Future Async Support

While the current API is synchronous, async/await versions can be added:
```swift
// Future API (not yet implemented)
public static func withFile<T>(
    reading url: URL,
    perform body: (NetCDFFile) async throws -> T
) async throws -> T
```

This would allow integration with Swift's structured concurrency while maintaining the C library's synchronous nature.

## Source Directory Naming

- `Sources/CNetCDF/` - C module (must match target name)
- `Sources/NetCDF/` - Swift wrapper (must match target name)
- `Tests/NetCDFTests/` - Test suite

Note: Directory names must exactly match target names in Package.swift.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/1amageek) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
