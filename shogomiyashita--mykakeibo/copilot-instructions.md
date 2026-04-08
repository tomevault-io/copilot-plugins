## mykakeibo

> This project is a mobile application development project using Flutter. Based on MVVM architecture pattern and Clean Architecture principles, we aim to achieve a highly maintainable, extensible, and testable codebase.

# Development Guidelines

## Project Overview

This project is a mobile application development project using Flutter. Based on MVVM architecture pattern and Clean Architecture principles, we aim to achieve a highly maintainable, extensible, and testable codebase.

### Supported Platforms
- iOS 16.0 or higher
- Android API 30 or higher

### Development Environment Requirements
- Flutter SDK 3.24.0 or higher
- Dart 3.7 or higher (Null Safety required)

## Mandatory Rules

### After modifying code, format it using dart commands at the end of the task

Always execute the following commands after code changes:

```bash
# Format execution
dart format .

# Format specific directory only
dart format lib/features/
```

### Add Japanese comments for implementations that don't follow official patterns or have unclear intentions, except when obvious from function/variable names

```dart
// ❌ Bad example
Future<void> processData() async {
  await Future.delayed(Duration(milliseconds: 100));
  _updateState();
}

// ✅ Good example
Future<void> processData() async {
  // APIレスポンスの競合状態を防ぐため、意図的に100ms遅延を入れている
  // これにより、連続したリクエストで古いレスポンスが新しいレスポンスを上書きする問題を回避
  await Future.delayed(Duration(milliseconds: 100));
  _updateState();
}
```

### After code changes, always run tests and confirm they pass. If tests fail, don't make random fixes - analyze the error, form a hypothesis, then fix

```bash
# Run all tests
flutter test

# Run specific test file only
flutter test test/features/auth/auth_viewmodel_test.dart

# Run tests with coverage
flutter test --coverage

# Example response to test failures
# 1. Read error messages in detail
# 2. Identify problem location from stack trace
# 3. Check related code and expected values
# 4. Form hypothesis and implement fix
# 5. Run tests again after fixing
```

## Technical Stack

### Programming Language

- **Dart 3.7+**
  - Null Safety support required
  - Utilize Records and Patterns features
  - Use async/await for asynchronous operations

### Libraries

**Note**: You don't need to add all these libraries from the start. Add them as needed during implementation. For example, dio and retrofit are not needed until you implement API communication.

Recommended libraries for medium-sized projects:

```yaml
dependencies:
  # State Management
  flutter_riverpod: ^2.5.1
  
  # Routing
  go_router: ^14.6.1
  
  # Network Communication (add when implementing APIs)
  dio: ^5.7.0
  retrofit: ^4.6.0
  
  # Local Database (add when data persistence is needed)
  drift: ^2.20.3
  
  # Dependency Injection
  get_it: ^8.0.2
  injectable: ^2.4.6
  
  # Model Class Generation
  freezed_annotation: ^2.6.0
  json_annotation: ^4.9.0
  
  # Utilities
  equatable: ^2.0.7
  dartz: ^0.10.1
  intl: ^0.19.0

dev_dependencies:
  # Code Generation
  build_runner: ^2.4.0
  freezed: ^2.6.0
  json_serializable: ^6.9.5
  retrofit_generator: ^9.1.4
  injectable_generator: ^2.6.2
  drift_dev: ^2.20.3
  
  # Testing
  flutter_test:
    sdk: flutter
  mocktail: ^1.0.0
  
  # Static Analysis
  flutter_lints: ^5.0.0
```

### Architecture Configuration

This project adopts a 3-layer architecture combining MVVM pattern and Clean Architecture principles.

#### Layer Structure

1. **Presentation Layer**
   - View: Responsible for UI rendering
   - ViewModel: Handles UI logic and state management

2. **Domain Layer**
   - Entities: Business models
   - Use Cases: Business logic
   - Repository Interfaces: Data access abstraction

3. **Data Layer**
   - Repository Implementations: Data access implementation
   - Data Sources: External data sources (API, DB)
   - Models/DTOs: Data transfer objects

#### Directory Structure

```
lib/
├── core/                          # Common functionality across the app
│   ├── constants/                # Constant definitions
│   ├── exceptions/               # Custom exceptions
│   ├── failures/                 # Error handling
│   ├── utils/                    # Utility functions
│   └── di/                       # Dependency injection setup
│
├── features/                     # Feature-based modules
│   └── authentication/          # Example: Authentication feature
│       ├── data/               # Data layer
│       │   ├── datasources/
│       │   ├── models/
│       │   └── repositories/
│       ├── domain/             # Domain layer
│       │   ├── entities/
│       │   ├── repositories/
│       │   └── usecases/
│       └── presentation/       # Presentation layer
│           ├── viewmodels/
│           ├── views/
│           └── widgets/
│
└── main.dart                    # Entry point
```

#### Data Flow Principles

1. **Unidirectional Data Flow**
   - View → ViewModel → UseCase → Repository → DataSource
   - Data changes are notified in reverse direction

