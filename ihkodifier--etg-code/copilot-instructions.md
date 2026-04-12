## etg-code

> use this rule anytime you are writing flutter code


You are an expert in Flutter, Dart, Riverpod, Freezed, Flutter Hooks, and Firebase and AI integrations.

# Key Principles
- Write concise, technical Dart code with accurate examples.
- Use functional and declarative programming patterns where appropriate.
- Prefer composition over inheritance.
- Use descriptive variable names with auxiliary verbs (e.g., isLoading, hasError).
- Structure files: exported widget, subwidgets, helpers, static content, types.
- Always use the Single Responsibility Widget principle.
- The Main Screens of the app/project shoulds just be acomposition of child widgets
- Always write null-aware code.
- Always implement both light and dark themes
-  Test Dark/Light theme switching functionality



# Dart/Flutter
- Use const constructors for immutable widgets.
- Leverage Freezed for immutable state classes and unions.
- Use arrow syntax for simple functions and methods.
- Prefer expression bodies for one-line getters and setters.
- Use trailing commas for better formatting and diffs.
- Always Use the SelectableText widget instead of plain old Text widgets.
- always prefer to use the MediaQuery.maybePlatformBrightnessOf
- Always use ThemeMode.brightness to detect  current theme mode or switch theme 
- Use AsyncNotifier/StreamNotifier classes for business logic
- Keep UI widgets as pure presentation layer
- Use Riverpod providers as your "controllers"
- Use GoRouter for navigation and deep linking
- Use GoRouter's extra parameter to pass data between routes
- Implement proper route guards for authentication
- app should remember to check for existing 





# Error Handling and Validation
- Implement error handling in views using SelectableText.rich instead of SnackBars.
- always catch Exceptions for code that is likely to throw an exception at runtime(especially network calls). 
- when an exception is thrown, present it gracefully to the users in the UI and print entire stacktrace and entire error object to debug log
- Display errors in SelectableText.rich with redish color for visibility.
- Handle empty states within the displaying screen.
- Use AsyncValue for proper error handling and loading states.


# State Management 
- Always use RiverPod for State Management instead of SetState() calls
- Always prefer corresponding NotifierProvider over mutable providers that do not notify
- Use AsyncNotifierProvider for async state that can be mutated
- Use StreamNotifierProvider for real-time data streams  
- Use Provider for simple immutable values

## Riverpod-Specific Guidelines
- Always User RiverPod for State Management instead of SetState() calls.
<!-- - Use @riverpod annotation for generating providers. -->
- prefer immutable providers over the mutable counterparts.
- Prefer AsyncNotifierProvider and StreamNotifierProvider over StateProvider.
- Use AsyncNotifierProvider for async state that can be mutated
- Use StreamNotifierProvider for real-time data streams
 - avoid legacy providers that are deprecated above Riverpod 2.0+ or marked for depreation
- prefer AsyncNotifierProvider/StreamNotifierProvider over legacy providers
- Use ref.invalidate() for manually triggering provider updates.
- Implement proper cancellation of asynchronous operations when widgets are disposed.
- when chaining AsyncProviders, handle all states of Data, error and loading in each of the constituent AsyncProvider within that chain.

# Performance Optimization
- Use const widgets where possible to optimize rebuilds.
- Implement list view optimizations (e.g., ListView.builder).
-use Widgets over nested build methods.
- Always wrap Buttons in a Tooltip Widget.
- Use AssetImage for static images and cached_network_image for remote images.
- Implement proper error handling for Firebase operations, including network errors.

# Key Conventions
1. Use GoRouter  for navigation and deep linking.
2. Optimize for Flutter performance metrics (first meaningful paint, time to interactive).
3. Prefer stateless widgets:
   - Use ConsumerWidget with Riverpod for state-dependent widgets.
   - Use HookConsumerWidget when combining Riverpod and Flutter Hooks.

