## appstore-analytics-cli

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

App Store Analytics CLI - A command-line tool for interacting with Apple's App Store Connect Analytics API. Enables programmatic creation, monitoring, and downloading of analytics reports for iOS apps.

**Language:** Swift 5.9+
**Platform:** macOS 13.0+ (Ventura)
**Primary Dependency:** AppStoreConnect-Swift-SDK v4.0.0+

## Build and Run Commands

### Building
```bash
# Debug build
swift build

# Release build
swift build -c release

# Run directly (development)
swift run appstore-analytics <command>

# Install to /usr/local/bin
sudo cp .build/release/appstore-analytics /usr/local/bin/
```

### Testing
Note: Currently no test suite exists. Tests would be added under `Tests/` directory.

```bash
# Run tests (when implemented)
swift test

# Run specific test
swift test --filter <TestName>
```

### Development Commands
```bash
# Clean build artifacts
swift package clean

# Update dependencies
swift package update

# Show resolved dependencies
swift package show-dependencies
```

## Architecture Overview

### Command Pattern Architecture
The CLI uses a command-based architecture with clear separation of concerns:

- **Commands Layer** (`Sources/AppStoreAnalyticsCLI/Commands/`)
  - Each command is a separate file with static `execute()` method
  - Command routing happens in `Command.swift` enum via argument parsing
  - `main.swift` dispatches to appropriate command based on parsed arguments

- **Core Layer** (`Sources/AppStoreAnalyticsCLI/Core/`)
  - `APIClient`: Actor-based wrapper around AppStoreConnect-Swift-SDK
  - `JWTManager`: Actor managing JWT token lifecycle (18-min refresh cycle)
  - `ConfigManager`: Singleton for config file I/O with secure permissions
  - `RateLimiter`: Token bucket algorithm enforcing API rate limits
  - `CSVDownloader`: Parallel download manager with progress tracking

- **Models Layer** (`Sources/AppStoreAnalyticsCLI/Models/`)
  - `Configuration`: User config stored in `~/.appstore-analytics-config.json`
  - `ReportType`: 20+ report types across 5 categories (discovery, commerce, usage, performance, subscriptions)
  - `ReportRequest`: API request/response models

- **Utilities Layer** (`Sources/AppStoreAnalyticsCLI/Utilities/`)
  - `Logger`: Formatted console output ([OK], [ERROR], [SUCCESS], [INFO])
  - `UserInput`: Interactive prompts and tilde path expansion

### Concurrency Model
- Uses Swift structured concurrency (async/await)
- Two actors for thread-safe state management:
  - `APIClient`: Protects API configuration and rate limiter state
  - `JWTManager`: Protects JWT token cache and expiry dates
- All commands are async and called from `main.swift` with top-level await

### Authentication Flow
1. User runs `configure` command, providing credentials
2. `ConfigManager` saves credentials to `~/.appstore-analytics-config.json` with 600 permissions
3. `JWTManager` validates credentials by creating `APIConfiguration` instance
4. JWT tokens are generated on-demand and cached for 18 minutes (buffer before 20-min expiry)
5. `APIClient` uses `APIProvider` from AppStoreConnect-Swift-SDK with auto-managed JWT

