## termi

> This is a cross-platform terminal emulator with:

# Next-Gen Terminal Project Rules

## Project Context
This is a cross-platform terminal emulator with:
- Rust core (PTY, VFS, ANSI parsing)
- Android: Kotlin + Jetpack Compose
- iOS (future): Swift + SwiftUI
- JNI/FFI bridges between native and platform code

## Architecture
- Rust core is platform-agnostic
- Platform-specific code in android/ and ios/
- VFS abstracts over Android SAF and iOS file picker
- Session state tracks process lifecycle

## Critical Safety Rules

### JNI Safety (MANDATORY)
1. NEVER use unwrap() in JNI boundary code
2. ALWAYS check for JNI exceptions after call_method():
```rust
   if env.exception_check().unwrap_or(false) {
       env.exception_describe().ok();
       env.exception_clear().ok();
       return error_code;
   }
```
3. ALWAYS validate handles before dereferencing
4. Use safe wrappers from rust/src/jni_safe.rs
5. Return error codes, never panic across FFI

### VFS Safety (MANDATORY)
1. ALWAYS check capabilities before operations:
```rust
   if !vfs.supports(path, VfsOperation::Chmod) {
       // Handle gracefully
   }
```
2. Warn users about SAF limitations in UI
3. Cache metadata to avoid slow ContentResolver calls
4. Batch operations when possible

### Session State (MANDATORY)
1. Track session state everywhere (Active/Checkpointed/Restored/Failed)
2. Checkpoint state periodically and on background
3. Show state clearly in UI
4. Design for "expect to die" not "hope to survive"

## Code Style
- Rust: Follow rustfmt defaults
- Kotlin: Follow ktlint defaults
- Error handling: Use Result<T, E>, not panic
- Logging: Use log crate (Rust) and Timber (Android)

## Documentation
- Reference docs/LIMITATIONS.md for constraints
- All public functions need doc comments
- Explain WHY not just WHAT in complex code

## Testing
- Unit tests for all core logic
- JNI safety tests mandatory
- Test on Samsung/Xiaomi devices before claiming stability

## When Suggesting Code
- Show complete examples with error handling
- Use safe patterns from jni_safe.rs
- Explain Android/iOS platform differences
- Point out performance implications

---
> Source: [MannanSaood/termi](https://github.com/MannanSaood/termi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
