## sport-league-board

> - **Language**: Kotlin

# SportLeague — Android Project

## Project Assignment Requirements

### Core Requirements (all satisfied)
- **Language**: Kotlin
- **UI**: Jetpack Compose
- **Screens**: 3 screens (Standings, Team Detail, Settings)
- **Navigation**: Jetpack Navigation Compose
- **Architecture**: MVVM + Repository pattern (recommended Android architecture)
- **API**: Retrofit → `https://api.football-data.org/v4/` (free, HTTPS/TLS encrypted)

### Bonus Points Achieved
| Bonus | Status | Where |
|-------|--------|-------|
| Room database | ✅ | `data/local/` — caches standings offline |
| SQL injection sanitization | ✅ | `utils/InputValidator.kt` + Room parameterized queries |
| Encrypted communication | ✅ | All API calls use HTTPS (TLS) via OkHttp |
| Settings screen | ✅ | `ui/screens/settings/` — dark mode + accent color |
| Unit tests | ✅ | `ExampleUnitTest.kt`, `StandingsViewModelTest.kt` |
| Clean code / modularization | ✅ | Layered architecture: api/local/repository/ui |
| Coroutines / Dispatchers | ✅ | `viewModelScope`, `Dispatchers.IO` in repository |

---

## Architecture

```
com.upb.sportleague/
├── data/
│   ├── api/            Retrofit service + OkHttp client
│   ├── local/          Room database, DAO, Entity
│   ├── model/          API response data classes (Gson)
│   ├── preferences/    DataStore for user settings
│   └── repository/     StandingsRepository (single source of truth)
├── ui/
│   ├── navigation/     NavHost + Screen sealed class
│   ├── screens/
│   │   ├── standings/  Standings list screen + ViewModel
│   │   ├── teamdetail/ Team stats screen + ViewModel
│   │   └── settings/   Dark mode / accent color + ViewModel
│   └── theme/          Material3 theme with Premier League colors
├── utils/
│   └── InputValidator  Input sanitization helpers
└── MainActivity.kt     Single activity, reads dark mode pref
```

### Data Flow
```
API (football-data.org)
        ↓ Retrofit (HTTPS)
StandingsRepository  ←→  Room DB (cache)
        ↓ Flow<List<StandingEntity>>
StandingsViewModel   (viewModelScope + StateFlow)
        ↓ collectAsState()
StandingsScreen      (Jetpack Compose)
```

---

## Setup

### 1. Get a free API key
Register at [football-data.org](https://www.football-data.org/client/register) (free tier: 10 req/min).

### 2. Add your key to `local.properties`
```properties
FOOTBALL_API_KEY=your_actual_key_here
```
This is injected into `BuildConfig.FOOTBALL_API_KEY` at build time and never committed to VCS.

### 3. Verify KSP version
The KSP version must match your Kotlin version exactly. Current config:
- Kotlin: `2.3.21`
- KSP: `2.3.8` (in `gradle/libs.versions.toml`) — KSP 2.3.x uses new versioning scheme, compatible with AGP 9.x

If the build fails, check [KSP releases](https://github.com/google/ksp/releases) for the matching version and update `ksp` in `libs.versions.toml`.

### 4. Sync & Build
Open in Android Studio → Sync Project with Gradle Files → Run.

---

## Key Design Decisions

- **No Hilt**: Manual dependency injection via `ViewModelProvider.Factory` keeps the project self-contained without the extra plugin overhead.
- **Cache strategy**: Network-first with 5-minute TTL. If network fails and cache exists, cached data is shown with no error.
- **Room + parameterized queries**: All Room `@Query` methods use bound parameters — no string concatenation, making SQL injection impossible.
- **Input sanitization**: `InputValidator` validates and sanitizes all user-controlled strings before they reach the database layer.
- **HTTPS only**: `android:usesCleartextTraffic="false"` in the manifest blocks any accidental unencrypted traffic.

---

## Running Tests

```bash
./gradlew test          # unit tests
./gradlew connectedTest # instrumented tests (needs device/emulator)
```

---

## API Response Reference

`GET https://api.football-data.org/v4/competitions/PL/standings`

Header: `X-Auth-Token: <your_key>`

Returns JSON with `competition`, `season`, and `standings[].table[]` containing position, team info, W/D/L, goals, points.

---
> Source: [edi16banulescu/Sport-League-Board](https://github.com/edi16banulescu/Sport-League-Board) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
