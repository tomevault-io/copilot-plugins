## sundialkitcombine

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SundialKitCombine is a Combine-based observation plugin for SundialKit that provides reactive network monitoring and WatchConnectivity communication for SwiftUI applications. It uses `@Published` properties and Combine publishers to deliver state updates, with strict `@MainActor` isolation for thread safety.

**Key characteristics:**
- Swift Package Manager library (no executables)
- Depends on SundialKit (main package with core protocols)
- Swift 6.1+ with strict concurrency enabled
- Platform support: iOS 16+, watchOS 9+, tvOS 16+, macOS 11+
- Targets Combine-based SwiftUI apps requiring backward compatibility

## Development Commands

### Building and Testing

```bash
# Build the package
swift build

# Build including tests
swift build --build-tests

# Run all tests
swift test

# Run tests with parallel execution
swift test --parallel

# Run a specific test
swift test --filter SundialKitCombineTests
```

### Code Quality

```bash
# Run all linting (format + lint + build)
LINT_MODE=STRICT ./Scripts/lint.sh

# Format only (no linting or build)
FORMAT_ONLY=1 ./Scripts/lint.sh

# Run linting in normal mode (without strict mode)
./Scripts/lint.sh

# Run periphery to detect unused code (requires mise)
mise exec -- periphery scan --disable-update-check
```

**Note:** Linting requires `mise` to be installed. The script checks for mise at:
- `/opt/homebrew/bin/mise`
- `/usr/local/bin/mise`
- `$HOME/.local/bin/mise`

Tools managed by mise (defined in `.mise.toml`):
- `swift-format` (version 602.0.0)
- `swiftlint` (version 0.61.0)
- `periphery` (version 3.2.0)

### Documentation

```bash
# Preview documentation locally
./Scripts/preview-docs.sh
```

## Architecture

### Three-Layer Architecture

SundialKitCombine sits in Layer 2 of the SundialKit architecture:

**Layer 1: Core Protocols** (dependencies from SundialKit package)
- `SundialKitCore`: Base protocols, errors, and types
- `SundialKitNetwork`: Network monitoring protocols (`PathMonitor`, `NetworkPing`)
- `SundialKitConnectivity`: WatchConnectivity protocols and messaging

**Layer 2: Observation Plugin** (this package)
- `NetworkObserver`: Combines `@MainActor` isolation with `@Published` properties for network state
- `ConnectivityObserver`: Manages WatchConnectivity session with Combine publishers

### Key Components

**NetworkObserver** (`Sources/SundialKitCombine/NetworkObserver.swift`)
- Generic over `MonitorType: PathMonitor` and `PingType: NetworkPing`
- `@MainActor` isolated with `@Published` properties:
  - `pathStatus`: Current network path status
  - `isExpensive`: Whether connection is cellular/expensive
  - `isConstrained`: Whether low data mode is active
  - `pingStatus`: Optional ping verification result
- Delegates to underlying monitor (typically `NWPathMonitor` via `NWPathMonitorAdapter`)
- Optional periodic ping for verification beyond path availability

**ConnectivityObserver** (`Sources/SundialKitCombine/ConnectivityObserver.swift`)
- `@MainActor` isolated observer for WatchConnectivity
- `@Published` properties for session state:
  - `activationState`: Session activation state
  - `isReachable`: Whether counterpart device is available
  - `isPairedAppInstalled`: Whether companion app is installed
  - `isPaired`: (iOS only) Whether Apple Watch is paired
  - `activationError`: Last activation error if any
- Combine `PassthroughSubject` publishers:
  - `messageReceived`: Raw messages with context
  - `typedMessageReceived`: Decoded `Messagable` types
  - `sendResult`: Message send confirmations
  - `activationCompleted`: Activation results with errors
- Supports `MessageDecoder` for type-safe message handling

**Message Types**
- `Messagable`: Protocol for dictionary-based type-safe messages (~65KB limit)
- `BinaryMessagable`: Protocol for efficient binary serialization (no size limit, works with Protobuf/custom formats)
- `MessageDecoder`: Decodes incoming messages to typed `Messagable` instances

### Concurrency Model

- All observers are `@MainActor` isolated
- No `@unchecked Sendable` conformances (strict concurrency compliance)
- Network/connectivity events marshaled to main actor via `Task { @MainActor in ... }`
- Natural integration with SwiftUI's `ObservableObject` pattern

