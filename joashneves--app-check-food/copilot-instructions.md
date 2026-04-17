## app-check-food

> This is a Flutter project named **CheckFood**. It is a mobile application designed to help users manage meal planning and shopping lists. The application allows users to create "Comidas" (Meals) and associate a list of "Ingredientes" (Ingredients) with each one.

# Gemini Context: CheckFood Project

## Project Overview

This is a Flutter project named **CheckFood**. It is a mobile application designed to help users manage meal planning and shopping lists. The application allows users to create "Comidas" (Meals) and associate a list of "Ingredientes" (Ingredients) with each one.

The app is built with cross-platform support in mind, including configurations for Android, iOS, Linux, macOS, and Windows.

### Core Technologies

*   **Framework:** Flutter
*   **Language:** Dart
*   **State Management:** `provider` (using `ChangeNotifier`)
*   **Database:** `sqflite` for local persistence on a SQLite database.
*   **Testing:** `flutter_test` with `sqflite_common_ffi` to run database tests on the host machine.
*   **Linting:** `flutter_lints` for maintaining code quality.

### Architecture

The project follows a clean, layered architecture that separates concerns:

1.  **UI Layer (`lib/ui/`)**: Contains all the Flutter widgets and screens. This layer is responsible for displaying data and capturing user input. It interacts with the `providers`.
2.  **State Management Layer (`lib/providers/`)**: Uses the `provider` package to manage the application's state. `FoodProvider` and `IngredientProvider` extend `ChangeNotifier` to hold the state and notify the UI of any changes. They orchestrate data flow between the UI and the repository.
3.  **Repository Layer (`lib/model/repository/`)**: The `FoodRepository` acts as a single source of truth for data. It abstracts the underlying data source from the rest of the app, providing a clean API (e.g., `getComidas()`, `addIngrediente()`) for the providers to use.
4.  **Data Layer (`lib/db/`)**: The `DatabaseHelper` class is a singleton that directly manages the SQLite database. It handles the database connection, schema creation/migration (`_onCreate`, `_onUpgrade`), and all CRUD (Create, Read, Update, Delete) operations using raw SQL queries.
5.  **Model Layer (`lib/model/`)**: Defines the core data structures of the application:
    *   `Comida`: Represents a meal with properties like name, type, and a list of ingredients.
    *   `Ingrediente`: Represents a single ingredient with properties like name and whether it has been acquired.

## Building and Running

### 1. Install Dependencies

Ensure all the necessary packages are downloaded and linked.

```bash
flutter pub get
```

### 2. Run the Application

Execute the following command to build and run the app on a connected device or emulator.

```bash
flutter run
```

### 3. Run Tests

The project contains tests to verify the database logic. To run them, use:

```bash
flutter test
```

To run a specific test file:

```bash
flutter test test/database_test.dart
```

## Development Conventions

*   **State Management**: State is managed using the `provider` package. For global state or state that needs to be shared across screens, use a `ChangeNotifierProvider`. When the data in a provider changes, `notifyListeners()` is called to trigger a UI rebuild in the listening widgets (`Consumer` or `context.watch`).
*   **Data Flow**: The UI should never interact directly with the database. The flow is always **UI <-> Provider <-> Repository <-> DatabaseHelper**.
*   **Asynchronicity**: All database and repository operations are asynchronous and should return a `Future`. Use `async/await` to handle these operations.
*   **Database Schema**: The database schema is defined in `lib/db/database_helper.dart`. The tables are `comida` and `ingrediente`. When making changes to the schema, the database `version` in `_initDatabase` must be incremented, and a migration path must be provided in the `_onUpgrade` method.
*   **Navigation**: The app uses named routes defined in `lib/main.dart`. Use `Navigator.pushNamed()` for navigation. Data can be passed between routes using the `arguments` parameter.
*   **Code Style**: The project uses `flutter_lints` to enforce a consistent and idiomatic Dart code style.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joashneves) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
