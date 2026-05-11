## trmnl-companion

> This is a Calendar Sync iOS app built for TRMNL that fetches calendar events via EventKit and syncs them to a remote service. The app is designed to run on iOS 17.0+ and uses SwiftUI with SwiftData for persistence.

# CLAUDE.md - AI Assistant Project Context

## Project Overview
This is a Calendar Sync iOS app built for TRMNL that fetches calendar events via EventKit and syncs them to a remote service. The app is designed to run on iOS 17.0+ and uses SwiftUI with SwiftData for persistence.

## Key Implementation Details

### Architecture
- **MVVM Pattern**: Views → ViewModels → Services → Models
- **Singleton Services**: CalendarService and APIService use shared instances
- **SwiftData**: Used for sync history persistence
- **@Observable macro (iOS 17+)**: Modern observation framework replacing ObservableObject
- **Result Types**: Type-safe error handling with Result<Success, Error> pattern
- **ErrorKit Integration**: Consistent user-friendly error messages via Throwable protocol

### Error Handling System
The app uses typed errors with ErrorKit for comprehensive error handling.

Read https://github.com/FlineDev/ErrorKit to know more about ErrorKit usage

```swift
// API Layer - Strongly typed network/HTTP errors
enum APIError: Throwable {
    case invalidURL
    case encodingFailed(String)
    case networkError(NetworkErrorType)
    case httpError(HTTPErrorType)
    case invalidResponse
    
    var userFriendlyMessage: String { ... }
}

// Service Layer - Domain-specific errors that wrap lower-level errors
enum SyncError: Throwable {
    case noCalendarsSelected
    case noEventsFound
    case apiError(APIError)  // Composition pattern
    case persistenceError(String)
    
    var userFriendlyMessage: String { ... }
}
```

#### Result Pattern Usage
```swift
// API Service returns Result with event count on success
func syncEvents(_ events: [EventModel]) async -> Result<Int, APIError>

// Sync Service returns Result with rich success data
func performSync(...) async -> Result<SyncSuccess, SyncError>

// Success type carries meaningful data
struct SyncSuccess {
    let eventCount: Int
    let syncDuration: TimeInterval
    let timestamp: Date
}
```

#### Error Propagation Flow
1. Network/HTTP errors originate in APIService as `APIError`
2. SyncService wraps API errors and adds domain-specific errors as `SyncError`
3. ViewModels extract user-friendly messages via `error.userFriendlyMessage`
4. UI displays consistent, helpful error messages to users

### Calendar Identifier Strategy
```swift
// In EventModel.swift
self.calendarIdentifier = event.calendarItemExternalIdentifier ?? event.calendarItemIdentifier
```
- Primary: `calendarItemExternalIdentifier` - Best for cross-device consistency
- Fallback: `calendarItemIdentifier` - Stable on device
- **Known Issue**: Apple warns about possible duplicates with external IDs
- **Solution**: Remote service should deduplicate using combination of calendar_identifier + start_full + summary

### Memory Management
- Single `EKEventStore` instance maintained in CalendarService
- Event enumeration instead of array building for memory efficiency
- Date range hardcoded: today -6 to today +30 days

## @Observable Migration (iOS 17+)

### Why We Migrated
The app was migrated from `ObservableObject` to the new `@Observable` macro to simplify observation and avoid data flow issues, such as with nested objects.

### What Changed
```swift
// Before: ObservableObject + @Published
class SomeViewModel: ObservableObject {
    @Published var property = value
}

// After: @Observable
@Observable
class SomeViewModel {
    var property = value  // No @Published needed
}
```

### View Updates
- `@StateObject` → `@State` for owned objects
- `@ObservedObject` → Direct reference or `@Bindable` for two-way binding
- Automatic observation of nested objects

### Key Benefits
- Cleaner code with less boilerplate
- Automatic nested object observation
- Better performance (only observes used properties)
- No manual Combine subscriptions needed

## API Integration

### Production Endpoints
- **Base URL**: `https://usetrmnl.com/api`
- **Authentication**: Bearer token with API key in Authorization header

### Core Endpoints

#### 1. GET /me
- Validates API key and retrieves user information
- Returns: User object with name, email, timezone, API key
- Used for: Initial authentication validation