# UI and Styling
- Use Flutter's built-in widgets and create custom widgets.
- Implement responsive design using LayoutBuilder or MediaQuery making use of pub.dev Responsive_framework package
- Use themes for consistent styling across the app.
- Use Theme.of(context).textTheme.titleLarge instead of headline6, and headlineSmall instead of headline5 etc. aka Material3 font sub system.

# Model and Database Conventions
- Include createdAt, updatedAt, createdBy, updatedBy and isDeleted fields in database tables.
- for createdBy, updatedBy fields, use the currently loggedin  user of the app.
- Use @JsonSerializable(fieldRename: FieldRename.snake) for models.
- Implement @JsonKey(includeFromJson: true, includeToJson: false) for read-only fields.
- always implement toString , toJson, fromString and fromJson methods for Data classes. 
- always implement Equality via equatable  pub.dev package.
- for collections use deep collectionEquality
- prefer to use packages that are atleast available for all of the  web, android,iOs platforms.
- Only use packages that have a good rating on pub.dev
-before adding a package dependency always check whether it can create compatibility issues within packages, if found so, inform the developer in cursor chat

# Serialization and Deserialization Standards
- **ALWAYS use model-based serialization and deserialization** instead of manual dictionary construction/parsing
- **NEVER manually construct data dictionaries or parse JSON** for API calls or database operations
- Use `.toJson()`/`.fromJson()` for Freezed models and `.dict()`/`.parse_obj()` for Pydantic models
- Manual dictionary construction/parsing leads to:
  - Field name inconsistencies (camelCase vs snake_case)
  - Silent data corruption and parsing failures
  - Maintenance nightmares when models change
  - Serialization/deserialization bugs that are hard to debug
  - Type safety violations

## Serialization (Object → JSON/Dict)

- **Examples of what NOT to do:**
  ```dart
  // ❌ WRONG - Manual dictionary construction
  final questionData = {
    'id': question.id,
    'questionId': question.questionId,
    'examCategory': question.examCategory,  // camelCase - WRONG!
    'approvalStatus': question.approvalStatus,  // camelCase - WRONG!
  };
  ```

  ```python
  # ❌ WRONG - Manual dictionary construction
  question_dict = {
      "question_text": create_request.question_text,
      "options": [...],  # Manual field mapping
  }
  ```

- **Examples of what TO do:**
  ```dart
  // ✅ CORRECT - Model-based serialization
  final questionData = question.toJson();
  ```

  ```python
  # ✅ CORRECT - Model-based serialization
  question_dict = create_request.dict()
  ```

## Deserialization (JSON/Dict → Object)

- **Examples of what NOT to do:**
  ```dart
  // ❌ WRONG - Manual JSON parsing
  final question = Question(
    id: json['id'],
    questionId: json['questionId'],  // May be string or int inconsistently
    examCategory: json['exam_category'],  // Wrong field name mapping
    // ... manual field mapping with potential null errors
  );
  ```

  ```python
  # ❌ WRONG - Manual dictionary parsing
  question = Question(
      question_text=data.get("question_text"),
      options=data.get("options", []),  # Manual parsing
  )
  ```

- **Examples of what TO do:**
  ```dart
  // ✅ CORRECT - Model-based deserialization
  final question = Question.fromJson(json);
  ```

  ```python
  # ✅ CORRECT - Model-based deserialization
  question = QuestionCreateRequest.parse_obj(data)
  ```

- **Benefits**: Type safety, automatic field mapping, validation, consistency, maintainability
- **Enforcement**: All serialization/deserialization code must use model methods, never manual construction/parsing

# API Client and Networking Standards
- **ALWAYS use relative paths in API service classes** when ApiClient has a base URL with path prefixes
- **NEVER hardcode absolute API paths** that duplicate the base URL prefixes
- **ALWAYS verify actual request URLs** when debugging 401/404 API errors
- API path concatenation issues cause silent authentication failures and routing errors

## API Client Configuration

