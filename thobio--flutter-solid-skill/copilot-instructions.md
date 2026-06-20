## flutter-solid-skill

> Transform Flutter code into senior-engineer quality mobile applications through SOLID principles, Test-Driven Development (TDD), Clean Architecture, and professional software design patterns. This skill ensures production-ready, maintainable, and scalable Flutter applications.

# Flutter SOLID Development Skill

## Overview

Transform Flutter code into senior-engineer quality mobile applications through SOLID principles, Test-Driven Development (TDD), Clean Architecture, and professional software design patterns. This skill ensures production-ready, maintainable, and scalable Flutter applications.

## When to Use This Skill

**ALWAYS use this skill when:**
- Writing any Flutter code (features, widgets, services, repositories)
- Refactoring existing Flutter applications
- Planning or designing app architecture
- Reviewing code quality and performance
- Debugging Flutter-specific issues
- Creating unit, widget, or integration tests
- Making architectural or design decisions
- Implementing state management solutions
- Working with API integrations and data layers

## Core Workflow: TDD-First Approach

**CRITICAL:** Always follow the Red-Green-Refactor cycle:

### Step 1: RED - Write Failing Test First
```dart
test('should fetch user profile from repository', () async {
  // Arrange
  when(mockRepository.getUser(any))
      .thenAnswer((_) async => Right(tUser));
  
  // Act
  final result = await useCase(Params(userId: tUserId));
  
  // Assert
  expect(result, Right(tUser));
  verify(mockRepository.getUser(tUserId));
  verifyNoMoreInteractions(mockRepository);
});
```

### Step 2: GREEN - Write Minimum Code to Pass
```dart
class GetUser implements UseCase<User, Params> {
  final UserRepository repository;
  
  GetUser(this.repository);
  
  @override
  Future<Either<Failure, User>> call(Params params) {
    return repository.getUser(params.userId);
  }
}
```

### Step 3: REFACTOR - Improve Design
- Apply SOLID principles
- Extract value objects
- Remove code smells
- Ensure clean architecture boundaries

## Flutter-Specific Architecture

### Layer Structure

```
lib/
├── core/
│   ├── error/
│   │   ├── exceptions.dart
│   │   └── failures.dart
│   ├── usecases/
│   │   └── usecase.dart
│   ├── utils/
│   └── network/
│       └── network_info.dart
├── features/
│   └── [feature_name]/
│       ├── data/
│       │   ├── datasources/
│       │   │   ├── [feature]_local_datasource.dart
│       │   │   └── [feature]_remote_datasource.dart
│       │   ├── models/
│       │   │   └── [feature]_model.dart
│       │   └── repositories/
│       │       └── [feature]_repository_impl.dart
│       ├── domain/
│       │   ├── entities/
│       │   │   └── [feature]_entity.dart
│       │   ├── repositories/
│       │   │   └── [feature]_repository.dart
│       │   └── usecases/
│       │       └── [specific_usecase].dart
│       └── presentation/
│           ├── bloc/
│           │   ├── [feature]_bloc.dart
│           │   ├── [feature]_event.dart
│           │   └── [feature]_state.dart
│           ├── pages/
│           │   └── [feature]_page.dart
│           └── widgets/
│               └── [feature]_widget.dart
└── injection_container.dart
```

## SOLID Principles for Flutter

### 1. Single Responsibility Principle (SRP)
**Each class/widget has ONE reason to change.**

❌ **Bad:** Widget doing business logic
```dart
class UserProfilePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // WRONG: HTTP call in widget
    final response = await http.get('api/user');
    final user = json.decode(response.body);
    return Text(user['name']);
  }
}
```

✅ **Good:** Separated concerns
```dart
// Widget only displays UI
class UserProfilePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocBuilder<UserBloc, UserState>(
      builder: (context, state) {
        return state.when(
          loaded: (user) => UserProfileView(user: user),
          loading: () => LoadingWidget(),
          error: (message) => ErrorWidget(message),
        );
      },
    );
  }
}

// Bloc handles state
class UserBloc extends Bloc<UserEvent, UserState> {
  final GetUser getUser;
  
  UserBloc({required this.getUser}) : super(UserInitial());
  
  @override
  Stream<UserState> mapEventToState(UserEvent event) async* {
    if (event is GetUserRequested) {
      yield UserLoading();
      final result = await getUser(Params(userId: event.userId));
      yield result.fold(
        (failure) => UserError(failure.message),
        (user) => UserLoaded(user),
      );
    }
  }
}

// UseCase handles business logic
class GetUser implements UseCase<User, Params> {
  final UserRepository repository;
  
  GetUser(this.repository);
  
  @override
  Future<Either<Failure, User>> call(Params params) {
    return repository.getUser(params.userId);
  }
}
```