#### 2. GET /plugin_settings?plugin_id=58
- Fetches plugin settings for calendar plugin (ID: 58)
- Returns: Array of PluginSetting objects with id, name, pluginId
- Used for: Getting plugin setting IDs for data updates

#### 3. POST /plugin_settings/{id}/data
- Updates plugin with calendar events
- Payload structure:
```json
{
  "merge_variables": {
    "events": [
      // Array of EventModel objects
    ]
  }
}
```
- Returns: Success status
- Used for: Syncing calendar events to TRMNL

### Plugin System
- **Plugin ID 58**: Hardcoded calendar plugin identifier
- **One Request Per Plugin**: Events from multiple calendars are merged before sending
- **Plugin Mapping**: Calendars are mapped to plugins via PluginMappingManager
- **Persistence**: Plugin mappings stored in UserDefaults

## Authentication System

### API Key Storage
- **Storage**: AppStorage (UserDefaults wrapper) with key `"TRMNLAPIKey"`
- **No Keychain**: Simplified to use AppStorage instead of Keychain
- **Validation**: API key validated via /me endpoint before saving

### Login Methods

#### 1. Browser-Based (Primary)
- Uses SFSafariViewController for in-app browser
- Opens https://usetrmnl.com/account
- Clipboard monitoring every 0.5 seconds
- Auto-detects API key patterns (user_, key_, api_, tk_, etc.)
- Auto-closes browser on key detection

#### 2. Manual Entry (Fallback)
- SecureField in Settings for direct paste
- Real-time validation with server
- Shows progress indicator during validation

## Event Syncing
- Events sent as full sync (no delta/diff)
- Events merged by plugin before sending
- Date range: today -6 to today +30 days

## Known Issues & Resolved Bugs

### Resolved Issues
1. **Sync View Refresh Bug**: View wasn't updating when API key was set
   - **Solution**: Implemented observableApiKey mirror property pattern
   - **Added**: onAppear, onChange(selectedTab), improved refreshable

2. **Plugin Selection Indicators**: Selection UI wasn't updating
   - **Problem**: Passing struct copies instead of references
   - **Solution**: Pass plugin IDs and use computed properties

3. **@AppStorage with @Observable Conflict**: Compilation errors
   - **Problem**: @AppStorage generates computed properties incompatible with @Observable
   - **Solution**: Use @ObservationIgnored on AppStorage, mirror to observable property

### Potential Improvements
1. **Error Recovery**: Add retry logic for failed syncs with exponential backoff if needed
2. **Background Sync**: Implement background refresh for periodic syncing
3. **Conflict Resolution**: Handle calendar event conflicts/duplicates on server
4. **Analytics**: Add sync success/failure metrics with error type tracking
5. **Widget Extension**: Show last sync status in widget
6. **Sync Scheduling**: Allow users to schedule automatic syncs

## Testing Guidelines

### Manual Testing Checklist
- [ ] Login flow (browser-based and manual)
- [ ] API key clipboard detection
- [ ] Calendar permissions request flow
- [ ] Plugin mapping UI and persistence
- [ ] Tab switching refresh behavior
- [ ] Pull-to-refresh in all states
- [ ] Online/offline state transitions
- [ ] Calendar selection persistence
- [ ] Sync with 0, 1, many calendars per plugin
- [ ] Sync with no events
- [ ] Sync with many events (memory test)
- [ ] Error handling (network failure, API errors, invalid key)
- [ ] Light/dark mode appearance
- [ ] iPad layout
- [ ] Logout and re-login flow

### Common Debug Commands
```bash
# Run Periphery for unused code detection
./run_periphery.sh --clean
```