- **Examples of what NOT to do:**
  ```dart
  // ❌ WRONG - Hardcoded absolute paths with base URL prefix
  class ApiClient {
    static String get baseUrl => 'http://localhost:8080/api/v1';  // Includes /api/v1
  }

  class QuestionApiService {
    static const String _questionsPath = '/api/v1/questions';  // ❌ Double /api/v1
    // Results in: http://localhost:8080/api/v1/api/v1/questions
  }
  ```

- **Examples of what TO do:**
  ```dart
  // ✅ CORRECT - Relative paths with base URL prefix
  class ApiClient {
    static String get baseUrl => 'http://localhost:8080/api/v1';  // Includes /api/v1
  }

  class QuestionApiService {
    static const String _questionsPath = '/questions';  // ✅ Relative path
    // Results in: http://localhost:8080/api/v1/questions
  }
  ```

## API Debugging Patterns

- **When API calls fail with 401/404 errors:**
  - Check browser dev tools or logs for actual request URLs
  - Verify path concatenation (look for double prefixes like `/api/v1/api/v1`)
  - Ensure service paths are relative to ApiClient base URL
  - Test API endpoints directly before implementing fallbacks

- **Benefits**: Prevents silent authentication failures, ensures correct routing, easier debugging
- **Enforcement**: All API service paths must be relative to base URL, never absolute with duplicate prefixes

# UI-Backend Communication Architecture
- **NEVER allow UI widgets to communicate directly with the backend**
- **ALL backend communication MUST go through API service layers  (talking to the api end points) **
- **UI widgets are strictly presentation layer components** - they should only consume state and trigger actions through providers

## Architecture Layers (Bottom to Top)
1. **Backend API** - External REST/GraphQL endpoints
2. **API Service Layer** - Dart classes that handle HTTP requests/responses
3. **Repository Layer** - Data access abstraction (optional but recommended)
4. **Riverpod Providers** - State management and business logic
5. **UI Widgets** - Pure presentation components

## What UI Widgets CAN Do
- **Consume state** from Riverpod providers using `ref.watch()`
- **Trigger actions** through Riverpod providers using `ref.read()`
- **Handle user interactions** and pass them to providers
- **Display data** from provider state
- **Show loading/error states** from AsyncValue

## What UI Widgets CANNOT Do
- **Make direct HTTP requests** using `http`, `dio`, or similar packages
- **Import API service classes** directly
- **Handle authentication tokens** or API keys
- **Parse raw JSON responses**
- **Manage network state** or retry logic
- **Cache data** locally (except for UI-specific state)

## Correct Architecture Pattern

### ✅ CORRECT - UI Widget using Provider
```dart
class QuestionListWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final questionsState = ref.watch(questionsProvider);
    
    return questionsState.when(
      data: (questions) => ListView.builder(
        itemCount: questions.length,
        itemBuilder: (context, index) => QuestionCard(question: questions[index]),
      ),
      loading: () => const CircularProgressIndicator(),
      error: (error, stack) => SelectableText.rich(
        TextSpan(
          text: 'Error: $error',
          style: TextStyle(color: Colors.red),
        ),
      ),
    );
  }
}

// Provider handles all backend communication
@riverpod
class QuestionsNotifier extends _$QuestionsNotifier {
  @override
  Future<List<Question>> build() async {
    return await _questionApiService.getQuestions();
  }
  
  Future<void> createQuestion(QuestionCreateRequest request) async {
    state = const AsyncValue.loading();
    try {
      final newQuestion = await _questionApiService.createQuestion(request);
      // Update state with new question
      ref.invalidateSelf();
    } catch (error, stackTrace) {
      state = AsyncValue.error(error, stackTrace);
    }
  }
}
```

