## mutlu-log

> This is a local Flutter logging package that provides structured, configurable logging for the EduStation application.

# mutlu_log Package - Cursor AI Rules

## Package Overview
This is a local Flutter logging package that provides structured, configurable logging for the EduStation application.

## Core Principles
1. **Always use mutlu_log for logging** - Never use `print()` or `dart:developer.log()` directly
2. **Tag everything** - Every log should have a descriptive tag
3. **Use appropriate levels** - Match log severity to the level
4. **Include context** - Use the `data` parameter for structured information
5. **Never log sensitive data** - Passwords, tokens, PII must not be logged

## Import Statement
```dart
import 'package:mutlu_log/mutlu_log.dart';
```

## Log Levels (Use Appropriately)

### AppLog.d() - Debug
Use for detailed debugging information that's only needed during development.
```dart
AppLog.d('User tapped button', tag: 'UI', data: {'button_id': 'submit'});
AppLog.d('Cache hit for key', tag: 'Cache', data: {'key': 'user_123'});
```

### AppLog.i() - Info
Use for general informational messages about app flow and state changes.
```dart
AppLog.i('User logged in', tag: 'Auth', data: {'user_id': userId});
AppLog.i('Navigation to screen', tag: 'Navigation', data: {'route': '/profile'});
```

### AppLog.w() - Warning
Use for potentially problematic situations that aren't errors.
```dart
AppLog.w('API response took longer than expected', tag: 'Performance', data: {'duration_ms': 3000});
AppLog.w('Using cached data due to network error', tag: 'Cache');
```

### AppLog.e() - Error
Use for errors and exceptions. Always include error and stackTrace when available.
```dart
AppLog.e(
  'Failed to load user profile',
  tag: 'Profile',
  data: {'user_id': userId},
  error: exception,
  stackTrace: stackTrace,
);
```

## Standard Tags by Feature

Use these standard tags for consistency:

- **Network**: All HTTP/API calls (automatically used by DioLogInterceptor)
- **Auth**: Authentication and authorization
- **Database**: Local database operations
- **Cache**: Caching operations
- **Storage**: File and secure storage operations
- **Navigation**: Route changes and navigation
- **UI**: User interface interactions
- **ChatWS**: WebSocket for chat
- **Firebase**: Firebase services (messaging, analytics, crashlytics)
- **Payment**: Payment processing
- **Profile**: User profile operations
- **[FeatureName]**: Use the feature name for feature-specific logs (e.g., 'PrizeWheel', 'Events', 'Training')

## When to Add Logging

### DO Log:
- ✅ Entry/exit of important operations
- ✅ API calls (automatic via DioLogInterceptor)
- ✅ State changes in providers
- ✅ User actions that trigger business logic
- ✅ Error conditions and exceptions
- ✅ Performance-critical operations
- ✅ Background tasks and scheduled jobs
- ✅ WebSocket connections and messages

### DON'T Log:
- ❌ Widget build methods (too verbose)
- ❌ Getters/setters (unless complex logic)
- ❌ Simple utility functions
- ❌ Every line of code (log strategically)
- ❌ Sensitive data (passwords, tokens, PII)

## Common Patterns

### In Providers (Riverpod)
```dart
class MyFeatureProvider extends StateNotifier<MyState> {
  Future<void> loadData() async {
    AppLog.d('Loading data started', tag: 'MyFeature');
    
    try {
      final result = await repository.fetchData();
      AppLog.i('Data loaded successfully', tag: 'MyFeature', data: {'count': result.length});
      state = state.copyWith(data: result);
    } catch (e, stackTrace) {
      AppLog.e('Failed to load data', tag: 'MyFeature', error: e, stackTrace: stackTrace);
      state = state.copyWith(error: e.toString());
    }
  }
}
```

### In Repositories
```dart
class UserRepository {
  Future<User> getUser(int userId) async {
    AppLog.d('Fetching user from API', tag: 'UserRepo', data: {'user_id': userId});
    
    try {
      final response = await networkHelper.get(path: '/users/$userId');
      // DioLogInterceptor automatically logs request/response
      return User.fromJson(response.data);
    } catch (e, stackTrace) {
      AppLog.e('Failed to fetch user', tag: 'UserRepo', error: e, stackTrace: stackTrace);
      rethrow;
    }
  }
}
```

### In Use Cases
```dart
class LoginUseCase {
  Future<LoginResult> execute(String email, String password) async {
    AppLog.i('Login attempt', tag: 'Auth', data: {'email': email});
    // Note: NEVER log passwords
    
    try {
      final result = await authRepository.login(email, password);
      AppLog.i('Login successful', tag: 'Auth', data: {'user_id': result.userId});
      return result;
    } catch (e, stackTrace) {
      AppLog.e('Login failed', tag: 'Auth', error: e, stackTrace: stackTrace);
      rethrow;
    }
  }
}
```

### In WebSocket/Stream Handlers
```dart
void _handleWebSocketMessage(Map<String, dynamic> data) {
  AppLog.d('WebSocket message received', tag: 'ChatWS', data: {'type': data['type']});
  
  try {
    final message = Message.fromJson(data);
    _messageController.add(message);
  } catch (e, stackTrace) {
    AppLog.e('Failed to parse WebSocket message', tag: 'ChatWS', error: e, stackTrace: stackTrace);
  }
}
```

### For Performance Monitoring
```dart
Future<void> heavyOperation() async {
  final stopwatch = Stopwatch()..start();
  
  try {
    await performOperation();
    stopwatch.stop();
    
    final duration = stopwatch.elapsedMilliseconds;
    if (duration > 1000) {
      AppLog.w('Slow operation detected', tag: 'Performance', data: {'duration_ms': duration});
    } else {
      AppLog.d('Operation completed', tag: 'Performance', data: {'duration_ms': duration});
    }
  } catch (e, stackTrace) {
    stopwatch.stop();
    AppLog.e(
      'Operation failed',
      tag: 'Performance',
      error: e,
      stackTrace: stackTrace,
      data: {'duration_ms': stopwatch.elapsedMilliseconds},
    );
    rethrow;
  }
}
```

