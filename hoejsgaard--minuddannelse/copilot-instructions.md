## minuddannelse

> description: Logging standards and best practices

---
description: Logging standards and best practices
globs: "**/*.cs"
alwaysApply: false
---
# Logging Rules

- Remove LogDebug calls - use LogInformation or higher
- Remove try/catch->log patterns - let middleware handle exceptions  

- Use ILoggerFactory injection, not ILogger<T> in constructors

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Hoejsgaard) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
