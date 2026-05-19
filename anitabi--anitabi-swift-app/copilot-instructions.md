## swiftui-project-structure-rules

> Enforces a specific project structure for SwiftUI projects with main files, views, shared components, models, view models, services, utilities, resources and tests.

- Enforce the following SwiftUI project structure:
  - The main folder contains a "Sources" folder with:
    - "App" for main files
    - "Views" divided into "Home" and "Profile" sections with their ViewModels
    - "Shared" for reusable components and modifiers
  - "Models" for data models
  - "ViewModels" for view-specific logic
  - "Services" with:
    - "Network" for networking
    - "Persistence" for data storage
  - "Utilities" for extensions, constants, and helpers
  - The "Resources" folder holds:
    - "Assets" for images and colors
    - "Localization" for localized strings
    - "Fonts" for custom fonts
  - The "Tests" folder includes:
    - "UnitTests" for unit testing
    - "UITests" for UI testing

---
> Source: [anitabi/anitabi-swift-app](https://github.com/anitabi/anitabi-swift-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
