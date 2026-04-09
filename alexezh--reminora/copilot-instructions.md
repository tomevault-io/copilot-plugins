## reminora

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Reminora (Wahi) is a geotagged photo sharing app built with SwiftUI (iOS) and Kotlin (Android). The app allows users to view their photo library, save photos with location data, view them on an interactive map, and share pins with other users. It includes cloud synchronization, user authentication, and social features like following users and commenting on pins.

## Development Commands

### Build and Run
```bash
# Build main app
xcodebuild -project reminora.xcodeproj -scheme reminora -configuration Debug build

# Run tests
xcodebuild -project reminora.xcodeproj -scheme reminora -configuration Debug test -destination 'platform=iOS Simulator,name=iPhone 15'

# Build share extension
xcodebuild -project reminora.xcodeproj -scheme ReminoraShareExt -configuration Debug build
```

### Open in Xcode
```bash
open reminora.xcodeproj
```

## Architecture

### Core Components

1. **Main App (`ios/reminora/`)**
   - `reminoraApp.swift` - SwiftUI App entry point, sets up Core Data environment
   - `ContentView.swift` - Main view controller with custom navigation system (VStack + ZStack replacing TabView)
   - `PinMainView.swift` - Interactive map showing geotagged photos with sliding panel
   - `PhotoMainView.swift` - Photo library browser with smart stacking and similarity detection
   - `SwipePhotoView.swift` - Full-screen photo viewer with swipe navigation
   - `AddPinFromPhotoView.swift` - Create pins from photos with reverse geocoding

2. **Cloud Services (`ios/reminora/cloud/`)**
   - `AuthenticationService.swift` - User authentication and session management
   - `CloudSyncService.swift` - Synchronization between local and cloud data
   - `APIService.swift` - HTTP API client and networking
   - `PinSharingService.swift` - Centralized deep link handling and pin sharing functionality
   - `UserProfileView.swift` - User profile management and following
   - `ProfileView.swift` - User profile display with debug deep link testing

3. **Pin Management (`ios/reminora/pin/`)**
   - `PinDetailView.swift` - Detailed pin view with address management
   - `SelectLocationsView.swift` - Multi-select location picker for addresses
   - `NearbyLocationsPageView.swift` - Explore nearby places and locations
   - `CommentsView.swift` - Pin comments and social interactions

4. **Backend (`backend/`)**
   - Cloudflare Workers with D1 SQLite database
   - Session-based authentication with Bearer tokens
   - RESTful API for pins, users, follows, and comments

5. **Android App (`android/`)**
   - Kotlin-based Android implementation
   - Matching functionality with iOS version

### Data Model

The app uses Core Data (iOS) with main entities:

**Place Entity:**
- `imageData: Binary` - Downsampled JPEG image data
- `location: Binary` - Archived CLLocation with GPS coordinates
- `locations: String` - JSON array of PlaceAddress objects for multiple addresses
- `dateAdded: Date` - When the photo was added
- `post: String` - Caption/text content
- `url: String` - Optional original file URL
- `cloudId: String` - Unique ID for cloud synchronization
- `isPrivate: Bool` - Privacy setting for pin sharing
- `originalUserId: String` - ID of the original pin creator (for shared pins)
- `originalUsername: String` - Username of the original creator
- `originalDisplayName: String` - Display name of the original creator

**Additional Entities:**
- `Comment` - User comments on pins
- `UserList` - Following relationships and user lists
- `ListItem` - Items within user lists

### Key Features

- **Cloud Synchronization**: Real-time sync between local storage and Cloudflare Workers backend
- **User Authentication**: Session-based auth with Google/Facebook OAuth integration
- **Reverse Geocoding**: Automatically extracts place names, cities, and countries from GPS coordinates
- **Address Management**: Multi-address support for pins with location picker
- **Social Features**: User following, pin sharing, comments, and user profiles
- **Deep Link Support**: Comprehensive deep link handling for shared pins with proper ownership tracking
- **Map Interaction**: Interactive map with pin clustering and detailed pin views
- **Photo Management**: Smart photo library integration with EXIF data extraction and AI-powered similarity detection
- **LazySnapPager**: Optimized photo swiping with robust spring animations for smooth transitions
- **Photo Stack Management**: Dynamic stacking/expanding of similar photos with boundary-aware navigation
- **Offline Support**: Local-first architecture with cloud sync when available
- **Cross-Platform**: iOS (SwiftUI) and Android (Kotlin) implementations

### Navigation System

**Modern NavigationStack Architecture:**
- **NavigationStack**: Uses iOS 16+ NavigationStack for proper navigation patterns
- **Route-Based**: AppRoute enum defines all main navigation destinations
- **Dynamic Toolbar**: Custom bottom toolbar with context-aware buttons and universal FAB
- **Route Management**: ContentView manages route switching via `currentRoute` state

