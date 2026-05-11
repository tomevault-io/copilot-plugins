## postwoman

> This file provides guidance to Claude Code (claude.ai/claude-code) when working with this codebase.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/claude-code) when working with this codebase.

## Project Overview

PostWoman 💅 is a native macOS REST API client built with SwiftUI and SwiftData. It's a Postman clone designed for testing REST APIs with folder organization and code generation.

## Tech Stack

- **Language**: Swift 5.9+
- **UI Framework**: SwiftUI 5.0
- **Data Persistence**: SwiftData
- **Networking**: URLSession with async/await
- **Minimum Target**: macOS 14.0 (Sonoma)

## Build Commands

```bash
# Build the project
xcodebuild -project PostWoman.xcodeproj -scheme PostWoman -configuration Debug build

# Build for release
xcodebuild -project PostWoman.xcodeproj -scheme PostWoman -configuration Release build

# Clean build
xcodebuild -project PostWoman.xcodeproj -scheme PostWoman clean

# Open in Xcode
open PostWoman.xcodeproj
```

## Architecture

### Data Models (SwiftData)

All models are in `PostWoman/Models/`:

- **`Folder`** - Collection containers with nested hierarchy (parent/subfolders relationship)
- **`APIRequest`** - Request configuration (URL, method, headers, body, auth)
- **`RequestHistory`** - Request/response logging

### Key Services

Located in `PostWoman/Services/`:

- **`HTTPClient`** - Actor-based URLSession wrapper for executing requests
- **`AuthenticationHandler`** - Applies Bearer/Basic auth to requests
- **`CodeGenerator/`** - Generates cURL, Swift, Python code from requests

### View Structure

The app uses a 3-column `NavigationSplitView`:

1. **Primary**: Workspace categories (History) and Collections (Folder list)
2. **Content**: Request list for the selected folder or History list
3. **Detail**: Request editor and response viewer

## Important Conventions

### Naming
- Response body property in views is named `responseBody` to avoid conflicts with SwiftUI's `body` computed property

### Notifications
Menu commands use `NotificationCenter` to communicate with views:
- `.createNewRequest`
- `.createNewFolder`
- `.sendRequest`
- `.saveRequest`
- `.duplicateRequest`
- `.generateCode`

### SwiftData Relationships
- `Folder` has cascade delete rules for both `subfolders` and `requests`

## File Organization

```
PostWoman/
├── App/
│   ├── PostWomanApp.swift      # @main entry, ModelContainer setup
│   ├── ContentView.swift       # Root NavigationSplitView
│   └── AppCommands.swift       # Menu commands and shortcuts
├── Models/                     # SwiftData @Model classes
├── Views/
│   ├── Sidebar/               # HistoryListView, FolderRequestsListView, FolderRowView, etc.
│   ├── RequestEditor/         # URLBarView, tab views
│   ├── ResponseViewer/        # Response display components
│   ├── CodeGeneration/        # Code generator sheet
│   └── Shared/                # Reusable components
├── Services/                  # Business logic
└── Utilities/                 # Helpers (JSONFormatter)
```

## Common Tasks

### Adding a New HTTP Method
1. Add case to `HTTPMethod` enum in `Models/HTTPMethod.swift`
2. Assign a color for the method badge

### Adding a New Code Generator
1. Create new generator struct in `Services/CodeGenerator/`
2. Add case to `CodeLanguage` enum in `CodeGeneratorView.swift`
3. Call generator in `generateCode()` function

### Adding a New Auth Type
1. Add case to `AuthenticationType` enum
2. Add UI in `AuthTabView.swift`
3. Handle in `AuthenticationHandler.swift`

---
> Source: [King-Bong-Software/postwoman](https://github.com/King-Bong-Software/postwoman) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