## HTTP Logging (DioLogInterceptor)

The package includes an enhanced Dio interceptor that automatically logs all HTTP requests/responses.

### Setup in NetworkManager
```dart
import 'package:mutlu_log/mutlu_log.dart';

dio.interceptors.add(
  DioLogInterceptor(
    requestHeader: true,      // Log request headers (sensitive data auto-sanitized)
    requestBody: true,         // Log request body
    responseHeader: true,      // Log response headers
    responseBody: true,        // Log response body
    logErrors: true,           // Log errors with details
    maxBodyLength: 1000,       // Truncate long bodies
  ),
);
```

### What Gets Logged Automatically
- Request: method, URL, headers (sanitized), body, query params
- Response: status code, headers, body, **duration in milliseconds**
- Errors: error type, status code, error body, stack traces
- Headers like `Authorization`, `X-API-Key`, `Cookie` are automatically redacted

## Configuration

### Initialization (in bootstrap.dart or main.dart)
```dart
import 'package:mutlu_log/mutlu_log.dart';

// Initialize with dotenv
AppLog.init(dotenvMap: dotenv.env);

// Or with explicit config
AppLog.init(
  logLevel: 'debug',
  logFormat: 'pretty',
  logTags: 'Network,Auth',
);
```

### Runtime Flags
```bash
# Pretty format (default)
flutter run --dart-define=LOG_FORMAT=pretty

# JSON format for parsing
flutter run --dart-define=LOG_FORMAT=json

# Filter by tags
flutter run --dart-define=LOG_TAGS=Network,Auth,ChatWS

# Set minimum level
flutter run --dart-define=LOG_LEVEL=warn
```

## Security Rules

### NEVER Log These:
- ❌ Passwords (plain or hashed)
- ❌ API keys or secrets
- ❌ Authorization tokens
- ❌ Credit card numbers
- ❌ Personal identification numbers
- ❌ Full email addresses in production (use masked: `u***@example.com`)
- ❌ Phone numbers
- ❌ Addresses

### Safe to Log:
- ✅ User IDs (numeric identifiers)
- ✅ Usernames (non-sensitive)
- ✅ Operation types
- ✅ Status codes
- ✅ Timestamps
- ✅ Durations
- ✅ Counts and aggregates
- ✅ Feature flags

## Code Review Checklist

When reviewing code or writing new code with logging:

1. ✅ Is `mutlu_log` imported instead of using `print()`?
2. ✅ Does every log have a descriptive tag?
3. ✅ Is the log level appropriate for the message?
4. ✅ Are structured data included in the `data` parameter?
5. ✅ Are errors logged with `error` and `stackTrace` parameters?
6. ✅ Is there no sensitive information in the logs?
7. ✅ Are logs meaningful and actionable?
8. ✅ Is logging not too verbose (avoid logging in tight loops)?

## Example: Complete Feature Implementation

```dart
import 'package:mutlu_log/mutlu_log.dart';

class PrizeWheelProvider extends StateNotifier<PrizeWheelState> {
  PrizeWheelProvider(this._claimPrizeUseCase) : super(PrizeWheelState());
  
  final ClaimPrizeUseCase _claimPrizeUseCase;

  int spinWheel(BuildContext context) {
    AppLog.d('spinWheel() called', tag: 'PrizeWheel');
    
    if (state.isSpinning || state.selectedPrizeIndex != null) {
      AppLog.d(
        'Spin prevented',
        tag: 'PrizeWheel',
        data: {
          'isSpinning': state.isSpinning,
          'selectedPrizeIndex': state.selectedPrizeIndex,
        },
      );
      return state.selectedPrizeIndex ?? 0;
    }

    final selectedIndex = Random().nextInt(8);
    AppLog.d('Selected prize index', tag: 'PrizeWheel', data: {'index': selectedIndex});
    
    state = state.copyWith(isSpinning: true);
    return selectedIndex;
  }

  Future<void> claimPrize(String prizeText) async {
    AppLog.i('Claiming prize', tag: 'PrizeWheel', data: {'prize': prizeText});
    state = state.copyWith(isLoading: true, error: null);

    try {
      final response = await _claimPrizeUseCase.execute(prizeText);
      AppLog.i('Prize claimed successfully', tag: 'PrizeWheel');
      state = state.copyWith(
        isLoading: false,
        claimResponse: response,
        error: null,
      );
    } catch (e, stackTrace) {
      AppLog.e('Prize claim failed', tag: 'PrizeWheel', error: e, stackTrace: stackTrace);
      state = state.copyWith(
        isLoading: false,
        error: e.toString(),
        claimResponse: null,
      );
      rethrow;
    }
  }
}
```

## Quick Reference

```dart
// Standard logging
AppLog.d('message', tag: 'Tag');
AppLog.i('message', tag: 'Tag', data: {'key': 'value'});
AppLog.w('message', tag: 'Tag');
AppLog.e('message', tag: 'Tag', error: e, stackTrace: st);

// HTTP logging (automatic via DioLogInterceptor)
// No manual logging needed - interceptor handles it

// Check configuration
print('Log level: ${AppLog.minLevel}');
print('Log format: ${AppLog.format}');
```

## Additional Resources

- Full documentation: `packages/mutlu_log/README.md`
- Examples: `packages/mutlu_log/example/example.md`
- Tests: `packages/mutlu_log/test/`
- Architecture: See "Logging" section in `ARCHITECTURE.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Melsaeed276) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
