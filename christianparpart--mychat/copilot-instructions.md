## mychat

> @~/.claude/rules/cpp.md

@~/.claude/rules/cpp.md

# Building

- Use CMake with preset "clang-debug" for building and testing on Linux with Clang in debug mode.
- Use CMake with preset "clang-release" for performance testing on Linux with Clang in release mode.

# Testing

- Run the tests using `ctest --preset=clang-debug` or `ctest --preset=clang-release` depending on the build type.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/christianparpart)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/christianparpart)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
