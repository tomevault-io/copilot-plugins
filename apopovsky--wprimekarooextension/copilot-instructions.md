## wprimekarooextension

> We are developing an **Android application (in Kotlin)** for the **Hammerhead Karoo 3 cycling computer**, using the official **karoo-ext** library. The goal is to create a **custom data field** that displays **W′ (W prime) remaining** in real-time during a ride. Future projects may include additional data fields or standalone apps (e.g., to improve phone notifications on the Karoo).

# GitHub Copilot Custom Instructions for Karoo 3 Extension Development

## Project Context and Goals

We are developing an **Android application (in Kotlin)** for the **Hammerhead Karoo 3 cycling computer**, using the official **karoo-ext** library. The goal is to create a **custom data field** that displays **W′ (W prime) remaining** in real-time during a ride. Future projects may include additional data fields or standalone apps (e.g., to improve phone notifications on the Karoo).

- **Platform:** Hammerhead Karoo 3 (Android-based device)
- **Language:** Kotlin
- **Library:** [karoo-ext](https://github.com/hammerheadnav/karoo-ext)
- **Documentation:** [https://hammerheadnav.github.io/karoo-ext/](https://hammerheadnav.github.io/karoo-ext/)
- **Sample Code:** [Sample Extension](https://github.com/hammerheadnav/karoo-ext/tree/master/app/src/main/kotlin/io/hammerhead/sampleext)
- **Template:** [karoo-ext-template](https://github.com/hammerheadnav/karoo-ext-template)
- **Community:** [r/Karoo subreddit](https://www.reddit.com/r/Karoo), [Hammerhead Forum](https://support.hammerhead.io/hc/en-us/community/topics/31298804001435-Hammerhead-Extensions-Developers)

## Coding Guidelines

- Use **modern Android best practices**:
  - Kotlin idioms (`data class`, lambdas, coroutines)
  - Architecture patterns (MVVM, DI with Hilt)
  - Jetpack Compose (if UI is needed)
  - Lifecycle awareness

- Ensure **clean and efficient code**:
  - Clear naming and self-explanatory logic
  - Comment only where necessary (e.g. W′ computation logic)
  - Avoid main-thread blocking (use coroutines/background threads)
  - Release resources when not needed (e.g., remove consumers)

- **Performance matters** on Karoo:
  - Optimize sensor subscriptions and dispatch frequency
  - Minimize memory and battery usage
  - Keep UI simple, high contrast, and readable

## Karoo Extension Integration

- Define a `KarooExtension` subclass (e.g. `MyWPrimeExtension`) with a unique `id` and version
- Register the service in `AndroidManifest.xml`:

```xml
<service android:name=".MyWPrimeExtension">
    <intent-filter>
        <action android:name="io.hammerhead.karooext.KAROO_EXTENSION" />
    </intent-filter>
    <meta-data
        android:name="io.hammerhead.karooext.EXTENSION_INFO"
        android:resource="@xml/extension_info" />
</service>
```

- Add `extension_info.xml` with `<DataType>` for W′, including attributes like `typeId`, `name`, `graphical="false"`

- Use `KarooSystemService`:
  - Connect in `onStart()` and disconnect in `onStop()`
  - Subscribe to events like `RideDataEvent` or `RideState`
  - Remove consumers properly to avoid leaks
  - Dispatch data to update DataType value (e.g. with `UpdateData`)

- For **custom graphical fields**, implement `RemoteViews` layout updates

## W′ Data Handling

- Use power data events to compute real-time W′ balance
- Implement integration logic based on critical power and recovery
- Dispatch updated W′ values to Karoo regularly (at reasonable intervals)

## Glance Layouts and Responsive Design

When working with Jetpack Glance for custom graphical data fields:

- **Use `ViewConfig.viewSize` NOT `LocalSize.current`**: 
  - `LocalSize.current` returns `NaN` in Karoo's Glance implementation
  - `ViewConfig.viewSize` provides actual pixel dimensions (e.g., `Pair(480, 240)`)
  - Convert pixels to dp using Karoo's density: `widthDp = (pixels / 2.0f).dp`

- **Responsive breakpoints based on actual pixel dimensions**:
  - Small field (media pantalla): ~480x240px = 240x120dp
  - Large field (pantalla completa): ~960x480px = 480x240dp
  - Use pixel values for breakpoints, not dp values

- **Text size adaptation by character count**:
  - Apply `widthFactorAdjustment` to force smaller font for longer text
  - Example: 5-digit numbers need ~1.65x more horizontal space to avoid truncation
  - Balance between readability and avoiding truncation

- **Icon/arrow sizing**:
  - Scale icons proportionally to field height (typically 20-25% of height)
  - Reserve minimal horizontal space (`iconSize + 2dp`) for text calculation

- **Glance limitations**:
  - NO support for custom `weight()` values (e.g., `weight(2f)`)
  - Use `defaultWeight()` for flex items, fixed `width()` for fixed columns
  - Use `fillMaxWidth()` carefully - can push other columns out of view

- **Debug logging**:
  - Always log actual dimensions when implementing responsive layouts
  - Use consistent prefix (e.g., `WPRIME_SIZE`) for easy filtering: `adb logcat | grep WPRIME_SIZE`

## Copilot Instructions Summary

GitHub Copilot should:

- Prioritize karoo-ext API usage and correct integration patterns
- Follow clean Kotlin and Android design principles
- Reference official documentation and sample code when suggesting API usage
- Avoid suggesting generic Android code when Karoo-specific solutions exist
- Assist with modular design for future expansion (e.g., notifications)
- **NEVER create separate MD documentation files for resolved issues or fixes** - just fix the code and commit

Copilot should behave like a **Karoo-aware Android assistant**, not a general Android suggester.

Copilot should leverage the `README.md` file to stay aligned with the existing architecture and project structure. For example, when working on W′ calculations, configuration handling, or Karoo system integration, prioritize files like:

- `WPrimeCalculator.kt`
- `WPrimeDataType.kt`
- `WPrimeSettings.kt`

---
> Source: [apopovsky/WPrimeKarooExtension](https://github.com/apopovsky/WPrimeKarooExtension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