### ❌ WRONG - UI Widget making direct API calls
```dart
class QuestionListWidget extends StatefulWidget {
  @override
  _QuestionListWidgetState createState() => _QuestionListWidgetState();
}

class _QuestionListWidgetState extends State<QuestionListWidget> {
  List<Question> questions = [];
  bool isLoading = false;
  
  @override
  void initState() {
    super.initState();
    _loadQuestions(); // ❌ WRONG - Direct API call from UI
  }
  
  Future<void> _loadQuestions() async {
    setState(() => isLoading = true);
    try {
      // ❌ WRONG - Direct HTTP call from UI widget
      final response = await http.get(
        Uri.parse('${ApiClient.baseUrl}/questions'),
        headers: {'Authorization': 'Bearer $token'},
      );
      final data = json.decode(response.body);
      setState(() {
        questions = (data as List).map((q) => Question.fromJson(q)).toList();
        isLoading = false;
      });
    } catch (e) {
      // ❌ WRONG - Error handling in UI
      setState(() => isLoading = false);
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Error: $e')),
      );
    }
  }
}
```

## Benefits of This Architecture
- **Separation of Concerns**: UI focuses on presentation, providers handle business logic
- **Testability**: Easy to mock providers for widget testing
- **Reusability**: Providers can be consumed by multiple widgets
- **State Management**: Centralized state with automatic UI updates
- **Error Handling**: Consistent error handling across the app
- **Caching**: Providers can implement intelligent caching strategies
- **Loading States**: Built-in loading state management with AsyncValue

## Enforcement Rules
- **No direct imports** of `http`, `dio`, or other HTTP packages in UI files
- **No direct imports** of API service classes in UI files
- **All network calls** must be encapsulated in Riverpod providers
- **All state management** must use Riverpod providers, never setState()
- **All error handling** must be done through AsyncValue, not try-catch in UI

## Widgets and UI Components
- Create small, private widget classes instead of methods like Widget _build....
- Implement RefreshIndicator for pull-to-refresh functionality.
- In TextFields, set appropriate textCapitalization, keyboardType, and textInputAction.
- Always include an errorBuilder when using Image.network.
- for Forms always use TextFormField's vaildation and error checking 
- for any TextFormField that is bound to receive numbers, use appropriate KeyboardInputType on mobile devices(android/ios)
- For text input fields that are supposed to receive an email, always use RegEx for validation.
- dispose  controllers as soon as possible
- for multistep Forms, dispose TextField controllers as soon as possible and pass their values to subsequent step via providers
- for Forms always use keyboard shortcuts 
- for TextFormFields always use valid RegEx validation

# Miscellaneous
- Use log instead of print for debugging.
- Use Flutter Hooks / Riverpod Hooks where appropriate.
- Keep lines no longer than 80 characters, adding commas before closing brackets for multi-parameter functions.
- Use @JsonValue(int) for enums that go to the database.

# Code Generation
- Utilize build_runner for generating code from annotations (Freezed, Riverpod, JSON serialization).
- Run 'flutter pub run build_runner build --delete-conflicting-outputs' after modifying annotated classes.

# Documentation
- Document complex logic and non-obvious code decisions with comments
- Follow official Flutter, Riverpod, and Supabase documentation for best practices.

# Best Practices

Prefer immutable providers over mutable counterparts
Use ref.invalidate() for manually triggering provider updates
Implement proper cancellation of asynchronous operations when widgets are disposed
When chaining AsyncProviders, handle all states (data, error, loading) in each constituent provider


Refer to Flutter, Riverpod, and Supabase documentation for Widgets, State Management, and Backend Integration best practices.
    










<!-- You are a senior Dart programmer with experience in the Flutter framework and a preference for clean programming and design patterns.

Generate code, corrections, and refactorings that comply with the basic principles and nomenclature. -->

## Dart General Guidelines

### Basic Principles

- Use English for all code and documentation.
- Always declare the type of each variable and function (parameters and return value).
  - Avoid using any.
  - Create necessary types.
- Don't leave blank lines within a function.
- One export per file.

### Nomenclature

- Use PascalCase for classes.
- Use camelCase for variables, functions, and methods.
- Use underscores_case for file and directory names.
- Use UPPERCASE for environment variables.
  - Avoid magic numbers and define constants.
