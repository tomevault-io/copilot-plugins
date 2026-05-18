## stepcounter

> StepCounter is an Android step tracking app that uses accelerometer sensor data to detect steps and calculate distance walked. The app runs as a foreground service for background tracking.

# AGENTS.md - AI Agent Guide for StepCounter

## Project Overview

StepCounter is an Android step tracking app that uses accelerometer sensor data to detect steps and calculate distance walked. The app runs as a foreground service for background tracking.

## Quick Navigation

| Component | Path | Purpose |
|-----------|------|---------|
| Step Detection | `core/sensor/StepDetector.kt` | Core algorithm for detecting steps from accelerometer |
| Stride Calculation | `core/stride/StrideCalculator.kt` | Calculates stride length from GPS or height |
| GPS Tracking | `core/location/LocationTracker.kt` | Collects GPS fixes for distance/calibration |
| GPS Calibration | `core/location/GpsCalibrator.kt` | Calibrates stride using GPS distance |
| Session Manager | `core/session/SessionManager.kt` | Manages walk sessions (start/stop/pause) |
| Database | `data/database/StepDatabase.kt` | Room database with WalkSession, DailySummary, UserProfile |
| Main UI | `ui/dashboard/` | Dashboard screen with step counter |
| Background Service | `service/StepTrackingService.kt` | Foreground service for background tracking |

## Key Logic Explained

### Step Detection Algorithm (`StepDetector.kt`)
- Processes accelerometer sensor data
- Detects peaks in acceleration magnitude
- Uses dynamic threshold based on signal variance
- Considers phone position (hand, pocket) via `PhonePositionDetector`
- Implements refractory period to avoid double-counting

### Stride Length Calculation
Two methods:
1. **Initial estimate** (`GenderHeightCalibrator.kt`):
   - Male: `stride = height_meters × 0.415`
   - Female: `stride = height_meters × 0.413`

2. **GPS calibration** (`StrideCalculator.kt`):
   - `stride = gps_distance / step_count`
   - Validates stride is 0.3m - 1.5m
   - Uses rolling average of last 10 calibrations
   - GPS distance calculated via Haversine formula

### Distance Calculation
- **From steps**: `distance = step_count × stride_length`
- **From GPS**: Haversine formula between GPS points
- GPS filters: accuracy >20m or speed >5.5 m/s (~20 km/h) rejected

### Data Flow
```
[Accelerometer] → [StepDetector] → [SessionManager] → [Repository] → [Room DB]
                                                    ↓
[GPS Location] → [GpsCalibrator] → [StrideCalculator] (calibrates stride)
```

## Build Commands

```bash
./gradlew assembleDebug     # Build debug APK
./gradlew assembleRelease   # Build release APK
./gradlew test              # Run unit tests
./gradlew lint             # Run lint checks
```

## Architecture Patterns

- **Clean Architecture**: Domain, Data, UI layers
- **Dependency Injection**: Hilt with `@HiltViewModel`, `@Inject`
- **State Management**: Compose state hoisting pattern
- **Database**: Room with KSP annotation processing
- **Navigation**: Compose Navigation with typed routes

## Important Conventions

- All sensor processing runs on background threads (coroutines)
- GPS updates every 60 seconds with `PRIORITY_BALANCED_POWER_ACCURACY`
- Foreground service shows persistent notification with step count
- `BootCompletedReceiver` restarts service after device reboot
- User profile stored in Room: height (cm), gender, daily step goal

## Common Tasks

### Adding a new screen
1. Create composable in `ui/<screen_name>/`
2. Add route to navigation graph in `ui/navigation/`
3. Create ViewModel with `@HiltViewModel`
4. Add to navigation bottom bar if needed

### Modifying step detection
- Edit `StepDetector.kt` threshold/peak detection logic
- Test with `PhonePositionDetector` for different phone positions
- Verify `CircularBuffer.kt` size is appropriate for sampling rate

### Database changes
1. Update entity in `data/database/entities/`
2. Update DAO in `data/database/daos/`
3. Increment database version in `StepDatabase.kt`
4. Provide migration or set `fallbackToDestructiveMigration`

## Linting and Type Checking

Before committing changes, run:
```bash
./gradlew lint
./gradlew ktlintCheck  # if ktlint is configured
```

## Testing

- Unit tests: `src/test/` directory
- Instrumentation tests: `src/androidTest/`
- Run with: `./gradlew test` or `./gradlew connectedAndroidTest`

---
> Source: [xezy/StepCounter](https://github.com/xezy/StepCounter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