**Navigation Flow:**
1. **Photos Route**: PhotoMainView with FAB-only mode (centered floating button)
2. **Map Route**: MapView with navigation toolbar (Photos, Pins, Lists + FAB)
3. **Pins Route**: PinMainView with navigation toolbar + pin actions (Sort, Add Pin, Open Invite)
4. **Lists Route**: AllRListsView with navigation toolbar + refresh action
5. **Profile Route**: ProfileView with FAB-only mode
6. **Editor Routes**: ECard and Clip editors as full-screen destinations

**Universal FAB System:**
- **Icon**: "r.circle.fill" - Blue circular button with "R" icon
- **Position**: Always at bottom of screen (centered in toolbar or floating)
- **Action**: Opens UniversalActionSheet with context-aware actions
- **ActionSheet Contents**: 
  - All tabs: Photo, Pin, List navigation buttons
  - Pins tab: Additional Sort, Add Pin, Open Invite buttons
  - Lists tab: Additional Refresh button
  - Other tabs: Settings button

**IMPORTANT - SwipePhotoView Integration:**
- SwipePhotoView MUST restore toolbar state when dismissed
- **FIX IMPLEMENTED**: Use `NotificationCenter.default.post(name: NSNotification.Name("RestoreToolbar"), object: nil)` in onDismiss
- ContentView listens for "RestoreToolbar" notification and calls `setupToolbarForTab(selectedTab)`
- This ensures the correct toolbar configuration is restored for the current tab after SwipePhotoView dismissal
- **Scroll Position Preservation**: Uses `NotificationCenter.default.post(name: NSNotification.Name("RestoreScrollPosition"), object: nil)` to restore exact scroll position in PhotoMainView using SwiftUI's native `.scrollPosition(id:)` system

**LazySnapPager Improvements:**
- **Robust Snapping**: Fixed image snapping with optimized spring animation parameters
- **Animation Config**: `response: 0.4, dampingFraction: 0.8, blendDuration: 0` prevents partial transitions
- **Performance**: Only renders previous/current/next images for memory efficiency
- **Gesture Handling**: Threshold-based swipe detection with proper boundary checking

### Technical Details

**iOS:**
- **Target**: iOS 18.2+, Swift 5.0, SwiftUI
- **Frameworks**: Core Data, MapKit, Photos, PhotosUI, Core Location, CoreLocationUI
- **Bundle ID**: `com.alexezh.reminora`
- **App Group**: `group.com.alexezh.reminora` for shared data access
- **Permissions**: Location (when in use), Photo Library access

**Backend:**
- **Platform**: Cloudflare Workers with TypeScript
- **Database**: D1 SQLite for structured data
- **Authentication**: Session-based with Bearer tokens
- **API**: RESTful endpoints for pins, users, follows, comments

**Android:**
- **Target**: Android API level TBD, Kotlin
- **Architecture**: MVVM with Room database
- **Matching iOS functionality and UI patterns

### Main Services Architecture

The app is built around key singleton services that manage global state and functionality:

**1. SelectionService (`shared/SelectionService.swift`)**
- **Purpose**: Manages multi-selection state for photos/assets across the entire app
- **Usage**: `@Environment(\.selectedAssetService)` or `SelectionService.shared`
- **Key Methods**:
  - `addSelectedPhoto(String)` / `removeSelectedPhoto(String)` - Manage photo selection
  - `isPhotoSelected(String) -> Bool` - Check selection state
  - `selectedPhotoIdentifiers: Set<String>` - Current selected photos
  - `setCurrentPhotoStack(RPhotoStack?)` - Track currently viewed photo
- **Observable**: SwiftUI reactive updates when selection changes
- **Scope**: Global across all views (PhotoMainView, SwipePhotoView, RListView, etc.)

**2. ToolbarManager (`shared/DynamicToolbar.swift`)**
- **Purpose**: Manages dynamic toolbar state and button configurations
- **Usage**: `@Environment(\.toolbarManager)`
- **Key Methods**:
  - `setFABOnlyMode()` - Show only floating action button
  - `setCustomToolbar(buttons: [ToolbarButtonConfig])` - Custom button set
  - `updateCustomToolbar(buttons: [ToolbarButtonConfig])` - Update existing toolbar
- **Integration**: Works with UniversalActionSheet for context-aware actions
- **State Management**: Tracks toolbar visibility, button configs, and FAB state

**3. SheetStack (`shared/SheetStack.swift`)**
- **Purpose**: Centralized sheet presentation management with stack-based navigation
- **Usage**: `@Environment(\.sheetStack)` or `SheetStack.shared`
- **Key Methods**:
  - `push(SheetType)` - Present new sheet
  - `pop()` - Dismiss current sheet
  - `clear()` - Dismiss all sheets
