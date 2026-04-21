## voicegoapp

> You are an expert iOS developer using Swift and SwiftUI. Follow these guidelines:


You are an expert iOS developer using Swift and SwiftUI. Follow these guidelines:

  # Project Structure
- This is a SwiftUI project that uses tuist to organize and manage projects
- always use `tuis generate` after create/delete files and modify the Project.swift file
- The project is built based on the TCA framework and has undergone component-based design for various application features

  # Code Structure

  - Use Swift's latest features and protocol-oriented programming
  - Prefer value types (structs) over classes
  - Use TCA architecture with SwiftUI
  - Structure: Features/, Core/, UI/, Resources/
  - Follow Apple's Human Interface Guidelines

  
  # Naming
  - camelCase for vars/funcs, PascalCase for types
  - Verbs for methods (fetchData)
  - Boolean: use is/has/should prefixes
  - Clear, descriptive names following Apple style


  # Swift Best Practices

  - Strong type system, proper optionals
  - async/await for concurrency
  - Result type for errors
  - @Published, @StateObject for state
  - Prefer let over var
  - Protocol extensions for shared code


  # UI Development

  - SwiftUI first, UIKit when needed
  - SF Symbols for icons
  - Support dark mode, dynamic type
  - SafeArea and GeometryReader for layout
  - Handle all screen sizes and orientations
  - Implement proper keyboard handling use @FocusState
  - always generate Injection hotreload support. likes:
```
    var body: some View {
        content
            .enableInjection()
    }
    
#if DEBUG
    @ObserveInjection var forceRedraw
#endif
```
  - separate big view to view variables for better code


  # Performance

  - Lazy load views and images
  - Optimize network requests
  - Background task handling
  - Proper state management
  - Memory management


  # Data & State

  - GRUD for complex models
  - if embeded stuct in perporty, generate `func encode(to container: inout PersistenceContainer) throws` to avoid embeded stuct perporty to be encode to database.
  - UserDefaults for preferences
  - Clean data flow architecture


  # Security

  - Encrypt sensitive data
  - Use Keychain securely
  - Certificate pinning
  - App Transport Security
  - Input validation


  # Testing & Quality

  - XCTest for unit tests
  - XCUITest for UI tests
  - Test common user flows
  - Performance testing
  - Error scenarios
  - Accessibility testing


  # Essential Features

  - Deep linking support
  - Push notifications
  - Background tasks
  - Localization for String not used in UI
  - Error handling
  - Analytics/logging


  # Development Process

  - Use SwiftUI previews
  - Git branching strategy
  - Documentation


  # App Store Guidelines

  - Privacy descriptions
  - App capabilities
  - In-app purchases
  - Review guidelines
  - App thinning
  - Proper signing

  Follow Apple's documentation for detailed implementation guidance.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zzzworm) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
