## immigru-app

> Manages data fetching/transformation (via models, datasources)


mmigru Architecture Guidelines (Feature-First Clean Architecture)
🧭 Guiding Principles
The Immigru application follows:

Clean Architecture for maintainability, testability, and scalability

Feature-First Organization for modular development and better ownership

Dependency Inversion for clean layering and mocking in tests

📁 Folder Structure Overview
bash
Copy
Edit
/lib
  /features
    /<feature_name>
      ├── data
      ├── domain
      ├── presentation
      └── di
  /core
  /shared
  main.dart
🔍 Inside a Feature (e.g., auth)
bash
Copy
Edit
/features/auth
  ├── data
  │   ├── datasources         # API clients, local storage access
  │   ├── models              # DTOs and serializers
  │   └── repositories        # Implements domain interfaces
  ├── domain
  │   ├── entities            # Business models
  │   ├── repositories        # Abstract contracts
  │   ├── usecases            # Business operations
  │   └── utils               # Feature-specific utilities
  ├── presentation
  │   ├── bloc                # State management
  │   ├── routes              # Navigation config
  │   ├── screens             # UI pages
  │   └── widgets             # Shared UI components
  └── di                      # Dependency injection for this feature
🧱 Layered Architecture in Each Feature
Presentation Layer
Contains screens, widgets, BLoCs/Cubits

Calls use cases via BLoCs

No direct data access or business logic

Domain Layer
Defines Entities, UseCases, Repositories

Pure Dart (no Flutter or data package imports)

Interfaces that the data layer implements

Data Layer
Implements domain Repository interfaces

Manages data fetching/transformation (via models, datasources)

Maps between domain entities and data models

🔁 Dependency Rule
Dependencies point inward:

Presentation → Domain

Data → Domain

Domain → Core only

Shared utilities go in core or shared

🛠️ Implementation Steps (New Feature)
Define the Domain Layer

Create entities, repository interfaces, and use cases

Build the Data Layer

Implement repository

Setup data sources and models

Map models ↔ entities

Setup Presentation Layer

Create screens and widgets

Implement state management (Bloc/Cubit)

Trigger use cases via events

Register Dependencies

Use the feature’s di/ folder

Add to global container via core/di/modules/

🎨 UI Guidelines
Design & UX
Follow Material Design

Use shared/widgets/ for reusable UI

Support animations and transitions

Theming
Define all theming in shared/theme/

Use ThemeProvider for light/dark support

Error Handling
Display errors via shared/widgets/error_message_widget.dart

Handle all states: loading, success, error

🔐 Authentication Flow
Phone & Email Auth
Separate BLoCs and screens for each auth step

Clear error feedback and form validation

Use dependency injection via auth/di/

♻️ Shared & Core Usage
shared/
Widgets: Common UI components

Theme: App-wide colors and styles

core/
Storage: Secure and local storage

Network: API clients and interceptors

DI: Global dependency registration

Config: Static app configuration

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WellingtonDevBR) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