- **Sheet Types**: Enum-based type safety for all sheet presentations
- **Benefits**: Consistent sheet management, proper navigation stack handling

**4. UniversalActionSheetModel (`shared/UniversalActionSheetModel.swift`)**
- **Purpose**: Context-aware action sheet with dynamic content based on current view
- **Usage**: `UniversalActionSheetModel.shared`
- **Context Types**: `.photos`, `.pins`, `.lists`, `.swipePhoto`, `.quickList`, etc.
- **Integration**: Automatically adapts actions based on current context
- **Method**: `setContext(ActionSheetContext)` - Update available actions

**5. AuthenticationService (`cloud/AuthenticationService.swift`)**
- **Purpose**: User authentication, session management, and account state
- **Usage**: `@EnvironmentObject private var authService: AuthenticationService`
- **Key Properties**:
  - `currentAccount: Account?` - Current user account info
  - `authState: AuthState` - Authentication status (.signedIn, .signedOut, etc.)
- **OAuth Integration**: Google/Facebook authentication flows
- **Session Management**: Bearer token handling and refresh

**6. ClipManager (`clip/Clip.swift`)**
- **Purpose**: Manages clip creation, editing, and persistence
- **Usage**: `@Environment(\.clipManager)` or `ClipManager.shared`
- **Storage**: UserDefaults for primary clip data + RListData for list integration
- **Key Methods**:
  - `getAllClips() -> [Clip]` - Retrieve all clips
  - `addClip(Clip)` / `updateClip(Clip)` / `deleteClip(UUID)` - CRUD operations
- **Dual Storage**: Maintains UserDefaults as primary with RListData for integration

**7. ECardEditor (`ecard/ECardEditor.swift`)**
- **Purpose**: Manages ECard editing sessions and state
- **Usage**: `@Environment(\.eCardEditor)` or `ECardEditor.shared`
- **Session Management**: Start/end editing sessions with asset management
- **Integration**: Works with UniversalActionSheet for "Make ECard" actions

**8. ActionRouter (`shared/ActionRouter.swift`)**
- **Purpose**: Centralized action execution with dependency injection
- **Usage**: `ActionRouter.shared.execute(ActionType)`
- **Configuration**: Requires setup with SheetStack, SelectionService, ToolbarManager
- **Benefits**: Decoupled action handling, testable architecture

**9. PinSharingService (`cloud/PinSharingService.swift`)**
- **Purpose**: Centralized deep link handling and pin sharing functionality
- **Usage**: `PinSharingService.shared`
- **Key Methods**:
  - `handleReminoraLink(URL)` - Process deep links from external sources
  - `sharePin(PinData)` - Share pins with external users
  - `isSharedFromOtherUser(PinData)` - Check if pin came from another user
- **Ownership Tracking**: Maintains `originalUserId`, `originalUsername`, `originalDisplayName` for shared pins
- **Integration**: Used by reminoraApp.swift and ProfileView for deep link processing

### Service Integration Patterns

**Environment Injection:**
```swift
.environment(\.toolbarManager, toolbarManager)
.environment(\.selectedAssetService, selectedAssetService)
.environment(\.sheetStack, sheetStack)
```

**Shared Access:**
```swift
// For services that need global access
SelectionService.shared.addSelectedPhoto(assetId)
UniversalActionSheetModel.shared.setContext(.photos)
ActionRouter.shared.execute(.sharePhoto(asset))
```

**Reactive Updates:**
- All services use `@Published` properties for SwiftUI reactivity
- Views automatically update when service state changes
- ObservableObject conformance enables `@StateObject` and `@ObservedObject` usage

**Notification Integration:**
- Services use NotificationCenter for loose coupling
- Example: "RestoreToolbar" notification for view transitions
- Cross-service communication without tight dependencies

### Common Patterns

- All views use `@Environment(\.managedObjectContext)` for Core Data access
- `@FetchRequest` with `Place` entity for reactive data updates
- `PersistenceController.shared` singleton for Core Data operations
- Async image loading with `PHImageManager` for photo thumbnails
- Location updates via `@StateObject private var locationManager = LocationManager()`
- LazySnapPager for memory-efficient horizontal photo swiping with robust animations
- UniversalActionSheet context management via `UniversalActionSheetModel.shared.setContext()`
- Toolbar state restoration using NotificationCenter for view transitions
- Service-based architecture with dependency injection and reactive state management

### Testing

- `reminoraTests/` - Unit tests for main app
- `reminoraUITests/` - UI tests for main app
- Run tests via Xcode Test navigator or command line with `xcodebuild test`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alexezh)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/alexezh)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