### 2. Open/Closed Principle (OCP)
**Open for extension, closed for modification.**

✅ **Good:** Use abstract classes and composition
```dart
// Abstract datasource
abstract class AuthDataSource {
  Future<UserModel> login(LoginParams params);
  Future<void> logout();
}

// Implementation can be extended without modifying interface
class FirebaseAuthDataSource implements AuthDataSource {
  final FirebaseAuth firebaseAuth;
  
  FirebaseAuthDataSource(this.firebaseAuth);
  
  @override
  Future<UserModel> login(LoginParams params) async {
    final credential = await firebaseAuth.signInWithEmailAndPassword(
      email: params.email,
      password: params.password,
    );
    return UserModel.fromFirebase(credential.user!);
  }
  
  @override
  Future<void> logout() => firebaseAuth.signOut();
}

// Can add new implementation without changing existing code
class SupabaseAuthDataSource implements AuthDataSource {
  final SupabaseClient supabase;
  
  SupabaseAuthDataSource(this.supabase);
  
  @override
  Future<UserModel> login(LoginParams params) async {
    final response = await supabase.auth.signIn(
      email: params.email,
      password: params.password,
    );
    return UserModel.fromSupabase(response.user!);
  }
  
  @override
  Future<void> logout() => supabase.auth.signOut();
}
```

### 3. Liskov Substitution Principle (LSP)
**Subtypes must be substitutable for their base types.**

✅ **Good:** Proper abstraction
```dart
abstract class Cache<T> {
  Future<void> save(String key, T value);
  Future<T?> get(String key);
  Future<void> delete(String key);
}

class SecureCache<T> implements Cache<T> {
  final FlutterSecureStorage storage;
  
  SecureCache(this.storage);
  
  @override
  Future<void> save(String key, T value) async {
    await storage.write(key: key, value: jsonEncode(value));
  }
  
  @override
  Future<T?> get(String key) async {
    final value = await storage.read(key: key);
    return value != null ? jsonDecode(value) as T : null;
  }
  
  @override
  Future<void> delete(String key) => storage.delete(key: key);
}

class SharedPrefsCache<T> implements Cache<T> {
  final SharedPreferences prefs;
  
  SharedPrefsCache(this.prefs);
  
  @override
  Future<void> save(String key, T value) async {
    await prefs.setString(key, jsonEncode(value));
  }
  
  @override
  Future<T?> get(String key) {
    final value = prefs.getString(key);
    return Future.value(value != null ? jsonDecode(value) as T : null);
  }
  
  @override
  Future<void> delete(String key) => prefs.remove(key);
}
```

### 4. Interface Segregation Principle (ISP)
**Don't force classes to depend on interfaces they don't use.**

❌ **Bad:** Fat interface
```dart
abstract class AuthService {
  Future<void> login(String email, String password);
  Future<void> register(String email, String password);
  Future<void> resetPassword(String email);
  Future<void> updateProfile(UserProfile profile);
  Future<void> deleteAccount();
  Future<void> enableTwoFactor();
  Future<void> verifyEmail();
}
```

✅ **Good:** Segregated interfaces
```dart
abstract class AuthenticationService {
  Future<void> login(String email, String password);
  Future<void> logout();
}

abstract class RegistrationService {
  Future<void> register(String email, String password);
  Future<void> verifyEmail();
}

abstract class PasswordService {
  Future<void> resetPassword(String email);
  Future<void> changePassword(String oldPassword, String newPassword);
}

abstract class ProfileService {
  Future<void> updateProfile(UserProfile profile);
  Future<void> deleteAccount();
}

abstract class SecurityService {
  Future<void> enableTwoFactor();
  Future<void> disableTwoFactor();
}
```

### 5. Dependency Inversion Principle (DIP)
**Depend on abstractions, not concretions.**

