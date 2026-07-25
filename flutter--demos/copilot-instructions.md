## demos

> This document outlines the best practices and coding standards to be followed

# Gemini Code-Gen Best Practices for This Project

This document outlines the best practices and coding standards to be followed
during the development of this Flutter project. Adhering to these guidelines
will ensure the codebase is clean, maintainable, and scalable.

## Architectural Principles

- **DRY (Don’t Repeat Yourself)** – eliminate duplicated logic by extracting
  shared utilities and modules.
- **Separation of Concerns** – each module should handle one distinct
  responsibility.
- **Single Responsibility Principle (SRP)** – every class/module/function/file
  should have exactly one reason to change.
- **Clear Abstractions & Contracts** – expose intent through small, stable
  interfaces and hide implementation details.
- **Low Coupling, High Cohesion** – keep modules self-contained, minimize
  cross-dependencies.
- **Scalability & Statelessness** – design components to scale horizontally and
  prefer stateless services when possible.
- **Observability & Testability** – build in logging, metrics, tracing, and
  ensure components can be unit/integration tested.
- **KISS (Keep It Simple, Sir)** - keep solutions as simple as possible.
- **YAGNI (You're Not Gonna Need It)** – avoid speculative complexity or
  over-engineering.

## Coding Standards

### Linting
This project uses the standard set of lints provided by the `flutter_lints`
package. Ensure that all code adheres to these rules to maintain code quality
and consistency. Run `flutter analyze` frequently to check for linting issues.

### Naming Conventions
- **Files:** Use `snake_case` for file names (e.g., `user_profile.dart`).
- **Classes:** Use `PascalCase` for classes (e.g., `UserProfile`).
- **Methods and Variables:** Use `camelCase` for methods and variables (e.g.,
  `getUserProfile`).
- **Constants:** Use `camelCase` for constants (e.g., `defaultTimeout`).

### Cross-Platform Compatibility
This application targets Android, iOS, web, and macOS. All code must be written
to be platform-agnostic.

-   **Avoid Platform-Specific APIs:** Do not use platform-specific libraries or
    APIs directly (e.g., `dart:io`'s `File` class for UI rendering). When
    platform-specific code is unavoidable, it is abstracted away behind a common
    interface using an adapter pattern, as seen in the `lib/platform` directory.
-   **Use Flutter-Native Solutions:** Prefer Flutter's built-in, cross-platform
    widgets and utilities (e.g., `Image.memory` with byte data for displaying
    images from `image_picker`, which works on all platforms).
-   **Verify Plugin Compatibility:** Before using a new package, ensure it
    supports all target platforms (Android, iOS, web).

### Don't Swallow Errors
- **Don't Swallow Errors** by catching expections, silently filling in required
  but missing values or adding timeouts when something hangs unexpectedly. All
  of those are exceptions that should be thrown so that the errors can be seen,
  root causes can be found and fixes can be applied.
-   **Use Assertions for Invariants:** Use `assert` statements to validate
    assumptions and logical invariants in your code. For example, if a function
    requires a list to be non-empty before proceeding, assert that condition at
    the beginning of the function. This practice turns potential silent failures
    into loud, immediate errors during development, making complex bugs
    significantly easier to track down.

### Null Value Handling
- Prefer using required parameters in constructors and methods when a value is
  not expected to be null.
- When the compiler requires a non-null value and you are certain a value is not
  null at that point, use the `!` (bang) operator. This turns invalid null
  assumptions into runtime exceptions, making them easier to find and fix.
- Avoid providing default values for nullable types simply to satisfy the
  compiler, as this can hide underlying data issues.

### Widget Development
- **`const` Constructors:** Use `const` constructors for widgets whenever
  possible to improve performance by allowing Flutter to cache and reuse widget
  instances.
- **Break Down Large Widgets:** Decompose large widget build methods into
  smaller, more manageable widgets. This improves readability, reusability, and
  performance.

### No Placeholder Code
- We're building production code here, not toys. Avoid placeholder code.

### No Comments for Removed Functionality
- The source is not the place to keep a history of what's changed; it's the
  place to implement the current requirements only. Use version control for
  history.

## Styling and Theming

### Avoid Hardcoded Values
- **Do not** hardcode colors, dimensions, text styles, or other style values
  directly in widgets.
- All centralized style-related code should be consolidated into
  `lib/styles.dart`.
- Create descriptive, `camelCase` constants in a dedicated `lib/styles.dart`
  file for any reusable style values that are not part of the main theme.

### Theme Architecture
- The app uses Material Design 3 with a centralized theme defined in
  `main.dart`.
- All UI components should inherit styles from this central theme. Avoid custom,
  one-off styling for individual widgets.
- Only use per-widget theme or style overrides when a particular widget requires
  a value that is explicitly different from the application-wide theme (e.g., a
  special-purpose button with a unique color).

#### Prioritize Blame Correctly
When debugging, assume the bug is in the local, new, application-specific code
before assuming a bug in a mature framework.

## State Management
- **Provider:** use the provider package for state management

## Testing
- Write unit tests for business logic (e.g., services, state management
  controllers).
- Write widget tests to verify the UI and interactions of your widgets.
- Aim for a reasonable level of test coverage to ensure application stability
  and prevent regressions.

## Project Structure
- **`lib/`**: Contains all Dart code.
  - **`main.dart`**: The application entry point and theme definition.
  - **`styles.dart`**: Centralized file for style constants.
  - **`models/`**: Directory for data model classes.
    - `clue_answer.dart`: Model for a clue and its answer.
    - `clue.dart`: Model for a single clue.
    - `crossword_data.dart`: Model for the entire crossword puzzle data.
    - `crossword_grid.dart`: Model for the crossword grid.
    - `crossword_state.dart`: State management for the crossword puzzle.
    - `grid_cell.dart`: Model for a single cell in the grid.
    - `todo_item.dart`: (likely unused example code)
  - **`platform/`**: Platform-specific implementations.
    - `platform_io.dart`: IO-specific implementation.
    - `platform_web.dart`: Web-specific implementation.
    - `platform.dart`: Common platform interface.
  - **`screens/`**: Top-level screen widgets.
    - `crossword_screen.dart`: The main screen of the application.
  - **`services/`**: Business logic services.
    - `gemini_service.dart`: Service for interacting with the Gemini API.
    - `image_picker_service.dart`: Service for picking images.
    - `puzzle_solver.dart`: Service for solving the puzzle.
  - **`widgets/`**: Reusable, shared widgets.
    - `clue_list.dart`: Widget for displaying the list of clues.
    - `grid_view.dart`: Widget for displaying the crossword grid.
    - `step_state_base.dart`: Base class for step state management.
    - `step1_select_image.dart`: Widget for the first step (selecting an image).
    - `step2_verify_grid_size.dart`: Widget for the second step (verifying grid
      size).
    - `step3_verify_grid_contents.dart`: Widget for the third step (verifying
      grid contents).
    - `step4_verify_clue_text.dart`: Widget for the fourth step (verifying clue
      text).
    - `step5_solve_puzzle.dart`: Widget for the fifth step (solving the puzzle).
    - `todo_list_widget.dart`: (likely unused example code)
- **`assets/`**: Contains static assets like images and fonts.
- **`test/`**: Contains tests for the application.
- **`web/`**: Contains web-specific files.
- **`macos/`**: Contains macOS-specific files.
- **`specs/`**: Contains project specifications and design documents.

## Technical Accuracy and Verification

To ensure the highest level of accuracy, the following verification steps are
mandatory when dealing with technical details like API names, library versions,
or other critical identifiers.

1.  **Prioritize Primary Sources:** Official documentation, API references, and
    the project's own source code are the highest authority. Information from
    secondary sources (e.g., blog posts, forum answers) must be cross-verified
    against a primary source before being used. When a user provides a link to
    official documentation, it must be treated as the ground truth.

2.  **Mandate Exact Identifier Verification:** When using a specific
    identifier—such as a model name, package version, or function name—you must
    find and use the **exact, literal string** from the primary source. Do not
    shorten, paraphrase, or infer the name from surrounding text or titles.

3.  **Quote Before Use:** Before implementing a critical identifier obtained
    from documentation, you must first quote the specific line or code block
    from the source that confirms the identifier. This acts as a final
    verification step to ensure you have found the precise value.

## Project-Specific Implementation

This Crossword Companion project serves as a practical example of the principles
outlined above:

-   **State Management:** The application uses the `provider` package for state
    management, with a central `CrosswordState` class that acts as a
    `ChangeNotifier`. This single source of truth manages the application's
    data, such as the puzzle details and solver status.

-   **Event-Driven Navigation:** Step transitions are handled by a robust
    two-phase state machine (`enteringStep`/`enteredStep`) within
    `CrosswordState`. This allows each step widget to listen for when it is
    being entered and run its own initialization logic in a self-contained
    manner.

-   **Abstracted State Management:** To adhere to the DRY principle, the common
    state management logic for each stepper page is encapsulated in a
    `StepStateBase` abstract class. This base class handles the listener
    registration and the two-phase state machine logic for entering a step. Each
    step's state class then extends this base class and provides its `stepIndex`
    and the specific logic to execute when the step is entered.

-   **Widget Decomposition:** The UI is broken down into small, single-purpose
    widgets. For example, the main `CrosswordScreen` is composed of a `Stepper`
    widget, which in turn uses a series of `Step...Content` widgets for each
    step in the process. This makes the code more readable, reusable, and easier
    to test.

-   **Centralized Theme:** The application's theme is defined in `main.dart` and
    applied to the entire `MaterialApp`. This ensures a consistent look and feel
    across all widgets and avoids hardcoded style values.

-   **Services:** Business logic is separated into a `GeminiService`. This
    service is configured with a detailed system prompt that instructs the
    `gemini-2.5-flash` model to act as a crossword-solving expert. This
    decouples the UI from the underlying AI logic, making the code more modular
    and easier to maintain.

-   **App-Driven Solving:** The puzzle-solving logic is not a simple API call
    but an intelligent, app-driven loop managed by a dedicated `PuzzleSolver`
    service, which is coordinated by `CrosswordState`. For each clue, the app
    calculates the word's length and current letter pattern from the grid. It
    then sends a highly focused prompt to the expert model. The app validates
    the model's response, updates the grid, and automatically retries clues that
    were answered incorrectly, creating a robust and resilient solving process.

## Verification and Maintenance

### Post-Change Verification
After any significant refactoring or feature addition, the following steps are
required to maintain code quality:

1.  **Run Static Analysis:** Execute `dart analyze` and fix all reported issues.
2.  **Audit Against Best Practices:** Review the changes against the principles
    outlined in the "Architectural Principles" and "Coding Standards" sections
    of this document to ensure the code remains clean, robust, and maintainable.

## Git Workflow

- **Committing Changes:** After the changes are complete and verified, I will not
  commit them to the repository. You, the user, are responsible for all git
  commits.

---
> Source: [flutter/demos](https://github.com/flutter/demos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