2. **Dependency Direction**
   - Outer layers depend on inner layers
   - Inner layers don't know about outer layers

3. **Interface Abstraction**
   - Define interfaces in Domain layer
   - Provide concrete implementations in Data layer

#### Implementation Examples

**ViewModel (Presentation Layer)**
```dart
@riverpod
class LoginViewModel extends _$LoginViewModel {
  @override
  LoginState build() {
    return const LoginState();
  }
  
  Future<void> login(String email, String password) async {
    state = state.copyWith(isLoading: true, errorMessage: null);
    
    // View calls ViewModel methods
    // ViewModel executes business logic through UseCase
    final loginUseCase = ref.read(loginUseCaseProvider);
    final result = await loginUseCase(
      LoginParams(email: email, password: password),
    );
    
    result.fold(
      (failure) => state = state.copyWith(
        isLoading: false,
        errorMessage: failure.message,
      ),
      (user) => state = state.copyWith(
        isLoading: false,
        user: user,
      ),
    );
  }
}

@freezed
class LoginState with _$LoginState {
  const factory LoginState({
    @Default(false) bool isLoading,
    User? user,
    String? errorMessage,
  }) = _LoginState;
}
```

**UseCase (Domain Layer)**
```dart
@riverpod
LoginUseCase loginUseCase(LoginUseCaseRef ref) {
  return LoginUseCase(ref.watch(authRepositoryProvider));
}

class LoginUseCase {
  final AuthRepository repository;
  
  LoginUseCase(this.repository);
  
  Future<Either<Failure, User>> call(LoginParams params) async {
    // Business rule implementation
    if (!_isValidEmail(params.email)) {
      return Left(ValidationFailure('Invalid email format'));
    }
    
    return await repository.login(params.email, params.password);
  }
  
  bool _isValidEmail(String email) {
    return RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(email);
  }
}
```

**Repository (Data Layer)**
```dart
@riverpod
AuthRepository authRepository(AuthRepositoryRef ref) {
  return AuthRepositoryImpl(
    ref.watch(authRemoteDataSourceProvider),
    ref.watch(authLocalDataSourceProvider),
  );
}

class AuthRepositoryImpl implements AuthRepository {
  final AuthRemoteDataSource remoteDataSource;
  final AuthLocalDataSource localDataSource;
  
  AuthRepositoryImpl(this.remoteDataSource, this.localDataSource);
  
  @override
  Future<Either<Failure, User>> login(String email, String password) async {
    try {
      final userModel = await remoteDataSource.login(email, password);
      final user = userModel.toEntity();
      await localDataSource.cacheUser(user);
      return Right(user);
    } on ServerException {
      return Left(ServerFailure());
    }
  }
}
```

## Coding Standards

### Naming Conventions

- **Class names**: UpperCamelCase (e.g., `UserViewModel`)
- **File names**: lowercase_with_underscores.dart
- **Variable/method names**: lowerCamelCase
- **Constants**: lowerCamelCase (`const defaultTimeout = 30;`)
- **Private members**: _lowerCamelCase

### Import Order

```dart
// 1. Dart standard libraries
import 'dart:async';
import 'dart:convert';

// 2. Flutter packages
import 'package:flutter/material.dart';

// 3. External packages
import 'package:dio/dio.dart';
import 'package:get_it/get_it.dart';

// 4. Project files
import '../../domain/entities/user.dart';
import '../widgets/custom_button.dart';

// 5. Part statements
part 'user_model.g.dart';
```

### State Management Principles

```dart
// Using Riverpod Generator for state management
@riverpod
class ExampleViewModel extends _$ExampleViewModel {
  @override
  ExampleState build() {
    // Return initial state
    return const ExampleState();
  }
  
  // State changes must be done through methods
  void addItem(Item item) {
    state = state.copyWith(
      items: [...state.items, item],
    );
  }
  
  Future<void> loadItems() async {
    state = state.copyWith(isLoading: true);
    
    try {
      final repository = ref.read(itemRepositoryProvider);
      final items = await repository.fetchItems();
      state = state.copyWith(
        items: items,
        isLoading: false,
      );
    } catch (e) {
      state = state.copyWith(
        isLoading: false,
        errorMessage: e.toString(),
      );
    }
  }
}

// Immutable state class using Freezed
@freezed
class ExampleState with _$ExampleState {
  const factory ExampleState({
    @Default([]) List<Item> items,
    @Default(false) bool isLoading,
    String? errorMessage,
  }) = _ExampleState;
}
```

### Error Handling

```dart
// Result representation using Either type
Future<Either<Failure, Success>> performOperation() async {
  try {
    final result = await someAsyncOperation();
    return Right(Success(result));
  } on NetworkException {
    return Left(NetworkFailure('No network connection'));
  } catch (e) {
    return Left(UnknownFailure('An unexpected error occurred'));
  }
}
```

### Widget Optimization