✅ **Good:** Using dependency injection
```dart
// Domain layer - abstract repository
abstract class ProductRepository {
  Future<Either<Failure, List<Product>>> getProducts();
  Future<Either<Failure, Product>> getProduct(ProductId id);
}

// Data layer - concrete implementation
class ProductRepositoryImpl implements ProductRepository {
  final ProductRemoteDataSource remoteDataSource;
  final ProductLocalDataSource localDataSource;
  final NetworkInfo networkInfo;
  
  ProductRepositoryImpl({
    required this.remoteDataSource,
    required this.localDataSource,
    required this.networkInfo,
  });
  
  @override
  Future<Either<Failure, List<Product>>> getProducts() async {
    if (await networkInfo.isConnected) {
      try {
        final products = await remoteDataSource.getProducts();
        await localDataSource.cacheProducts(products);
        return Right(products.map((m) => m.toEntity()).toList());
      } on ServerException {
        return Left(ServerFailure());
      }
    } else {
      try {
        final products = await localDataSource.getCachedProducts();
        return Right(products.map((m) => m.toEntity()).toList());
      } on CacheException {
        return Left(CacheFailure());
      }
    }
  }
  
  @override
  Future<Either<Failure, Product>> getProduct(ProductId id) async {
    // Implementation
  }
}

// Presentation layer - depends on abstraction
class ProductBloc extends Bloc<ProductEvent, ProductState> {
  final GetProducts getProducts;
  
  ProductBloc({required this.getProducts}) : super(ProductInitial());
  // Implementation
}
```

## Value Objects for Domain Primitives

**ALWAYS use value objects for domain concepts.**

```dart
// Email value object
class Email extends Equatable {
  final String value;
  
  Email._(this.value);
  
  factory Email(String input) {
    if (!_isValid(input)) {
      throw InvalidEmailException(input);
    }
    return Email._(input);
  }
  
  static bool _isValid(String input) {
    final emailRegex = RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$');
    return emailRegex.hasMatch(input);
  }
  
  @override
  List<Object?> get props => [value];
}

// UserId value object
class UserId extends Equatable {
  final String value;
  
  UserId._(this.value);
  
  factory UserId(String input) {
    if (input.isEmpty) {
      throw InvalidUserIdException();
    }
    return UserId._(input);
  }
  
  factory UserId.generate() {
    return UserId._(Uuid().v4());
  }
  
  @override
  List<Object?> get props => [value];
}

// Money value object
class Money extends Equatable {
  final double amount;
  final Currency currency;
  
  Money._(this.amount, this.currency);
  
  factory Money(double amount, Currency currency) {
    if (amount < 0) {
      throw NegativeAmountException();
    }
    return Money._(amount, currency);
  }
  
  Money add(Money other) {
    if (currency != other.currency) {
      throw CurrencyMismatchException();
    }
    return Money(amount + other.amount, currency);
  }
  
  @override
  List<Object?> get props => [amount, currency];
}
```

## Flutter Widget Best Practices

### Keep Widgets Small and Focused

❌ **Bad:** Large widget
```dart
class UserDashboard extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Dashboard'),
        actions: [
          IconButton(icon: Icon(Icons.settings), onPressed: () {}),
          IconButton(icon: Icon(Icons.notifications), onPressed: () {}),
        ],
      ),
      body: Column(
        children: [
          // 100+ lines of widget tree
        ],
      ),
    );
  }
}
```

✅ **Good:** Extracted widgets
```dart
class UserDashboard extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: DashboardAppBar(),
      body: DashboardBody(),
      bottomNavigationBar: DashboardNavigationBar(),
    );
  }
}

class DashboardAppBar extends StatelessWidget implements PreferredSizeWidget {
  @override
  Widget build(BuildContext context) {
    return AppBar(
      title: const Text('Dashboard'),
      actions: const [
        SettingsButton(),
        NotificationsButton(),
      ],
    );
  }
  
  @override
  Size get preferredSize => const Size.fromHeight(kToolbarHeight);
}

class DashboardBody extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return const Column(
      children: [
        UserProfileCard(),
        RecentActivityList(),
        QuickActionsGrid(),
      ],
    );
  }
}
```

### Use Composition Over Inheritance

