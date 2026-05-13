## approuter

> AppRouter is a SwiftUI navigation router library that provides both simple single-stack navigation (`SimpleRouter`) and complex tab-based navigation (`Router`) with independent navigation stacks and sheet presentation. The library targets iOS 17+, macOS 14+, tvOS 17+, and watchOS 10+ using Swift 5.9+.

# AppRouter - Claude.md

## Project Overview
AppRouter is a SwiftUI navigation router library that provides both simple single-stack navigation (`SimpleRouter`) and complex tab-based navigation (`Router`) with independent navigation stacks and sheet presentation. The library targets iOS 17+, macOS 14+, tvOS 17+, and watchOS 10+ using Swift 5.9+.

## Build & Test Commands
```bash
# Build the package
swift build

# Run tests
swift test

# Build for specific platform
swift build --arch arm64 --target AppRouter

# Test with verbose output
swift test --verbose

# Check for Swift format/lint (if available)
swift-format --configuration .swift-format Sources/ Tests/
```

## Project Structure
- `Sources/AppRouter/` - Main library code
  - `Router.swift` - Tab-based navigation router
  - `SimpleRouter.swift` - Single-stack navigation router  
  - `Protocols.swift` - Core protocols (DestinationType, SheetType, TabType)
  - `Extensions.swift` - URL navigation helpers and extensions
  - `URLNavigationHelper.swift` - Deep linking utilities
- `Tests/AppRouterTests/` - Unit tests
- `Examples/` - Usage examples
- `Package.swift` - Swift Package Manager configuration

## Key Components

### Core Classes
- `Router<Tab, Destination, Sheet>` - Tab-based navigation with independent stacks per tab
- `SimpleRouter<Destination, Sheet>` - Single NavigationStack with sheet support
- Both are `@Observable @MainActor` for SwiftUI integration

### Protocols
- `DestinationType: Hashable` - Navigation destinations with URL parsing support
- `SheetType: Hashable, Identifiable` - Sheet presentations  
- `TabType: Hashable, CaseIterable, Identifiable, Sendable` - Tab definitions

### Navigation Methods
- `navigateTo(_:)` / `navigateTo(_:for:)` - Push destinations
- `popNavigation()` / `popToRoot()` - Pop navigation stack
- `presentSheet(_:)` / `dismissSheet()` - Sheet management
- `navigate(to: URL)` - Deep linking navigation

## Development Guidelines

### Code Style
- Follow Swift API Design Guidelines
- Use `@MainActor` for UI-related classes
- Prefer `@Observable` over `ObservableObject` for iOS 17+
- Use protocol-oriented design for flexibility
- Include comprehensive documentation comments

### Testing
- Unit tests in `Tests/AppRouterTests/`
- Test both `Router` and `SimpleRouter` functionality
- Mock destinations and sheets for testing
- Verify navigation state changes
- Test URL deep linking parsing

### URL Deep Linking
- Implement `DestinationType.from(path:parameters:)` for URL parsing
- URL format: `scheme://destination1/destination2?param1=value1`
- Support navigation through multiple destinations
- Handle URL validation and error cases

### Common Tasks
- Add new navigation methods to both Router classes
- Extend protocols for additional functionality  
- Update Examples/ with new features
- Ensure thread safety with `@MainActor`
- Maintain backwards compatibility within major versions

## Example Usage Patterns

### Simple Router Setup
```swift
@State private var router = SimpleRouter<Destination, Sheet>()

NavigationStack(path: $router.path) {
    // Root view
}.environment(router)
```

### Tab Router Setup  
```swift
@State private var router = Router<AppTab, Destination, Sheet>(initialTab: .home)

TabView(selection: $router.selectedTab) {
    // Tab content with NavigationStack(path: $router[tab])
}.environment(router)
```

### Navigation Usage
```swift
@Environment(AppRouter.self) private var router

router.navigateTo(.detail(id: "123"))
router.presentSheet(.settings)
router.navigate(to: "myapp://detail?id=456")
```

## Dependencies
- No external dependencies
- Uses only SwiftUI and Foundation
- Swift Package Manager for distribution

## Platform Support
- iOS 17.0+
- macOS 14.0+ 
- tvOS 17.0+
- watchOS 10.0+
- Swift 5.9+

---
> Source: [Dimillian/AppRouter](https://github.com/Dimillian/AppRouter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
