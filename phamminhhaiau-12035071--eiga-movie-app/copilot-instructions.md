## eiga-movie-app

> ├── feature_cubit.dart  # Or feature_bloc.dart

# State Management Guidelines (Application Layer)

## Directory Structure

```
lib/
  └── features/
      └── feature_name/
          └── application/
              ├── feature_cubit.dart  # Or feature_bloc.dart
              └── feature_state.dart
```

## State Management Conventions

-   **Primary Pattern:** Use **BLoC/Cubit** for state management.
-   **State Immutability:** Use **`freezed`** to generate immutable state classes.
-   **Dependency Injection:** Use **`@injectable`** or `@factoryMethod` for Cubits/Blocs.
-   **Data Flow:** Maintain unidirectional data flow.

### State Class Structure

1.  State classes must:
    *   Use the `@freezed` annotation.
    *   Have a private constructor `const FeatureState._();` if defining helper getters.
    *   Have a `factory FeatureState.initial()` constructor for the initial state.
    *   Be immutable.
    *   Group related state properties logically (e.g., data, status flags, error messages).

### Cubit/Bloc Class Structure

1.  Cubit/Bloc classes should:
    *   Have a single, well-defined responsibility (managing the state for a specific feature or screen).
    *   Contain the business logic required to react to events/calls and update the state.
    *   Inject required **Repositories** (from the domain layer) via the constructor.
    *   Be annotated with `@injectable` (or similar) for DI.

2.  Constructor must:
    *   Accept all required repository interfaces (e.g., `FeatureRepository`) as parameters.
    *   Call `super()` with the `FeatureState.initial()` state.

### Naming Conventions

1.  Files:
    *   State file: `feature_state.dart`
    *   Cubit file: `feature_cubit.dart`
    *   Bloc file: `feature_bloc.dart` (if using Bloc)

2.  Classes:
    *   State class: `FeatureState`
    *   Cubit class: `FeatureCubit`
    *   Bloc class: `FeatureBloc`

3.  Methods (in Cubit/Bloc):
    *   Use clear, descriptive names indicating the action (e.g., `loadFeatureData()`, `submitForm()`, `retryInitialization()`).
    *   Avoid generic names like `updateState`.

## State Properties & Patterns

-   **Async Operation State:** A common pattern is to track the status of async operations. The `DataState` approach mentioned in the original rules or a similar enum (`LoadingStatus { initial, loading, success, failure }`) combined with data and error fields is recommended.

    ```dart
    // Example using a status enum
    enum LoadingStatus { initial, loading, success, failure }

    @freezed
    class FeatureState with _$FeatureState {
      const factory FeatureState({
        @Default(LoadingStatus.initial) LoadingStatus status,
        FeatureModel? featureData, // Nullable data
        String? errorMessage,    // Nullable error
        @Default(false) bool isSubmitting, // Example other state flag
      }) = _FeatureState;

      factory FeatureState.initial() => const FeatureState();
    }
    ```

-   **Helper Getters:** Define getters in the state class for common checks if it improves readability.

    ```dart
    @freezed
    class FeatureState with _$FeatureState {
      const FeatureState._(); // Need private constructor for getters

      const factory FeatureState({
         @Default(LoadingStatus.initial) LoadingStatus status,
         // ... other properties
      }) = _FeatureState;

      factory FeatureState.initial() => const FeatureState();

      bool get isLoading => status == LoadingStatus.loading;
      bool get hasError => status == LoadingStatus.failure;
      bool get isSuccess => status == LoadingStatus.success;
    }
    ```

## Error Handling

1.  Catch errors from Repository calls within the Cubit/Bloc.
2.  Map repository errors (e.g., `FeatureError`) to user-friendly messages or specific state flags.
3.  Update the state to reflect the error (`status: LoadingStatus.failure`, `errorMessage: '...'`).
4.  The Presentation layer will listen to these state changes and display appropriate UI feedback.

## Best Practices

1.  **State Management:**
    *   Keep Cubits/Blocs focused; extract complex logic into Use Cases (if needed, resides in Application layer).
    *   Emit new states for every change; avoid mutating the current state object directly.
    *   Repositories handle *how* to get/save data; Cubits/Blocs handle *when* and *what to do with it*.

2.  **Testing:**
    *   Use `bloc_test` package for testing Cubits/Blocs.
    *   Use `mocktail` to mock repository dependencies.
    *   Test all state transitions based on method calls/events.
    *   Verify that the correct repository methods are called.
    *   Test error handling paths thoroughly.
    *   Follow Arrange-Act-Assert pattern.

## Code Example (Cubit)

```dart
// feature_state.dart
import 'package:app/features/feature_name/domain/models/feature_model.dart'; // Adjust path
import 'package:freezed_annotation/freezed_annotation.dart';

part 'feature_state.freezed.dart';

enum LoadingStatus { initial, loading, success, failure }

@freezed
class FeatureState with _$FeatureState {
  const factory FeatureState({
    @Default(LoadingStatus.initial) LoadingStatus status,
    FeatureModel? featureData,
    String? errorMessage,
  }) = _FeatureState;

  factory FeatureState.initial() => const FeatureState();
}

// feature_cubit.dart
import 'package:app/features/feature_name/domain/feature_repository.dart'; // No 'I' prefix
import 'package:app/features/feature_name/domain/models/feature_model.dart'; // If needed
import 'package:bloc/bloc.dart';
import 'package:injectable/injectable.dart';
import 'feature_state.dart'; // Adjust path

part 'feature_cubit.freezed.dart'; // If you freeze the cubit itself (less common)

@injectable
class FeatureCubit extends Cubit<FeatureState> {
  final FeatureRepository _featureRepository; // No 'I' prefix

  FeatureCubit(this._featureRepository) : super(FeatureState.initial());

  Future<void> loadFeatureData(String id) async {
    emit(state.copyWith(status: LoadingStatus.loading, errorMessage: null));

    final result = await _featureRepository.getFeature(id);

    result.when(
      (featureModel) => emit(state.copyWith(
        status: LoadingStatus.success,
        featureData: featureModel,
      )),
      (error) => emit(state.copyWith(
        status: LoadingStatus.failure,
        // Map domain error to user-friendly message
        errorMessage: _mapErrorToMessage(error),
      )),
    );
  }

  String _mapErrorToMessage(FeatureError error) {
    switch (error) {
      case FeatureError.notFound:
        return 'Feature not found.';
      case FeatureError.networkError:
        return 'Network error. Please check your connection.';
      case FeatureError.validation:
        return 'Invalid data provided.';
      case FeatureError.unknown:
      default:
        return 'An unexpected error occurred.';
    }
  }

  // Example other method
  void clearError() {
    if (state.status == LoadingStatus.failure) {
      emit(state.copyWith(status: LoadingStatus.initial, errorMessage: null));
    }
  }
}
``` 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PhamMinhHaiAu-12035071)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/PhamMinhHaiAu-12035071)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
