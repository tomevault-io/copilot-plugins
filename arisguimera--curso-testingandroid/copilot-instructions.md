## curso-testingandroid

> This file defines working conventions for coding agents in this repository.

This file defines working conventions for coding agents in this repository.
- Platform: Android
- Build system: Gradle (Kotlin DSL)
- Main module: `app`
- Use Java 17+ and Android SDK configured in Android Studio.
- Keep local secrets and machine-specific settings out of version control.
Run from repository root:
```bash
./gradlew tasks
./gradlew assembleDebug
```
Before finishing changes, run:
```bash
./gradlew test
./gradlew lint
```
If UI or instrumentation code is changed, also run:
```bash
./gradlew connectedAndroidTest
```
- Prefer Kotlin idioms and clear naming.
- Keep functions focused and side effects explicit.
- Add tests for behavior changes.
- Avoid broad refactors unless requested.
- Do not commit generated artifacts or local IDE noise.
- Make small, focused commits.
- Use non-interactive Git commands.
- Never rewrite or discard user-authored changes unless explicitly requested.
1. Inspect existing code paths before editing.
2. Implement the smallest correct change.
3. Run relevant checks.
4. Summarize changes with touched files and verification status.
- Build passes for affected modules.
- Tests for changed behavior pass.
- Lint is clean or known issues are explicitly called out.
- Documentation is updated when behavior or setup changes.

---
> Source: [ArisGuimera/Curso-TestingAndroid](https://github.com/ArisGuimera/Curso-TestingAndroid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