## Project Structure
```
Companion/
├── Models/                 # Data models
│   ├── EventModel.swift   # Event → JSON conversion
│   ├── CalendarModel.swift # Calendar selection state
│   ├── PluginModel.swift  # Plugin data and mapping manager
│   ├── SyncHistory.swift  # SwiftData sync records
├── Services/              # Business logic services
│   ├── CalendarService.swift # EventKit wrapper
│   ├── SyncService.swift  # Sync orchestration with plugin support
│   └── APIService.swift   # TRMNL API integration
├── ViewModels/            # View business logic
│   ├── SyncViewModel.swift # Sync tab logic with plugin mapping
│   └── SettingsViewModel.swift # Settings with auth management
├── Views/                 # SwiftUI views
│   ├── SyncView.swift     # Main sync interface
│   ├── SettingsView.swift # Settings with login flow
│   ├── PluginMappingView.swift # Plugin-calendar mapping UI
│   ├── SafariView.swift   # In-app browser with clipboard monitoring
│   └── Components/        # Reusable components
│       ├── SyncButton.swift
│       ├── StatusIndicator.swift
│       └── PluginMappingCard.swift # Plugin mapping card UI
├── Theme/                 # Styling
│   └── AppTheme.swift     # Colors and constants
├── ContentView.swift      # Tab container with binding
└── CompanionApp.swift     # App entry point
```

## State Management Patterns

### API Key Management
- Stored in AppStorage with key `"TRMNLAPIKey"`
- Observable property pattern for @Observable compatibility:
  - `apiKey`: @AppStorage property (marked @ObservationIgnored)
  - `observableApiKey`: Mirror property for view observation
  - Synchronized in `initialize()` method

### Plugin Mappings
- Managed by `PluginMappingManager` class
- Stored in UserDefaults with key `"trmnl_plugin_mappings"`
- Structure: `[PluginModel]` with `mappedCalendarIds` sets
- Calendars are now mapped to plugins instead of being independently selected

### Sync State
- Managed by `SyncService` (@Observable class)
- Last sync info stored in UserDefaults
- Keys: `"LastSyncDate"`, `"LastSyncStatus"`
- All properties automatically observable (no @Published needed)
- Use @ObservationIgnored to ignore properties that don't need observation

## Build Configuration
- Bundle ID: `com.usetrmnl.app`
- Deployment Target: iOS 17.0
- Entitlements & Permissions:
  - Calendar access (full)

## Common Pitfalls to Avoid
1. Don't create multiple EKEventStore instances
2. Don't forget to handle iOS 17+ calendar permissions differently
3. Don't use ObservedObject, use modern @Observable instead
4. Use `@Bindable` when passing @Observable objects to child views that need two-way binding
5. Remember to test with calendars that have recurring events
6. Don't use string-based errors - always use typed errors and / or throw, if needed with Result<T,E>
7. Always provide userFriendlyMessage for custom errors via ErrorKit's Throwable
8. Don't ignore error context - wrap lower-level errors to preserve the error chain
9. Test error paths thoroughly - network failures, API errors, empty states
10. **@AppStorage with @Observable**: Can't use @AppStorage directly as observable property
    - Solution: Use mirror property pattern (observableApiKey)
11. **Plugin Updates**: Pass plugin IDs instead of structs to avoid SwiftUI update issues
12. **Single API Request**: Ensure ONE request per plugin regardless of calendar count
13. **Clipboard Monitoring**: Check clipboard content changes, not just existence

## Future Feature Ideas
- Widget support for last sync status
- Apple Watch companion app for easier sync
- Siri Shortcuts integration
- CloudKit sync for settings

## Key Implementation Decisions

### Why AppStorage Instead of Keychain
- Simplified implementation for non-sensitive API keys
- Better integration with SwiftUI property wrappers
- Automatic persistence without additional dependencies
- API keys are already visible in TRMNL account page

### Why Plugin-Based Architecture
- Allows users to organize calendars logically
- Supports multiple TRMNL displays/widgets
- Reduces API calls (one per plugin vs one per calendar)
- Future-proof for additional plugin types

### Why Clipboard Monitoring
- Seamless UX - users don't need to manually paste
- Common pattern in modern apps
- Fallback manual entry still available
- Auto-closes browser for clean flow

## Contact & Resources
- TRMNL Platform: https://usetrmnl.com
- TRMNL Account/API Key: https://usetrmnl.com/account
- EventKit Documentation: https://developer.apple.com/documentation/eventkit
- SwiftData: https://developer.apple.com/documentation/swiftdata
- ErrorKit: https://github.com/FlineDev/ErrorKit

---
*Last updated: 2025-08-29*
*This file helps AI assistants understand the project context for future work sessions.*
*Key changes: Added TRMNL API integration, plugin system, browser-based auth, and resolved sync view refresh issues.*

---
> Source: [usetrmnl/trmnl-companion](https://github.com/usetrmnl/trmnl-companion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