✅ **Good:** Composition
```dart
class CustomButton extends StatelessWidget {
  final String label;
  final VoidCallback onPressed;
  final ButtonStyle style;
  
  const CustomButton({
    required this.label,
    required this.onPressed,
    required this.style,
    Key? key,
  }) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: onPressed,
      style: style,
      child: Text(label),
    );
  }
}

// Use factory constructors for variants
class PrimaryButton extends CustomButton {
  PrimaryButton({
    required String label,
    required VoidCallback onPressed,
    Key? key,
  }) : super(
    label: label,
    onPressed: onPressed,
    style: ElevatedButton.styleFrom(
      backgroundColor: Colors.blue,
      foregroundColor: Colors.white,
    ),
    key: key,
  );
}

class SecondaryButton extends CustomButton {
  SecondaryButton({
    required String label,
    required VoidCallback onPressed,
    Key? key,
  }) : super(
    label: label,
    onPressed: onPressed,
    style: ElevatedButton.styleFrom(
      backgroundColor: Colors.white,
      foregroundColor: Colors.blue,
    ),
    key: key,
  );
}
```

## State Management with BLoC Pattern

### Event-Driven Architecture

```dart
// Events
abstract class UserEvent extends Equatable {
  const UserEvent();
  
  @override
  List<Object?> get props => [];
}

class LoadUserRequested extends UserEvent {
  final UserId userId;
  
  const LoadUserRequested(this.userId);
  
  @override
  List<Object?> get props => [userId];
}

class UpdateUserRequested extends UserEvent {
  final User user;
  
  const UpdateUserRequested(this.user);
  
  @override
  List<Object?> get props => [user];
}

// States
abstract class UserState extends Equatable {
  const UserState();
  
  @override
  List<Object?> get props => [];
}

class UserInitial extends UserState {}

class UserLoading extends UserState {}

class UserLoaded extends UserState {
  final User user;
  
  const UserLoaded(this.user);
  
  @override
  List<Object?> get props => [user];
}

class UserError extends UserState {
  final String message;
  
  const UserError(this.message);
  
  @override
  List<Object?> get props => [message];
}

// BLoC
class UserBloc extends Bloc<UserEvent, UserState> {
  final GetUser getUser;
  final UpdateUser updateUser;
  
  UserBloc({
    required this.getUser,
    required this.updateUser,
  }) : super(UserInitial()) {
    on<LoadUserRequested>(_onLoadUserRequested);
    on<UpdateUserRequested>(_onUpdateUserRequested);
  }
  
  Future<void> _onLoadUserRequested(
    LoadUserRequested event,
    Emitter<UserState> emit,
  ) async {
    emit(UserLoading());
    final result = await getUser(Params(userId: event.userId));
    result.fold(
      (failure) => emit(UserError(failure.message)),
      (user) => emit(UserLoaded(user)),
    );
  }
  
  Future<void> _onUpdateUserRequested(
    UpdateUserRequested event,
    Emitter<UserState> emit,
  ) async {
    emit(UserLoading());
    final result = await updateUser(UpdateParams(user: event.user));
    result.fold(
      (failure) => emit(UserError(failure.message)),
      (user) => emit(UserLoaded(user)),
    );
  }
}
```

## Testing Strategy

### Unit Tests for Business Logic

```dart
void main() {
  late GetUser useCase;
  late MockUserRepository mockRepository;
  
  setUp(() {
    mockRepository = MockUserRepository();
    useCase = GetUser(mockRepository);
  });
  
  group('GetUser', () {
    final tUserId = UserId('123');
    final tUser = User(
      id: tUserId,
      name: Name('John Doe'),
      email: Email('john@example.com'),
    );
    
    test('should return user from repository', () async {
      // Arrange
      when(mockRepository.getUser(any))
          .thenAnswer((_) async => Right(tUser));
      
      // Act
      final result = await useCase(Params(userId: tUserId));
      
      // Assert
      expect(result, Right(tUser));
      verify(mockRepository.getUser(tUserId));
      verifyNoMoreInteractions(mockRepository);
    });
    
    test('should return failure when repository fails', () async {
      // Arrange
      when(mockRepository.getUser(any))
          .thenAnswer((_) async => Left(ServerFailure()));
      
      // Act
      final result = await useCase(Params(userId: tUserId));
      
      // Assert
      expect(result, Left(ServerFailure()));
      verify(mockRepository.getUser(tUserId));
    });
  });
}
```

### Widget Tests for UI

