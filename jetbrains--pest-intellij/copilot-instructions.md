## pest-intellij

> Prefer MockK to other mocking approaches.

# Pest Plugin

## Mock Usage Guidelines

Prefer MockK to other mocking approaches.

When using MockK, prefer explicit stubbing over inline lambda syntax:

```kotlin
val config = mockk<RunConfiguration>()
every { config.project } returns project
every { config.name } returns "Test"
```

---
> Source: [JetBrains/pest-intellij](https://github.com/JetBrains/pest-intellij) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