## Swift Settings

The package enables extensive Swift 6.2 upcoming features and experimental features (see `Package.swift:8-45`):

**Upcoming Features:**
- `ExistentialAny`, `InternalImportsByDefault`, `MemberImportVisibility`, `FullTypedThrows`

**Experimental Features:**
- Noncopyable generics, move-only types, borrowing switch, init accessors, etc.

**Important:** When adding new code, maintain strict concurrency compliance. Avoid `@unchecked Sendable`.

## Code Style

- **Indentation:** 2 spaces (configured in `.swift-format`)
- **Line length:** 100 characters max
- **Documentation:** All public declarations must have documentation comments (`AllPublicDeclarationsHaveDocumentation` rule enabled)
- **ACL:** Explicit access control required on all declarations (`explicit_acl`, `explicit_top_level_acl`)
- **Concurrency:** Prefer `@MainActor` isolation over manual dispatch queue management
- **Imports:** Use `public import` for re-exported dependencies

## Testing

Tests are minimal stubs (`Tests/SundialKitCombineTests/SundialKitCombineTests.swift`). The package primarily provides observer wrappers around SundialKit protocols, with integration testing typically done in consuming applications.

When adding tests:
- Use `@MainActor` isolation where needed
- Test with mock implementations of `PathMonitor` and `ConnectivitySession`
- Verify `@Published` property updates and Combine publisher emissions

## Common Patterns

### Creating a NetworkObserver

```swift
// Default NWPathMonitor without ping
let observer = NetworkObserver()
observer.start()

// With custom monitor and ping
let observer = NetworkObserver(
  monitor: NWPathMonitorAdapter(),
  ping: CustomPing(),
  queue: .main
)
```

### Creating a ConnectivityObserver

```swift
// Without message decoder (raw dictionaries)
let observer = ConnectivityObserver()

// With message decoder for typed messages
let observer = ConnectivityObserver(
  messageDecoder: MessageDecoder(messagableTypes: [
    ColorMessage.self,
    TemperatureReading.self
  ])
)
try observer.activate()
```

### Observing State Changes

```swift
// Bind @Published properties
observer.$pathStatus
  .sink { status in
    print("Status: \(status)")
  }
  .store(in: &cancellables)

// Listen to event publishers
connectivityObserver.messageReceived
  .sink { result in
    print("Received: \(result.message)")
  }
  .store(in: &cancellables)
```

## Dependencies

The package depends on SundialKit 2.0.0-alpha.1+ (`Package.swift:62`):
- `SundialKitCore`: Base protocols and types
- `SundialKitNetwork`: Network monitoring abstractions
- `SundialKitConnectivity`: WatchConnectivity abstractions

All SundialKit products are external dependencies resolved via SPM.

## Platform Conditionals

- `#if canImport(Combine)`: Entire package wrapped (graceful degradation on platforms without Combine)
- `#if canImport(Network)`: Convenience initializers for `NWPathMonitor`
- `#if os(iOS)`: iOS-specific WatchConnectivity properties (e.g., `isPaired`)

## Scripts

- `Scripts/lint.sh`: Runs swift-format, swiftlint, periphery, and header checks
- `Scripts/header.sh`: Ensures MIT license headers on all source files
- `Scripts/preview-docs.sh`: Generates and serves DocC documentation locally
- `Scripts/ensure-remote-deps.sh`: Ensures dependencies are resolved remotely (not local overrides)

## CI/CD

GitHub Actions workflows (`.github/workflows/`):
- `SundialKitCombine.yml`: Main build and test workflow
- `codeql.yml`: CodeQL security analysis
- `claude.yml`: Claude-specific automation
- `claude-code-review.yml`: Automated code review

## Documentation Structure

DocC documentation is in `Sources/SundialKitCombine/SundialKitCombine.docc/Documentation.md`. This file contains:
- Usage examples for `NetworkObserver` and `ConnectivityObserver`
- Type-safe messaging patterns
- SwiftUI integration examples
- Comparison with SundialKitStream (the AsyncStream-based alternative)

**Important:** Keep README.md and Documentation.md in sync for major architectural changes.

## Related Packages

- **SundialKit**: Main package containing core protocols and implementations
- **SundialKitStream**: Alternative observation plugin using `AsyncStream` and actor-based concurrency (iOS 16+, macOS 13+)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brightdigit) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
