## pocketly

> Follow a feature-first architecture with clear separation of concerns:

# Flutter Clean Architecture - Cursor Rules

## Project Structure

Follow a feature-first architecture with clear separation of concerns:
```
lib/
├── core/
│   ├── services/
│   ├── utils/
│   ├── theme/
│   ├── locator/
│   └── constants/
├── features/
│   ├── feature_name/
│   │   ├── data/
│   │   │   ├── datasources/
│   │   │   │   ├── local/
│   │   │   │   └── remote/
│   │   │   ├── models/
│   │   │   └── repositories/
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   ├── repositories/
│   │   │   └── usecases/
│   │   └── presentation/
│   │       ├── providers/
│   │       ├── views/
│   │       ├── widgets/
│   │       └── components/
│   └── shared/
│       ├── components/
│       └── widgets/
```

## Layer Responsibilities

### Data Layer
- **Purpose**: Handle all data operations (local and remote)
- **Contains**:
  - `datasources/`: API calls, database operations, cache management
  - `models/`: Data transfer objects (DTOs) with JSON serialization
  - `repositories/`: Implementation of domain repository interfaces
- **Rules**:
  - Never import from `domain` or `presentation` layers
  - Handle data transformation between external sources and domain entities
  - Implement error handling and data validation

### Domain Layer
- **Purpose**: Business logic and core entities (framework-independent)
- **Contains**:
  - `entities/`: Plain Dart classes representing business objects
  - `repositories/`: Abstract repository interfaces
  - `usecases/`: Single-responsibility business logic operations
- **Rules**:
  - No Flutter imports allowed
  - No external package dependencies (except Dart core)
  - Must be 100% testable with unit tests
  - Each use case should do ONE thing

### Presentation Layer
- **Purpose**: UI and state management
- **Contains**:
  - `providers/`: State management (Riverpod/Provider/Bloc)
  - `views/`: Full-screen pages/routes
  - `widgets/`: Reusable widgets specific to this feature
  - `components/`: Bottom sheets, dialogs, modals
- **Rules**:
  - Views should be thin - delegate logic to providers
  - Providers call use cases, never repositories directly
  - Widgets should be stateless when possible
  - Components are feature-specific UI elements that appear over views

## Shared Resources

### features/shared/
- **components/**: Dialogs, bottom sheets, modals used across multiple features
- **widgets/**: Reusable widgets used in 2+ features
- **Rule**: If it's used in only one feature, keep it in that feature's folder

### core/
- **services/**: App-wide services (API client, storage, navigation)
- **utils/**: Helper functions, extensions, validators
- **theme/**: Colors, text styles, theme data
- **locator/**: Dependency injection setup (get_it)
- **constants/**: App-wide constants, enums
- **Rule**: Only truly global utilities belong here

## Code Standards

### Naming Conventions
- Features: `snake_case` folder names (e.g., `user_profile`)
- Files: `snake_case.dart`
- Classes: `PascalCase`
- Variables/functions: `camelCase`
- Private members: prefix with `_`

### Use Cases
```dart
// Each use case should have a single execute method
class GetUserProfile {
  final UserRepository repository;
  
  GetUserProfile(this.repository);
  
  Future<Either<Failure, User>> execute(String userId) {
    return repository.getUser(userId);
  }
}
```

### Repositories
```dart
// Domain: Abstract interface
abstract class UserRepository {
  Future<Either<Failure, User>> getUser(String id);
}

// Data: Concrete implementation
class UserRepositoryImpl implements UserRepository {
  final UserRemoteDataSource remoteDataSource;
  final UserLocalDataSource localDataSource;
  
  UserRepositoryImpl({
    required this.remoteDataSource,
    required this.localDataSource,
  });
  
  @override
  Future<Either<Failure, User>> getUser(String id) async {
    // Implementation with error handling
  }
}
```

### Providers (State Management)
```dart
// Providers handle state and orchestrate use cases
class UserProfileProvider extends ChangeNotifier {
  final GetUserProfile getUserProfile;
  
  User? _user;
  bool _isLoading = false;
  String? _error;
  
  // Getters, methods, and state management logic
}
```

### Views
```dart
// Views are full pages - keep them thin
class UserProfileView extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Profile')),
      body: Consumer<UserProfileProvider>(
        builder: (context, provider, child) {
          // UI logic here
        },
      ),
    );
  }
}
```

### Widgets vs Components
- **Widgets**: Reusable UI elements (cards, list items, buttons)
  - Example: `UserCard`, `ProfileAvatar`
- **Components**: Overlays and modals (dialogs, bottom sheets)
  - Example: `UserFilterBottomSheet`, `DeleteConfirmationDialog`

## SOLID Principles

1. **Single Responsibility**: Each class has one reason to change
2. **Open/Closed**: Open for extension, closed for modification
3. **Liskov Substitution**: Subtypes must be substitutable for base types
4. **Interface Segregation**: Many specific interfaces over one general
5. **Dependency Inversion**: Depend on abstractions, not concretions

## Testing Requirements

- **Unit Tests**: All use cases and business logic must have unit tests
- **Widget Tests**: Test individual widgets and components
- **Integration Tests**: Test feature flows
- **Test Structure**: Mirror the source structure in `test/` folder
```
test/
├── features/
│   └── feature_name/
│       ├── data/
│       ├── domain/
│       └── presentation/
```

## Dependency Flow
```
Presentation → Domain → Data
     ↓           ↓        ↓
  Providers → UseCases → Repositories → DataSources
```

**Rules**:
- Higher layers can depend on lower layers
- Lower layers NEVER depend on higher layers
- Use dependency injection (locator) for all dependencies

## Error Handling

- Use `Either<Failure, Success>` pattern (dartz package)
- Create specific `Failure` classes in domain layer
- Handle errors in repositories, expose clean failures to presentation

## Import Rules

- **Domain**: Only Dart core imports
- **Data**: Can import from domain, external packages
- **Presentation**: Can import from domain, external packages
- Feature code should never import from other features (use `shared` instead)

## When Creating New Features

1. Create folder structure: `data/`, `domain/`, `presentation/`
2. Define domain entities first
3. Create repository interface in domain
4. Implement repository in data layer
5. Create use cases in domain
6. Build presentation layer (providers, views, widgets)
7. Register dependencies in locator
8. Write tests for each layer

## Checklist Before Committing

- [ ] Feature follows the prescribed folder structure
- [ ] No cross-feature imports (use shared instead)
- [ ] Domain layer has no Flutter/external dependencies
- [ ] Use cases are single-purpose
- [ ] Repository interfaces defined in domain, implemented in data
- [ ] Views are thin, logic is in providers
- [ ] Dependency injection properly configured
- [ ] Unit tests written for business logic
- [ ] Code follows naming conventions
- [ ] SOLID principles respected

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aderemi-alo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/aderemi-alo)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
