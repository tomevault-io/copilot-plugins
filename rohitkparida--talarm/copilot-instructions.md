## talarm

> Follow strict MVVM architecture: **UI → ViewModel → Repository → Data**

# Android MVVM Architecture Guidelines

## Architecture Pattern
Follow strict MVVM architecture: **UI → ViewModel → Repository → Data**

## Layer Responsibilities
- **UI Layer**: Only handle user interactions and display data
- **ViewModel**: Manage UI state and handle business logic
- **Repository**: Abstract data access and provide clean API to ViewModels
- **Data Layer**: Handle raw data from network, database, or local storage

## Key Rules
- **Never put business logic** inside Activities or Fragments
- UI components should only observe data and pass events to ViewModels
- ViewModels should expose data via LiveData or StateFlow
- Use dependency injection (preferably Hilt) instead of global singletons

## Data Flow
```
User Action → UI → ViewModel → Repository → Data Source
Data Source → Repository → ViewModel → UI → User

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rohitkparida)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rohitkparida)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