```dart
void main() {
  testWidgets('UserProfilePage displays user data', (tester) async {
    // Arrange
    final user = User(
      id: UserId('123'),
      name: Name('John Doe'),
      email: Email('john@example.com'),
    );
    
    // Act
    await tester.pumpWidget(
      MaterialApp(
        home: BlocProvider(
          create: (_) => MockUserBloc()
            ..add(LoadUserRequested(user.id)),
          child: UserProfilePage(),
        ),
      ),
    );
    await tester.pumpAndSettle();
    
    // Assert
    expect(find.text('John Doe'), findsOneWidget);
    expect(find.text('john@example.com'), findsOneWidget);
  });
  
  testWidgets('UserProfilePage shows loading indicator', (tester) async {
    // Arrange
    final mockBloc = MockUserBloc();
    whenListen(
      mockBloc,
      Stream.fromIterable([UserLoading()]),
      initialState: UserInitial(),
    );
    
    // Act
    await tester.pumpWidget(
      MaterialApp(
        home: BlocProvider.value(
          value: mockBloc,
          child: UserProfilePage(),
        ),
      ),
    );
    await tester.pump();
    
    // Assert
    expect(find.byType(CircularProgressIndicator), findsOneWidget);
  });
}
```

### Integration Tests

```dart
void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();
  
  group('User Authentication Flow', () {
    testWidgets('complete login flow', (tester) async {
      await tester.pumpWidget(MyApp());
      
      // Navigate to login
      await tester.tap(find.text('Login'));
      await tester.pumpAndSettle();
      
      // Enter credentials
      await tester.enterText(
        find.byType(EmailField),
        'test@example.com',
      );
      await tester.enterText(
        find.byType(PasswordField),
        'password123',
      );
      
      // Submit
      await tester.tap(find.text('Submit'));
      await tester.pumpAndSettle();
      
      // Verify navigation to home
      expect(find.byType(HomePage), findsOneWidget);
    });
  });
}
```

## Code Quality Checklist

Before considering code complete, verify:

- [ ] All tests pass (unit, widget, integration)
- [ ] Test coverage > 80%
- [ ] No code smells detected
- [ ] SOLID principles applied
- [ ] Value objects used for domain primitives
- [ ] Clean architecture layers respected
- [ ] Widget methods < 10 lines
- [ ] Classes < 50 lines
- [ ] No comments needed (code is self-explanatory)
- [ ] Proper error handling with Either type
- [ ] Dependency injection configured
- [ ] No business logic in widgets
- [ ] State management properly implemented
- [ ] Performance optimized (const constructors, keys)

## Common Flutter Code Smells

### 1. Business Logic in Widgets
❌ Widgets should only handle UI rendering

### 2. Tight Coupling
❌ Direct instantiation of dependencies

### 3. God Widgets
❌ Widgets with 100+ lines

### 4. Missing Tests
❌ Any feature without tests

### 5. Primitive Obsession
❌ Using String for Email, int for UserId

### 6. Data Class in Domain
❌ Using models instead of entities

### 7. Breaking Clean Architecture
❌ Domain depending on data layer

## Performance Optimization

### Use const Constructors
```dart
const MyWidget({Key? key}) : super(key: key);
```

### Provide Keys for Lists
```dart
ListView.builder(
  itemBuilder: (context, index) {
    return MyListItem(
      key: ValueKey(items[index].id),
      item: items[index],
    );
  },
)
```

### Avoid Rebuilds
```dart
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocBuilder<CounterBloc, CounterState>(
      buildWhen: (previous, current) => previous.count != current.count,
      builder: (context, state) {
        return Text('${state.count}');
      },
    );
  }
}
```

## Reference Documentation

For detailed information, refer to:
- `references/solid-principles.md` - SOLID with Dart/Flutter examples
- `references/tdd.md` - Test-Driven Development for Flutter
- `references/testing.md` - Testing strategies (unit, widget, integration)
- `references/clean-code.md` - Clean code guidelines for Dart
- `references/code-smells.md` - Flutter-specific code smell detection
- `references/design-patterns.md` - GoF patterns in Flutter
- `references/clean-architecture.md` - Clean Architecture for Flutter
- `references/state-management.md` - BLoC pattern and alternatives
- `references/performance.md` - Flutter performance optimization

## Workflow Summary

1. **Understand Requirement** → Define acceptance criteria
2. **Write Failing Test** → Red phase (TDD)
3. **Write Minimum Code** → Green phase
4. **Refactor** → Apply SOLID, remove smells
5. **Review Architecture** → Ensure clean boundaries
6. **Check Quality** → Run all checks
7. **Commit** → Only when all checks pass

Remember: **Quality over speed. Always.**

---
> Source: [thobio/flutter-solid-skill-](https://github.com/thobio/flutter-solid-skill-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
