## fedman-admin

> enableScroll: true, // default is true

# Claude Development Instructions

## Screen Development Guidelines

When creating new screens, use `@lib/core/common_widgets/screen_body.dart` for the screen scaffold instead of creating custom Scaffold widgets.

The ScreenBody widget provides:
- Consistent scaffold structure
- Optional scrolling with `enableScroll` parameter
- Built-in SafeArea handling
- Support for AppBar and bottom navigation
- Keyboard dismiss behavior on drag

Example usage:
```dart
ScreenBody(
  appBar: AppBar(title: Text('Screen Title')),
  enableScroll: true, // default is true
  bottomNAvBar: BottomNavigationBar(...), // optional
  child: YourScreenContent(),
)
```

## Asset Management Guidelines

Always add assets to `@lib/core/constants/app_assets.dart` first before using them in widgets.

Steps:
1. Add asset path constant to `AppAssets` class
2. Import `AppAssets` in your widget file
3. Use the constant instead of hardcoded asset path

Example:
```dart
// In app_assets.dart
static const String myIcon = 'assets/images/my_icon.svg';

// In your widget
import 'package:fedman_admin/core/constants/app_assets.dart';

SvgPicture.asset(AppAssets.myIcon)
```
- **Custom Widgets**: Always use custom widgets from `lib/core/common_widgets/` instead of creating new ones. Available widgets include `CustomButton`, `SocialButton`, `CustomTextFormField`, etc.
- **Text Styling**: Always use predefined text styles from `@lib/core/theme/app_text_styles.dart` instead of creating inline TextStyle. Available styles include `heading1`, `heading2`, `subHeading1`, `subHeading2`, `body1`, `body2`, `navlinks1`, `navlinks2`, `cta1`, `cta2`
- **Spacing**: Always use spacing extensions from `@lib/core/extensions/space.dart` instead of hardcoded SizedBox values. Available extensions: `horizontalSpace`, `verticalSpace`, `responsiveHorizontalSpace`, `responsiveVerticalSpace`. Example: `16.verticalSpace` instead of `SizedBox(height: 16)`
- **Navigation**: Always use go_router for navigation instead of Navigator. Use `context.go('/route')` for navigation, `context.push('/route')` for overlay navigation, and `context.pop()` to go back. Import go_router: `import 'package:go_router/go_router.dart'`

## Dependency Injection Guidelines

All repository classes must be registered in the dependency injection container at `@lib/core/di/injection.dart` using GetIt.

Steps:
1. Import the repository class in `injection.dart`
2. Register it as a lazy singleton with required dependencies
3. Ensure all dependencies (like ApiClient, managers) are registered first

Example:
```dart
// Register repository with dependencies
getIt.registerLazySingleton<YourRepo>(
  () => YourRepo(
    apiClient: getIt<ApiClient>(),
    otherDependency: getIt<OtherDependency>(),
  ),
);
```

## Repository Response Guidelines

All repository methods must return `ApiResponse<T>` for consistent error handling.

- Use `ApiResponse.success(data)` for successful responses
- Use `ApiResponse.failure(message)` for error responses
- DO NOT add try-catch in repository methods - let errors bubble up to BLoC layer

Example:
```dart
Future<ApiResponse<User>> getUser() async {
  final response = await apiClient.get('/user');
  return ApiResponse.success(User.fromJson(response.data));
}
```

## BLoC Error Handling Guidelines

All BLoC classes should handle errors with try-catch blocks, not the repository layer.

- Add try-catch in BLoC event handlers
- Convert exceptions to appropriate error states
- Repository methods should throw errors directly for BLoC to catch

Example:
```dart
Future<void> _onGetUserRequested(
  GetUserRequested event,
  Emitter<UserState> emit,
) async {
  try {
    emit(UserLoading());
    final result = await userRepo.getUser();
    
    if (result.success && result.data != null) {
      emit(UserSuccess(user: result.data!));
    } else {
      emit(UserError(message: result.message ?? 'Failed to get user'));
    }
  } catch (e) {
    emit(UserError(message: 'Error: $e'));
  }
}
```

## State Management Guidelines

Use `ValueNotifier` with `ValueListenableBuilder` instead of `setState` for reactive UI updates.

**Benefits:**
- Better performance (only rebuilds specific widgets)
- Cleaner code separation
- More predictable state updates
- Easier testing

**Implementation:**
```dart
class MyWidget extends StatefulWidget {
  @override
  State<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  final ValueNotifier<String?> _selectedValueNotifier = ValueNotifier(null);
  final ValueNotifier<int> _counterNotifier = ValueNotifier(0);

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        ValueListenableBuilder<String?>(
          valueListenable: _selectedValueNotifier,
          builder: (context, selectedValue, child) {
            return DropdownButton<String>(
              value: selectedValue,
              onChanged: (value) {
                _selectedValueNotifier.value = value;
              },
              items: ['Option 1', 'Option 2'].map((item) =>
                DropdownMenuItem(value: item, child: Text(item))
              ).toList(),
            );
          },
        ),
        ValueListenableBuilder<int>(
          valueListenable: _counterNotifier,
          builder: (context, counter, child) {
            return Text('Count: $counter');
          },
        ),
        ElevatedButton(
          onPressed: () => _counterNotifier.value++,
          child: Text('Increment'),
        ),
      ],
    );
  }

  @override
  void dispose() {
    _selectedValueNotifier.dispose();
    _counterNotifier.dispose();
    super.dispose();
  }
}
```

**Rules:**
- NEVER use `setState()` - always use `ValueNotifier`
- Wrap reactive widgets with `ValueListenableBuilder`
- Always dispose ValueNotifiers in the dispose method
- Use descriptive names ending with "Notifier" (e.g., `_selectedLocationNotifier`)

### Git Commit Guidelines
- Do not include "Generated with Claude Code" attribution in commit messages
- Write clean, concise commit messages that focus on the changes made
- Use conventional commit format when appropriate (feat:, fix:, etc.)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mehroze-zaidi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
