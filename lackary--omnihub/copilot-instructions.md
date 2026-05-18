## omnihub

> You are an expert Kotlin Multiplatform developer. Your role is to help maintain and expand the OmniHub project using MVI and Clean Architecture.

# Gemini AI Context for OmniHub

You are an expert Kotlin Multiplatform developer. Your role is to help maintain and expand the OmniHub project using MVI and Clean Architecture.

## Project Vision
OmniHub is a multiplatform application built with Compose Multiplatform, targeting Android (including XR), iOS, Desktop, and Web.

## Architectural Enforcement (STRICT)

### 1. Clean Architecture Layers
- **Domain Layer**: No platform dependencies. Contains `UseCase` and repository interfaces.
- **Data Layer**: Implements repositories, Ktor API calls, and DTO-to-Domain mappers.
- **Presentation Layer**: Compose UI and MVI ViewModels.

### 2. MVI Pattern (Contract-Based)
All new features must implement a `Contract` interface in the following format:
- `State`: Data class for UI state.
- `Event`: Sealed class/interface for user intents.
- `Effect`: Sealed class/interface for one-time side effects (e.g., `Navigation`).

Example Structure:
```kotlin
interface FeatureContract {
    data class State(...)
    sealed interface Event { ... }
    sealed interface Effect { ... }
}

---
> Source: [lackary/omnihub](https://github.com/lackary/omnihub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
