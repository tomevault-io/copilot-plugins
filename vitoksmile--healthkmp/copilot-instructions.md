## healthkmp

> - **Common-First Policy:** All business logic, health data interfaces, and models reside in the `health` module's `commonMain` source set.

# HealthKMP - Project Rules & Context

## 1. Core Architecture
- **Common-First Policy:** All business logic, health data interfaces, and models reside in the `health` module's `commonMain` source set.
- **Health Management:** The core interface is `HealthManager`, which defines methods for checking availability, requesting authorization, and reading/writing health data.
- **Factory Pattern:** Use `HealthManagerFactory` (an `expect`/`actual` class) to instantiate the appropriate platform-specific implementation of `HealthManager`.
- **Platform Implementations:**
    - **Android:** Utilizes `HealthConnectManager` (for Health Connect) and `GoogleFitManager` (for legacy Google Fit support).
    - **Apple (iOS/watchOS):** Utilizes `HealthKitManager` to interact with Apple HealthKit.
- **Sample Apps:**
    - `sample/composeApp`: Demonstrates usage with **Compose Multiplatform** for Android and iOS.
    - `sample/appleApps`: Demonstrates usage with **SwiftUI** for watchOS.

## 2. Coding Standards
- **Platform Purity:** No `java.*`, `android.*`, or `Foundation/UIKit` imports in `commonMain`. Use KMP-native alternatives like `kotlinx-datetime` and `kotlinx-coroutines`.
- **Async & Flow:** Use `kotlinx-coroutines` and `suspend` functions for asynchronous operations.
- **API Consistency:** Public methods in `HealthManager` should return `Result<T>` for robust error handling.
- **Data Integrity:** Record data models (e.g., `StepsRecord`, `WeightRecord`) should include `init` blocks with `require` statements to validate input ranges and logic.
- **Named Arguments:** **MANDATORY** in tests and highly encouraged for data model instantiation. This prevents parameter swaps (e.g., systolic vs. diastolic) and improves readability.
- **Language Style:** Standard software engineering English (American English) is used for all code, comments, and documentation (e.g., `isAuthorized`, `initialization`).

## 3. Testing & Verification
- **Test Locations:**
    - `commonTest`: Unit tests for data models and shared logic.
    - `androidHostTest`: Validation for Health Connect mapping (runs on JVM).
    - `appleTest`: Validation for HealthKit mapping (shared by iOS and watchOS).
- **Mapping Verification:** Any new `HealthRecord` must have corresponding mapping tests in both `androidHostTest` and `appleTest` to ensure data integrity during platform translation.
- **CI Enforcement:** GitHub Actions require all tests to pass (`:health:allTests`) before merging Pull Requests or publishing releases.

## 4. Tooling & Libraries
- **Kotlin:** Version 2.3.20.
- **Java:** Version 21.
- **Swift Interop:** Use **SKIE** for enhanced Swift interop, especially for `suspend` functions and `Flow`. Configuration is in `health/build.gradle.kts`.
- **API Stability:** The project uses the **Binary Compatibility Validator** to ensure public API stability.
- **Dependencies:** Always check `gradle/libs.versions.toml` before adding new dependencies. Use the version catalog for all declarations.

## 5. Interaction Guidelines
- **Discovery:** Always verify the existence of health data types and units in `health/src/commonMain/kotlin/com/viktormykhailiv/kmp/health/` before proposing changes.
- **Cross-Platform Awareness:** When modifying `commonMain` interfaces, ensure that implementations in `androidMain` and `appleMain` (shared by `iosMain` and `watchosMain`) are updated accordingly.
- **Build System:** If modifying the build configuration, be mindful of the custom XCFramework setup for Apple platforms and the specific dependencies for Android Health Connect/Google Fit.
- **Testing Requirements:** Every feature or bug fix must include tests. Use `./gradlew :health:allTests` to verify changes across all supported platforms.

## 7. Building and Verification
### 7.1. API Stability
The project uses the **Binary Compatibility Validator**. When public API changes are made (e.g., adding new members to `HealthDataType` or new records), you MUST update the API dump:
```bash
./gradlew :health:apiDump
```
Ensure the generated `.api` files are committed.

### 7.2. Running Tests
Verify changes across all platforms using the following commands:
- **Android Unit Tests:** `./gradlew :health:testAndroid`
- **Android Mapping (Host) Tests:** `./gradlew :health:testAndroidHostTest`
- **Apple (iOS/watchOS) Tests:** `./gradlew :health:iosArm64Test` (runs on macOS host)
- **All Tests:** `./gradlew :health:allTests`

### 7.3. Xcode Framework Assembly
To verify that the Swift framework builds correctly and to update it for sample apps, use:
```bash
./gradlew :health:linkDebugFrameworkIosAmr64 :health:linkDebugFrameworkWatchosArm64
```

### 7.4. Sample App Verification
- **Android/Compose:** Build and run via Android Studio or `./gradlew :sample:androidApp:installDebug`.
- **Apple (macOS with Xcode):** 
  ```bash
  # Build for simulator with ad-hoc signing to embed entitlements
  xcodebuild build -project sample/appleApps/appleApps.xcodeproj -scheme watchosApp -destination 'id=<SIMULATOR_ID>' CODE_SIGN_IDENTITY="-"
  
  # Install and launch
  xcrun simctl install <SIMULATOR_ID> <PATH_TO_APP>
  xcrun simctl launch <SIMULATOR_ID> <BUNDLE_ID>
  ```

### 7.5. Releasing
To prepare a new release:
1. Update `CHANGELOG.md` with the new version and changes.
2. Update `VERSION_NAME` in `gradle.properties`.
3. Update version references in `README.md` if necessary.
4. Run all tests and update the API dump (`./gradlew :health:allTests :health:apiDump`).
5. Commit changes with message "Release X.Y.Z".
6. Push changes: `git push`.
7. The `publish.yml` workflow will automatically publish the release to Maven Central, create tag and Github release.

---
> Source: [vitoksmile/HealthKMP](https://github.com/vitoksmile/HealthKMP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
