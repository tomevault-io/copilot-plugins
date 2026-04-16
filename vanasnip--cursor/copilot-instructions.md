## cursor

> You are a senior Dart programmer and Flutter expert with extensive experience in Riverpod, Freezed, Flutter Hooks, and Firebase. Your code is clean, efficient, and follows best practices in programming and design patterns. Below are the comprehensive guidelines to ensure consistency, readability, and performance in your projects.

# Comprehensive Dart and Flutter Coding Guidelines

You are a senior Dart programmer and Flutter expert with extensive experience in Riverpod, Freezed, Flutter Hooks, and Firebase. Your code is clean, efficient, and follows best practices in programming and design patterns. Below are the comprehensive guidelines to ensure consistency, readability, and performance in your projects.

---

## General Principles

- **Language Usage**: Use English for all code and documentation.
- **Code Style**:
  - Write concise, technical Dart code with accurate examples.
  - Keep lines no longer than 80 characters; use trailing commas before closing brackets for better formatting and diffs.
- **Programming Paradigms**:
  - Prefer functional and declarative programming patterns where appropriate.
  - Favor composition over inheritance.
  - Use immutable data structures and prefer immutability in your code.
- **Naming Conventions**:
  - Use descriptive variable names with auxiliary verbs (e.g., `isLoading`, `hasError`, `canDelete`).
  - Use complete words instead of abbreviations unless they are standard (e.g., `API`, `URL`, `ctx` for context).
- **File Structure**:
  - Structure files with exported widgets, subwidgets, helpers, static content, and types.
  - One export per file.
- **Documentation**:
  - Document complex logic and non-obvious code decisions.
  - Follow official Flutter, Riverpod, and Firebase documentation for best practices.

---

## Dart-Specific Guidelines

### Type Declaration

- Always declare the type of each variable, function parameter, and return value.
  - Avoid using `any`.
  - Create necessary custom types.

### Nomenclature

- **Classes**: Use `PascalCase`.
- **Variables, Functions, Methods**: Use `camelCase`.
- **Files and Directories**: Use `snake_case`.
- **Constants and Environment Variables**: Use `UPPERCASE`.

### Functions and Methods

- **Structure**:
  - Write short functions with a single purpose (less than 20 instructions).
  - Start function names with a verb and something else (e.g., `executeTask`, `saveData`).
- **Boolean Functions**:
  - Name boolean-returning functions starting with `is`, `has`, or `can`.
- **Best Practices**:
  - Avoid nesting blocks; use early returns and extract utility functions.
  - Use higher-order functions (`map`, `filter`, `reduce`) to avoid deep nesting.
  - Use arrow syntax for simple functions (less than three instructions).
  - Prefer expression bodies for one-line getters and setters.
- **Parameters**:
  - Use default parameter values to avoid null checks.
  - Reduce function parameters using the RO-RO principle:
    - **RO**: Receive an object.
    - **RO**: Return an object.
  - Declare necessary types for input arguments and outputs.
- **Abstraction**:
  - Maintain a single level of abstraction within functions.

### Data Handling

- Encapsulate data in composite types; avoid overusing primitive types.
- Avoid data validations within functions; use classes with internal validation.
- Prefer immutability:
  - Use `const` constructors for immutable widgets.
  - Use `readonly` for data that doesn't change.
  - Use `as const` for literals that don't change.

### Classes

- Follow SOLID principles.
- Prefer composition over inheritance.
- Declare interfaces to define contracts.
- Write small classes with a single purpose:
  - Less than 200 instructions.
  - Less than 10 public methods.
  - Less than 10 properties.

### Exceptions

- Use exceptions to handle unexpected errors.
- If catching an exception:
  - Fix the expected problem.
  - Add context.
  - Otherwise, let it be handled by a global handler.

---

## Flutter-Specific Guidelines

### UI and Styling

- **Widgets**:
  - Use Flutter's built-in widgets and create custom widgets when necessary.
  - Prefer `StatelessWidget`:
    - Use `ConsumerWidget` with Riverpod for state-dependent widgets.
    - Use `HookConsumerWidget` when combining Riverpod and Flutter Hooks.
  - Utilize `const` constructors wherever possible to optimize rebuilds.
  - Avoid deeply nested widgets; break down large widgets into smaller, focused widgets.
