## todomemoapp

> This project is a high-functionality ToDo list and memo application built with SwiftUI. The primary goal is to master modern UI construction and data persistence using Apple's latest frameworks. The application should be intuitive, feature-rich, and adhere to the best practices of iOS development.

# GEMINI Prompt Engineering Context for TodoMemoApp

## 1. Project Goal & Core Mission

This project is a high-functionality ToDo list and memo application built with SwiftUI. The primary goal is to master modern UI construction and data persistence using Apple's latest frameworks. The application should be intuitive, feature-rich, and adhere to the best practices of iOS development.

## 2. Technical Stack & Architecture

- **UI Framework**: SwiftUI
- **Data Persistence**: SwiftData
- **Widgets**: WidgetKit
- **Live Updates**: Live Activities
- **Architecture**: Model-View-ViewModel (MVVM)
- **Dependency Management**: Manual Dependency Injection via a `DIContainer` singleton.

The application follows a clear MVVM pattern:
- **Model**: `Item.swift` (defined with `@Model` macro).
- **View**: SwiftUI views like `ContentView`, `TodoListView`, `TaskInputView`, etc.
- **ViewModel**: `TodoListViewModel.swift` encapsulates the UI logic and state.
- **Repository**: `TodoRepository.swift` abstracts the data layer (SwiftData operations), separating data access from the ViewModel.

## 3. File-by-File Breakdown

This section provides a detailed analysis of each file in the project, explaining its purpose and key components.

---

### 📄 `.github/copilot-instructions.md`

**Purpose**: This is the high-level instruction manual for the AI assistant. It outlines the project's goals, required features, technology stack, coding conventions, and development priorities. It serves as the foundational document for any AI-driven development.

---

### 📄 `.gitignore`

**Purpose**: Standard Xcode/Swift `.gitignore` file. It correctly ignores build artifacts, user-specific settings (`xcuserdata`), Swift Package Manager caches (`.build/`), and sensitive files like `APIKeys.swift`. It's configured to keep essential project files (`project.pbxproj`) and shared schemes under version control.

---

### 📁 `TodoMemoApp.xcodeproj`

- **`project.pbxproj`**: The core Xcode project file. It defines targets, build settings, file references, and project structure. It's configured for Swift 5.0+ and iOS 18.5, with standard Debug and Release configurations.
- **`xcshareddata/xcschemes/TodoMemoApp.xcscheme`**: The shared scheme for the main application target. It defines settings for building, testing, running, profiling, and archiving the app.

---

### 📱 `TodoMemoApp/` (Main Application Source)

#### **Entry Point & Configuration**

- **`TodoMemoAppApp.swift`**: The main entry point of the application (`@main`). It sets up the `ModelContainer` for SwiftData, making it available to the entire view hierarchy via the `.modelContainer` view modifier.
- **`DIContainer.swift`**: A singleton class for Dependency Injection. It's responsible for creating and providing instances of the `TodoRepository` and `TodoListViewModel`. It is configured with the `ModelContext` from the environment to ensure the repository can access the database.

#### **Data Layer (Model & Repository)**

- **`Item.swift`**: The data model for a single ToDo item. It's a `final class` decorated with the `@Model` macro, making it a SwiftData model. It includes properties for the task description, completion status, timestamp, and a memo.
- **`TodoRepository.swift`**: Implements the `TodoRepositoryProtocol`. This class is the single source of truth for all interactions with SwiftData. It abstracts away the `ModelContext` operations (`insert`, `delete`, `fetch`, `save`) from the ViewModel, making the code cleaner and easier to test.

#### **ViewModel**

- **`TodoListViewModel.swift`**: The brain of the main screen. It's an `@Observable` class that holds the UI state (`newTask`, `items`, `editingItem`, etc.) and contains all the business logic for adding, deleting, and updating tasks. It communicates with the `TodoRepository` to perform data operations and exposes computed properties like `sortedItems` and `incompleteTasksCount` to the views. It also handles error states.

#### **Views (UI)**

- **`ContentView.swift`**: The root view of the application. It initializes the `TodoListViewModel` using the `DIContainer` and sets up the main `NavigationStack`. It composes the `TaskInputView` and `TodoListView` to build the main screen. It's also responsible for passing the `modelContext` to the `DIContainer` when the view appears.
- **`TodoListView.swift`**: A view dedicated to displaying the list of tasks. It receives the array of `Item` objects and callback closures for actions (toggle completion, edit, delete). It uses a `List` with a `ForEach` loop to render the `TodoRowView` for each item.
- **`TodoRowView.swift`**: The view for a single row in the ToDo list. It displays the task's completion status, title, and an indicator if a memo exists. It uses a `NavigationLink` to go to the `TaskDetailView`. It handles user taps for toggling completion and initiating an edit.
- **`TaskInputView.swift`**: A reusable view component for the input text field and the "add" button. It uses a `@Binding` to the `newTask` string in the ViewModel and calls the `onAddTask` closure when the user submits.
- **`EditTaskView.swift`**: A modal sheet for editing a task's title. It uses `@Bindable` on the `Item` object, which allows the `TextField` to directly modify the `task` property of the model object in real-time. The changes are automatically persisted when the user hits "Save" (which just dismisses the sheet).
- **`TaskDetailView.swift`**: A view for displaying and editing the details of a task, specifically the multi-line memo. It also uses `@Bindable` on the `Item` to bind the `TextEditor` directly to the `memo` property.

#### **Assets**

- **`Assets.xcassets`**: Contains the `AccentColor` and `AppIcon` definitions for the application. Standard Xcode asset catalog.

## 4. Development & Interaction Guidelines

Based on the context, please adhere to the following when assisting with development:

- **Follow MVVM + Repository**: All new features should respect this pattern. UI logic goes in Views, state management and business logic in ViewModels, and data persistence in Repositories.
- **Use the DIContainer**: When a view or viewmodel needs a dependency (like the repository), it should be provided by the `DIContainer`.
- **Modify via ViewModel**: Views should not modify data directly. They should call methods on the `TodoListViewModel`, which then orchestrates the necessary changes through the repository.
- **SwiftData Best Practices**: Use `@Bindable` for direct UI-to-model binding in edit views. Use the `modelContext` from the environment for data operations, preferably abstracted away in the repository.
- **Adhere to `copilot-instructions.md`**: This file is the primary source of truth for high-level requirements, coding style, and feature priorities. Refer to it for guidance.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/massan02)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/massan02)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