- Start each function name  with a verb.
- Use verbs for boolean variables. Example: isLoading, hasError, canDelete, etc.
- Use complete words instead of abbreviations and correct spelling.
  - Except for standard abbreviations like API, URL, etc.
  - Except for well-known abbreviations:
    - i, j for loops
    - err for errors
    - ctx for contexts
    - req, res, next for middleware function parameters

### Functions

- In this context, what is understood as a function will also apply to a method.
- Write short functions with a single purpose. Less than 20 instructions.
- Name functions with a verb and something else.
  - If it returns a boolean, use isX or hasX, canX, etc.
  - If it doesn't return anything, use executeX or saveX, etc.
- Avoid nesting blocks by:
  - Early checks and returns.
  - Extraction to utility functions.
- Use higher-order functions (map, filter, reduce, etc.) to avoid function nesting.
  - Use arrow functions for simple functions (less than 3 instructions).
  - Use named functions for non-simple functions.
- Use default parameter values instead of checking for null or undefined.
- Reduce function parameters using RO-RO
  - Use an object to pass multiple parameters.
  - Use an object to return results.
  - Declare necessary types for input arguments and output.
- Use a single level of abstraction.

### Data

- Don't abuse primitive types and encapsulate data in composite types.
- Avoid data validations in functions and use classes with internal validation.
- Prefer immutability for data.
  - Use readonly for data that doesn't change.
  - Use as const for literals that don't change.

### Classes

- Follow SOLID principles.
- Prefer composition over inheritance.
- Declare interfaces to define contracts.
- Write small classes with a single purpose.
  - Less than 200 instructions.
  - Less than 10 public methods.
  - Less than 10 properties.

### Exceptions

- Use exceptions to handle errors you don't expect.
- If you catch an exception, it should be to:
  - Fix an expected problem.
  - Add context.
  - Otherwise, use a global handler.

### Testing

- Follow the Arrange-Act-Assert convention for tests.
- Name test variables clearly.
  - Follow the convention: inputX, mockX, actualX, expectedX, etc.
- Write unit tests for each public function.
  - Use test doubles to simulate dependencies.
    - Except for third-party dependencies that are not expensive to execute.
- Write acceptance tests for each module.
  - Follow the Given-When-Then convention.

## Specific to Flutter

### Basic Principles

- Use clean architecture
  - see modules if you need to organize code into modules
  - see controllers if you need to organize code into controllers
  - see services if you need to organize code into services
  - see repositories if you need to organize code into repositories
  - see entities if you need to organize code into entities
- Use repository pattern for data persistence
  - see cache if you need to cache data
- Use controller pattern for business logic with Riverpod
- Use Riverpod to manage state
  - see keepAlive if you need to keep the state alive
- Use freezed to manage UI states
- Controller always takes methods as input and updates the UI state that effects the UI
<!-- - Use getIt to manage dependencies -->
  - Use singleton for services and repositories
  - Use factory for use cases
  - Use lazy singleton for controllers

  - Use extras to pass data between pages
- Use extensions to manage reusable code
- Use ThemeData to manage themes
- Use AppLocalizations to manage translations
- Use constants to manage constants values
- When a widget tree becomes too deep, it can lead to longer build times and increased memory usage. Flutter needs to traverse the entire tree to render the UI, so a flatter structure improves efficiency
- A flatter widget structure makes it easier to understand and modify the code. Reusable components also facilitate better code organization
- Avoid Nesting Widgets Deeply in Flutter. Deeply nested widgets can negatively impact the readability, maintainability, and performance of your Flutter app. Aim to break down complex widget trees into smaller, reusable components. This not only makes your code cleaner but also enhances the performance by reducing the build complexity
- Deeply nested widgets can make state management more challenging. By keeping the tree shallow, it becomes easier to manage state and pass data between widgets
- Break down large widgets into smaller, focused widgets
- Utilize const constructors wherever possible to reduce rebuilds

### Testing

- Use the standard widget testing for flutter
- Use integration tests for each api module.






    

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IHKodifier)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/IHKodifier)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
