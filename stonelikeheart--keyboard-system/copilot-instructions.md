## keyboard-system

> This is a C++20 learning project for a future keyboard lighting system. Keep

# Repository Guidance

## Scope

This is a C++20 learning project for a future keyboard lighting system. Keep
the core logic portable and independent of any microcontroller SDK.

## Development guidelines

- Use C++20 and configure targets through CMake.
- Prefer small, focused classes and functions over early abstractions.
- Put public headers under `include/keyboard_system/`.
- Put implementations and application entry points under `src/`.
- Add or update tests for behavioral changes.
- Keep hardware-specific implementations in separate targets when they are
  introduced; do not couple the core library to ESP32 APIs.

## Verification

Configure, build, and test with:

```sh
cmake -S . -B build
cmake --build build
ctest --test-dir build --output-on-failure
```

---
> Source: [StoneLikeHeart/keyboard-system](https://github.com/StoneLikeHeart/keyboard-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
