## ktor-quickstart

> - Always use `--quiet` flag to avoid Gradle's verbose output

# Build with Gradle

- Always use `--quiet` flag to avoid Gradle's verbose output
- Build any module `./gradlew :<module>:build`
- Run all checks for a module `./gradlew :<module>:test`
    - Test selection accepts the pipe | character to separate test elements:
      `./gradlew clean test --tests "com.example.TestSuite|inner suite|*" --no-build-cache`

---
> Source: [nomisRev/ktor-quickstart](https://github.com/nomisRev/ktor-quickstart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