```dart
// Use ConsumerWidget to watch providers
class OptimizedWidget extends ConsumerWidget {
  const OptimizedWidget({super.key});
  
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Watch only necessary providers
    final itemCount = ref.watch(
      exampleViewModelProvider.select((state) => state.items.length),
    );
    
    return Column(
      children: [
        Text('Item count: $itemCount'),
        const SizedBox(height: 16),
        const _StaticContent(), // Separate static content into another widget
      ],
    );
  }
}

// Implement static content with StatelessWidget
class _StaticContent extends StatelessWidget {
  const _StaticContent();
  
  @override
  Widget build(BuildContext context) {
    return const Column(
      children: [
        Text('Static text'),
        Icon(Icons.home),
      ],
    );
  }
}
```

## Testing Strategy

### Test Types and Purposes

1. **Unit Tests**
   - Test individual functionality of ViewModels, UseCases, Repositories
   - Use mocks for external dependencies

2. **Widget Tests**
   - Test UI component behavior
   - Simulate user interactions

3. **Integration Tests**
   - Test integration between multiple components
   - Verify actual user scenarios

### Test Coverage Goals

- ViewModels: 90% or higher
- Use Cases: 100%
- Repositories: 80% or higher
- Overall: 80% or higher

### Test Implementation Examples

**ViewModel Test**
```dart
void main() {
  test('should login successfully', () async {
    // Arrange
    final container = ProviderContainer(
      overrides: [
        loginUseCaseProvider.overrideWithValue(
          LoginUseCase(MockAuthRepository()),
        ),
      ],
    );
    
    when(() => container.read(loginUseCaseProvider)(any())).thenAnswer(
      (_) async => Right(User(id: '1', email: 'test@example.com')),
    );
    
    final viewModel = container.read(loginViewModelProvider.notifier);
    
    // Act
    await viewModel.login('test@example.com', 'password');
    
    // Assert
    final state = container.read(loginViewModelProvider);
    expect(state.user, isNotNull);
    expect(state.errorMessage, isNull);
    expect(state.isLoading, false);
  });
  
  test('should show error on login failure', () async {
    // Arrange
    final container = ProviderContainer(
      overrides: [
        loginUseCaseProvider.overrideWithValue(
          LoginUseCase(MockAuthRepository()),
        ),
      ],
    );
    
    when(() => container.read(loginUseCaseProvider)(any())).thenAnswer(
      (_) async => Left(ServerFailure('Authentication error')),
    );
    
    final viewModel = container.read(loginViewModelProvider.notifier);
    
    // Act
    await viewModel.login('test@example.com', 'wrong_password');
    
    // Assert
    final state = container.read(loginViewModelProvider);
    expect(state.user, isNull);
    expect(state.errorMessage, 'Authentication error');
    expect(state.isLoading, false);
  });
}
```

**Widget Test**
```dart
void main() {
  testWidgets('login process executes when login button is tapped', (tester) async {
    // Arrange
    await tester.pumpWidget(
      ProviderScope(
        overrides: [
          loginViewModelProvider.overrideWith(() => MockLoginViewModel()),
        ],
        child: const MaterialApp(
          home: LoginView(),
        ),
      ),
    );
    
    // Act
    await tester.enterText(find.byType(TextField).first, 'test@example.com');
    await tester.enterText(find.byType(TextField).last, 'password');
    await tester.tap(find.byType(ElevatedButton));
    await tester.pump();
    
    // Assert
    // Verify that ViewModel's login method was called
    final container = tester.element(find.byType(LoginView)).findAncestorWidgetOfExactType<ProviderScope>()!;
    verify(() => container.read(loginViewModelProvider.notifier).login(
      'test@example.com',
      'password',
    )).called(1);
  });
  
  testWidgets('shows progress indicator when loading', (tester) async {
    // Arrange
    await tester.pumpWidget(
      ProviderScope(
        overrides: [
          loginViewModelProvider.overrideWithValue(
            const LoginState(isLoading: true),
          ),
        ],
        child: const MaterialApp(
          home: LoginView(),
        ),
      ),
    );
    
    // Assert
    expect(find.byType(CircularProgressIndicator), findsOneWidget);
  });
}

// View implementation example
class LoginView extends ConsumerWidget {
  const LoginView({super.key});
  
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final state = ref.watch(loginViewModelProvider);
    
    if (state.isLoading) {
      return const Scaffold(
        body: Center(
          child: CircularProgressIndicator(),
        ),
      );
    }
    
    return Scaffold(
      body: LoginForm(
        onSubmit: (email, password) {
          ref.read(loginViewModelProvider.notifier).login(email, password);
        },
        errorMessage: state.errorMessage,
      ),
    );
  }
}
```

### Test Execution Commands

```bash
# Run all tests
flutter test

# Generate coverage report
flutter test --coverage

# Run specific test only
flutter test test/unit/login_viewmodel_test.dart

# Run tests in watch mode (auto-run on file changes)
flutter test --watch
```

## CI/CD

*(Currently in preparation)*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ShogoMiyashita)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ShogoMiyashita)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
