## aranet-radon-sensor-measurement

> - Swift 5.5+ with async/await concurrency


# Swift Code Conventions

## Language and Framework

- Swift 5.5+ with async/await concurrency
- CoreBluetooth framework for BLE communication
- Foundation framework for file I/O and data handling
- Use `@main` attribute for entry point

## Code Style

### Naming Conventions

- Constants: `UPPER_SNAKE_CASE` (e.g., `SERVICE_UUID`, `RADON_CHAR_UUID`)
- Global vars: `UPPER_SNAKE_CASE` (e.g., `VERBOSE_MODE`, `JSON_MODE`)
- Types: `PascalCase` (e.g., `DeviceConfig`, `Measurements`)
- Functions/methods: `camelCase` (e.g., `parseRadonData`, `loadConfig`)
- Private methods: Prefix with `private` keyword

### Async/Await Patterns

- Use `async throws` for BLE operations
- Use `CheckedContinuation` to bridge delegate-based APIs to async/await
- Always handle timeout scenarios with `Timer`
- Clean up resources (stop scanning, disconnect) on completion or error

### Error Handling

- Define custom `AranetError` enum conforming to `LocalizedError`
- Provide descriptive error messages with context
- Include troubleshooting hints in error output for common issues
- Use `do-catch` blocks at the top level, not throughout the code

## CoreBluetooth Specifics

### Delegate Pattern

- Use `NSObject` base class for delegate conformance
- Mark delegate classes as `@unchecked Sendable` (required for async context)
- Store continuations as properties and resume them in delegate callbacks
- Always invalidate timers when operations complete

### BLE Best Practices

- Always check `CBCentralManager.state` before operations
- Set timeouts for all BLE operations (scanning, connecting, reading)
- Clean up connections immediately after reading data
- Handle both `poweredOn` state and state changes in delegate

### UUID Handling

- Use `CBUUID` for service and characteristic UUIDs
- Use `UUID` for device identifiers (CBPeripheral.identifier)
- Store UUIDs as strings in config files

## Data Parsing

### Binary Data Handling

- Use `Data.withUnsafeBytes` with `loadUnaligned` for reading multi-byte values
- Always validate data length before parsing (`guard data.count >= expected`)
- Track byte offset explicitly when parsing structured data
- Use appropriate integer types (UInt8, UInt16, UInt32)

### Numeric Conversions

- Convert raw sensor values using documented multipliers:
  - Temperature: `raw × 0.05`
  - Pressure: `raw × 0.1`
  - Humidity: `raw × 0.1`
- Use `Decimal` type for temperature/pressure/humidity (precise decimal values)
- Use `Int` for radon concentrations (whole numbers)

## Output Formatting

### Display Modes

- Default: Human-readable text with labels and units
- JSON: Structured output for programmatic consumption
- Verbose: Include progress messages (only when `VERBOSE_MODE` is true)

### JSON Keys

Use snake_case for JSON output:

- `radon_bqm3`, `temperature_c`, `humidity_percent`, `pressure_hpa`
- `battery_percent`, `interval_s`, `updated_s_ago`
- `radon_24h_avg_bqm3`, `radon_7d_avg_bqm3`, `radon_30d_avg_bqm3`

## Configuration Management

### File Operations

- Store config in `~/.config/aranet-radon/`
- Use `FileManager.default.homeDirectoryForCurrentUser`
- Create directories with `withIntermediateDirectories: true`
- Use `JSONEncoder` with `.prettyPrinted` for config files

### Config Structure

```swift
struct DeviceConfig: Codable {
    let uuid: String
    let name: String
    let lastSeen: String  // ISO8601 format
}
```

## Testing Considerations

- Test both advertisement-based and GATT-based reading modes
- Verify timeout handling (scan timeout, operation timeout)
- Test with device out of range to validate error messages
- Verify config file creation and loading

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kaanozdokmeci) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
