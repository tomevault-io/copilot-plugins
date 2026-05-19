## android-viewmodel

> Android ViewModel conventions — StateFlow, repository abstraction, coroutines, no LiveData


# Android ViewModel Rules

## StateFlow only

ViewModels expose state via `StateFlow` — do not use `LiveData`. Use `MutableStateFlow` internally and expose as `StateFlow`.

```kotlin
private val _uiState = MutableStateFlow(MyUiState())
val uiState: StateFlow<MyUiState> = _uiState.asStateFlow()
```

## Repository abstraction

ViewModels must not access `SharedPreferences` directly. All persistence goes through repository classes in the `data/` package.

## Coroutines only

All async work uses Kotlin coroutines. No RxJava, no callback-based APIs. Use `viewModelScope` for ViewModel-scoped coroutine launches.

## Existing ViewModels

`FretboardViewModel`, `TunerViewModel`, `PitchMonitorViewModel`, `ChordLibraryViewModel`, `FavoritesViewModel`, `SongbookViewModel`, `SettingsViewModel`, `ScalePracticeViewModel`, `MetronomeViewModel`, `MelodyViewModel`, `BackupRestoreViewModel`, `CustomProgressionViewModel`, `LearningProgressViewModel`, `SetlistViewModel`, `TtsAnnouncementThrottler`

---
> Source: [baijum/ukulele-companion](https://github.com/baijum/ukulele-companion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
