## mushaf-app

> Directly using `print()` for debugging or logging is **forbidden** in this project. Use `debugPrint()` instead, which automatically handles debug/release mode and prevents leaking sensitive information.

# Use `debugPrint()` for Debug Logging

Directly using `print()` for debugging or logging is **forbidden** in this project. Use `debugPrint()` instead, which automatically handles debug/release mode and prevents leaking sensitive information.

## Preferred Method: `debugPrint()`

`debugPrint()` is Flutter's built-in debug logging function that:
- Only outputs in debug builds (stripped from release builds)
- Automatically throttles output to prevent log flooding
- Handles long messages appropriately

```dart
import 'package:flutter/foundation.dart'; // For debugPrint and kDebugMode

// Good: debugPrint is automatically debug-only
debugPrint('This is a debug message: $variable');

// For conditional logging or additional context
if (kDebugMode) {
  debugPrint('Detailed debug info: $variable');
  // TODO: Include stackTrace when implementing crash analytics
  // catch (e, stackTrace) { ... debugPrint(stackTrace.toString()); }
}
```

## For Persistent Logging

For more permanent or structured logging, consider using a dedicated logging framework like `package:logger`. This provides more control over log levels, output formatting, and destinations.

## Linter Enforcement

The project's `analysis_options.yaml` is configured to flag `print()` calls, reminding developers to follow these best practices. Any new code should not introduce `print()` statements outside of a `kDebugMode` block.

## Why `debugPrint()` over `print()`

- **Security**: `debugPrint()` is automatically stripped from release builds, preventing accidental data leaks
- **Performance**: `debugPrint()` is optimized and throttled to prevent log flooding
- **Cleanliness**: Keeps production logs clean automatically without manual `kDebugMode` checks
- **Consistency**: Ensures all debug output follows the same pattern across the codebase

## When to use `kDebugMode` with `debugPrint()`

Use `if (kDebugMode)` guards when:
- The logging operation itself might have side effects
- You want to conditionally include additional debug context
- You need to prepare data for logging that's expensive to compute

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kfkhalili) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