### Rate Limiting Strategy
- Hourly limit: 3,500 requests (buffer below Apple's 3,600 limit)
- Minute limit: 300 requests (buffer below Apple's limit)
- Token bucket implementation with automatic refill
- Blocks requests when limits exceeded until tokens available

## Key Design Patterns

### Configuration Management
- Single config file at `~/.appstore-analytics-config.json`
- Automatic chmod 600 on save for security
- Validates private key file existence and permissions on load
- Tilde path expansion for cross-user compatibility

### Error Handling
All major components have dedicated error enums conforming to `LocalizedError`:
- `ConfigManagerError`: Configuration and file permission issues
- `JWTManagerError`: JWT generation and validation failures
- `APIClientError`: API communication and rate limit errors
- `DownloadError`: Download and file operation failures

Errors provide actionable messages guiding users to resolution.

### Download Architecture
CSV downloads use parallel execution model:
- Up to 5 concurrent segment downloads
- Each report can have multiple instances
- Each instance can have multiple segments
- Directory structure: `{output-dir}/{report-id}/instance-{id}/segment-NNN.csv`
- Optional merge functionality combines segments into single CSV

## Important Implementation Notes

### API Integration Status
The tool has **placeholder implementations** for API calls. The infrastructure is complete but actual API endpoints need implementation:

- `APIClient.createReportRequest()` - Returns mock report ID
- `APIClient.listReports()` - Returns empty array
- `APIClient.getReportStatus()` - Returns `.processing`
- `APIClient.getReportInstances()` - Returns empty array
- `APIClient.getReportSegments()` - Returns empty array

When implementing real API calls, use the `provider` property (type `APIProvider`) which is already configured with JWT authentication.

### File Permissions Security
The code enforces strict file permissions:
- Config file: Automatically set to 600 (owner read/write only)
- Private key: Warns if more permissive than 600
- Always use `chmod 600` for sensitive files

### Date Validation
`CreateReportCommand` enforces:
- Date format: YYYY-MM-DD
- Maximum range: 365 days between start and end
- Start date must be before end date

### Adding New Commands
To add a new command:

1. Create `NewCommand.swift` in `Sources/AppStoreAnalyticsCLI/Commands/`
2. Add case to `Command` enum in `Command.swift`
3. Add argument parsing logic in `Command.parse()`
4. Add switch case in `main.swift` to dispatch to new command
5. Implement static `execute()` method with required parameters

### Working with Reports
Report types are strongly typed via `ReportType` enum. Each report has:
- Raw API value (e.g., "APP_STORE_PRODUCT_PAGE_VIEWS")
- Display name for user output
- Category association (discovery, commerce, usage, performance, subscriptions)

Access via:
```swift
let reportType = ReportType.appStoreProductPageViews
print(reportType.rawValue)      // "APP_STORE_PRODUCT_PAGE_VIEWS"
print(reportType.displayName)   // "App Store Product Page Views"
print(reportType.category)      // .discovery
```

## Common Development Workflows

### Adding a New Report Type
1. Add case to `ReportType` enum in `Models/ReportType.swift`
2. Add to appropriate category in `category` computed property
3. Add display name in `displayName` computed property

### Modifying Rate Limits
Edit `RateLimiter.swift`:
- Change `hourlyLimit` or `minuteLimit` properties
- Token buckets auto-refill at configured intervals

### Changing JWT Token Lifetime
Edit `JWTManager.swift`:
- Modify `tokenLifetime` constant (default: 18 minutes)
- Keep below 20 minutes (Apple's token expiry)

### Adding Progress Indicators
Use `Logger` utility for consistent output:
```swift
Logger.info("Processing...")     // [INFO] message
Logger.ok("Success")             // [OK] message
Logger.success("Complete!")      // [SUCCESS] message
Logger.error("Failed")           // [ERROR] message
```

## Dependencies and External APIs

### AppStoreConnect-Swift-SDK
The project depends on AvdLee's AppStoreConnect-Swift-SDK for:
- Type-safe API models
- JWT authentication handling
- `APIProvider` client
- `APIConfiguration` setup

Key types used from SDK:
- `APIConfiguration` - JWT credentials setup
- `APIProvider` - HTTP client for API calls
- Various analytics models (when implementing real API calls)

### App Store Connect Analytics API
Reference documentation:
- https://developer.apple.com/documentation/appstoreconnectapi/analytics
- Requires API key with Analytics permission from App Store Connect
- Rate limits: 3,600 requests/hour, exact per-minute limit varies

## Configuration File Format

`~/.appstore-analytics-config.json`:
```json
{
  "apiKeyId": "YOUR_KEY_ID",
  "appId": "YOUR_APP_ID",
  "issuerId": "YOUR_ISSUER_ID",
  "outputDirectory": "./analytics-reports",
  "privateKeyPath": "~/path/to/AuthKey_XXXXXXXXXX.p8"
}
```

File is automatically created with 600 permissions by `ConfigManager`.

## Security Considerations

- Never log or print JWT tokens
- Never commit `.p8` private key files
- Config file contains sensitive credentials - always 600 permissions
- JWT tokens cached in memory only, never written to disk
- Private key path supports tilde expansion for user-specific paths
- Validate file permissions before reading sensitive files

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lismondbernard)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lismondbernard)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
