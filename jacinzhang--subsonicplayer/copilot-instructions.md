## subsonicplayer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SubsonicPlayer is an iOS audio player app for Subsonic servers, built with SwiftUI. The project implements a simplified Clean Architecture pattern optimized for Subsonic API characteristics.

## Build and Development Commands

### Build
```bash
xcodebuild -project SubsonicPlayer.xcodeproj -scheme SubsonicPlayer build
```

### Open in Xcode
```bash
open SubsonicPlayer.xcodeproj
```

### Run
Use Xcode (⌘+R) to run on iOS Simulator or device.

## Architecture

### Layered Structure

The app follows a simplified Clean Architecture with clear separation of concerns:

```
LoginView (UI)
    ↓
LoginViewModel (State Management)
    ↓
LoginUseCase (Business Logic)
    ↓
LoginRepository (Data Access)
    ↓
NetworkService (Network Layer)
```

### Key Design Decisions

**Simplified Architecture**: The codebase intentionally avoids over-abstraction. After initial implementation with extensive protocol layers and token storage (see SIMPLIFIED_ARCHITECTURE.md), the architecture was simplified by:
- Removing unnecessary protocol abstractions for Repository and NetworkService
- Eliminating complex AuthToken storage (Subsonic doesn't provide real tokens)
- Using concrete implementations where protocols don't add value

**Why This Matters**: When adding new features, follow the existing simplification philosophy. Use protocols only for UseCases where they provide clear testing value. Avoid creating protocol abstractions for every class.

### Directory Structure

- **Models/**: Request/response data structures with validation logic
- **Repositories/**: Data access layer containing `LoginRepository` and `NetworkService`
- **UseCases/**: Business logic layer with protocol definitions
- **ViewModels/**: SwiftUI state management with `@Published` properties
- **Injection/**: Dependency injection container (`DIContainer`)
- **Views/**: SwiftUI UI components

## Subsonic API Integration

### Critical: HTTP 200 Does NOT Mean Success

**The most important thing to understand about Subsonic API:**

Subsonic returns HTTP 200 even for authentication failures. Business errors are indicated in the JSON response.

**Example - Wrong Password:**
```bash
curl "http://server:4533/rest/ping.view?u=user&p=wrong&v=1.16.0&c=client&f=json"

# Response: HTTP 200 OK
{
  "subsonic-response": {
    "status": "failed",  // ← Business error here
    "error": {"code": 40, "message": "Wrong username or password"}
  }
}
```

**Correct Error Handling Pattern:**

1. **NetworkService** (Transport Layer):
   - Only handles HTTP connection errors (timeout, network unreachable)
   - Only handles JSON decoding errors
   - Does NOT parse business errors from JSON
   - Returns decoded `LoginAPIResponse` for any valid JSON

2. **LoginRepository** (Business Layer):
   - Checks `response.subsonicResponse.status` field
   - If `status == "failed"`: extract error message, return failure response
   - If `status == "ok"`: validate required fields, return success response

**Anti-Pattern to Avoid:**
```swift
// ❌ WRONG: Parsing Subsonic errors in NetworkService
private func parseSubsonicError(from data: Data) -> String? {
    // This violates layer separation
}
```

**Correct Pattern:**
```swift
// ✅ CORRECT: Business layer checks status
private func mapToLoginResponse(...) -> LoginResponse {
    if response.subsonicResponse.status == "failed" {
        return LoginResponse(success: false, errorMessage: ...)
    }
    // Process success case
}
```

### Authentication Method

Subsonic uses **query parameter authentication** (not Basic Auth, not tokens):

```
GET /rest/ping.view?u=username&p=password&v=1.16.0&c=SubsonicPlayer&f=json
```

Parameters:
- `u`: Username
- `p`: Password (plain text in query)
- `v`: API version (1.16.0)
- `c`: Client name (SubsonicPlayer)
- `f`: Response format (json)

### Response Structure

All responses wrapped in `subsonic-response`:
```json
{
  "subsonic-response": {
    "status": "ok" | "failed",
    "version": "1.16.1",
    "type": "navidrome",
    "serverVersion": "0.53.3",
    "openSubsonic": true,
    "error": {  // Only present when status == "failed"
      "code": 40,
      "message": "Wrong username or password"
    }
  }
}
```

## Key Implementation Details

### Dependency Injection
All dependencies are registered in `DIContainer.shared`:
```swift
let container = DIContainer.shared
container.registerDependencies()
let viewModel = container.makeLoginViewModel()
```

### Reactive State Management
Uses Combine framework with `@Published` properties for reactive UI updates. ViewModels subscribe to UseCase publishers and update state accordingly.

### Input Validation
`LoginRequest` model includes `validate()` method that checks:
- Non-empty server address, username, password
- Valid URL format for server address

### Error Handling
Multi-layer error handling:
1. Model validation errors
2. Network errors (connectivity, timeouts)
3. HTTP status code errors
4. Subsonic API errors (parsed from error element)

## Testing Considerations

The architecture supports testing through:
- Protocol-based UseCases (can be mocked)
- Dependency injection (can substitute implementations)
- Separate layers (can test in isolation)

## Network Configuration

- **Timeout**: 30 seconds (configured in `NetworkService.Configuration`)
- **Error Types**: Custom `NetworkError` and `LoginRepositoryError` enums
- **Threading**: All network responses dispatched to main queue via Combine

## Common Pitfalls to Avoid

1. **Don't assume HTTP status code indicates business success** - Always check JSON `status` field
2. **Don't add protocols everywhere** - Use concrete classes unless testing requires protocols
3. **Don't parse business errors in NetworkService** - That's the Repository's job
4. **Don't force-unwrap optional fields in SubsonicResponse** - They may be missing in error responses

## Future Extensions

When implementing new features (music library, playback, search), follow the established pattern:
1. Create request/response models with validation
2. Add methods to Repository (concrete class, no protocol unless needed)
3. Create UseCase with protocol for testability
4. Build ViewModel with `@Published` state
5. Implement SwiftUI view

Available Subsonic API endpoints for future work:
- `/rest/getMusicFolders.view`
- `/rest/stream.view`
- `/rest/search2.view`
- `/rest/getRandomSongs.view`

---
> Source: [jacinzhang/SubsonicPlayer](https://github.com/jacinzhang/SubsonicPlayer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