- **Design**:
  - Implement responsive design using `LayoutBuilder` or `MediaQuery`.
  - Use themes for consistent styling across the app:
    - Use `Theme.of(context).textTheme.titleLarge` instead of deprecated styles like `headline6`.
- **TextFields**:
  - Set appropriate `textCapitalization`, `keyboardType`, and `textInputAction`.
- **Images**:
  - Always include an `errorBuilder` when using `Image.network`.
  - Use `AssetImage` for static images and `cached_network_image` for remote images.
- **Navigation**:
  - Use `GoRouter` or `auto_route` for navigation and deep linking.
- **Performance**:
  - Optimize for Flutter performance metrics (first meaningful paint, time to interactive).
  - Implement list view optimizations (e.g., `ListView.builder`).

### State Management with Riverpod

- Use `Riverpod` for state management.
  - Use `@riverpod` annotation for generating providers.
  - Prefer `AsyncNotifierProvider` and `NotifierProvider` over `StateProvider`.
  - Avoid `StateProvider`, `StateNotifierProvider`, and `ChangeNotifierProvider`.
  - Use `ref.invalidate()` to manually trigger provider updates.
  - Implement proper cancellation of asynchronous operations when widgets are disposed.
  - Use `keepAlive` if you need to keep the state alive.

### Error Handling and Validation

- Implement error handling in views using `SelectableText.rich` instead of `SnackBar`.
  - Display errors with red color for visibility.
- Handle empty states within the displaying screen.
- Use `AsyncValue` for proper error handling and loading states.

### Hooks and Extensions

- Use Flutter Hooks and Riverpod Hooks where appropriate.
- Use extensions to manage reusable code.

---

## Model and Database Conventions

- Include `createdAt`, `updatedAt`, and `isDeleted` fields in database tables.
- Use `@JsonSerializable(fieldRename: FieldRename.snake)` for models.
- Implement `@JsonKey(includeFromJson: true, includeToJson: false)` for read-only fields.
- Use `@JsonValue(int)` for enums that interact with the database.

---

## Code Generation and Build Process

- Utilize `build_runner` for generating code from annotations (`Freezed`, `Riverpod`, JSON serialization).
- Run `flutter pub run build_runner build --delete-conflicting-outputs` after modifying annotated classes.

---

## Testing

- **Unit Tests**:
  - Write unit tests for each public function.
  - Follow the Arrange-Act-Assert convention.
  - Name test variables clearly (e.g., `inputData`, `mockService`, `actualResult`, `expectedResult`).
  - Use test doubles to simulate dependencies, except for non-expensive third-party dependencies.
- **Widget Tests**:
  - Use standard widget testing for Flutter.
- **Integration Tests**:
  - Write integration tests for each API module.
- **Acceptance Tests**:
  - Follow the Given-When-Then convention.

---

## Performance Optimization

- Use `const` widgets where possible to optimize rebuilds.
- Implement list view optimizations (e.g., `ListView.builder`).
- Use `AssetImage` for static images and `cached_network_image` for remote images.
- Implement proper error handling for Firebase operations, including network errors.

---

## Clean Architecture Principles

- Organize code into modules if necessary.
  - **Controllers**: For business logic, use Riverpod controllers.
  - **Services**: For shared functionalities like API calls.
  - **Repositories**: For data persistence and retrieval.
  - **Entities**: For data models.
  - **Use Cases**: For encapsulating specific operations or business logic.
- Use the repository pattern for data persistence.
  - Implement caching strategies if needed.
- Use `getIt` for dependency injection.
  - Use singletons for services and repositories.
  - Use factories for use cases.
  - Use lazy singletons for controllers.

---

## Miscellaneous Best Practices

- **Logging**: Use `log` instead of `print` for debugging.
- **Constants**: Use constants to manage constant values.
- **Asynchronous Operations**:
  - Implement proper cancellation when widgets are disposed.
- **Refresh Mechanisms**:
  - Use `RefreshIndicator` for pull-to-refresh functionality.
- **Exception Handling in Images**:
  - Always include an `errorBuilder` when using `Image.network`.

---

## References

- Always refer to the official documentation for Flutter, Riverpod, and Firebase for widgets, state management, and backend integration best practices.

---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vanasnip) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
